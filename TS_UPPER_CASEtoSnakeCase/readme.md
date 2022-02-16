# How to UPPER_CASE to camelCase in raw Typescript generics

## TLDR:
Today's challenge is to retype an `UPPER_CASE` static string into `camelCase` and apply this transformation recursively to the object keys.

Preview
![upperCaseToCamelCase](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/upperCaseToCamelCase.png)

![transformKeysToCamelCase](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/transformKeysToCamelCase.png)


As you can see, we transformed static type text written in `UPPER_CASE` format into `camelCase`. Then we applied the transformation recursively to all the object keys.


[You can play with full source code here](https://www.typescriptlang.org/play?target=1#code/C4TwDgpgBAMg9gdwgJwCpwKpkmu8nIDCAhgM4QCyx2KUAvFAN4CwAUFFMQFxQDkAgrzYcARj14AhIeygBjcYWkcAJuIAiSqBHEBRTQDNxAMU0BzcQHFNAC3EAJTQEtxASU0ArcQClNAa3EA0poANuIwmgC24hSaAHbiAHKacOIA8ppg4gAKmgCO4gCKmsjiAEqapOIAyprA4qiaAK7iGJoAbuIAapoI4gDqmgAe4gAamiDiAJqaAF7iAFrSAL5sbKCQUFg46PgoJORUNMj0TMJQ-OLEmhLiIpqE4rKaauLKmjriEJpG4vqaFuJTJo7OJrJoXOJHJovOJ3JoAuJfJoYOJgpoKOIIpoEuJYppUuI4JosuIwJoCuJcppSuJkJoquJSJpUOJgJoMOJGppOuI2po+uIEJoRuJBppJuIQJp5uIZstVqw1uBoHYIMRlDAIMBgCgADyoAB8J1QWkGOtiylIUAABgASRiOWL6WhGRzIUjATXalBLe2O53HAD6pQgHqW1qgAH4oK73Z6tTrjjxYhA2iglRtUMRHMEvYnSPqjQwTRAzRALVa7Q6nbRA7GPXmfX6a8cQ2GI9G28AoMnU+nFax1tBG7gtnrDcbTebLTbmwGY26Gwmm9X58HQ8Bw2do-X497jqXp1bfBAQHB9LBECh0GPcLsiGRKNQcGcONH7zejjsrw+Ds+UAA2ruI4ALqvj2C5xiOZw8KgCpDrAy53j+haToe5YzlW-ououe6Jr6q61l2W4yDuuEjlOGHHqe56bF+eA-vsT5HOB0a3t+BBMYcOBAeRSFgTIHA8MBSEwVAcEDghHF7I+qHFpRFZ8Jo0YSUJs6MCO0nILqqrqiOhYGgRWlMfq2a5khBaGoZ1oKgA9LZkEelAwRIVAjhWo0RxQMgG5uVawQ-hm0DoFU5bmrIEAmRODBVppmBHDpaoakhBlGQxnGyVmOYjpZBrWfByriXAWRkLIxDBFFRbiQpmE2aR4liSWZaKVhLaOcAfRwMgyhLIGc60F2OUkW+6khWF5YRSZu6dd1hn2ugJWkGVFWyYNFn5TIsFwKFsThZFmUGgqpZgF13YIbeTHoCQEQQCt5ByaN6UyfdunJfuqXzWZOX6sVpXlZVG1sPZnDYMEIBQKQsTECeciPm5u1wFAcAiO4ECyMApBBVA+zAPqAA0mxVU1R6bFG1U8BgWMWFqqQo505WNKGD3E1RUAhrIXXKLqxCxCABPYccnRGtGnQQSmabIAVGwkMEwTs407qOGmqDIDzpD6F1EQuPotPuMzNVWuznPc7zBM8yAwviarsTq5rASnqQV3EDdd0QKhsFHYMJ3IGdhVVAgjjALI1j2yA9PBIzupnKgePRwAjAbbNo8bHrII6phm7zVUsIJUAAQE8NQCeZ4XqgIE8IwRenjwAQANxQG0DPaOJ+cgVAKwyEsscyKgABMJw5xwHD54X1PALr4eM5ZccGgBvCNxHXwgeXUA6Gaqvo7qY8T0308GgTlcL4zNft7PvDF7wAkcB3RO91LwXW7byARKHjtwNdt0mdHidG91uqp+nTOFtu4cFQAABgHnnAujoq4l3EivC6j4nYuxMgEI0Xd44nH9oHYOodJ5u3AfvaO-cGCVxHjA4utFUBxxXjLOWaMFakCVhAFWasNbP21rrfUAEcb6jjq3AmADYimANCBdBbBb4KixgkDcEBlChxDBLbsxZH7sJfg7ZBn9ZKDxjKkVIgYJD8BpBDYAadhFnH4AkVIqA7A6FKHWPRBijE8FMYzKAAAfKAsRGgRBECgEBUAEg6CqKgHQahAwBB0BKU4ucgkhIcQSExZjTDgTiagJxxiRBwDgC5HmZwMGsCWIdVgQA)


Typescript 4.2 is already in beta version, so we should be prepared for new incoming features to fully use the power it offers. You can find all the new Typescript 4.2 features there: https://devblogs.microsoft.com/typescript/announcing-typescript-4-2-beta/

## Let's deep dive into code

To change the case from `UPPER_CASE` to camel Case, we have to use the parser to convert uppercase letters to lowercase ones and to remove unwanted `_`.  


### Letters mapper
First of all, we create the Lower/Upper Mapper type which describes dependencies between lowercase and uppercase letters.

```typescript
type LowerToUpperToLowerCaseMapper = {
  a: 'A'
  b: 'B'
  c: 'C'
  d: 'D'
  e: 'E'
  f: 'F'
  g: 'G'
  h: 'H'
  i: 'I'
  j: 'J'
  k: 'K'
  l: 'L'
  m: 'M'
  // ... and so on 
}

type UpperToLowerCaseMapper = {
  A: 'a'
  B: 'b'
  C: 'c'
  // ... and so on 
}

```

### Parse strings Utils

We have to write a small parser that will read `UPPER_CASE` format and try to parse it to the new structure which will be transformed into `camelCase`. So let's start with a text parser util function.

#### HeadLetter

This generic infers the first letter and just returns it.

```typescript
type HeadLetter<T> = T extends `${infer FirstLetter}${infer _Rest}` ? FirstLetter : never

```

![headLetter](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/headLetter.png)

#### TailLetters

This generic infers all the letters except the first one and returns them.

```typescript
type TailLetters<T> = T extends `${infer _FirstLetter}${infer Rest}` ? Rest : never
```

![tailLetters](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/tailLetters.png)


#### LetterToUpper

This generic calls the proper LowerCase Mapper structure to convert one char.

```typescript
type LetterToUpper<T> = T extends `${infer FirstLetter}${infer _Rest}`
  ? FirstLetter extends keyof LowerToUpperToLowerCaseMapper
    ? LowerToUpperToLowerCaseMapper[FirstLetter]
    : FirstLetter
  : T
```

![letterToUpper](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/letterToUpper.png)

#### LetterToLower

```typescript
type LetterToLower<T> = T extends `${infer FirstLetter}${infer _Rest}`
  ? FirstLetter extends keyof UpperToLowerCaseMapper
    ? UpperToLowerCaseMapper[FirstLetter]
    : FirstLetter
  : T
```

![letterToLower](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/letterToLower.png)

#### ToLowerCase

Now we're albe to recursively call `HeadLetter`, `Tail` and `LetterToLower` to iterate through the whole `string` and apply lowecase to them.

```typescript

type ToLowerCase<T> = T extends ''
  ? T
  : `${LetterToLower<HeadLetter<T>>}${ToLowerCase<TailLetters<T>>}`

``` 

![toLowerCase](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/toLowerCase.png)

#### ToSentenceCase

This generic transforms the first letter to uppercase and the rest of the letters to lowercase.

```typescript
type ToSentenceCase<T> = `${LetterToUpper<HeadLetter<T>>}${ToLowerCase<TailLetters<T>>}`

```


![toSentenceCase](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/toSentenceCase.png)


We're done with all our utils Generics, so we can jump into the final type implementation.

### UpperCaseToPascalCase

We're almost there. Now we can write the generic which will transform `CAMEL_CASE` into `PascalCase`.

```typescript
type ToPascalCase<T> = T extends ``
  ? T
  : T extends `${infer FirstWord}_${infer RestLetters}`
  ? `${ToSentenceCase<FirstWord>}${ToPascalCase<RestLetters>}`
  : ToSentenceCase<T>
```

As you can see, we recursively split words by `_` delimiter. Each word converts to `Sentencecase` and joins them together.

![toPascalCase](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/toPascalCase.png)

### UpperCaseToCamelCase
The last step is to use `PascalCase` but to keep the first letter of the first word lowercase.

We use previously created generics and just combine them together.

```typescript
export type UpperCaseToCamelCase<T> = `${ToLowerCase<HeadLetter<T>>}${TailLetters<ToPascalCase<T>>}`
```

![upperCaseToCamelCase](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/upperCaseToCamelCase-2.png)


Pretty amazing and kinda simple code, right? 

## Apply case transformation to object keys

Now we want to build a static type that applies recursively `UpperCaseToCamelCase` generic to Object nested keys.

Before we start, let's define three helper generics.


#### GetObjValues
```typescript
type GetObjValues<T> = T extends Record<any, infer V> ? V : never
```

This simple generic helps us to extract data outside of a `Record<any, T>` wrapper.

![getObjValues](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/getObjValues.png)


#### Cast

This generic helps us to bypass the Typescript compiler to pass invalid types. We will use Cast<X, Y> to "shrink" a union type to the other type which is defined as the second parameter.


```typescript
type Cast<T, U> = T extends U ? T : any
```

```typescript
type T4 = string | number
type T5 = Cast<T4, string>
```

![cast](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/cast.png)

#### SwitchKeyValue

We use our previously defined generic `GetObjValues<T>` to switch the to the value. 

The goal of this generic is to transform the string value into the key and vice-versa, like in the preview.

```typescript
type Foo = SwitchKeyValue<{ a: 'key-a', b: 'key-b' }>
```

![switchKeyValue](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/switchKeyValue.png)


```typescript
type GetObjValues<T> = T extends Record<any, infer V> ? V : never

export type SwitchKeyValue<
  T,
  // step 1
  T1 extends Record<string, any> = {
    [K in keyof T]: { key: K; value: T[K] }
  },
  // step 2
  T2 = {
    [K in GetObjValues<T1>['value']]: Extract<GetObjValues<T1>, { value: K }>['key']
  }
> = T2
```


The whole procedure takes, two steps so I decided to keep the code less nested and to save partial values into variables. Sub results variables are saved due to generic parameters. Thanks to that Typescript feature I can "save" the results of transformations into "variables" `T1` and `T2`. This is a pretty useful pattern of writting static types with less nesting.

Everything works fine so let's dive into recursive nested keys transformation.

### TransformKeysToCamelCase

Now we will combine the generics from the whole article into one single piece of art.


```typescript

type TransformKeysToCamelCase<
  T extends Record<string, any>,
  T0 = { [K in keyof T]: UpperCaseToCamelCase<K> },
  T1 = SwitchKeyValue<T0>,
  T2 = {
    [K in keyof T1]:T[Cast<T1[K], string>]
  }
> = T2
```

```typescript
type NestedKeyRevert = TransformKeysToCamelCase<{
  FOO_BAR: string
  ANOTHER_FOO_BAR: true | number,
}>
```

![transformKeysToCamelCase3](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/transformKeysToCamelCase3.png)



As you can see, the generic has 3 steps which are saved into `T0`, `T1` and `T2` variables. 

#### The First step

The first step creates an Object type where keys are UPPER_CASE and values are just keys transformed into camelCase   

```typescript
T0 = { [K in keyof T]: UpperCaseToCamelCase<K> },
```

![nestedKeyRevertType](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/nestedKeyRevertType.png)

#### The second step
The second step just applies the previously created generic and switch keys to values

```typescript
T1 = SwitchKeyValue<T0>,
```

![nestedKeyRevert](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/nestedKeyRevert.png)


#### The third step

The third step connects `T1` with the data type from `T`.

```typescript
T2 = { [K in keyof T1]: T[Cast<T1[K], string>] }
```

![transform Keys to camel Case 2](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/transformKeysTOCamelCase2.png)

### Add nested deep recursion

To provide this, we will create a generic which will check if the value is of type Object and will call recursion.

```typescript
type CallRecursiveTransformIfObj<T> = T extends Record<any, any> ? TransformKeysToCamelCase<T> : T

```

And updates the third step of TransformKeysToCamelCase generic.

```typescript

type TransformKeysToCamelCase<
  T extends Record<string, any>,
  T0 = { [K in keyof T]: UpperCaseToCamelCase<K> },
  T1 = SwitchKeyValue<T0>,
  T2 = { [K in keyof T1]: CallRecursiveTransformIfObj<T[Cast<T1[K], string>]> }
> = T2
```



And voilà! 🎉🎉🎉


If we test the nested data structure as a generic parameter

```typescript
type NestedKeyRevert = TransformKeysToCamelCase<{
  FOO_BAR: string
  ANOTHER_FOO_BAR: true | number,
  NESTED_KEY: {
    NEST_FOO: string
    NEST_BAR: boolean
  },
}>
```

![transformKeysToCamelCase](https://raw.githubusercontent.com/Svehla/articles-backup/main/TS_UPPER_CASEtoSnakeCase/imgs/transformKeysToCamelCase.png)


Everything works well.

Congratulations that you have read this article until the end. We successfully added a nested key case transformation which is a pretty advanced task in raw typescript.


[You can play with the full source code here](https://www.typescriptlang.org/play?target=1#code/C4TwDgpgBAMg9gdwgJwCpwKpkmu8nIDCAhgM4QCyx2KUAvFAN4CwAUFFMQFxQDkAgrzYcARj14AhIeygBjcYWkcAJuIAiSqBHEBRTQDNxAMU0BzcQHFNAC3EAJTQEtxASU0ArcQClNAa3EA0poANuIwmgC24hSaAHbiAHKacOIA8ppg4gAKmgCO4gCKmsjiAEqapOIAyprA4qiaAK7iGJoAbuIAapoI4gDqmgAe4gAamiDiAJqaAF7iAFrSAL5sbKCQUFg46PgoJORUNMj0TMJQ-OLEmhLiIpqE4rKaauLKmjriEJpG4vqaFuJTJo7OJrJoXOJHJovOJ3JoAuJfJoYOJgpoKOIIpoEuJYppUuI4JosuIwJoCuJcppSuJkJoquJSJpUOJgJoMOJGppOuI2po+uIEJoRuJBppJuIQJp5uIZstVqw1uBoHYIMRlDAIMBgCgADyoAB8J1QWkGOtiylIUAABgASRiOWL6WhGRzIUjATXalBLe2O53HAD6pQgHqW1qgAH4oK73Z6tTrjjxYhA2iglRtUMRHMEvYnSPqjQwTRAzRALVa7Q6nbRA7GPXmfX6a8cQ2GI9G28AoMnU+nFax1tBG7gtnrDcbTebLTbmwGY26Gwmm9X58HQ8Bw2do-X497jqXp1bfBAQHB9LBECh0GPcLsiGRKNQcGcONH7zejjsrw+Ds+UAA2ruI4ALqvj2C5xiOZw8KgCpDrAy53j+haToe5YzlW-ououe6Jr6q61l2W4yDuuEjlOGHHqe56bF+eA-vsT5HOB0a3t+BBMYcOBAeRSFgTIHA8MBSEwVAcEDghHF7I+qHFpRFZ8Jo0YSUJs6MCO0nILqqrqiOhYGgRWlMfq2a5khBaGoZ1oKgA9LZkEelAwRIVAjhWo0RxQMgG5uVawQ-hm0DoFU5bmrIEAmRODBVppmBHDpaoakhBlGQxnGyVmOYjpZBrWfByriXAWRkLIxDBFFRbiQpmE2aR4liSWZaKVhLaOcAfRwMgyhLIGc60F2OUkW+6khWF5YRSZu6dd1hn2ugJWkGVFWyYNFn5TIsFwKFsThZFmUGgqpZgF13YIbeTHoCQEQQCt5ByaN6UyfdunJfuqXzWZOX6sVpXlZVG1sPZnDYMEIBQKQsTECeciPm5u1wFAcAiO4ECyMApBBVA+zAPqAA0mxVU1R6bFG1U8BgWMWFqqQo505WNKGD3E1RUAhrIXXKLqxCxCABPYccnRGtGnQQSmabIAVGwkMEwTs407qOGmqDIDzpD6F1EQuPotPuMzNVWuznPc7zBM8yAwviarsTq5rASnqQV3EDdd0QKhsFHYMJ3IGdhVVAgjjALI1j2yA9PBIzupnKgePRwAjAbbNo8bHrII6phm7zVUsIJUAAQE8NQCeZ4XqgIE8IwRenjwAQANxQG0DPaOJ+cgVAKwyEsscyKgABMJw5xwHD54X1PALr4eM5ZccGgBvCNxHXwgeXUA6Gaqvo7qY8T0308GgTlcL4zNft7PvDF7wAkcB3RO91LwXW7byARKHjtwNdt0mdHidG91uqp+nTOFtu4cFQAABgHnnAujoq4l3EivC6j4nYuxMgEI0Xd44nH9oHYOodJ5u3AfvaO-cGCVxHjA4utFUBxxXjLOWaMFakCVhAFWasNbP21rrfUAEcb6jjq3AmADYimANCBdBbBb4KixgkDcEBlChxDBLbsxZH7sJfg7ZBn9ZKDxjKkVIgYJD8BpBDYAadhFnH4AkVIqA7A6FKHWPRBijE8FMYzKAAAfKAsRGgRBECgEBUAEg6CqKgHQahAwBB0BKU4ucgkhIcQSExZjTDgTiagJxxiRBwDgC5HmZwMGsCWIdVgQA)

Don't forget to 🫀 if you like this article.

