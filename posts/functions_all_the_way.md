---
category: Functional Programming
tags:
  - y
  - combinators
  - anonymous functions
  - elixir
  - church
---

# Functions Everywhere

In the previous [post][1] we used a small subset of a given programming language and tried (successfully) to define our own implementation of _recursion_.
Let's try to use even smaller subset.

Imagine... We don't have numbers, strings, operations...
Everything we have is the ability to define functions (abstraction) and to call (invoke) them (application).
Let's start with this idea in mind and see where it'll take us.

## In The Beginning Was The Function

So, what do we have? We can define functions:

```elixir
fn (x) -> x end
```

...and we can call (invoke) them:

```elixir
fn (x) -> x end.(fn (x) -> x end)
```

That's all we can do.

But what should we pass to a function as its arguments?

We only have one type of data - functions.
We can define them (creating instances of this data type) and invoke them.
If we look at the code sample above, we can see that we are calling the function with a function as an argument.
Yes. This is it! The only thing we can pass to a function call is another (or the same) function.

Now, one last constraint : we only have the ability to define functions of *one* argument.

So to summarize:
1. We only have one type of data - *functions*.
2. We know how to define a function of *one* argument.
3. We know how to call a function. From (1) : one function can be called with a parameter of type `function`.

And now, to make our lives a bit easier, we'll add one additional rule:
4. We can _'give a name'_ to our functions, so we can reuse them:

```elixir
id = fn(x) -> x end
id.(id)
```

What should we do with the rules we have?
The answer is simple : let's think of a few challenges we could solve using only these four base rules.

