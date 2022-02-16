# Typescript inferring - stop writing tests & avoid runtime-errors. pt1

 

## TLDR:

This is the first chapter of the series where I show you the way how to avoid runtime-errors without writing `static types` and `tests` using strong Typescript inferring principles.

You can copy-paste source code from examples into your IDE or [online Typescript playground](https://www.typescriptlang.org/play) and play with it by yourself.

**“Minimalist Typescript” chapters:**

1. Inferring (current read)

2. [Generics](https://dev.to/svehla/typescript-generics-stop-writing-tests-avoid-runtime-errors-pt2-2k62)

## Introduction

The whole article series is about changing the Typescript mindset on how to use minimalistic static-types in modern Javascript projects. The problem with Typescript is that when programmers discover static-types they start to overuse and over-engineer them. This results in transforming our beloved Javascript into a language similar to C# or Java.

We’re going to try to forget standard type-safe interface best practices where programmers have to create type interface APIs for everything and then implement business logic compatible with these interface declarations. We can see that in the diagram below where two modules (you can also imagine function etc..)communicate via some abstract interface in the middle.

```md
## approach 1

                     +-------------+
                     |  interface  |
            +--------+-----+-------+-----------+
            |              |                   |
            |              |                   |
    +-------v----+         |            +------v------+
    |   module 1 |         |            |  module 2   |
    |            |         |            |             |
    +------------+         |            +-------------+
                           |
```

Ughh… We are Javascript developers and we love dynamic prototyping, that is the reason why the diagram does not look very nice to me. I want to have a type-safe code without runtime errors but at the top of it. I don’t want to write static-types by hand at all. The good news is that Typescript has tools that can help us to “obtain” static-types (known as *inferring*) from pure Javascript implementation. And that's it. Inferring is the key to this whole Typescript series.

**Type Inferring** enables the compiler to generate type interfaces in the compile-time and check the correctness of our implementation. We will be able to use inferring for creating logical connections between layers of programming abstraction (*like functions/files/and so on*).
The final code should be type-safe without writing extra type interface APIs like in the diagram below.

```md
## approach 2

    +---------------+   interface 2   +----------------+
    |               +---------------> |                |
    |               |                 |                |    
    | module 1      |    interface 1  |  module 2      |
    |               |                 |                |
    |               | <---------------+                |
    +---------------+                 +----------------+
```

Our goal is to alter our mindset to think that we will just continue with writing our **good old dynamic Javascript.** But we will get an extra type-safe layer based on our implementation.

## Let’s change the mindset!


Do you remember when you were 15 and started to learn C?

```c
int main() {
  int a = 3;
  int b = 4; 
  int c = a + b;
  return 0;
}
```

I don’t like that I have to define that a variable `c` is an integer because it’s obvious! Variables `a` and `b` are integers so `a + b` should return integer too!

We can forgive this behavior because C is almost 50 years old and a low-level programming language that is not suitable for quick prototyping in the application layer but It’s fast as hell.

### Remove redundant data types

Let’s look at how can we write strongly-typed Javascript and avoid writing redundant type annotations.

First of all, we are going to rewrite the previous C function into Typescript exactly the same way.


```typescript
const main = (): number => {
  const a: number = 3
  const b: number = 4
  const c: number = a + b
  return c
}
```

![main ts 1](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt1/imgs/main-ts-1.gif)

Ugh… awful right?
Hmm so let’s apply Typescript “*type inference*”.

```typescript
const main = () => {
  const a = 3
  const b = 4
  const c = a + b
  return c
}
```

![main ts 2](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt1/imgs/main-ts-2.gif)

This looks way better. Typescript is smart and understands that `3` is a `number` and plus operator returns a `number`.

**Type inferring** is a Typescript feature that can “obtain” (*infer*) data types from your code implementation. As you can see in the demo, Typescript checks the code, infers types of variables, and performs static analyses. The beauty of that solution is that 100% of your code is pure Javascript just enhanced by static-type checking.

### Advanced Typescript “inferring”

This is a crucial feature that separates Typescript from other type-safe programming languages.

The problem with pure Javascript started with an escalating number of lines of code. Your brain (and `unit tests`😃) is just a thin layer that has to check if your newly implemented refactored data structures are compatible with the rest of your code. When you are done with your code you have to check that your documentation is compatible with your latest implementation.

Typescript can fully work like your brain and perform static analyses of code without extra typing by hand. For example, you may write code like:

```typescript
const foo = ({ bar, baz }) => [bar, baz]
```

You as a programmer have no idea what type of `bar` and `baz` are. Obviously, Typescript has no idea about that as well.

Let’s compare the previous example with the next one:

```typescript
const main = () => {
  const bar = 3
  const baz = 4
  const foo = { bar, baz } 
  return [foo.bar, foo.baz]
}
```

![main ts 3](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt1/imgs/main-ts-3.gif)

It’s much clearer in this more “spaghetti-like” implementation. Variables `foo` and `bar` are just `numbers`.

Don’t forget that if your code contains many “redundant” layers of abstraction, code readability rapidly decreases. In the first example, our brain had no idea what variables `bar` and `baz` were.

Many people start to get frustrated with incomprehensible, unclear code, and start to write functions with type interfaces like this one:

```typescript
type FooArg = {
  bar: number,
  baz: number
}
const foo = ({ bar, baz }: FooArg) => [bar, baz]]
```

In this example, we add an extra 4 lines just for typing an interface of the `foo` micro function. Then the code grows, the codebase starts to get less flexible and you have just lost the flexibility of Javascript.

> # The best statically analyzed code without type defining is the spaghetti one!
> #  — — Jakub Švehla — —

## Skip redundant interface definition — use `typeof`

Do you know the DRY **(Don’t repeat yourself)** programming philosophy? 
Every time you create a type interface with defined keys and so on, you start to duplicate your code (and one cat will die).

```typescript
const user = {
  id: 3,
  name: 'Foo'
}
```

vs

```typescript
type User = {
  id: number
  name: string
}
const user: User = {
  id: 3,
  name: 'Foo'
}
```

We can solve this issue with the Typescript `typeof` type guard, which takes a Javascript object and infers data types from it.

```typescript
const user = {
  id: 3,
  name: 'Foo'
};
type User = typeof user 
```

![typeof-preview](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt1/imgs/typeof-preview.png)

You can see that this new code does not create declaration duplicates and our Javascript object is the source of truth for the type `User`. And at the top of it, we can still use Typescript types to check the correctness of the code implementation.

The next example demonstrates how type-checking finds an issue in the code using just 2 lines of Typescript code.

```typescript
const user = {
  id: 3,
  name: 'Foo'
};
type User = typeof user
const changeUserName = (userToEdit: User, age: number) => {
  userToEdit.name = age;
};

```
![change-user-name](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt1/imgs/change-user-name.png)


If Typescript is not able to 100% correctly infer your static types, you can help the compiler by defining a sub-value of an object with `as` syntax. In this example: `state: 'nil' as 'nil' | 'pending' | 'done'` we set that the state attribute contains only `nil`, `pending` or `done` value.

```typescript
const user = {
  id: 3,
  name: 'Foo',
  // Help the compiler to correctly infer string as the enum optional type
  state: 'nil' as 'nil' | 'pending' | 'done'
};
type User = typeof user
const changeUserName = (useToEdit: User, newName: string) => {
  useToEdit.name = newName;
  useToEdit.state = 'pendingggggg';
};
```

![userToEdit](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt1/imgs/userToEdit.png)


as you can see:
> # the only place where you have to define static types by hand are just function arguments

and the rest of the code can be inferred by the Typescript compiler. If you want to be more strict with inferring you can help the Typescript compiler by using the `as` keyword and write a more strict type inferring Javascript code.

> You should not forget that a newly defined function outside of the previous function scope is just another “reusable” layer of abstraction which decreases compiler & human readability and simplicity.

### Algebraic data type — Enumerated values

One of the best features of Typescript is `Pattern matching` based on *enumerated values*.

Let’s have 3 types of animals. Each kind of animal has different attributes. Your target is to create the custom print function differently for each of your animals.

Your data model layer could look like:

```typescript

const elephantExample = {
  trunkSize: 10,
  eyesColor: 'red'
}
const pythonExample = {
  length: 50
}
const whaleExample = {
  volume: 30
}
```

First of all, we can simply get static types from values by using the `typeof` keyword.

```typescript
type Elephant = typeof elephantExample
type Python = typeof pythonExample
type Whale = typeof whaleExample
type Animal = 
  | Elephant
  | Python
  | Whale
```

![animal](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt1/imgs/animal.png)

Let’s add a `type` attribute for each of our animals to make a unique standardized way of identifying an “instance” of the animal type and check the correctness of objects.

```typescript
// & operator merge 2 types into 1
type Elephant = typeof elephantExample & { type: "Elephant" }
type Python = typeof pythonExample & { type: "Python" }
type Whale = typeof whaleExample & { type: "Whale" }
type Animal = 
  | Elephant
  | Python
  | Whale
const animalWhale: Animal = {
  type: "Whale",
  volume: 3
}
const animalWhaleErr: Animal = {
  length: 100,
  type: "Whale",
}
```

![length-error](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt1/imgs/length-error.png)


You can see that we use the Typescript `&` operator for merging two Typescript’s data types.

Now we can create a print function that uses a `switch-case` pattern matching over our inferred javascript object.

![print Animal Attrs](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt1/imgs/printAnimalAttrs.gif)

```typescript
const elephantExample = {
  trunkSize: 10,
  eyesColor: 'red'
}
const pythonExample = {
  length: 50
}
const whaleExample = {
  volume: 30
}

// & operator merge 2 types into 1
type Elephant = typeof elephant & { type: "Elephant" }
type Python = typeof python & { type: "Python" }
type Whale = typeof whale & { type: "Whale" }

type Animal = 
  | Elephant
  | Python
  | Whale

const printAnimalAttrs = (animal: Animal) => {
  // define custom business logic for each data type
  switch (animal.type) {
    case 'Elephant':
      console.log(animal.trunkSize)
      console.log(animal.eyesColor)
      break
    case 'Python':
      console.log(animal.size)
      break
    case 'Whale':
      console.log(animal.volume)
      break
  }
}
```

As you see in this example, we just took a simple Javascript code and added a few lines of types for creating relations between data structures and function arguments. The beauty of that solution is that Typescript does not contain *business logic or *data shape declaration so Javascript code is **the only source of truth**. Typescript still checks 100% of your source code interface compatibility and adds a nice self-documentation feature.

### Use `as const` for constant values

Typescript has an `as const` syntax feature that helps with defining constant values instead of basic data types. If the Typescript compiler found an expression like:

![css-type](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt1/imgs/css-type.png)

it obviously infers `justifyContent` key as a `string`. But we as programmers know that `justifyContent` is an enum with values:
 `'flex-start' | 'flex-end' | 'start' | .. | .. | etc ...`

We have no option of getting this `justifyContent` data-type information from the code snippet because CSS specification is not related to the Typescript specification. So let’s transform this static object into a type with exact compile-time values. To do this, we are going to use an `as const` expression.

![css-as-const](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt1/imgs/css-as-const.png)

Now we can use `justifyContent` as a `readonly` constant value `flex-start`.

In the next example, we combine `as const`, `as`, and `typeof` for a one-line configuration type interface.

![configuration-typeof](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt1/imgs/configuration-typeof.png)

### Conclusion

In this chapter, we went through the basics of Typescript smart inferring. We used Typescript as a type-safe **glue** for our Javascript code. We were also able to get perfect IDE help and documentation with a minimal amount of effort.

**We learned how to:**

* Infer and check basic data types.

* Add static types for arguments of a function.

* Use `typeof` for inferring Typescript types from a static Javascript implementation.

* Merge type objects with `&` operator.

* Make options types with `|` operator.

* Use `switch-case` pattern matching on different data types.

* Use `as {{type}}` for correcting inferred data types.

* Use `as const` for type values.

### Next chapter:

* **[In chapter 2](https://dev.to/svehla/typescript-generics-stop-writing-tests-avoid-runtime-errors-pt2-2k62)**, we will look at more advanced type inferring and type reusing with Typescript generics. In the second part of the article, we will declare custom generics for “inferring” from external services.

*If you enjoyed reading the article don’t forget to like it to tell me that it makes sense to continue.*
