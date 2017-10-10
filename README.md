`do` Generators for ECMAScript
------------------------------

A convenient way of generating arrays/sets/maps of values in a contained expression. An alternative to [array comprehensions](http://tc39wiki.calculist.org/es6/array-comprehensions/).

## Motivation

A strong argument allowing traditional for comprehensions in the language is that when they become complex, they're often better structured as combinator methods with arrow functions. Therefore it's better to just start with combinators from the start to make that transition easier.

However, we've observed that when combinator chains become complex, rather than staying in the combinator form, people tend to break out of the combinator form and switch to imperative loops.

The idea around this proposal is to allow a more natural transition by making a convenient shorthand imperative style that can be expanded to more complex cases.

## `do` Generators

The primitive is a new kind of do expression that allows `yield`ing. It evaluates to a generator.

```js
let gen = do* {
  yield 1;
  yield 2;
};
```

This desugars to an immediately invoked generator function:

```js
let gen = (function*() {
  yield 1;
  yield 2;
})();
```

This can be combined with various initializers to create an Array, Maps or Set.

```js
let arrayOfUsers = [...do* {
  for (let user of users)
    if (user.name.startsWith('A'))
      yield user;
}];
```

```js
let arrayOfUsers = Array.from(do* {
  for (let user of users)
    if (user.name.startsWith('A'))
      yield user;
});
```

```js
let setOfUsers = new Set(do* {
  for (let user of users)
    if (user.name.startsWith('A'))
      yield user;
});
```

```js
let mapOfUsers = new Map(do* {
  for (let user of users)
    if (user.name.startsWith('A'))
      yield [user.id, user];
});
```

Since these do expressions naturally expand to more complex examples, you can keep expanding these with more complex logic as requirements expand. While still remaining in an isolated expression.

```js
let allUsers = Array.from(do* {
  let i = 0;
  for (let user of activeUsers) {
    i++;
    let isEven = (i % 2 === 0);
    let id = user.id;
    let name = user.name;
    if (id && name !== 'DELETED') {
      yield { id, name, isEven };
    }
  }
  for (let user of inactiveUsers) {
    i++;
    let isEven = (i % 2 === 0);
    yield { id: null, name: user.name, isEven };
  }
});
```

### Completion Values

Just like normal do-expressions, the completion value is like a return value. That means that the completion value is the final value in the generator.

This means that this yields a generator whose final value is `z`.

```js
let xyz = do* {
  for (let x in a)
    yield x;
  for (let y in b)
    yield y;
  z;
};
```

It desugars to:

```js
let xyz = (function* {
  for (let x in a)
    yield x;
  for (let y in b)
    yield y;
  return z;
})();
```

## Drop the `do` - It's cleaner

It might be possible to drop the `do` keyword and simply use `* {...}` instead. This provides a short hand form that still allows for new bindings to be declared outside the loop.

```js
let mapOfUsers = new Map(*{
  let i = 0;
  for (let user of users)
    if (user.name.startsWith('A') && (i++ % 2) === 0)
      yield [user.id, user];
});
```

This form could be different than `do *` in that completion values are not implicitly returned from the generator. Perhaps this should be the main proposal and `do * {}` is just a variant of `* {}`?

It looks pretty nice with spread:

```js
let arrayOfUsers = [...*{
  for (let user of users)
    if (user.name.startsWith('A'))
      yield user;
}];
```

## Implicit `*` Loop Expressions

A potential future enhancement to the array literal syntax could be used to have `for`/`while` expressions be implicit generator expressions while inside an array literal.

```js
let arrayOfUsers = [
  for (let user of users)
    if (user.name.startsWith('A'))
      yield user;
];
```

`do { } while()` expressions doesn't work as naturally in this position since that syntax is occupied by `do` expressions.

## `return` and `break` Statements

It's unclear what the `return` statement should do inside these generators. It could either return out of the outer function or abruptly stop the iteration of the generator. Unfortunately this leaves no possibility for an early return from a do-generator.

`break` without a label is easier. It should probably break out of the generator expression. However, with a label the control flow doesn't make as much sense since in a free standing generator, that scope doesn't exist anymore.

I'm leaning to just forbidding them for now. `throw` works just fine though.

## Related Proposals

[`do` expressions](https://github.com/tc39/proposal-do-expressions)

[Array comprehensions](http://tc39wiki.calculist.org/es6/array-comprehensions/)

## [Status of this Proposal](https://github.com/tc39/ecma262)

This has not yet been presented to TC39. I'm still gathering feedback.
