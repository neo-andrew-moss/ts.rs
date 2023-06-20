# ts.rs

> Write typescript while thinking in rust

## Either

The `Either` data type represents a value of one of two possible types (a disjoint union). An instance of `Either` is either an instance of `Left` or `Right`. It is super useful for modeling the outcome of an operation that might fail.

The `Either` data type is used natively in rust for async operation, operations that may return an error, and other network operations such as io. The return value of these operations is unknown at runtime, thus it is beneficial to expressively model them in code.

In typescript/javascript, it is common to model these operations with a try catch and a throw of the error. The problem with this is that the throw of the exception is not modeled in the function signature. The caller of the function has no knowledge that the function houses an unsafe operation

```ts
const divide = (a: number, b: number): number => {
  if (b === 0) {
    throw new Error("Divide by zero error");
  }
  return a / b;
};

try {
  console.log(divide(1, 0)); // Throws "Divide by zero error"
} catch (error) {
  console.log(error.message);
}
```

The signature of divide is `const divide: (a: number, b: number) => number`, this does nothing to inform the caller that an error may occur.

Lets re-write this using `Either`:

```ts
import { either, pipe } from "fp-ts";
import { fold } from "fp-ts/Either";

const safeDivide = (a: number, b: number): either.Either<Error, number> => {
  return b === 0
    ? either.left(new Error("Divide by zero error"))
    : either.right(a / b);
};

pipe(
  safeDivide(1, 0),
  fold(
    (error) => console.error(error.message), // Handle Left
    (result) => console.log(result) // Handle Right
  )
);
```

The `Either` data type represents values with two possibilities - a value of type `Either<A, B>` is either `Left<A>` or `Right<B>`.

The `Either` type is commonly used to handle computations that can fail or return an error. The two possibilities capture two different outcomes:

- `Right<A>`: Represents a successful outcome containing a value of type A.
- `Left<B>`: Represents a failure or error containing a value of type B.

The standard convention is to use `Right` for a successful computation and `Left` for an error or failed computation. It is tantamount to an enhanced `Option` type where `Left` carries information about why the operation failed, unlike `Option` where `None` just signifies the absence of a value without any additional context.

One common use case for `Either` is in function return types for operations that could fail. The `Either` type makes it explicit in the type signature that the function could fail. Modeling this as an `Either` directly represents the nature of the operation in the functions return type. Further, the caller MUST (by must I mean that failing to do so is a compile time check) account for each possible disjointed union of outcomes prior to using the return value of the function.

The `Either` type works like a wrapper over the actual value, encapsulating the result of an operation within a context. This context expresses whether the operation was successful (Right) or failed (Left). This forces the caller to handle both cases explicitly, leading to safer code and preventing runtime exceptions.

Here is what this looks like in rust:

```rs
fn safe_divide(a: i32, b: i32) -> Result<i32, &'static str> {
    if b == 0 {
        Err("Divide by zero error")
    } else {
        Ok(a / b)
    }
}

match safe_divide(1, 0) {
    Ok(result) => println!("{}", result),
    Err(error) => println!("Error: {}", error),
}
```

[Excerpt from fp-ts Either code](https://github.com/gcanti/fp-ts/blob/master/src/Either.ts)

````ts
/**
 * ```ts
 * type Either<E, A> = Left<E> | Right<A>
 * ```
 *
 * Represents a value of one of two possible types (a disjoint union).
 *
 * An instance of `Either` is either an instance of `Left` or `Right`.
 *
 * A common use of `Either` is as an alternative to `Option` for dealing with possible missing values. In this usage,
 * `None` is replaced with a `Left` which can contain useful information. `Right` takes the place of `Some`. Convention
 * dictates that `Left` is used for failure and `Right` is used for success.
 *
 */
````

## Option

What is nothingness? Well, it is the absence of something. How do we model absence in typescript/javascript? We use `null` or `undefined`. I don't know what the difference between them is, and I don't care to know. In most codebases, they are used interchangeably with little heed or consideration. The negatives of this are substantial.

This is what I referrer to nothingness hell (A term has not punctured the cultural zeitgeist)

```ts
let user = {
  address: {
    street: {
      name: null,
    },
  },
};

if (user && user.address && user.address.street && user.address.street.name) {
  console.log(user.address.street.name);
} else {
  console.log("Street name not available");
}
```

This yields the most common logical error in ts/js, “cannot read property of undefined”.

Enter `Option`.

The `Option` datatype represents the possible absence of a value. Instead of using `null` or `undefined` , `Option` expressively handles cases where a value might not exist, avoiding null-pointer exceptions and similar issues.

The `Option` data type encapsulates a value into two possible variants:

- `Some(A)`: Indicates the presence of a value A.
- `None`: Indicates the absence of a value.

A function that might not return a value will instead return an `Option` type, indicating to the caller that they must handle the possibility of the absence of a value. This effectively integrates error-checking into the type system, forcing developers to consciously deal with the "missing value" scenario. Again, a compile time consideration.

Tantamount to `Either`, the `Option` encodes the nature of the operation within the function signature. This expresses to the caller that optionality is present, and that means something.

Here is the `Option` in typescript:

```ts
import { pipe } from "fp-ts";
import { fold } from "fp-ts/Option";

let user = {
  address: {
    street: {
      name: "Main St", // Can be null or undefined
    },
  },
};

pipe(
  user.address.street.name,
  fold(
    () => console.log("Street name not available"), // Handle None
    (name) => console.log(name) // Handle Some
  )
);
```

And now rust:

```rs
let user = Some("Moss");

match user {
    Some(name) => println!("User's name is {}", name),
    None => println!("User's name is not available"),
}
```

Lovely!

Excerpt from fp-ts: https://github.com/gcanti/fp-ts/blob/master/src/Option.ts

````ts
/**
 * ```ts
 * type Option<A> = None | Some<A>
 * ```
 *
 * `Option<A>` is a container for an optional value of type `A`. If the value of type `A` is present, the `Option<A>` is
 * an instance of `Some<A>`, containing the present value of type `A`. If the value is absent, the `Option<A>` is an
 * instance of `None`.
 *
 * An option could be looked at as a collection or foldable structure with either one or zero elements.
 * Another way to look at `Option` is: it represents the effect of a possibly failing computation.
 *
 */
````
