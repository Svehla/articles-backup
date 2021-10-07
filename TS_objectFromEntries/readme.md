# How to add types for Object.fromEntries

Step by step tutorial on how to create a proper type for `Object.fromEntries()` which can work with tuples and read-only data structures.


## TLDR:
Source code for `Object.fromEntries` type generic is at bottom of the article.
You can copy-paste it into your IDE and play with it.


VS-code preview


```typescript

const data = [
  ['key1', 'value1' as string],
  ['key2', 3]
]  as const

const result = Object.fromEntries(data)
```


![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/434sbq3rq8t9hq33gn0e.png)

## Motivation
The default typescript type for `Object.fromEntries` definition looks like this

```typescript
interface ObjectConstructor {
  // ...
  fromEntries(entries: Iterable<readonly any[]>): any;
}
```

As you can see the usage of return value `: any` it's not the best one. So we will redeclare static types for this method via the usage of the strongest Typescript tools which are described below.



## Prerequisite
before we will continue we have to know the typescript keyword `infer` and some basic generics usage.
https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#type-inference-in-conditional-types

If you want to deep dive into advanced typescript types I recommend this typescript series full of useful examples.

- Basic static types inferring: https://dev.to/svehla/typescript-inferring-stop-writing-tests-avoid-runtime-errors-pt1-33h7

- More advanced generics https://dev.to/svehla/typescript-generics-stop-writing-tests-avoid-runtime-errors-pt2-2k62



---------------------------------


## Let's start hacking

First of all, we will define `Cast<X, Y>` generic which helps us to build our target `FromEntries<T>` type.

### `Cast<X, Y>` 

This generic help us to bypass the typescript compiler for passing invalid types. We will use `Cast<X, Y>` to "shrink" a union type to the other type which is defined as the second parameter.

```typescript
type Cast<X, Y> = X extends Y ? X : Y
```

Preview

```typescript
type T4 = string | number
type T5 = Cast<T4, string>
```

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/9kna6qn3ouxt9wdt5m25.png)

Okay... it should be enough for this moment. We can start with the `FromEntries<T>` generic.

## `FromEntries<T>`

So let's define a new type `FromEntriesV1<T>`. It takes one argument `T` and checks if the argument is a two-dimensional matrix `[any, any][]` if yes, create proper type. if no return default behavior which just returns unknown untyped Object `{ [key in string]: any } `.

```typescript
type FromEntriesV1<T> = T extends [infer Key, any][]
  // Cast<X, Y> ensure TS Compiler Key to be of type `string`
  ? { [K in Cast<Key, string>]: any }
  : { [key in string]: any } 
```

```typescript
type ResFromEV1 = FromEntriesV1<[
  ['key1', 'value1'],
  ['key2', 3],
]>
```

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/8r1k2ir4zpzmtpcp9zpu.png)


<sub>*It works the same even without `Cast<Key, string>` generic but Typescript compiler still warning you that there is a potential error so we have to bypass it with the `Cast<X, Y>`*</sub>

This generic works thanks to `infer` which extracts out all keys into a union type which is used as target object keys.

Now we have to set the correct values of the object but before we will do it let's introduce another generics `ArrayElement<A>`.


### `ArrayElement<A>`

this simple generic helps us to extract data outside of an `Array<T>` wrapper.

```typescript
export type ArrayElement<A> = A extends readonly (infer T)[]
  ? T
  : never
```

Preview

```typescript
type T1 = ArrayElement<['foo', 'bar']>
```



```typescript
const data = ['foo', 'bar'] as const
type Data = typeof data
type T2 = ArrayElement<Data>
```

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/cf8ngi1a1vcb83qgmd2u.png)


Okey we can continue with adding proper `value` into the new object. We just simply set that value is second item of nested tuple `ArrayElement<T>[1]`.

```typescript
type FromEntriesV2<T> = T extends [infer Key, any][]
  ? { [K in Cast<Key, string>]: ArrayElement<T>[1] }
  : { [key in string]: any }
```

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/0jais3wttdvg0jfhsve3.png)

we successfully extracted all possible values but as we can see, there is a missing connection between `key` and `value` in our new type.


If we want to fix it we have to know another generic `Extract<T>`. `Extract<T>` is included in the official standard typescript library called `utility-types`.

This generic is defined as:

```typescript
type Extract<T, U> = T extends U ? T : never;
```
official documentation: https://www.typescriptlang.org/docs/handbook/utility-types.html#extracttype-union

Thanks to this generic we can create connections between keys and values of nested tuples

```typescript
type FromEntries<T> = T extends [infer Key, any][]
  ? { [K in Cast<Key, string>]: Extract<ArrayElement<T>, [K, any]>[1] }
  : { [key in string]: any }
```

Preview

```typescript
type Result = FromEntries<[
  ['key1', 'value1'],
  ['key2', 3],
]>
```

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/gxdy5w6ud2wwg2ca3c4i.png)

And... that's all!!! Good job! we did it 🎉 now the generics can transfer an Array of tuples into object type.

_________________________________________

Oh, wait. there is still some major issues which we should solve

Generic does not work well with readonly Notations like in the example below

```typescript
const data = [['key1', 1], ['key2', 2]] as const
type Data = typeof data
type Res = FromEntries<Data>
```

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/9y8ybguqoqn0bs7tdsya.png)

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/ul6786dg5dnyuut7od2p.png)

To resolve this issue let's introduce another generic `DeepWriteable`

### `DeepWriteable<T>`

this generic is used to recursively remove all `readonly` notations from the data type. 
If you create type by `typeof (data as const)` all keys start with the `readonly` prefix so we need to remove it to make all objects consistent.


```typescript
type DeepWriteable<T> = { -readonly [P in keyof T]: DeepWriteable<T[P]> }
```

Preview

```typescript
const data = ['foo', 'bar'] as const
type Data = typeof data
type T3 = DeepWriteable<Data>
```

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/xwmuko5wmxg63nwhcv9p.png)


With this new knowledge, we can fix unexpected behavior and make it all works again.

```typescript
const data = [['key1', 1], ['key2', 2]] as const
type Data = typeof data

type T6 = FromEntries<DeepWriteable<Data>>
```

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/83f4as0aq9xmajvh4i40.png)




## Final source code + Redeclare global Object behavior

If you don't know what `declare {module}` annotations in typescript is, You can check official documentation https://www.typescriptlang.org/docs/handbook/modules.html

We will use this feature to redeclare the global type behavior of `Object.fromEntries`.

All you need to do is just paste the code below to your `index.d.ts` or `global.d.ts`.


```typescript

export type ArrayElement<A> = A extends readonly (infer T)[] ? T : never;
type DeepWriteable<T> = { -readonly [P in keyof T]: DeepWriteable<T[P]> };
type Cast<X, Y> = X extends Y ? X : Y
type FromEntries<T> = T extends [infer Key, any][]
  ? { [K in Cast<Key, string>]: Extract<ArrayElement<T>, [K, any]>[1]}
  : { [key in string]: any }

export type FromEntriesWithReadOnly<T> = FromEntries<DeepWriteable<T>>


declare global {
   interface ObjectConstructor {
     fromEntries<T>(obj: T): FromEntriesWithReadOnly<T>
  }
}
```


And voilá 🎉 🎉 🎉 🎉 🎉 🎉 
We are done

I hope that you enjoyed this article the same as me and learned something new. If yes don't forget to like this article 
