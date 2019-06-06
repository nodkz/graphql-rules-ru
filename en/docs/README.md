[CC-BY-NC-4.0](https://creativecommons.org/licenses/by-nc/4.0/)

# Дизайн GraphQL-схем — делаем АПИ удобным, предотвращаем боль и страдания

Rules and recommendations mentioned in this paper were results of 3-years experience of using GraphQL both on front-end and back-end side. Also, we use recommendations and experience of Caleb Meredith (PostGraphQL author, Facebook ex-employee) and Shopify engineers.

This article could be changed in the future, cause current rules are advisory and may be improved, changed, or even become antipattern. Но то что здесь написано, выстрадано временем и болью от использования кривых GraphQL-схем.

**Если вы считаете, что какое-либо правило полная лажа или оно не полностью раскрыто, или хотите добавить свое – пожалуйста откройте issue или стукните меня в телеграмме по нику @nodkz.** Ошибки и опечатки можно поправить, нажав на карандашик в правом верхнем углу. Я только начал формировать правила, но довести дело до конца и привести всё божеский вид – мы сможем только вместе.

## TL;DR всех правил

- **1. Naming rules** 
  - [1.1.](./01-naming/1.1-fields-args.md) Используйте `camelCase` для именования GraphQL-полей и аргументов.
  - [1.2.](./01-naming/1.2-types.md) Используйте `UpperCamelCase` для именования GraphQL-типов.
  - [1.3.](./01-naming/1.3-enum.md) Используйте `CAPITALIZED_WITH_UNDERSCORES` для именования значений ENUM-типов.
- **2. Type rules** 
  - [2.1.](./02-types/2.1-custom-scalars.md) Используйте кастомные скалярные типы, если вы хотите объявить поля или аргументы с определенным семантическим значением.
  - [2.2.](./02-types/2.2-enumerable.md) Используйте Enum для полей, которые содержат определенный набор значений.
- **3. Fields Rules (Output)** 
  - [3.1.](./03-fields-output/3.1-semantic-names.md) Давайте полям понятные смысловые имена, а не то как они реализованы.
  - [3.2.](./03-fields-output/3.2-non-null-output.md) Делайте поля обязательными `NonNull`, если данные в поле возвращаются при любой ситуации.
  - [3.3.](./03-fields-output/3.3-grouping.md) Достигайте максимальной группировки взаимосвязанных полей.
- **4. Argument rules (Input)** 
  - [4.1.](./04-fields-input/4.1-grouping-input.md) Группируйте взаимосвязанные аргументы вместе в новый input-тип.
  - [4.2.](./04-fields-input/4.2-custom-scalar-for-input.md) Используйте строгие скалярные типы для аргументов, например `DateTime` вместо `String`.
  - [4.3.](./04-fields-input/4.3-non-null-input.md) Помечайте аргументы как `NonNull`, если они обязательны для выполнения запроса.
- **5. Правила списков** 
  - [5.1.](./05-list/5.1-filter.md) Для фильтрации списков используйте аргумент `filter`, который содержит в себе все доступные фильтры.
  - [5.2.](./05-list/5.2-sort.md) Для сортировки списков используйте аргумент `sort`, который должен быть `Enum` или `[Enum!]`.
  - [5.3.](./05-list/5.3-limit-skip.md) Для ограничения возвращаемых элементов в списке используйте аргументы `limit` со значением по умолчанию и `skip`.
  - [5.4.](./05-list/5.4-pagination.md) Для пагинации используйте аргументы `page`, `perPage` и возвращайте output-тип с полями `items` с массивом элементов и `pageInfo` с метаданными для удобной отрисовки страниц на клиенте.
  - [5.5.](./05-list/5.5-cursor-connection.md) Для бесконечных списков (infinite scroll) используйте [Relay Cursor Connections Specification](https://facebook.github.io/relay/graphql/connections.htm).
- **6. Mutation rules** 
  - [6.1.](./06-mutations/6.1-mutation-namespaces.md) Используйте Namespace-типы для группировки мутаций в рамках одного ресурса!
  - [6.2.](./06-mutations/6.2-business-operations.md) Выходите за рамки CRUD – cоздавайте небольшие мутации для разных бизнес операций над ресурсами.
  - [6.3.](./06-mutations/6.3-batch-changes.md) Рассмотрите возможность выполнения мутаций сразу над несколькими элементами (однотипные batch-изменения).
  - [6.4.](./06-mutations/6.4-required-args.md) У мутаций должны быть четко описаны все обязательные аргументы, не должно быть вариантов либо-либо.
  - [6.5.](./06-mutations/6.5-input-arg.md) У мутации вкладывайте все переменные в один уникальный `input` аргумент.
  - [6.6.](./06-mutations/6.6-payload.md) Мутация должна возвращать свой уникальный Payload-тип. 
    - [6.6.1.](./06-mutations/6.6.1-payload-record.md) В ответе мутации возвращайте измененный ресурс и его `id`.
    - [6.6.2.](./06-mutations/6.6.2-payload-status.md) В ответе мутации возвращайте статус операции.
    - [6.6.3.](./06-mutations/6.6.3-payload-query.md) В ответе мутации возвращайте поле с типом `Query`.
    - [6.6.4.](./06-mutations/6.6.4-payload-errors.md) В ответе мутации возвращайте поле `errors` с типизированными пользовательскими ошибками.
- **7. Правила связей между типами (relationships)** 
  - [7.1.](./07-relations/7.1-hairy-graphql.md) GraphQL-схема должна быть "волосатой"
- **10. Other rules** 
  - [10.1.](./10-misc/10.1-docs-markdown.md) Use markdown for documentation
- **A. Appendix** 
  - [A-1](./a-appendix/#A-1) Useful links