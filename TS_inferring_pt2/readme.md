# Typescript generics - stop writing tests & avoid runtime-errors. pt2


## TLDR:

This is The second chapter of the series where I show you the way how to avoid runtime-errors without writing `tests`. We'll use just strong Typescript inferring principles and generics.

You can copy-paste source code from examples into your IDE or [online Typescript playground](https://www.typescriptlang.org/play) and play with it by yourself.



**Chapters:**

1. [Inferring](https://dev.to/svehla/typescript-inferring-stop-writing-tests-avoid-runtime-errors-pt1-33h7)

2. Generics (current read)


In this chapter, we will look at more advanced type inferring and type reusing with Typescript generics.

In the previous chapter about typescript inferring we introduced

* `type inferring`
* `typeof`
* `&`
* `as const`
* `|`

So if you did not read it or you don’t fully understand these concepts or Typescript syntax, check [chapter 1](https://dev.to/svehla/typescript-inferring-stop-writing-tests-avoid-runtime-errors-pt1-33h7).

## Generics

Generics are crucial for our new inferring Typescript mindset. It enables us to perform real one-liner Typescript magic. With generics, we will be able to infer whatever we want.

## In this chapter, we will introduce

1. Generics + Type inferring

1. Type checking using an `extends` subset

1. Conditions inside of generics

1. Type inference in conditional types

1. Promise wrapper

1. Utility types

1. Custom generics utils

I don’t want to duplicate Typescript documentation so you should spend some time reading `generics` documentation for a better understanding of this series.

You can inspire yourself with useful resources like:

* https://www.typescriptlang.org/docs/handbook/generics.html
* https://www.typescriptlang.org/docs/handbook/advanced-types.html

So let’s look at a brief overview of Typescript features which we have to know.

### 1. Generics + Type inferring

One of the main tools for creating reusable components is `generics`. We will be able to create a component that can work over a variety of data types rather than a single one.

We can combine `generics` with Typescript inferring. You can easily create a `generic` which will be used as the argument of our new function.

```typescript
const unwrapKey = <T>(arg: { key: T }) => arg.key;
```

Now we will just call this function and get a type based on implementation.

```typescript

const unwrapKey = <T>(arg: { key: T }) => arg.key;
// ts infer value1 as string
const value1 = unwrapKey({ key: 'foo' });
// ts infer value1 as boolean
const value2 = unwrapKey({ key: true });
// ts infer value1 as true
const value3 = unwrapKey({ key: true } as const);
```

![unwrapKey](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt2/imgs/unwrapKey.gif)

Typescript dynamically infers arguments and returns the value of the function by extracting the data type of `<T>` which is passed as a `generic` value. The function is 100% type-safe even if the property `key` is type-agnostic.

Documentation: [https://www.typescriptlang.org/docs/handbook/generics.html](https://www.typescriptlang.org/docs/handbook/generics.html)

### 2. Type checking using an `extends` subset

The typescript keyword extends works as a subset checker for incoming data types. We just define a set of possible options for the current generic.

```typescript
const unwrapKey = <T extends boolean | number>(arg: { key: T }) => arg.key;
const ok = unwrapKey({ key: true });

const willNotWork = unwrapKey({
  value: 'value should be boolean or number'
});

```

![willNotWork](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt2/imgs/willNotWork.png)


Documentation:
https://www.typescriptlang.org/docs/handbook/generics.html#generic-constraints

### 3. Conditions inside of generics

There is another usage of `extends` keyword for checking if the type matches the pattern. If it does, Typescript applies a type behind the question mark `?`. If not, it uses the type behind the column `:`. It behaves the same way as the ternary operator in Javascript.

```typescript
type Foo<T> = T extends number
  ? [number, string]
  : boolean

const a: Foo<number> = [2, '3']
const b: Foo<boolean> = true
```

![foo](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt2/imgs/foo.png)


If the type of `T` is a `number`, the resulting type is a tuple if not, it‘s just boolean.

Documentation:
https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#conditional-types

This feature can be nicely used with Typescripts type-guards.
https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-guards-and-differentiating-types

### 4. Type inferring in conditional types

The typescript [keyword infer](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#type-inference-in-conditional-types) is a more advanced feature. It can infer a type inside of the generic type condition declaration like in the example below.

```typescript
type ReturnFnType<T> = T extends (...args: any[]) => infer R ? R : any;
const getUser = (name: string) => ({
  id: `${Math.random()}`,
  name,
  friends: [],
})
type GetUserFn = typeof getUser

type User = ReturnType<GetUserFn>
```
![returnFnType](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt2/imgs/returnFnType.png)

You will read more about ReturnType generic later in this chapter.

I’ll recommend reading the documentation for type inferring in condition types (and usage ofinfer keyword)
[https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#type-inference-in-conditional-types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#type-inference-in-conditional-types)

### 5. Promise Wrapper

Typescript also perfectly works with [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

There is a built-in `Promise<...>` generic which we will use in asynchronous operations. The `Promise` generic is just a wrapper that wraps your data into Promise “class”.

The Typescript has perfect Promise support for `async`, `await` syntax sugar such as:

```typescript
const getData = () => {
  return Promise.resolve(3)
}

// each async function wrap result into Promise()
const main = async () => {
  // await unwrap Promise wrapper
  const result = await getData()
}
```
![getAwaitData](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt2/imgs/getAwaitData.png)

### 6. Utility types

Typescript provides utility types to simplify common type transformations. These utilities are globally available in your project by default.

Documentation: [https://www.typescriptlang.org/docs/handbook/utility-types.html](https://www.typescriptlang.org/docs/handbook/utility-types.html)

We will focus on two of them `ReturnType<...>` and `Partial<...>`.

### 6.1 ReturnType<...>

ReturnType is an absolutely **phenomenal** Typescript feature which we will see in many more examples!

The definition of this generic looks like this:

```typescript
type ReturnType<T extends (...args: any) => any> =
  T extends (...args: any) => infer R
    ? R
    : any;
```

As you can see, ReturnType just takes some function and obtains the type of the return value. It enables us to perform more hardcore type inferring. Let’s take a look it in this example

```typescript
const getUser = (name: string) => ({
  id: Math.random(),
  name,
  isLucky: Math.random() % 2 === 0 
})
type User = ReturnType<typeof getUser>
```
![returnTypeTypeof](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt2/imgs/returnTypeTypeof.png)

This is a great feature for our new Typescript inferring programming mental model which we presented in the [previous chapter](https://dev.to/svehla/typescript-inferring-stop-writing-tests-avoid-runtime-errors-pt1-33h7).

Another cool example of `ReturnType<...>` is getting some specific read-only value from an object inside of a function.

```typescript
const foo = () => ({ foo: 'bar' } as const);
type FooReturnValue= ReturnType<typeof foo>
type bar = FooReturnValue['foo']
```

![fooReturnValue](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt2/imgs/fooReturnValue.png)

### 6.2 Partial<…>

In this example, we will use an `in keyof` syntax feature. If you want to know more about that, read advanced Typescript documentation. [https://www.typescriptlang.org/docs/handbook/advanced-types.html#index-types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#index-types).

Generic `Partial` definition looks like:

```typescript
/**
 * Make all properties in T optional
 */
type Partial<T> = {
  [P in keyof T]?: T[P];
};
```

As you can see, it just wraps a Javascript object and set its keys to be possibly undefined. A question mark after the key name makes the key optional. You can use this generic if you want to use just apart of an object.

```typescript
const user = {
  id: Math.random(),
  name: 'Foo',
  isLucky: Math.random() % 2 === 0
}

type PartialUser = Partial<typeof user>

```
![Partial](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt2/imgs/Partial.png)


### 7. Custom generics utils

In this section, we are going to create helper generics.

### 7.1 Await

`Await` is a utility generic which takes `Promise<...>` wrapped value and remove the `Promise` wrapper and leaves only extracted data.

Try to imagine that you already have `async` Javascript function. As we know, each `async` function wraps the result into a `Promise` generic wrapper. So if we call `ReturnType` for an async function, we get some value wrapped into `Promise<T>` generic.

We are able to extract a return value out of a Promise using `ReturnType<T>` and `Await<T>`:

```typescript

export type Await<T> = T extends Promise<infer R> ? R : T

// helper function to emit server delay
const delay = (time: number) => {
  return new Promise(res => {
    setTimeout(() => {
      res()
    }, time)
  })

}

const getMockUserFromServer = async () => {
  // some asynchronous business logic 
  await delay(2000)
  return {
    data: {
      user: {
        id: "12",
      }
    }
  }
}

type Response = Await<ReturnType<typeof getMockUserFromServer>>
```


![Await](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt2/imgs/Await.png)

It adds another possibility of inferring more advanced hidden data types in Javascript code.

### 7.2 RecursivePartial

This is just enhanced `Partial<...>` generic which we introduce a few paragraphs ago. The declaration looks like this:

```typescript
// inspiration: https://stackoverflow.com/a/51365037
type RecursivePartial<T> = {
  [P in keyof T]?:
    // check that nested value is an array
    // if yes, apply RecursivePartial to each item of it
    T[P] extends (infer U)[] ? RecursivePartial<U>[] :
    T[P] extends object ? RecursivePartial<T[P]> :
    T[P];
};
```

RecursivePartial is initially inspired by this Stack-overflow question https://stackoverflow.com/a/51365037 

As you see, it just recursively sets all keys of the nested object to be possibly `undefined`.

## Combine all generics to one monstrous masterpiece

Okay, we learned a lot about Typescript generics. Now we will combine our knowledge together in the next paragraphs.

Imagine that we have an application that makes calls to a backend service. Backend returns data about a currently logged user. For better development, we use mocked responses from the server. Our target is to extract the response data type from mocked API calls (like `getMeMock` function in the example).

We don’t believe in the correctness of the response from the server so we make all fields optional.

Let’s define our utils generics and just apply a one-line typescript sequence of generics to infer the type of `User` from the mock function.


```typescript
// ------------------- utils.ts ----------------------
// inspiration https://stackoverflow.com/a/57364353
type Await<T> = T extends {
  then(onfulfilled?: (value: infer U) => unknown): unknown;
} ? U : T;
// inspiration: https://stackoverflow.com/a/51365037
type RecursivePartial<T> = {
  [P in keyof T]?:
    T[P] extends (infer U)[] ? RecursivePartial<U>[] :
    T[P] extends object ? RecursivePartial<T[P]> :
    T[P];
};


// helper function to emit server delay
const delay = (time: number) => new Promise((res) => {
  setTimeout(() => {
    res();
  }, time);
});


// ----------------- configuration.ts ---------------
const USE_MOCKS = true as const;
// ----------------- userService.ts -----------------
const getMeMock = async () => {
  // some asynchronous business logic
  await delay(2000);
  return {
    data: {
      user: {
        id: '12',
        attrs: {
          name: 'user name'
        }
      }
    }
  };
};
const getMe = async () => {                     
  // TODO: call to server
  return getMeMock();
};

type GetMeResponse = Await<ReturnType<typeof getMeMock>>


type User = RecursivePartial<GetMeResponse['data']['user']>
```

![AwaitReturnType](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt2/imgs/AwaitReturnType.png)

Do you see it too? We took almost pure javascript code and using our Typescript utils, **we added only 2 lines of Typescript code and inferred all static data types from this Javascript implementation!** We can still write Javascript code and enhance it with Typescript micro annotations. All of that with a minimal amount of effort without boring interface typing.

And at top of it, every time you want to access some sub-property of User type, your IDE will automatically add an [optional chaining operator](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#optional-chaining) (name**?** ). Because we made all fields optional, accessing nested values can’t throw a new error.

![Alt Text](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_inferring_pt2/imgs/readUser.png)


If optional chaining does not work, you have to set up `“strictNullChecks”: true`, in your `tsconfig.json`

And that’s it! At this moment you’re able to infer whatever you want from your Javascript implementation and you’re able to use a type-safe interface without extra static types.

## Pay attention! Don’t overuse Generics!
> # Everytime you manually pass a static type as a parameter into a generic your code starts to smell
> #  — — Jakub Švehla — —

I believe that in your average code there are not big tricky functions with hard to understand data models. So please, don’t overthink your `generics`. Every time you create new `generic` think about if it’s necessary to create that kind of redundant abstraction which decreases code/type readability. So if you write a type by hand, be **strict** and **clear**. Generics are awesome especially for some **general-purpose** utility types (`ReturnType`, `Await`, Etc.). But be aware, generics in your custom data model could add extra unwanted complexity. So pay attention and use your brain and heart to do it well ❤️.

**Bad practice 😒**

```typescript
type UserTemplate<T> = { id: string, name: string } & T
type User1 = UserTemplate<{ age: number }>
type User2 = UserTemplate<{ motherName: string }>
type User = User1 | User2
```

**Good practice 🎉**

```typescript
type UserTemplate = { id: string, name: string }
type User1 = UserTemplate & { age: number }
type User2 = UserTemplate & { motherName: string }
type User = User1 | User2
```


**An alternative notation for the good practice 🎉**

```typescript
type User = {
  id: string,
  name: string
} & (
    { age: number }
  | { motherName: string }
)
```


### Conclusion

**[In chapter one](https://dev.to/svehla/typescript-inferring-stop-writing-tests-avoid-runtime-errors-pt1-33h7)** we learned the basics of Typescript and its features. We have new ideas about using static type inferring for Javascript.

In this chapter, we learned how to use generics, and when it’s appropriate to use them.

## Do You want more?
If you're interested in more advanced type usage, look to my other articles.


### `Object.fromEntries<T>`
Retype `Object.fromEntries` to support all kinds of tuples
https://dev.to/svehla/typescript-object-fromentries-389c

### `DeepMerges<T, U>`
How to implement `DeepMerge` for static types
https://dev.to/svehla/typescript-how-to-deep-merge-170c


*If you enjoyed reading the article don’t forget to like it to tell me that it makes sense to continue.*

