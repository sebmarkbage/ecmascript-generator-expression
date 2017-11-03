Generator Expressions for ECMAScript
------------------------------------

A convenient way of generating arrays/sets/maps of values in a contained expression. An alternative to [array comprehensions](http://tc39wiki.calculist.org/es6/array-comprehensions/).

## Motivation

A strong argument allowing traditional for comprehensions in the language is that when they become complex, they're often better structured as combinator methods with arrow functions. Therefore it's better to just start with combinators from the start to make that transition easier.

However, we've observed that when combinator chains become complex, rather than staying in the combinator form, people tend to break out of the combinator form and switch to imperative loops.

The idea around this proposal is to allow a more natural transition by making a convenient shorthand imperative style that can be expanded to more complex cases.

## Generator Expressions

The primitive is a new kind of expression that allows `yield`ing. It evaluates to a generator.

```js
let gen = *{
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
let arrayOfUsers = [...*{
  for (let user of users)
    if (user.name.startsWith('A'))
      yield user;
}];
```

```js
let arrayOfUsers = Array.from(*{
  for (let user of users)
    if (user.name.startsWith('A'))
      yield user;
});
```

```js
let setOfUsers = new Set(*{
  for (let user of users)
    if (user.name.startsWith('A'))
      yield user;
});
```

```js
let mapOfUsers = new Map(*{
  let i = 0;
  for (let user of users)
    if (user.name.startsWith('A') && (i++ % 2) === 0)
      yield [user.id, user];
});
```

Since these generator expressions naturally expand to more complex examples, you can keep expanding these with more complex logic as requirements expand. While still remaining in an isolated expression.

```js
let allUsers = new Set(*{
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

If the code complexity keeps expanding to requiring a temporary array, the next refactor step is to switch to a `do` expression.

```js
let allUsers = new Set(do {
  let tmp = [];
  let i = 0;
  for (let user of activeUsers) {
    i++;
    let isEven = (i % 2 === 0);
    let id = user.id;
    let name = user.name;
    if (id && name !== 'DELETED') {
      tmp.push({ id, name, isEven });
    }
  }
  for (let user of inactiveUsers) {
    i++;
    let isEven = (i % 2 === 0);
    tmp.push({ id: null, name: user.name, isEven });
  }
  tmp.reverse();
});
```

### Completion Values

Just like do-expressions, the completion value is like a return value. That means that the completion value is the final value in the generator. This is an advanced and uncommon feature. In most common uses of this feature, the completion value is ignored. E.g. when used to construct Arrays or Sets.

This means that this yields a generator whose final value is `z`.

```js
let xyz = *{
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

This can be useful for scheduling code like Task.js.

```js
let promise = Task(*{
  const [foo, bar] = yield Task.join(
    read("foo.json"),
    read("bar.json")
  );
  foo.x + bar.y;
});
```

## `break` Statements

`break` without a label will break out of the generator. It behaves like returning the completion value of a generator function. Early returns can be accomplished using `break`.

```js
let promise = Task(*{
  const [foo, bar] = yield Task.join(
    read("foo.json"),
    read("bar.json")
  );
  if (!bar) {
    foo.x;
    break;
  }
  foo.x + bar.y;
});
```

You can break to labels inside of the generator expression just like normal. However, with a label defined outside the generator expression, the control flow doesn't make as much sense since in a free standing generator, that scope doesn't exist anymore. That's a syntax error.

```js
foo: {
  let items = [...*{
    break foo; // SyntaxError!
  }];
}
```

## `return` Statements

It's unclear what the `return` statement should do inside these generators. It could either return out of the outer function or abruptly stop the iteration of the generator. I'm leaning to just forbidding `return` for now.

`throw` works just fine though.

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

## Related Proposals

[`do` expressions](https://github.com/tc39/proposal-do-expressions)

[Array comprehensions](http://tc39wiki.calculist.org/es6/array-comprehensions/)

## [Status of this Proposal](https://github.com/tc39/ecma262)

This has not yet been presented to TC39. I'm still gathering feedback.
