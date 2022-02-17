# TypeScript or F#?!? WHICH IS BETTER?!

Today I'll try to compare two different programming languages, **F#** and **TypeScript**.
Both of them are multiplatform, high-level, and statically-typed so we're able to use them to write any kind of type-safe application like web-servers, native-apps, web-applications or we can use them for daily scripting on your favourite operating system.

## Comparison categories

I would like to focus on a few concepts of programming languages that we are going to compare. Those categories are:

1. [Data-type inferring](#1-data-type-inferring)
2. [Generic inferring](#2-generic-inferring)
3. [Dynamic object types](#3-dynamic-object-types)
4. [Null pointer exceptions](#4-null-pointer-exceptions)
5. [Pattern matching](#5-pattern-matching)
6. [Modules import system](#6-modules-import-system)
7. [Runtime optimization](#7-runtime-optimization)
8. [Not owned by mega-corporate](#8-not-owned-by-megacorporate)

 
[TLDR: goto final conclusion](#final-conclusion)


## 1. Data-type inferring

F# is a strongly-typed language that implements an algorithm called `Hindley–Milner`. That algorithm analyzes your codebase and infers as much static types as possible.

If we want to write a function to sum two numbers in TypeScript, we have to define the data types for both arguments, and the TypeScript compiler will infer the Return type

**TS**
```ts
// this is a function which takes 2 arguments: `a`, `b`, the function sums those numbers and returns a value
// here we have to define the data types of both arguments
let sum = (a: number, b: number) => a + b
```
![fs obj example](https://raw.githubusercontent.com/Svehla/articles-backup/main/ts_vs_fs/imgs/ts_sum.png)

On the other hand, F# analyses the source code and finds that the `+` operator can be called on two integers so it infers the data types of the arguments. 

**F#**
```fsharp
// this is a function which takes 2 arguments: `a`, `b`, the function sums those numbers and returns a value
let sum a b = a + b
```
![fs obj exapmle](https://raw.githubusercontent.com/Svehla/articles-backup/main/ts_vs_fs/imgs/fs_sum.png)
> F# is running on `.net` ecosystem so it makes a difference between `float` and `int` while TypeScript does not

As you can see, F# code looks like another scripting language without static types but because of the terrific F# compiler it's a type-safe and strongly typed snippet.

Thanks to the `Hindley-Milner` algorithm in the F# compiler, F# is the winner.
[Unfortunatelly it looks like TypeScript has no plan to implement this kind of type inference.](https://github.com/microsoft/TypeScript/issues/11218)

**🏆 F# won**







## 2. Generic inferring

TypeScript use angle brackets `<T>` for generics. This is very similar to languages like `C++`, `Java` or `C#`. F# use just simple apostrophe `'T` for it.

In TypeScript, we're able to infer simple return values and variable declarations, but if you want to have dynamic arguments like in this example:

**TS**
```ts
const toDoubleTuple = <T>(anything: T): [T, T] => [anything, anything]
```
  
you have to define that the argument `anything` is of some generic type `T` and the function takes the type of `anything` and returns a tuple of `[anything, anything]`

As you may suggest, F# is so smart that it can even infer generic types of our function.

**F#**
```fsharp
let toDoubleTuple anything = anything, anything
```

![fs head](https://raw.githubusercontent.com/Svehla/articles-backup/main/ts_vs_fs/imgs/fs_toDoubleTuple.png)

Again, F# analyzes the source code and finds that if a function takes `anything` of type `'a`, it returns a tuple of `anything, anything` of type `'a * 'a`. F# compiler infers that argument is of Generic type `'a`. This code looks like a dynamic scripting language similar to Python but it has a powerful 100% type-safe runtime.

**🏆 F# won**







## 3. Dynamic object types

Now, let's compare how to create data-type and structural data objects. Let's start with F#.

**F#**
```fsharp
type User = {
  id: string
  name: string
}

let kuba = {
  id = "my-id-123"
  name = "Kuba"
}
```

![fs obj example](https://raw.githubusercontent.com/Svehla/articles-backup/main/ts_vs_fs/imgs/fs_obj.png)

As you can see, F# has `structurally based data object types`. It means that F# compiler finds that there exists an object type with attributes `id: string` and `name: string` and it automatically infers its data type.

In TypeScript, if we create a data type, we have to assign that data type to a variable.

**TS**
```ts
type User = {
  id: string
  name: string
}
// assign the data type to the variable
let kuba: User = {
  id: 'my-id-123',
  name: 'Kuba'
}
```

It's verbose so we have an alternative in TypeScript and we can infer the data type directly from the data by using the `typeof` keyword.

**TS**
```ts
let kuba = {
  id: 'my-id-123',
  name: 'Kuba'
}
type User = typeof kuba
```

![ts obj exapmle](https://raw.githubusercontent.com/Svehla/articles-backup/main/ts_vs_fs/imgs/ts_obj.png)

Thanks to the TypeScript approach, we may use more advanced generics like `Omit<...>`, `Pick<...>`, `keyof` and so on.

F# structural-based data object types are awesome but here the TypeScript has much more powerful, nicer and minimalist tools to define dynamic data object types.

**🏆 TypeScript won**







## 4. Null pointer exceptions

In JavaScript we have a few nullable values: `null`, `undefined`, `0`, `NaN`, `""`. This makes it difficult to handle the JavaScript runtime properly. Thanks to TypeScript we can check the types more strictly.

**TS**
```ts
let pipePrint = <T>(data: T | undefined | null) => {
  if (data === undefined || data === null || isNan(data)) {
    console.log('no value provided')
  } else {
    console.log(data)
  }
}
```

F# decided not to implement those nullable values and focused on forcing you to handle edge-cases strictly. So in the F# core there is a defined union type called [`Option`](https://fsharpforfunandprofit.com/posts/the-option-type/)
It's defined as:

**F#**
```fsharp
type Option<'a> =       // use a generic definition
   | Some of 'a           // valid value
   | None                 // missing value
```

If we wrap some value in that `Option` type, we're able to check if the value exists or if the value is empty.

**F#**

```fsharp
let pipePrint data = 
   match data with
     | Some x -> printf x
     | None -> printf "no value provided"
```

Thanks to that, F# compiler forces us to handle all non-valid potential null pointer exception errors, which is good. But in TypeScript, we have the same result if we type 100% of our codebase correctly. So in this category it is a draw and there is no winner.

**🏆 no winner**







## 5. Pattern matching

Pattern matching is a really powerful part of F# language design. There are many sources where you can find more information, like [there](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/pattern-matching) or [there](https://fsharpforfunandprofit.com/posts/match-expression/).

TLDR:
Pattern matching in JavaScript/TypeScript is bad, not flexible, and bad again. So there the F# is the winner.

I put here one of many examples of the powerfulness of pattern-matching in F#.

**F#**
```fsharp
let vectorLength vec =
    match vec with
    | [| var1 |] -> var1
    | [| var1; var2 |] -> sqrt (var1 * var1 + var2 * var2)
    | [| var1; var2; var3 |] -> sqrt (var1 * var1 + var2 * var2 + var3 * var3)
    | _ -> failwith (sprintf "vectorLength called with an unsupported array size of %d." (vec.Length))
```

**🏆 F# won**







## Modules import system

F# has [linear module system](https://fsharpforfunandprofit.com/posts/cyclic-dependencies/) with [namespaces](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/namespaces) and [modules](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/modules).

TypeScript has a few module systems like commonjs, ES6, and so on. Today we're gonna talk about [ES6 imports](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import).

To be honest, I don't like namespaces in programming languages like C#, Java, PHP, and so on. I prefer a module system where each file is its namespace and `import` & `export` are the only sign if the value is local, or if the value can be imported into a different module.

**F#**
```fsharp
// file-a.fs
module FileA
let sum a b = a + b

// file-b.ts
module FileB
open FileA

```

ES6 modules enable us to have cyclic dependencies where names of files are part of our application architecture. In a language like C# and F# we have to create a file plus add a module or namespace declaration. I prefer a more minimalistic way of defining modules so I prefer the ES6 way.

**TS**
```ts
// file-a.ts
export const sum = (a, b) => a + b


// file-b.ts
import { sum } from './file-a.ts'
```


**🏆 TypeScript won**







## 7. Runtime optimization

Both languages have really strong type inference systems. This means that the compiler will check the code and suggest (infer) the best static type to use. Thanks to that, you are able not to define static-type in the code on your own. It helps you to make a better development experience with less writing, but more readable, self-documented, and less error-prone code.

TypeScript is a highly dynamic language so if we compile our TypeScript code into the vanilla JavaScript, it removes static types and we're not able to use those metadata to optimize JavaScript runtime like memory allocation and CPU time.

On the other hand, thanks to the fact that TypeScript is a tool for a better developer experience, we're able to use expressions like `as any`, `: any`, `@ts-expect-error`, `@ts-ignore` and not to be too strict.

It means that both approaches have pros and cons, so there is no winner.

**🏆 no winner**







## 8. Not owned by mega-corporate

As you may know, F# is developed by Microsoft and it runs on the .net platform created by Microsoft as well. TypeScript is also created by Microsoft but the final output is pure vanilla JavaScript which is not owned by any large mega-corporate. Thanks to that, it gave us (as developers) the option not to be locked in one ecosystem of some mega-corporate and we are able to feel more free and independent. 

Maybe this point could be stupid for you but I believe that it's better to write code in a language that is not so directly connected to anyone and there is more democracy.


**🏆 TypeScript won**







## Final conclusion

So, Let's check the results of categories:

|                                         | F# | TS |
| --------------------------------------- | -- | -- |
| 1. Data-type inferring                  | 🏆 |    |
| 2. Generic inferring                    | 🏆 |    |
| 3. Dynamic object types                 |    | 🏆 |
| 4. Null pointer exceptions              | -- | -- |
| 5. Pattern matching                     | 🏆 |    |
| 6. Modules import system                |    | 🏆 |
| 7. Runtime optimization                 | -- | -- |
| 8. Not owned by mega-corporate          |    | 🏆 |

So as you you can see, it's hard to decide which language I like more. 
In conclusion, my dream language will be F# with Javascript ES6 module system and object data inferring via `typeof`.


## Is there a solution?

So at the end of this article, it is worth mentioning that there is an awesome compiler [fable.io](https://fable.io/) that brings F# to the JavaScript ecosystem. 

On the next screenshot you can see demo example where `fable.io` transpile one of our example from F# into the JavaScript.

![fable example](https://raw.githubusercontent.com/Svehla/articles-backup/main/ts_vs_fs/imgs/fable_example.png)


**If you enjoyed reading the article don’t forget to like it to make my day**

