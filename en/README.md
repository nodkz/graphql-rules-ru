## 1. Правила именования

GraphQL для проверки имен полей и типов использует следующую регулярку `/[_A-Za-z][_0-9A-Za-z]*/`. Согласно нее можно использовать `camelCase`, `under_score`, `UpperCamelCase`, `CAPITALIZED_WITH_UNDERSCORES`. Слава богу `kebab-case` ни в каком виде не поддерживается.

Так что же лучше выбрать для именования?

Абстрактно можно обратиться к исследованию [Eye Tracking'а](http://www.cs.kent.edu/~jmaletic/papers/ICPC2010-CamelCaseUnderScoreClouds.pdf) по `camelCase` и `under_score`. В этом исследовании явного фаворита не выявлено.

Коль исследования особо не помогло. Давайте будем разбираться в каждом конкретном случае.

### <a name="rule-1.1"></a> 1.1. Используйте `camelCase` для именования GraphQL-полей и аргументов.

Поля:

```diff
type User {
+  isActive: boolean # GOOD
-  is_active: boolean # BAD
}
```

Аргументы:

```diff
type Query {
+  users(perPage: Int): boolean # GOOD
-  users(per_page: Int): boolean # BAD
}
```

Названиями полей активнее всего пользуются потребители GraphQL-апи, т.е. наши любимые клиенты — браузеры с JavaScript и разработчики мобильных приложений. Давайте посмотрим, что чаще всего используется по их конвенции для именования переменных. Ведь если клиенты дергают ваше апи, то скорее всего они будут использовать ваше именование для переменных у себя в коде. Ведь маппить (алиасить) названия полей в удобный формат не шибко интересная работа.

Согласно [википедии](https://en.wikipedia.org/wiki/Naming_convention_(programming)) следующие клиентские языки (потребители GraphQL-апи) придерживаются следующих конвенций по именованию переменных:

- JavaScript — `camelCase`
- Java — `camelCase`
- Swift — `camelCase`
- Kotlin — `camelCase`

Конечно каждый у себя "на кухне" может использовать `under_score`. Но в среднем по больнице используется `camelCase`. Если найдете какое-нибудь исследование по процентовке использования `camelCase` и `under_score` в том или ином языке программирования — дайте пожалуйста знать, очень тяжело гуглится вся эта тема. Кругом сплошной субъективизм.

А ещё, если залезть в кишки graphql и посмотреть его [IntrospectionQuery](https://github.com/graphql/graphql-js/blob/master/src/utilities/introspectionQuery.js), то он также написан используя `camelCase`.

PS. Мне очень печально видеть в документации MySQL или PostgreSQL примеры с названием полей через `under_score`. Потом все это дело кочует в код бэкенда, далее появляется в GraphQL апи, а потом и в коде клиентов. Блин, ведь эти БД спокойно позволяют сразу объявлять поля в `camelCase`. Но так как на начале не определились с конвенцией имен, а взяли как в документации, то потом происходят холивары что `under_score` лучше чем `camelCase`. Ведь переименовывать поля на уже едущем проекте больно и чревато ошибками. В общем тот кто выбрал `under_score`, тот и должен маппить поля в `camelCase` для клиентов, т.е. бэкендер!

### <a name="rule-1.2"></a> 1.2. Используйте `UpperCamelCase` для именования GraphQL-типов.

А вот именование типов, в отличии от полей уже происходит немного по другому.

```diff
- type blogPost { # BAD
- type Blog_Post { # so-so
+ type BlogPost { # GOOD
    title: String!
  }
```

В самом GraphQL уже есть скалярные типы `String`, `Int`, `Boolean`, `Float`. Они именуются через `UpperCamelCase`.

Также внутренние типы GraphQL-интроспекции `__Type`, `__Field`, `__InputValue` и пр. Именуются через `UpperCamelCase` с двумя символами подчеркивания в начале.

А еще GraphQL статический типизированный язык запросов. И из GraphQL-запросов много кто генерирует тайп-дефинишены для статического анализа кода. Так вот если посмотреть как в JS именуют сложные типы во Flowtype и TypeScript — то тут тоже обычно используется `UpperCamelCase`.

И опять таки согласно [википедии](https://en.wikipedia.org/wiki/Naming_convention_(programming)) по конвенции для классов и деклараций типов используется `UpperCamelCase` в JavaScript, Java, Swift и Kotlin.

### <a name="rule-1.3"></a> 1.3. Используйте `CAPITALIZED_WITH_UNDERSCORES` для именования значений ENUM-типов.

Enum в GraphQL используются для перечисления списка возможных значение у некого типа.

```diff
enum Sort {
+  NAME_ASC # GOOD
-  nameAsc # BAD
-  NameAsc # BAD
   NAME_DESC
   AGE_ASC
   AGE_DESC
}
```

В самом GraphQL для интроспекции в типе `__TypeKind` используются следующие значения: `SCALAR`, `OBJECT`, `INPUT_OBJECT`, `NON_NULL` и другие. Т.е. используется `CAPITALIZED_WITH_UNDERSCORES`.

Если относиться к Enum-типу как к типу с набором констант. То в большинстве языков программирования константы именуются через `CAPITALIZED_WITH_UNDERSCORES`.

Также `CAPITALIZED_WITH_UNDERSCORES` будет хорошо смотреться в самих GraphQL-запросах, чтобы четко идентифицировать Enum'ы:

```graphql
query {
  findUser(sort: ID_DESC) {
    id
    name
  }
}
```