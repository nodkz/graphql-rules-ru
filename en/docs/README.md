[CC-BY-NC-4.0](https://creativecommons.org/licenses/by-nc/4.0/)

# Graphql-scheme design — make API comfortable, prevent pain and suffering

Rules and recommendations mentioned in this paper were results of 3-years experience of using GraphQL both on front-end and back-end side. Also, we use recommendations and experience of Caleb Meredith (PostGraphQL author, Facebook ex-employee) and Shopify engineers.

This article could be changed in the future, cause current rules are advisory and may be improved, changed, or even become antipattern. But what is written here, suffered time and pain from the use of horrible GraphQL-schemes.

If you think that any rule is a complete mess or it is not fully disclosed, or want to add your own – please open the issue or hit me on the telegram by nickname @nodkz. Errors and typos can be corrected by clicking on the pencil in the upper right corner. I've only just started to make the rules, but to finish the job and stick it all together, only collaborative work can help us.

## TL;DR of all rules

- **1. Naming rules** 
  - [1.1.](./01-naming/1.1-fields-args.md) Use `camelCase` for GraphQL-fields and arguments.
  - [1.2.](./01-naming/1.2-types.md) Use `UpperCamelCase` for GraphQL-types.
  - [1.3.](./01-naming/1.3-enum.md) Use `CAPITALIZED_WITH_UNDERSCORES` to name ENUM-types.
- **2. Type rules** 
  - [2.1.](./02-types/2.1-custom-scalars.md) Use custom scalar types if you want to declare fields or args with specific semantic value.
  - [2.2.](./02-types/2.2-enumerable.md) Use Enum for fields which contain a specific set of values.
- **3. Fields Rules (Output)** 
  - [3.1.](./03-fields-output/3.1-semantic-names.md) Use semantic names for fields and avoid leaking of implementation details in fields names.
  - [3.2.](./03-fields-output/3.2-non-null-output.md) Use `Non-Null` field if field will always have a given field value.
  - [3.3.](./03-fields-output/3.3-grouping.md) Group as many related fields into custom Object type as possible.
- **4. Argument rules (Input)** 
  - [4.1.](./04-fields-input/4.1-grouping-input.md) Group coupled arguments to the new input-type.
  - [4.2.](./04-fields-input/4.2-custom-scalar-for-input.md) Use strict scalar types for arguments, eg. `DateTime` instead of `String`.
  - [4.3.](./04-fields-input/4.3-non-null-input.md) Mark arguments as `required`, if they are required for query execution.
- **5. Правила списков** 
  - 5.1. To filter the lists, use the `filter` argument, which contains all the available filters.
  - 5.2. Use argument `sort` of type `Enum` or `[Enum!]` for listings sorting.
  - 5.3 Use `limit` with default value and `skip` to limit number of returned items in list.
  - 5.4. Use `page`, `perPage` args for pagination and return output type with `items` (array of elements) and `pageInfo` (meta-data).
  - [5.5.](./05-list/5.5-cursor-connection.md) For infinite lists (infinite scroll) use [Relay Cursor Connections Specification](https://facebook.github.io/relay/graphql/connections.htm).
- **6. Mutation rules** 
  - 6.1. Use Namespace-types to group mutations within a single resource.
  - 6.2. Go beyond CRUD – create small mutations for different business operations on resources.
  - 6.3. Consider the ability to perform mutations on multiple items (same type batch changes).
  - 6.4. Mutations should clearly describe all the mandatory arguments, there should be no options either-either.
  - 6.5. In mutations, put all variables into one unique input argument.
  - [6.6.](./06-mutations/6.6-payload.md) Every mutation should have a unique payload type. 
    - [6.6.1.](./06-mutations/6.6.1-payload-record.md) In the mutation response, return the modified resource and its `id`.
    - [6.6.2.](./06-mutations/6.6.2-payload-status.md) Return operation status in mutation response.
    - [6.6.3.](./06-mutations/6.6.3-payload-query.md) In the mutation response, return a field of type `Query`.
    - [6.6.4.](./06-mutations/6.6.4-payload-errors.md) In the mutation response, return the `errors` field with typed user errors.
- **7. Rules of linkages between types (relationships)** 
  - 7.1. GraphQL schema should be "hairy"
- **10. Other rules** 
  - [10.1.](./10-misc/10.1-docs-markdown.md) Use markdown for documentation
- **A. Appendix** 
  - [A-1](./a-appendix/#A-1) Useful links