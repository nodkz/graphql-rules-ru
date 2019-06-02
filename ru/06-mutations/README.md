## 6. Правила Мутаций

Сколько схем я не смотрел, но больше всего бардака разводят в Мутациях. Следующие правила позволят вам сделать ваше АПИ сухим, чистым и удобным.

### <a name="rule-6.1"></a> 6.1. Используйте Namespace-типы для группировки мутаций в рамках одного ресурса.

В большенстве GraphQL-схем страшно заглядывать в мутации. На АПИ среднего размера кол-во мутаций может легко переваливать за 50-100 штук, и это все на одном уровне. Ковыряться и искать нужную операцию в таком списке достаточно сложно.

Shopify рекомендует придерживаться такого именования для мутаций `collection<Action>`. В списке это позволяет хоть как-то сгрупировать операции над одним ресурсом. Кто-то противник такого подхода, и форсит использование `<action>Collection`.

В любом случае есть способ получше – используйте `Namespace-типы`. Это такие типы которые содержат в себе набор операций над одним ресурсом. Если представить путь запроса в dot-нотации, то выглядит он так `Mutation.<collection>.<action>`.

На стороне сервера в Node.js это делается достаточно легко. Я приведу несколько примеров реализации такого подхода с использованием разных библиотек:

Стандартная реализация с пакетом `graphql`:

```js
// Create Namespace type for Article mutations
const ArticleMutations = new GraphQLObjectType({
  name: 'ArticleMutations',
  fields: () => {
    like: {
      type: GraphQLBoolean,
      resolve: () => { /* resolver code */ },
    },
    unlike: {
      type: GraphQLBoolean,
      resolve: () => { /* resolver code */ },
    },
  },
});

// Add `article` to regular mutation type with small magic
const Mutation = new GraphQLObjectType({
  name: 'Mutation',
  fields: () => {
    article: {
      type: ArticleMutations,
      resolve: () => ({}), // ✨✨✨ magic! which allows to proceed call of sub-methods
    }
  },
});
```

С помощью пакета `graphql-tools`:

```js
const typeDefs = gql`
type Mutation {
    article: ArticleMutations
  }

  type ArticleMutations {
    like: Boolean
    unlike: Boolean
  }
`;

const resolvers = {
  Mutation: {
    article: () => ({}), // ✨✨✨ magic! which allows to proceed call of sub-methods
  }
  ArticleMutations: {
    like: () => { /* resolver code */ },
    unlike: () => { /* resolver code */ },
  },
};
```

С помощью `graphql-compose`:

```js
schemaComposer.Mutation.addNestedFields({
  'article.like': { // ✨✨✨ magic! Just use dot-notation with `addNestedFields` method
    type: 'Boolean',
    resolve: () => { /* resolver code */ }
  },
  'article.unlike': {
    type: 'Boolean',
    resolve: () => { /* resolver code */ }
  },
});
```

Соответственно клиенты легко будут понимать и видеть по документации, какие именно методы доступны над "ресурсом/моделью" `article`. А их запросы к серверу будут иметь следующий вид:

```graphql
mutation {
  article {
    like(id: 15)
  }

  ### Forget about ugly mutations names!
  # artileLike
  # likeArticle
}
```

Если вдруг Фронтендеры в одном запросе выполняют серийно несколько мутаций (что является антипатерном – нужно делать общую мутацию), то они смогут получить тоже самое поведение на вложенных мутациях через алиасы. Детально можно ознакомиться с [этим тестом](__tests__/mutations-test.js), где мутации like/unlike работают асинхронно с таймаутом на время выполнения. Кратко тест и результаты его работы выглядят так:

```js
await graphql({
  schema,
  source: `
  mutation {
    op1: article { like(id: 1) }
    op2: article { like(id: 2) }
    op3: article { unlike(id: 3) }
    op4: article { like(id: 4) }
  }
