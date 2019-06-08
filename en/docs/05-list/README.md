## 5. Rules of lists

I never saw an API that does not return a list of items. Or it's page by page or something built on cursors for infinite lists. Списки надо фильтровать, сортировать, ограничивать кол-во возвращаемых элементов. Сам GraphQL никак не ограничивает свободу реализации, но для того чтобы сформировать некое единообразие, необходимо завести стандарт.

- **5. Rules of lists** 
  - 5.1. To filter the lists, use the `filter` argument, which contains all the available filters.
  - 5.2. Use argument `sort` of type `Enum` or `[Enum!]` for listings sorting.
  - 5.3 Use `limit` with default value and `skip` to limit number of returned items in list.
  - 5.4. Use `page`, `perPage` args for pagination and return output type with `items` (array of elements) and `pageInfo` (meta-data).
  - [5.5.](./5.5-cursor-connection.md) For infinite lists (infinite scroll) use [Relay Cursor Connections Specification](https://facebook.github.io/relay/graphql/connections.htm).