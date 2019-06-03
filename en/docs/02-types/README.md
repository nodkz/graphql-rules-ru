## 2. Type rules

GraphQL specification has 5 scalar types - `String`, `Int`, `Float`, `Boolean`, `ID` (string with unique identifier). All that types easily transmitted using JSON and available in every programming language.

But when it all comes to some scalar type that doesn't belong to JSON syntax, eg. `Date`. In that case backend developer has to figure out data format on his own, so that format would be serializable and transmittable using JSON. А также получить значение этого типа от клиента и обратно десериализовать его на сервере.

В таких случаях GraphQL позволяет создавать свои кастомные скалярные типы. О том как это делается [написано здесь](../types/README.md#custom-scalar-types).

- **2. Type rules** 
  - [2.1.](./2.1-custom-scalars.md) Используйте кастомные скалярные типы, если вы хотите объявить поля или аргументы с определенным семантическим значением.
  - [2.2.](./2.2-enumerable.md) Используйте Enum для полей, которые содержат определенный набор значений.