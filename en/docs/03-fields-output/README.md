## 3. Fields Rules (Output)

В GraphQL поля используются для передачи данных с сервера на клиент. Они могут быть описаны обычными скалярнвми типами, а также сложными структурами – Output-типами (GraphQLObjectType).

- **3. Fields Rules (Output)** 
  - [3.1.](./3.1-semantic-names.md) Use semantic names for fields and avoid leaking of implementation details in fields names.
  - [3.2.](./3.2-non-null-output.md) Делайте поля обязательными `NonNull`, если данные в поле возвращаются при любой ситуации.
  - [3.3.](./3.3-grouping.md) Group as many related fields into custom Object type as possible.