`,
});

expect(serialResults).toEqual([
  'like 1 executed with timeout 100ms',
  'like 2 executed with timeout 100ms',
  'unlike 3 executed with timeout 5ms',
  'like 4 executed with timeout 100ms',
]);
```

Итак правило, для избежания бардака в мутациях используйте Namespace-типы для группировки мутаций в рамках одного ресурса!

### <a name="rule-6.2"></a> 6.2. Выходите за рамки CRUD – создавайте небольшие мутации для разных логических операций над ресурсами.

```diff
type ArticleMutations {
   create(...): Payload
   update(...): Payload
+  like(...): Payload
+  unlike(...): Payload
+  publish(...): Payload
+  unpublish(...): Payload
}
```

С GraphQL надо отходить от CRUD (create, read, update, delete). Если вешать все изменения на мутацию `update`, то достаточно быстро она станет массивной и тяжелой в обслуживании. Здесь речь идет не о простом редактировании полей заголовок и описание, а о "сложных" операциях.  К примеру, для публикации статей создайте мутации `publish`, `unpublish`. Необходимо добавить лайки к статье - создайте две мутации `like`, `unlike`. Помните, что потребители вашего АПИ слабо представляют себе структуру взаимосвязей ваших данных, и за что каждое поле отвечает. А набор мутаций над ресурсом быстро позволяет фронтендеру "въехать" в набор возможных операций.

Да и бэкендеру в будущем будет легче отслеживать по логам, что пользователи чаще всего дергают. И оптимизировать узкие места.

### <a name="rule-6.3"></a> 6.3. Рассмотрите возможность выполнения мутаций сразу над несколькими элементами (однотипные batch-изменения).

Правило скорректировано по замечаниям: Ivan Goncharov #42
Дата последней корректировки: 17.05.2019

```diff
type ArticleMutations {
-  deleteArticle(id: Int!): Payload
+  deleteArticle(id: [Int!]!): Payload
}

```

Клиентские приложения становятся более умными и удобными. Часто пользователю предлагаются batch-операции – добавление нескольких записей, массового удаления или сортировки. Отправлять операции по одной будет накладно. Как-то агрегировать их в сложный GraphQL-запрос с несколькими мутациями, т.е. динамически генерировать один общий запрос на клиенте - совершенно отвратительная идея:

```graphql
mutation DeleteArticles { # BAD
  op1: deleteArticle(id: 1)
  op2: deleteArticle(id: 2)
  op3: deleteArticle(id: 5)
  op4: deleteArticle(id: 5)
}
```

Если GraphQL-запрос динамически создается в рантайме (само тело запроса, а не сбор переменных) – то скорее всего вы делаете что-то не так. Форма запроса должна задаваться разработчиками на этапе написания кода. Это позволяет проверять запросы линтерами и статическими анализаторами; а также позволяет "скомпилировать" запрос (сконвертировать в AST) для Relay/Apollo. При динамической генерации запроса на клиенте в браузере теряются все эти проверки и оптимизации.

