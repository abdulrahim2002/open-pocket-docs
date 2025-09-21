---
slug: to-string-or-not-to-string
title: "To String or Not to String"
authors: "openpocket"
tags: [OpenPocket]
---

# To string or not to string

The common debate about API design is that shall we use strings for
integers or integers should be represented as integers only. The JSON
data format supports integers natively. Therefore, we can use integers
in JSON directly.

However, bigint datatype is not supported in JSON data format.
Therefore, we need a way to deal with bigint's. One simple way to do
this is to use strings instead of numbers. With this approach, we use
strings when taking an input as bigint from a user.

We can then convert this string into bigint using
[`BigInt`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt)
constructor.

In the original getpocket API implementation also, all integers or
numeric types are represented as strings. And I suppose, they are then
converted internally into an integer/bigint etc.


## Comparison

The advantages of "integer as an integer" approch are as follows:

- we get schema validation working out of the box. We just need to
  define the variable as "{ type: "number" }"
- they natuarally align with standard javascript numbers

The disadvantages of using "intger as an integer in JSON" are:

- the behaviour when numbers are larger than `MAX_SAFE_INTEGER` is still
  undefined/unclear and not properly understood.


The advantages of using "integer as a string" are:

- the range of numbers can be arbitrarly large

The disadvantages of using "integer as a string" are:

- the conversion needs to be done by you, although we have `type
  coercion` in validation libraries like ajv that automatically coerce
  types, so this is less of a concern
- we need to check the range manually and determine weather the
  retrieved number is in range of the underlying type (bigint for
  example)

## Note on ajv and JSON schema

In general, when we define a variable as "number". Then the
minimum/maximum value of this number is the minimum/maximum value
javascript can handle. This is same as [`Number.MAX_VALUE =
1.7976931348623157 *
10^308`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_VALUE).

Therefore, when we just define a plain "number" in our ajv schema, then
it can take the values in `[-1.7976931348623157 * 10^308 ,
+1.7976931348623157 * 10^308]`. This is an extremely large range. For
comparison, consider that [`Number.MAX_SAFE_INTEGER =
2^53-1`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER).

There is an important distinction to be made here. Particularly the
difference between `Number.MAX_VALUE` and `Number.MAX_SAFE_INTEGER`.

1. `Number.MAX_VALUE`:  The largest positive finite value representable
   in JavaScript. It's approximately
2. `Number.MAX_SAFE_INTEGER`: The largest integer that can be safely
   represented in JavaScript (i.e., it can be accurately represented
   without losing precision). This is 9007199254740991 (or 2^53 - 1).

What does this mean?

It means that you can store values larger then `MAX_SAFE_INTEGER` in a
variable as long as they are less than `MAX_VALUE`. But you cannot use
them in arithematic operations (This will lead to incorrect results).

```javascript
> const x = Math.pow(10, 200) // x can be stored in a variable
undefined
> x
1e+200

> y = x     // passed around
1e+200
> y
1e+200

> x+1       // BUT CANNOT BE USED IN ARITHEMATIC OPERATIONS
1e+200
```

Note that **integers** larger than `MAX_SAFE_INTEGER` are automatically
converted into float. Hence, they are essentially floating point
numbers. For integers larger than this, we should still use BigInt.


## Conclusion

For now, the best option is to use "integer as a string" when we are
dealing with integers larger than 2^53 (i.e. `MAX_SAFE_INTEGER`). 

And for integers smaller than this value, both "integer as integer" and
"integer as string" approaches will work. However, for consistency
reasons, we should prefer "integer as a string".

On the sidenote, using "integer as string" presents a few challenges
like coercing the type as appropriate. And checking the range of
numbers. Try to use solutions provided natively by schema validation
libraries.

Therefore, always use "integer as a string" when exchanging numbers with
clients via JSON.


