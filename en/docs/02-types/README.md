## 2. Type rules

GraphQL specification has 5 scalar types - `String`, `Int`, `Float`, `Boolean`, `ID` (string with unique identifier). All that types easily transmitted using JSON and available in every programming language.

But when it all comes to some scalar type that doesn't belong to JSON syntax, eg. `Date`. In that case, backend developer has to figure out data format on his own, so that format would be serializable and transmittable using JSON. Get the value of this type from a client and deserialize it back on the server.

In such cases, GraphQL allows you to create your own custom scalar types. [This article](../types/README.md#custom-scalar-types) describes it.

- **2. Type rules** 
  - [2.1.](./2.1-custom-scalars.md) Use custom scalar types if you want to declare fields or args with specific semantic value.
  - [2.2.](./2.2-enumerable.md) Use Enum for fields which contain a specific set of values.