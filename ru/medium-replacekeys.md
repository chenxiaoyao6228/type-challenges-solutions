---
id: 1130
title: ReplaceKeys
lang: ru
level: medium
tags: object-keys
---

## Проблема

Реализовать тип `ReplaceKeys`, который заменяет ключи в объединениях.
Если какого-то ключа нет в элементе, просто пропускаем.
Такой тип принимает три аргумента.
Например:

```typescript
type NodeA = {
  type: "A";
  name: string;
  flag: number;
};

type NodeB = {
  type: "B";
  id: number;
  flag: number;
};

type NodeC = {
  type: "C";
  name: string;
  flag: number;
};

type Nodes = NodeA | NodeB | NodeC;

// would replace name from string to number, replace flag from number to string
type ReplacedNodes = ReplaceKeys<
  Nodes,
  "name" | "flag",
  { name: number; flag: string }
>;

// would replace name to never
type ReplacedNotExistKeys = ReplaceKeys<Nodes, "name", { aa: number }>;
```

## Решение

У нас на руках объединение интерфейсов и нам нужно по ним итерироваться.
В процессе итерации, нужно заменить ключи на требуемые, переданные через параметры.
Дистрибутивность здесь, очевидно, сильно поможет, как и сопоставляющие типы.

Я начну с такого небольшого факта, как "сопоставляющие типы в TypeScript также дистрибутивные".
То есть, если вы напишете сопоставляющий тип, который принимает объединение - поведение будет дистрибутивным.
Таким образом, мы сможем итерироваться как по объединению, так и по ключам самого интерфейса одновременно.
Давайте постараюсь объяснить.

Вы уже наверняка знаете, что условные типы дистрибутивные.
Такое свойство языка нам очень часто помогало в проблемах ранее.
Каждый раз, когда мы писали `U extends any ? U[] : never`, мы получали внутри правдивой ветки один элемент со всего объединения на каждой итерации.
Компилятор делает это за нас.

Аналогичное свойство присутствует и в сопоставляющих типах.
Мы можем написать сопоставляющий тип и, если на входе объединение, компилятор будет неявно применять дистрибутивность.
Следовательно, мы будем работать с отдельным элементом из объединения.

Так что давайте начнем с простого.
Мы возьмем каждый элемент из `U` (спасибо дистрибутивности) и на каждом элементе возьмем список ключей, вместе с его типами значений.

```typescript
type ReplaceKeys<U, T, Y> = { [P in keyof U]: U[P] };
```

Таким образом, мы скопировали один в один всё, что пришло через параметр `U`.
Теперь, нам нужно применить фильтр и работать только с теми ключами, которые есть в `T` и `Y`.

Для начала проверим, а есть ли текущее свойство в параметре `T` (в списке ключей, которые нам нужно обновить).

```typescript
type ReplaceKeys<U, T, Y> = {
  [P in keyof U]: P extends T ? never : never;
};
```

Если это так, это значит что разработчик попросил нас поменять указанный ключ.
И, скорее всего, он указал и тип, на который нужно произвести замену.
Но, мы не можем быть уверенны в этом.
Поэтому, дополнительно проверим, что это свойство присутствует в `Y`.

```typescript
type ReplaceKeys<U, T, Y> = {
  [P in keyof U]: P extends T ? (P extends keyof Y ? never : never) : never;
};
```

И только в ситуации, когда оба условия правдивые, мы можем быть уверенны в необходимости замены.
Берём тип из `Y` и производим замену.

```typescript
type ReplaceKeys<U, T, Y> = {
  [P in keyof U]: P extends T ? (P extends keyof Y ? Y[P] : never) : never;
};
```

Однако, если окажется, что такого ключа нет в `Y`, но он есть в `T`, нам нужно вернуть `never` (согласно постановке задачи).
Остается последний вариант, когда оба условия неправдивые.
При таком варианте, мы просто возвращаем тот же тип, который был и в оригинальном интерфейсе.

```typescript
type ReplaceKeys<U, T, Y> = {
  [P in keyof U]: P extends T ? (P extends keyof Y ? Y[P] : never) : U[P];
};
```

Имея дистрибутивные сопоставляющие типы, мы смогли реализовать очень даже читаемое и поддерживаемое решение.
Без них, мы были бы вынуждены использовать дистрибутивные условные типы с дальнейшим применением сопоставляющих внутри условного типа.
Что, очевидно, не так хорошо смотрится, как это решение.

## Что почитать

- [Сопоставляющие типы](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)
- [Условные типы](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)
- [Индексные типы поиска](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html)
