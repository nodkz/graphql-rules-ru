## 7. 7. Rules of linkages between types (relationships)**

The conceptual difference between GraphQL and REST API is that the implementation of the logic of obtaining related resources was transferred from the client to the server. While with REST API clients are wondering (without hypermedia) how to request related resources, writing a layer of gluing/pre-dragging data on the client. With GraphQL, people who perfectly understand their data domain are engaged in that business, and they do that much faster and more efficiently. Especially when the related resources are fetched within the server, the client does not spend a lot of time on long round-trips between the client and the server for subqueries. И вы не ограничены 4-мя браузерными подключениями на домен, внутри сервера отправляйте хоть 200 одновременных запросов, если хватает мощей.

- **7. Правила связей между типами (relationships)** 
  - [7.1.](./7.1-hairy-graphql.md) GraphQL-схема должна быть "волосатой"