So here are the challenges:
1. [Let's find a way to pass more than one argument to a function call][2]
2. [Let's define the simplest data type - the boolean][3]
3. [Let's define numbers and operations over those numbers][4]


## <a name="currying"></a> Multiple Arguments

The function, we defined and called above is known as _identity_ or _I combinator_.
When called with an argument, it just returns it.

We want to define another well known combinator - `K`.
This is a very simple function, also known as 'The Kestrel' (if you want to know why, take a look at [this book][5]).
The interesting thing about the Kestrel combinator is that its definition looks like this : `K(x, y) = x`, which means that it is a function of two arguments and it returns the first of them.

The question is : How to define it, when we can only define functions of _one_ argument?

To answer this question, we can look at a technique, which main idea is to change the logic of a function of multiple arguments to
a function of one argument, which returns a function of one argument, which returns a function of one argument and so on,
depending on the number of the arguments of the original function.
The last function of one argument executes the original function's calculation using the arguments of the functions wrapping it.
This technique is known as _'schönfinkelisation'_. Let's look at a code example:

```elixir
a = fn (x, y, z) -> (x + y) * z end
b = fn (x) ->
      fn (y) ->
        fn (z) -> (x + y) * z end
      end
    end

a.(2, 3, 4)
# 20

b.(2).(3).(4)
# 20
```

We can see that the result of calling the two functions is the same, but the invocations and the definitions differ.
The important thing is that the logic is kept the same.
And yes, we went out of our constraints a bit for this example.
Interesting fact: this technique was named after its inventor [Moses Schönfinkel][6] and further developed by [Haskell Curry][7].
That's why today we call it _'currying'_ and not _'schönfinkelisation'_.
Guess why? :)

This technique allows us to think of a function of one argument returning a function of one argument and so on, as a functions of multiple arguments.
That's how we are going to define the Kestrel:

```elixir
kestrel = fn (x) -> fn (y) -> x end end
```

We can define another combinator, named after a bird - _'The Starling'_, also known as _the S combinator_.
Its definition is: `S(x, y, z) = x(z, (y(z)))`.
This means that it expects three arguments: the first, `x`, should be a function of (at least) two arguments,
which `S` calls with its third argument, `z` and `y(z)`.
And here is _'The Starting'_, written in `Elixir`:

```elixir
starling =
  fn (x) ->
    fn (y) ->
      fn (z) ->
        x.(z).(y.(z))
      end
    end
  end
```

Interesting fact:

```elixir
starling.(kestrel).(kestrel)
```

is

```elixir
id
```

Notice that comparing functions is hard.
In this case is not so hard, but generally it is a hard thing to do.
We can assume that two functions are identical if they return the same value for all of the possible values we can pass to them in a function call.
This equality is hard to prove.
We say that two functions are _functionally equivalent_, if for every `y` they return the same value.

As mentioned above, proving that `id` and `starling.(kestrel).(kestrel)` are _functionally equivalent_ is not that hard:

```elixir
starling.(kestrel).(kestrel)

# By the definition of starling =>

fn (z) ->
  kestrel.(z).(kestrel.(z))
end

# By the definition of kestrel =>

fn (z) ->
  z
end

# By the definition of id =>

id
```

Now we are ready to define a new type of data.


## <a name="booleans"></a> Boolean Values

Let's define _TRUE_ and _FALSE_ like this:

```elixir
bool_true  = fn (a) -> fn (b) -> a end end
bool_false = fn (a) -> fn (b) -> b end end
```

Which means that we define the two boolean values as follows:
1. _TRUE_ is a function of two arguments, which always returns the first one.
2. _FALSE_ is a function of two arguments, which always returns the second one.

We could also define them using the **S** and **K** combinators:

```elixir
bool_true  = kestrel
bool_false = starling.(kestrel)
```

The first line is clear, and the second:

```
K(x, y)    = x
S(x, y, z) = x(z, y(z))

=>

S(K, a, b) = K(b, a(b)) = b === FALSE(a, b) = b
```

Now we have the two boolean values. Starting from this point, we can define
the simplest operation on them: _NOT_:

```elixir
bool_not = fn (p) -> p.(bool_false).(bool_true) end

bool_not.(bool_true)
# bool_true.(bool_false).(bool_true) => bool_false

bool_not.(bool_false)
# bool_false.(bool_false).(bool_true) => bool_true
```

It's easy to define a function for _conjunction_. Let's think about it:
1. We know, that if we pass to `bool_true` its two arguments, it will always return the first one.
2. We know, that if we pass to `bool_false` its two arguments, it will always return the second one.
3. We know, that _AND_ should be a function of two _boolean_ arguments; If the first one is `bool_false`, _AND_ should return `bool_false`, we don't need to check the second one.
4. From (1) and (2), `bool_true.(x).(bool_false)` is `x`, and `bool_false.(x).(bool_false)` is `bool_false`.
5. From (3) and (4), we can define _AND_ as:

```elixir
bool_and = fn (a) -> fn (b) -> a.(b).(bool_false) end end

bool_and.(bool_true).(bool_true)
# bool_true.(bool_true).(bool_false) => bool_true

bool_and.(bool_false).(bool_true)
# bool_false.(bool_true).(bool_false) => bool_false

bool_and.(bool_true).(bool_false)
# bool_true.(bool_false).(bool_false) => bool_false

bool_and.(bool_false).(bool_false)
# bool_false.(bool_false).(bool_false) => bool_false
```

In a similar fashion we can define _OR_ as:

```elixir
bool_or = fn (a) -> fn (b) -> a.(bool_true).(b) end end
```

So we have logical _NOT_, _conjunction_ and _disjunction_.
We can also define _IF_ or rather _COND_.
This is a function of three arguments.
If the first one is _TRUE_, it will return the second one, if the first one is instead _FALSE_, it will return the third one:

```elixir
bool_cond = fn (a) -> fn (b) -> fn (c) -> a.(b).(c) end end end
```

Let's break out of our constraint to have only one type of data and write a function,
that returns the right `Elixir` boolean value when we call it with `bool_true` or `bool_false`:

```elixir
church_to_bool =
  fn (church_bool) ->
    church_bool.(true).(false)
  end

church_to_bool.(bool_true)
# true

church_to_bool.(bool_false)
# false

church_to_bool.(bool_and.(bool_false).(bool_true))
# false
```

This way of representing the boolean values and the operations on them as functions is called [Church's encoding of booleans][8] and is named after [Alonzo Church][9].
He used this in the lambda calculus.
We'll be talking a lot about the lambda calculus and Church's encodings.
For example, in the following section we will define our own numbers.


## <a name="numbers"></a> Numbers And Arithmetic Operations

We'll do something very similar to the way we defined our boolean values. We'll assume that:


1. `0` is `zero(f, x)  = x`.
2. `1` is `one(f, x)   = f(x)`.
3. `2` is `two(f, x)   = f(f(x))`.
4. `3` is `three(f, x) = f(f(f(x)))`.

The idea is that a number is a function of two arguments.
When the number is zero, the function just returns its second argument.
When the number is one, the function applies its first argument on the second one : **f(x)**.
When the number is two, the function applies its first argument on itself and then the result on its second argument : **f(f(x))**.
We can continue forever like that, so **n** is **f<sup>n</sup>(x)**.

We can write this representation in `Elixir` as:

```elixir
zero  = fn (f) -> fn (x) -> x end end
one   = fn (f) -> fn (x) -> x |> f.() end end
two   = fn (f) -> fn (x) -> x |> f.() |> f.() end end
three = fn (f) -> fn (x) -> x |> f.() |> f.() |> f.() end end

# And so on...
```

Now, let's introduce a few operations on these numbers.

Our first task is to define a function `is_zero(n)`, that takes a number as an argument and returns `bool_true` if the number is `zero` or `bool_false` if it isn't.
Let's think about it:
1. The numbers are functions of two arguments.
2. We can pass two values to a given number and examine its behaviour.
3. If we pass the values `g` and `a` to a number, `g` will be called one or more times if the number is not `zero`. If the number is `zero`, `g` won't be called at all.
4. From (3) : `g` should be a function, which always returns `bool_false` and `a` could be `bool_true`. Now if we pass these functions to `zero`, we'll get the second argument : `a` or `bool_true`. If the number is not `zero`, we'll get the result of calling `g`, which is `bool_false`.

```elixir
bool_true  = fn (a) -> fn (b) -> a end end
bool_false = fn (a) -> fn (b) -> b end end

is_zero =
  fn (n) ->
    n.(fn (_) -> bool_false end).(bool_true)
  end

is_zero.(zero) |> church_to_bool.()
# true

is_zero.(one) |> church_to_bool.()
# false
```

The next task is to implement a function, which given a number returns the next number.
This is the function `succ(n)`, which when called with `zero` returns `one`, when called with `one`, returns `two` and so on.
As we know the numbers themselves are functions of two arguments, so `n` is actually **f<sup>n</sup>(x)** and `n+1` is **f(f<sup>n</sup>(x))** or **f(n(f, x))**.
We can define `succ` as a function of three arguments : the number `n`, `f` and `x`.
This function constructs a new number : **f(n(f, x))**:

```elixir
succ =
  fn (n) ->
    fn (f) ->
      fn(x) ->
        f.(n.(f).(x))
      end
    end
  end

succ.(zero)
# f.(x) => one

succ.(two)
# f.( f.(f.(x)) ) => three

four = succ.(three)
five = succ.(four)
```

The function for adding two numbers is very similar.
We construct a new number using the two numbers we want to _add_- **m** and **n**.
The new number is **m(f, n(f, x))** => **m(f, f<sup>n</sup>(x))** => **f<sup>m</sup>(f<sup>n</sup>(x))** => **f<sup>m + n</sup>(x)** => **m + n**:

```elixir
plus =
  fn (m) ->
    fn (n) ->
      fn (f) ->
        fn(x) ->
          m.(f).(n.(f).(x))
        end
      end
    end
  end

six = plus.(four).(two)
```

It is hard to debug the results we get by invoking these operations on numbers.
That's why let's leave our constraints behind for a bit and write a function which transforms our numbers based on functions to `Elixir` numbers:

```elixir
church_to_int =
  fn (n) ->
    n.(&(&1 + 1)).(0)
  end

six |> church_to_int.()
# 6

seven = succ.(six)
seven |> church_to_int.()
# 7

plus.(four).(five) |> church_to_int.()
# 9

twenty = plus.(plus.(four).(five)).(plus.(six).(five))
twenty |> church_to_int.()
# 20
```

Let's define another operation - `pred`.
This one will return the previous number when we apply it to a number.
Important thing : we don't have negative numbers, so `pred(zero) == zero`.

Let's think.
1. To calculate `n+1` from `n` (using `succ`), we pass `n`'s value ( **f<sup>n</sup>(x)** ) to its first argument **f**.
2. From (1) : to calculate `n-1` we just have to remove one **f** from `n`'s return value.

The problem is that doing this is not as easy as it sounds.
The idea is to construct `n` applying **f** to **x** *n* times and on every step to keep the value of the previous one.
In the end we are going to return the previous value for **f<sup>n</sup>(x)**, which is **f<sup>n-1</sup>(x)**.

Our task now is to think of a way to keep both the previous and current result of applying **f** on every step.
If we had a tuple of values it would work, wouldn't it?
But we don't have this data type in our little language subset.

The solution is : let's define a custom _PAIR_ type, based on functions, like we did it for the booleans and the numbers.
We assume that a _PAIR_ is defined as:

```elixir
pair =
  fn (a) ->
    fn (b) ->
      fn (c) -> c.(a).(b) end
    end
  end
```

The only thing left is to implement retrieving the first or the second element of a `pair`:

```elixir
first =
  fn (p) ->
    g = fn (x) -> fn (y) -> x end end
    p.(g)
  end

second =
  fn (p) ->
    g = fn (x) -> fn (y) -> y end end
    p.(g)
  end

first.(pair.(one).(two)) |> church_to_int.()
# 1
second.(pair.(one).(two)) |> church_to_int.()
# 2
```

The above idea is simple - the function, constructing the pair, expects _3_ arguments, but we call it with only two - the two elements of the pair.
The third argument is a function of two arguments, which will be called with the two elements of the pair as arguments.
So if we pass the right function as third argument of `pair`, we could get the right element.
Interesting fact is, that the function we use to read the first element of a pair in `first` is `bool_true` and the one in `second` is `bool_false`.
It is common to see the same function playing different role, depending on the type of data it operates on or represents.
Don't forget that the **K combinator** is also `bool_true`.

Let's return to the `pred` operation.
It is time to define the case, in which we call it with `zero`:

```elixir
zero_case = pair.(zero).(zero)
```

And now the case for every other number:

```elixir
main_step =
  fn (p) ->
    pair.(second.(p)).(succ.(second.(p)))
  end
```

Basically it takes a pair of numbers and returns a new pair of numbers.
The new pair's first element is the second element of the original pair and its second element is the second element of the original pair, incremented by one.
 * We know that the number itself is a function of two arguments, which applies its first argument to its second one as many times, as the value of the number it represents.
 * The `main_step` function expects a pair of numbers and creates a new pair using the following logic : `(n, m)` -> `(m, m+1)`.
 * If we start with passing `(0, 0)` to `main_step` once, we'll get `(0, 1)`. If we pass this result to `main_step`, we'll get `(1, 2)`. Passing this to `main_step` will produce `(2, 3)`. So, calling `main_step` on `(0, 0)` **n** times will return `(n-1, n)`. If we now read the first element of the pair we will get what we wanted - `n-1`.

If we have the number **n** (don't forget, it is a function of two arguments) and call it like this: `n.(main_step).(zero_case)`,
by the definition of `n`, `main_step` will get applied to `zero_case` exactly `n` times.
Returning the first element of this result will produce `n-1`:

```elixir
pred =
  fn (n) ->
    first.(n.(main_step).(zero_case))
  end

# And so:

pred.(zero) # =>
first.(zero.(main_step).(zero_case)) # =>
first.((fn (f) -> fn (x) -> x end end).(main_step).(zero_case)) # =>
first.(zero_case) # =>
zero

pred.(two) # =>
first.(two.(main_step).(zero_case)) # =>
first.((fn (f) -> fn (x) -> f.(f.(x)) end end).(main_step).(zero_case)) # =>
first.(main_step.(main_step.(zero_case))) # =>
first.(main_step.(pair.(second.(zero_case)).(succ.(second.(zero_case))))) # =>
first.(main_step.(pair.(zero).(one))) # =>
first.(pair.(second.(pair.(zero).(one))).(succ.(second.(pair.(zero).(one))))) # =>
first.(pair.(one).(succ.(one))) # =>
one
```

We can define subtraction very easy, using `pred`.
If we want to subtract `m` from `n` we just have to apply `pred` exactly *m* times to `n`:

```elixir
minus =
  fn (n) ->
    fn(m) ->
      m.(pred).(n)
    end
  end

minus.(seven).(three) |> church_to_int.()
# 4
```

Again, we used the fact that our numbers are just functions of two arguments which apply their first argument on their second
as many times as their numerical value.

Let's define multiplication using the same property:

```elixir
mult =
  fn (n) ->
    fn (m) ->
      m.(plus.(n)).(zero)
    end
  end

mult_without_plus =
  fn (n) ->
    fn (m) ->
      fn (f) ->
        n.(m.(f))
      end
    end
  end

one_hundred = mult.(twenty).(five)
one_hundred |> church_to_int.()
# 100
```

And what about comparing numbers?
Not a problem! Here is the operation for checking if two numbers are equal:

```elixir
num_eq =
  fn (n) ->
    fn (m) ->
      # bool_and.(is_zero.(minus.(n).(m))).(is_zero.(minus.(m).(n)))
      bool_and.(is_zero.((m).(pred).(n))).(is_zero.((n).(pred).(m)))
    end
  end

num_eq.(one_hundred).(one_hundred) |> church_to_bool.()
# true
```

We use `is_zero` and `minus`.
If we subtract `m` from `n` and we get `zero`, the numbers are equal.
We don't have negative numbers, so if we subtract bigger number from smaller one, we'll get `zero` (`pred(zero) == zero`) and `num_eq` will return `bool_true`.
That's why we use `bool_and` to subtract both `n` from `m` and `m` from `n` and check if both of these subtractions are `zero`.

The strict inequalities, _greater than_ and _less than_ are similar:

```elixir
num_gt =
  fn (n) ->
    fn (m) ->
      bool_and.(is_zero.((n).(pred).(m))).(bool_not.(is_zero.((m).(pred).(n))))
    end
  end

num_lt =
  fn (n) ->
    fn (m) ->
      bool_and.(is_zero.((m).(pred).(n))).(bool_not.(is_zero.((n).(pred).(m))))
    end
  end

num_gt.(two).(one) |> church_to_bool.()
# true
num_gt.(two).(three) |> church_to_bool.()
# false
num_lt.(two).(three) |> church_to_bool.()
# true
num_lt.(four).(three) |> church_to_bool.()
# false
```

We know that `(n).(pred).(m)` returns `zero`, if `n` is greater than `m` or equal to `m`.
We don't want to return `bool_true` if `n` and `m` are equal.
That's why we also check that `(m).(pred).(n)` is not `zero`.
And this is how the _strictly greater than_ check works.
The implementation of the _strictly less than_ check is equivalent.
Defining their _non-strict_ counterparts is even easier:

```elixir
num_gt_eq =
  fn (n) ->
    fn (m) ->
      is_zero.((n).(pred).(m))
    end
  end

num_lt_eq =
  fn (n) ->
    fn (m) ->
      is_zero.((m).(pred).(n))
    end
  end

num_gt_eq.(two).(one) |> church_to_bool.()
# true
num_gt_eq.(two).(two) |> church_to_bool.()
# true
num_gt_eq.(two).(three) |> church_to_bool.()
# false
num_lt_eq.(two).(three) |> church_to_bool.()
# true
num_lt_eq.(three).(three) |> church_to_bool.()
# true
num_lt_eq.(four).(three) |> church_to_bool.()
# false
```

Let's summarize : we can compare numbers, add them to one another, subtract them and multiply them.
It was not very easy to define the subtraction operation.
It required the `pred` function, which required working with *pairs*.

Our next tasks is even harder. We are going to define the _division_ operation.
The algorithm can be described like this:

```
a / b =
  if a >= b do
    1 + (a - b) / b
  else
    0
  end
```

We have all the operations we need to implement this algorithm, but the _division_ operation is used recursively.
That's why we'll have to use the [Y combinator][1]:

```elixir
y = fn (h) ->
  (fn (f) ->
    f.(f)
  end).(fn (g) ->
    h.(fn (x) -> g.(g).(x) end)
  end)
end
```

And with the *Y combinator*, the definition of the _division_ operation looks like this:

```elixir
divide =
  fn (n) ->
    fn (m) ->
      y.(fn (divide1) ->
        fn (current) ->
          num_gt_eq.(current).(m).(
            fn (_) -> succ.(divide1.(minus.(current).(m))) end
          ).(
            fn (_) -> zero end
          ).(zero)
        end
      end).(n)
    end
  end


divide.(twenty).(two) |> church_to_int.()
# 10
divide.(twenty).(three) |> church_to_int.()
# 6
divide.(twenty).(four) |> church_to_int.()
# 5
```

Actually it was not so hard.
But that's because we are using a lot of things we defined in this and [the previous][1] posts.
Basically we are implementing the pseudo-code from above with our own operations and with the *Y combinator*, providing the recursion.

When we need to do `n / m`, we call `divide.(n).(m)`.
This will pass `n` as the initial `current` value.
If `current` is greater than or equal to `m`, we add `1` to the recursive call to `divide` with `current - m` and `m` as arguments (the first time the arguments are `n - m` and `m`).
If `current` is less than `m`, we return `zero`.
Notice that we achieved that by only using functions of one argument.

The operation, returning the _remainder_ of a _division_ is very similar:

```elixir
remainder =
  fn (n) ->
    fn (m) ->
      y.(fn (remainder1) ->
        fn (current) ->
          num_gt_eq.(current).(m).(
            fn (_) -> remainder1.(minus.(current).(m)) end
          ).(
            fn (_) -> current end
          ).(zero)
        end
      end).(n)
    end
  end


remainder.(twenty).(two) |> church_to_int.()
# 0
remainder.(twenty).(three) |> church_to_int.()
# 2
remainder.(twenty).(seven) |> church_to_int.()
# 6
```

In the end, we return `current` and not the number of the divisions.

Let's add another operation as a bonus - the _exponentiation_:

```elixir
power =
  fn (n) ->
    fn (m) ->
      m.(n)
    end
  end

power.(two).(three) |> church_to_int.()
# 8
```

This one is very simple, so I'll leave it to you.

This post became a bit long, so let's conclude it with something we did in [the previous one][1].
We'll define a function calculating the factorial but this time we'll only be using functions of one argument as our data type and operations.
We've got all we need to do it.

Our previous definition of factorial was:

```elixir
proto_factorial = fn g ->
  fn
    0 -> 1
    n -> n * g.(n - 1)
  end
end

factorial = y.(proto_factorial)
```

By using our own numbers and booleans, based on functions and the operations we created,
we can transform the above `factorial` into:

```elixir
proto_factorial = fn g ->
  fn (n) ->
    is_zero.(n).(fn (_) -> one end).(
      fn (_) ->
        mult.(n).(g.(pred.(n)))
      end
    ).(zero)
  end
end

factorial = y.(proto_factorial)

factorial.(five) |> church_to_int.()
# 120
```

The problem with this implementation is that if you invoke it with even a small number like *ten*, it'll take time.
This applies to subtracting large numbers too.

The numbers we defined using functions are very unoptimized for real calculations.
Their idea is to prove that only using anonymous functions of one argument, we can define numbers and operations on them.
The anonymous functions of one argument are exactly what the lambda calculus is based on.
The untyped lambda calculus is one of the simples programming languages, which theoretically can be used to calculate any program written in another programming language, like `Elixir` for example.
And this is very interesting.

The lambda calculus is the base of the modern functional programming and is very important.
That's why we'll continue tackling with it in future posts.
The numbers, we defined and used in this post, are known as *'Church numerals'*.
There are even versions of these numerals supporting negative numbers.

## Related Books And Links

So a fellow *functional programming* zealot, [Phil][10], gave me the idea to add links in the end of my posts to some books or articles, related to the topics I write about.
I'll start adding such links from now on.

If the topic covered here was of interest to you, checkout the following books:

 * [To Mock a Mockingbird][5] - Puzzles, games and birds. This is the book providing bird names to some well known combinators.
 * [Types and Programming Languages][11] - This book will be an integral part of the topics I'm going to cover in the future. Types, lambda calculus, computability etc.
 * [Understanding Computation][12] - Contains similar exercises to the ones we did through this post, written in `Ruby`.

## Code

* The code I wrote for this post is located [here][13].
* The markdown source of the post can be found [here][14]. If you want to contribute, by fixing mistakes I made, or adding more content, you can do it in a PR there.


[1]: y
[2]: #currying
[3]: #booleans
[4]: #numbers
[5]: https://www.amazon.com/gp/product/0192801422?ie=UTF8&tag=raganwald001-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0192801422
[6]: https://en.wikipedia.org/wiki/Moses_Schönfinkel
[7]: https://en.wikipedia.org/wiki/Haskell_Curry
[8]: https://en.wikipedia.org/wiki/Church_encoding
[9]: https://en.wikipedia.org/wiki/Alonzo_Church
[10]: https://twitter.com/pkamenarsky
[11]: https://www.amazon.com/Types-Programming-Languages-MIT-Press/dp/0262162091
[12]: http://computationbook.com/
[13]: https://gist.github.com/meddle0x53/1a3bd176d29e4b17a9b279d79463a141
[14]: https://github.com/meddle0x53/themeddle/blob/master/posts/bg/functions_all_the_way.md