Благодаря [GraphQL спецификации - List Input Coercion](https://graphql.github.io/graphql-spec/draft/#sec-Type-System.List) вы можете в схеме объявить аргумент `id` как массив `id: [Int!]!`, тогда клиенты смогут передавать в своих запросах как просто число, так и массив:

```graphql
mutation DeleteArticles {
  op1: deleteArticle(id: [1, 2, 5]) # works
  op2: deleteArticle(id: 7) # works too
}
```

Тоже самое работает, если использовать переменные (можно посмотреть [тесты](./__tests__/list-coercion-test.js)):

```js
await graphql({
  schema,
  source: `
    mutation DeleteArticles($id: [Int!]!) {
      deleteArticle(id: $id)
    }
  `,
  variableValues: { id: 777 }, // Should be `Array`, but works with `Int` too
});
```

Также иногда бекендерам важно понимать, что происходит массовая операция. Т.к. можно оптимизировать логику или побочные эффекты, например отправить 1 нотификацию или 1000.

### <a name="rule-6.4"></a> 6.4. У мутаций должны быть четко описаны все обязательные аргументы, не должно быть вариантов либо-либо.

Бывают задачи, когда необходимо в зависимости от определенных входных значений принимать различные входные параметры. Проблема заключается в том, что в таком случае придется указывать входные параметры необязательными, что может привести к  ошибкам, т.к. клиент не знает наверняка, какие параметры являются обязательными к заполнению в том или ином случае.

К примеру ваше АПИ позволяет отправить разные письма с помощью мутации `sendEmail(type: PASSWORD_RESET, params: JSON)`. Для того чтобы выбрать шаблон, вы передаете Enum аргумент с типом письма и для него передаете какие-то параметры.

Проблема такого подхода в том, что клиент заранее точно не знает какие параметры необходимо передать для того или иного типа писем. К тому же, если в будущем проводить рефакторинг схемы, то статическая типизация нам не позволит отловить ошибки на клиенте.

Лучше разбивать мутации на несколько штук с жестким описанием аргументов. Например: `sendEmailPasswordReset(login: String!, note: String)`. При этом не забываем аргументы помечать как обязательные, если без них операция не отработает.

Также бывают ситуации, когда вы обязаны передать либо один аргумент, либо другой. К примеру, мы можем отправить письмо по сбросу пароля если укажут login или email – `sendResetPassword(login: String, email: String)`.

В таком случае мы не можем оба аргумента в нашей мутации сделать обязательными. О том что не передан обязательный аргумент, мы узнаем только в рантайме. Да и фронтендеру будет не сразу понятно, что надо передавать либо `login`, либо `email`. А что будет если передать оба аргумента от разных пользователей?

Для решения этой проблемы просто заводится две мутации, где жестко расписаны обязательные аргументы:

```diff
type Mutation {
-  sendResetPassword(login: String, email: Email)
+  sendResetPasswordByLogin(login: String!)  # login NonNull
+  sendResetPasswordByEmail(email: Email!)   # email NonNull
}
```

Не экономьте на мутациях и старайтесь избегать слабой типизации.

Если вы только начинаете знакомство с graphQL, Вы могли бы подумать, что проблему можно решить с помощью union инпутов, однако, их в graphQL еще не завезли. В официальном репозитории graphQL ведется обсуждение добавления union инпутов, за которым вы можете следить [здесь](https://github.com/facebook/graphql/issues/488). Вы должны понимать, что после добавления union инпутов некоторые схожие задачи будет правильнее решать с их помощью. В обсуждении вы можете ознакомиться с примерами задач от разработчиков, которые нуждаются в добавлении union инпутов.

### <a name="rule-6.5"></a> 6.5. У мутации вкладывайте все переменные в один уникальный `input` аргумент.

Старайтесь в мутациях использовать один аргумент `input`. Его гораздо легче использовать на клиентской стороне. Клиенту потребуется передать всего одну переменную, а не вагон для каждого аргумента в мутации.

```graphql
# Good:
mutation ($input: UpdatePostInput!) {
  updatePost(input: $input) { ... }
}

# Not so good – гороздо сложнее писать запрос (дубль переменных)
mutation ($id: ID!, $newText: String, ...) {
  updatePost(id: $id, newText: $newText, ...) { ... }
}
```

Если у мутации на верхнем уровне один-два аргумента, то при таком подходе они становятся более стройными и читабельными. При этом без дополнительных затрат, кроме нескольких дополнительных нажатий клавиш, вложение аргументов позволяет вам полностью использовать возможности GraphQL, в качестве version-less API (безверсионного апи). Вложенность дает вам возможность расширять типы с течением времени и избегать конфликтов в именовании полей.

Также при статической типизации с помощью Typescript или Flowtype гораздо легче отследить изменения в вашем АПИ, когда в коде идет привязка к одному сложному-типу, а не набору разрозненных аргументов.

Думайте о вложении аргументов в один общий аргумент `input`, как об инвестиции в будущие изменения вашего GraphQL API.

При этом не экономьте на типах – для каждой мутации заводите свой Input-тип с уникальным именем. Это позволит вам менять мутации, не оглядываясь на то, что новая семантика может поломать другие мутации.

Также по состоянию на конец 2018 года в спецификации GraphQL нет возможности деприкейтить аргументы (помечать их как устаревшие). Но вот деприкейтить поля внутри типа `input` можно. Это еще один повод использовать аргумент `input` со вложенностью.

### <a name="rule-6.6"></a> 6.6. Мутация должна возвращать свой уникальный Payload-тип.

Результат выполнения мутации тоже необходимо возвращать в виде вложенного Payload-типа с уникальным именем. Это позволяет вам расширять ответы мутаций дополнительными полями с данными. При этом не ломать другие мутации, когда вы редактируете уникальный тип ответа для текущей мутации.

Даже если вы хотите вернуть только одну запись из вашей мутации, не поддавайтесь искушению вернуть этот тип напрямую. Возвращая данные напрямую (без обертки в Payload-тип), вы лишаете себя возможности в будущем легко добавить дополнительные поля для возврата. GraphQL version-less API (безверсионного апи) хорошо работает когда типы расширяются, а не модифицируются. Вложение ответа в свой кастомный Payload-тип – это инвестиции в будущие изменения вашего АПИ.

```diff
type Mutation {
-  createPerson(input: ...): Person                # BAD
+  createPerson(input: ...): CreatePersonPayload   # GOOD
}

+ type CreatePersonPayload {
+   recordId: ID
+   record: Person
+   # ... любые другие поля, которые пожелаете
+ }
```

Важно отметить, что возвращаемые поля в вашем Payload-типе должны быть nullable (необязательными). Т.е. если вы будите возвращать ошибку например в поле `userErrors`, то вы не сможете гарантировать наличие данных в поле `record`. Этот момент может всплыть, когда фронтендеры начнут вас просить сделать эти поля обязательными, т.к. статический анализ заставляет их делать дополнительную проверку на наличие данных. Вы спокойно должны им сказать, что им необходимо делать проверку, ведь данные реально могут отсутствовать.

### <a name="rule-6.6.1"></a> 6.6.1. В ответе мутации возвращайте измененный ресурс и его `id`.

В Payload-типе мутации необходимо возвращать измененный ресурс. Лучше всего если поле, которое возвращает измененный ресурс будет иметь фиксированное имя – `record`. Тогда фронтендеры смогут на автомате считывать результаты вашей мутации и не тратить время на поиск поля, в котором возвращается измененый ресурс.

Также желательно не полениться и предоставить поле `recordId`, которое возвращает `ID` измененного ресурса. К примеру, при создании новой записи, если запросили только поле `recordId`, то это позволит не тянуть весь объект, а просто вернуть `id` созданной записи.

```diff
type CreatePersonPayload {
+  recordId: ID!
+  record: Person
  # ... любые другие поля, которые пожелаете
}
```

### <a name="rule-6.6.2"></a> 6.6.2. В ответе мутации возвращайте статус операции.

Любая мутация это операция на изменение данных. И желательно предоставить механизм для быстрого определения прошла мутация успешно или нет. В мире REST API, да и вообще в мире HTTP достаточно активно используются [HTTP status code](https://ru.wikipedia.org/wiki/%D0%A1%D0%BF%D0%B8%D1%81%D0%BE%D0%BA_%D0%BA%D0%BE%D0%B4%D0%BE%D0%B2_%D1%81%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%B8%D1%8F_HTTP) – так почему же не взять и не перенять в каком-то виде эту практику?!

Как вы знаете GraphQL не зависит от протокола – может быть http, web-sockets, telnet, ssh и прочее. В запросе может опрашиваться много ресурсов и возвращаться много разных ошибок. И многие REST разработчики "страдают" от отсутствия статус кодов, кто-то даже ошибочно пытается их прикрутить к GraphQL-серверу. Так вот почему не взять и не добавить `status` поле в ответе вашей мутации?!

```diff
type CreatePersonPayload {
   record: Person
+  status: CreatePersonStatus! # fail, success, etc. ИЛИ 201, 403, 404 etc.
   # ... любые другие поля, которые пожелаете
}
```

Желательно, чтобы поле `status` было типом `Enum` с фиксированным набором значений. Тип статуса может быть уникальным, либо общим для всех мутаций – надо смотреть по ситуации.

### <a name="rule-6.6.3"></a> 6.6.3. В ответе мутации возвращайте поле с типом `Query`.

Если ваши мутации возвращают Payload-тип, то обязательно добавляейте в него поле `query` с типом `Query`.

```diff
type Mutation {
  likePost(id: 1): LikePostPayload
}

type LikePostPayload {
   record: Post
+  query: Query
}
```

Это позволит клиентам за один раунд-трип не только вызвать мутацию, но и получить вагон данных для обновления своего приложения. К примеру, мы лайкаем какую-то статью `likePost` и тут же в ответе через поле `query` можем запросить любые данные которые доступны нам в АПИ (в нашем примере список последних статей с активностью `lastActivePosts`).

```graphql
mutation {
  likePost(id: 1) {
    record {
      id
      title
      likes
    }
    query {
      lastActivePosts {
        id
        title
        likes
      }
    }
  }
}
```

Если мутация возвращает `query`, то для фронтендеров открывается дичайший профит – возможность сразу запросить любые данные для своего приложения после какой-нибудь страшной мутации за один http-запрос. А если на клиенте используются Relay или ApolloClient с именованными фрагментами, то обновить половину приложения становиться проще простого. Не надо писать второй запрос на получение данных и как-то пробрасывать их в нужное место. Всё обновиться магическим образом само, просто надо написать такую мутацию с существующими фрагментами из вашего приложения:

```graphql
mutation {
  likePost(id: 1) {
    query {
      ...LastActivePostsComponent
      ...ActiveUsersComponent
    }
  }
}
```

На стороне сервера прикрутить `query` в Payload-типе можно следующим образом:

Стандартная реализация с пакетом `graphql`:

```js
const QueryType = new GraphQLObjectType({ name: 'Query', fields: ... });
const MutationType new GraphQLObjectType({ name: 'Mutation', fields: () => {
  likePost: {
    args: { id: { type: new GraphQLNonNull(GraphQLInt) } },
    type: new GraphQLObjectType({
      name: 'LikePostPayload',
      fields: {
        record: { type: PostType },
        recordId: { type: GraphQLInt },
        query: { type: new GraphQLNonNull(QueryType) }, // ✨✨✨ magic – add 'query' field with 'Query' root-type
      },
    }),
    resolve: async (_, { id }, context) => {
      const post = await context.DB.Post.find(id);
      post.like();
      return {
        record: post,
        recordId: post.id,
        query: {}, // ✨✨✨ magic - just return empty Object
      };
    },
  }
}});
```

С помощью пакета `graphql-tools`:

```js
const typeDefs = gql`
  type Query { ...}
  type Post { ... }
  type Mutation {
    likePost(id: Int!): LikePostPayload
  }
  type LikePostPayload {
    recordId: Int
    record: Post
    # ✨✨✨ magic – add 'query' field with 'Query' root-type
    query: Query!
  }
`;

const resolvers = {
  Mutation: {
    likePost: async (_, { id }, context) => {
      const post = await context.DB.Post.find(id);
      post.like();
      return {
        record: post,
        recordId: post.id,
        query: {}, // ✨✨✨ magic - just return empty Object
      };
    },
  }
};
```

С помощью `graphql-compose`:

```js
schemaComposer.Mutation.addFields({
  likePost: {
    args: { id: 'Int!' },
    type: `type LikePostPayload {
      record: Post
      recordId: Int
      # ✨✨✨ magic – add 'query' field with 'Query' root-type
      query: Query!
    }`,
    resolve: async (_, { id }, context) => {
      const post = await context.DB.Post.find(id);
      post.like();
      return {
        record: post,
        recordId: post.id,
        query: {}, // ✨✨✨ magic - just return empty Object
      };
    },
  }
}});
```

### <a name="rule-6.6.4"></a> 6.6.4. В ответе мутации возвращайте поле `errors` с типизированными пользовательскими ошибками.

В резолвере можно выбросить эксепшн, и тогда ошибка улетает на глобальный уровень, но так делать нельзя по следующим причинам:

- ошибки глобального уровня используются для ошибок парсинга и других серверных ошибок
- на клиентской стороне тяжело разобрать этот массив глобальных ошибок
- клиент не знает какие ошибки могут возникнуть, они не типизированы и в схеме отсутствуют.

```diff
type Mutation {
  likePost(id: 1): LikePostPayload
}

type LikePostPayload {
   record: Post
+  errors: [LikePostProblems!]
}
```

Мутации должные возвращать пользовательские ошибки или ошибки бизнес-логики сразу в Payload'е мутации в поле `errors`. Все ошибки необходимо описать с суффиксом `Problem`. И для самой мутации завести Union-тип ошибок, где будут перечислены возможные пользовательские ошибки. Это позволит легко определять ошибки на клиентской стороне, сразу понимать что может пойти не так. И более того позволит клиенту дозапросить дополнительные метаданные по ошибке.

Для начала необходимо создать интерфейс для ошибок и можно объявить пару глобальных ошибок. Интерфейс необходим, чтобы можно было считать текстовое сообщение в не зависимости от того, какая ошибка вернулась. А вот каждую конкретную ошибку уже можно расширить дополнительными значениями, например в ошибке `SpikeProtectionProblem` добавлено поле `wait`:

```graphql
interface ProblemInterface {
  message: String!
}

type AccessRightProblem implements ProblemInterface {
  message: String!
}

type SpikeProtectionProblem implements ProblemInterface {
  message: String!
  # Timout in seconds when the next operation will be executed without errors
  wait: Int!
}

type PostDoesNotExistsProblem implements ProblemInterface {
  message: String!
  postId: Int!
}
```

Ну а дальше можно описать нашу мутацию `likePost` с возвратом пользовательских ошибок:

```graphql
type Mutation {
  likePost(id: Int!): LikePostPayload
}

union LikePostProblems = SpikeProtectionProblem | PostDoesNotExistsProblem;

type LikePostPayload {
  recordId: Int
  # `record` is nullable! If there is an error we may return null for Post
  record: Post
  errors: [LikePostProblems!]
}
```

Благодаря union-типу `LikePostProblems` теперь через интроспекцию фронтендеры знаю какие ошибки могут вернутся при вызове мутации `likePost`. К примеру для такого запроса они для любого типа ошибки смогут считать название ошибки с поля `__typename`, а вот благодаря интерфейсу считать `message` из любого типа ошибки:

```graphql
mutation {
  likePost(id: 666) {
    errors {
      __typename
      ... on ProblemInterface {
        message
      }
    }
  }
}
```

А если клиенты умные, то можно запросить дополнительные поля по необходимым по ошибкам:

```graphql
mutation {
  likePost(id: 666) {
    recordId
    record {
      title
      likes
    }
    errors {
      __typename
      ... on ProblemInterface {
        message
      }
      ... on SpikeProtectionProblem {
        message
        wait
      }
      ... on PostDoesNotExistsProblem {
        message
        postId
      }
    }
  }
}
```

И получить ответ от сервера в таком виде:

```js
{
  data: {
    likePost: {
      errors: [
        {
          __typename: 'PostDoesNotExistsProblem',
          message: 'Post does not exists!',
          postId: 666,
        },
        {
          __typename: 'SpikeProtectionProblem',
          message: 'Spike protection! Please retry later!',
          wait: 20,
        },
      ],
      record: { likes: 0, title: 'Post 666' },
      recordId: 666,
    },
  },
}
```
