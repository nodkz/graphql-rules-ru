## 1. Naming rules

GraphQL uses regexp `/[_A-Za-z][_0-9A-Za-z]*/` to validate property and type names. That means you can use `camelCase`, `under_score`, `UpperCamelCase`, `CAPITALIZED_WITH_UNDERSCORES`. Pay attention that `kebab-case` is not supported at all.

Which name convention is better to use?

We could refer to [Eye Tracking'а](http://www.cs.kent.edu/~jmaletic/papers/ICPC2010-CamelCaseUnderScoreClouds.pdf) research, that tried to analyze `camelCase` and `under_score`. But that research didn't find any favourites.

For the current moment, we don't have any whitepapers to suggest something scientifically. But we want to give the following advice.

- **1. Naming rules** 
  - [1.1.](./1.1-fields-args.md) Используйте `camelCase` для именования GraphQL-полей и аргументов.
  - [1.2.](./1.2-types.md) Используйте `UpperCamelCase` для именования GraphQL-типов.
  - [1.3.](./1.3-enum.md) Используйте `CAPITALIZED_WITH_UNDERSCORES` для именования значений ENUM-типов.