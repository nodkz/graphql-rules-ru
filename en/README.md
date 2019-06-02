## 1. Правила именования

GraphQL для проверки имен полей и типов использует следующую регулярку `/[_A-Za-z][_0-9A-Za-z]*/`. Согласно нее можно использовать `camelCase`, `under_score`, `UpperCamelCase`, `CAPITALIZED_WITH_UNDERSCORES`. Слава богу `kebab-case` ни в каком виде не поддерживается.

Так что же лучше выбрать для именования?

Абстрактно можно обратиться к исследованию [Eye Tracking'а](http://www.cs.kent.edu/~jmaletic/papers/ICPC2010-CamelCaseUnderScoreClouds.pdf) по `camelCase` и `under_score`. В этом исследовании явного фаворита не выявлено.

Коль исследования особо не помогло. Давайте будем разбираться в каждом конкретном случае.

- **1. Правила именования** 
  - [1.1.](./1.1-fields-args.md) Используйте `camelCase` для именования GraphQL-полей и аргументов.
  - [1.2.](./1.2-types.md) Используйте `UpperCamelCase` для именования GraphQL-типов.
  - [1.3.](./1.3-enum.md) Используйте `CAPITALIZED_WITH_UNDERSCORES` для именования значений ENUM-типов.