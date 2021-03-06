[![build status](https://img.shields.io/travis/gcanti/newtype-ts/master.svg?style=flat-square)](https://travis-ci.org/gcanti/newtype-ts)
[![dependency status](https://img.shields.io/david/gcanti/newtype-ts.svg?style=flat-square)](https://david-dm.org/gcanti/newtype-ts)
![npm downloads](https://img.shields.io/npm/dm/newtype-ts.svg)

# Motivation

A common programming practice is to define a type whose representation is identical to an existing one but which has a separate identity in the type system.

```ts
type USD = number
type EUR = number

const myamount: USD = 1

declare function change(usd: USD): EUR
declare function saveAmount(eur: EUR): void

saveAmount(change(myamount)) // ok
saveAmount(myamount) // opss... this is also ok because both EUR and USD are type alias of number!
```

# Usage

## Newtypes

Let's define a newtype for the EUR currency

```ts
import { Newtype, iso } from 'newtype-ts'

interface EUR extends Newtype<{ readonly EUR: unique symbol }, number> {}

// isoEUR: Iso<EUR, number>
const isoEUR = iso<EUR>()

// myamount: EUR
const myamount = isoEUR.wrap(0.85)

// n: number = 0.85
const n = isoEUR.unwrap(myamount)

declare function saveAmount(eur: EUR): void

saveAmount(0.85) // static error: Argument of type '0.85' is not assignable to parameter of type 'EUR'
saveAmount(myamount) // ok
```

By definition a "newtype" must have the **exact same runtime representation** as the value that it stores, e.g. a value of type `EUR` is just a `number` at runtime.

For the `Iso` type, see the [monocle-ts](https://github.com/gcanti/monocle-ts) documentation.

## Refinements

An `Integer` is a refinement of `number`

```ts
import { Newtype, prism } from 'newtype-ts'

interface Integer extends Newtype<{ readonly Integer: unique symbol }, number> {}

const isInteger = (n: number) => n % 1 === 0

// prismInteger: Prism<number, Integer>
const prismInteger = prism<Integer>(isInteger)

// oi: Option<Integer>
const oi = prismInteger.getOption(2)

declare function f(i: Integer): void

f(2) // static error: Argument of type '2' is not assignable to parameter of type 'Integer'
oi.map(f) // ok
```

For the `Prism` type, see the [monocle-ts](https://github.com/gcanti/monocle-ts) documentation.

## Builtin refinements

- `Char`
- `Integer`
- `Negative`
- `NegativeInteger`
- `NonNegative`
- `NonNegativeInteger`
- `NonPositive`
- `NonPositiveInteger`
- `NonEmptyString`
- `NonZero`
- `NonZeroInteger`
- `Positive`
- `PositiveInteger`

```ts
import { NonZero, prismNonZero } from 'newtype-ts/lib/NonZero'

// a total function
const safeDivide = (numerator: number, denominator: NonZero): number => {
  return numerator / prismNonZero.reverseGet(denominator)
}

// result: Option<number>
const result = prismNonZero.getOption(2).map(denominator => safeDivide(2, denominator))
```

# TypeScript compatibility

The stable version is tested against TypeScript 3.1.6

# Performance

```ts
const double = n => n * 2
const doubleEUR = eurIso.modify(double)
```

Test `double(2)` vs `doubleEUR(eurIso.wrap(2))`

Results (`node v8.9.3`)

```
double x 538,301,203 ops/sec ±0.45% (87 runs sampled)
doubleEUR x 536,575,600 ops/sec ±0.27% (87 runs sampled)
```

# Recipes

## How to lift a function

```ts
const double = (n: number): number => n * 2

// doubleEUR: (s: EUR) => EUR
const doubleEUR = eurIso.modify(double)
```

## How to operate over newtypes

```ts
import { over } from 'newtype-ts'

interface USD extends Newtype<{ readonly USD: unique symbol }, number> {}

const USDFromEUR = (n: number): number => n * 1.18

// getter: Getter<EUR, USD>
const getter = over<EUR, USD>(USDFromEUR)

// usd: USD
const usd = getter.get(eur)
```

For the `Getter` type, see [monocle-ts](https://github.com/gcanti/monocle-ts) documentation.
