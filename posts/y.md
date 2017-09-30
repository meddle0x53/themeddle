---
category: Functional Programming
tags:
  - y
  - recursion
  - combinators
  - anonymous functions
  - elixir
  - erlang
  - ruby
  - javascript
  - js
---

# Defining Recursion

This is the English version of an article I wrote for [elixir-lang.bg](https://elixir-lang.bg/posts).
You can find it in [Български on this blog too](http://themeddle.com/bg/posts/y).

It is an interesting exercise, in which we are going to use a very limited subset of the language - a subset that doesn't even support recursion. Using only it, we will try to define a recursive function.
By language I actually mean _a few_ languages. I added an interesting new feature to my blog engine's front-end [BlogitWeb](https://github.com/meddle0x53/blogit_web),
which allows me to write examples in more than one programming language and give the reader the feeedom to select the one he/she likes.

In this article we will be using [elixir]() {: .multi-code}, [erlang]() {:.multi-code}, [ruby]() {:.multi-code} and [javascript]() {:.multi-code}.
Let's begin.

## Prerequisites

For each language of choice, we are going to use only its interpreter and won't use named functions.
We will forget for a moment that [elixir]() {: .multi-code}, [erlang]() {:.multi-code}, [ruby]() {:.multi-code} and [javascript]() {:.multi-code} support named functions.

So let's run our interpreter:

```elixir
iex
```
```erlang
erl
```
```ruby
irb
```
```javascript
node
# or some modern browser (Firefox?)
```
{: .multi-code}

We are going to limit ourselves by using only numbers, basic arithmetic and comparing the numbers:

```elixir
23

5 + 4
# 9

5 * 4
# 20

5 > 4
# true

5 < 4
# false

5 == 4
# false
```

```erlang
23.

5 + 4.
% 9

5 * 4.
% 20

5 > 4.
% true

5 < 4.
% false

5 == 4.
% false
```

```ruby
23

5 + 4
# 9

5 * 4
# 20

5 > 4
# true

5 < 4
# false

5 == 4
# false
```
```javascript
23

5 + 4
// 9

5 * 4
// 20

5 > 4
// true

5 < 4
// false

5 === 4
// false
```
{: .multi-code}

We are going to try to define and use a [Factorial function](https://en.wikipedia.org/wiki/Factorial),
so that's why we are going to use only integral numbers and arithmetic operations with them.
Let's see how we can define a simple anonymous function. Let's define a function, which adds its two arguments and returns the result:

```elixir
fn (a, b) -> a + b end
```
```erlang
fun (A, B) -> A + B end.
```
```ruby
proc { |a, b| a + b }

# or

lambda { |a, b| a + b }

# or

-> (a, b) { a + b }
# We are going to use the '->' syntax in this article.
```
```javascript
(function (a, b) { return a + b; });

// or

(a, b) => { return a + b; };

// We are going to use the '=>' syntax in this article.
```
{: .multi-code}

Now we can call this function like this:
```elixir
(fn (a, b) -> a + b end).(4, 5)
# 9
```
```erlang
(fun (A, B) -> A + B end)(4, 5).
% 9
```
```ruby
-> (a, b) { a + b }.call(4, 5)
# 9

# or

-> (a, b) { a + b }[4, 5]
# 9

# We are going to use the '[]' syntax in this article.
```
```javascript
(function (a, b) { return a + b; })(4, 5);
// 9

// or

((a, b) => { return a + b; })(4, 5);
// 9
```
{: .multi-code}

We can also have different logic depending on the parameters we pass:

```elixir
(fn
  0 -> 0
  x -> x - 1
end).(3)
# 2
```
```erlang
fun
  (0) -> 0;
  (X) -> X - 1
end(3).
% 2
```
```ruby
-> (x) do
  if x == 0
    0
  else
    x - 1
  end
end[3]
# 2

# or

-> (x) { x == 0 ? 0 : x - 1 }[3]
# 2
```
```javascript
(
  (x) => {
    if (x === 0) {
      return 0;
    } else {
      return x - 1;
    }
  }
)(3);
// 2
```
{: .multi-code}

That's basically a function which gives the previous number. For zero it returns zero as it doesn't have previous number.
We pretend to not know about negative numbers in our limited subset of the language.

Good! Now we know everything we need to know in order to define the Factorial function! Or do we?

## Factorial - The Beginning

So far we have learned how to define a function.
If we had factorial defined we would expect from it to behave like this:

```elixir
factorial.(0)
# 1

factorial.(1)
# 1 * factorial.(0) => 1

factorial.(2)
# 2 * factorial.(1) => 2

..........

factorial.(n)
# n * factorial.(n - 1) => n * n - 1 * n - 2 * ... * 1
```
```erlang
Factorial(0).
% 1

Factorial(1).
% 1 * Factorial(0) => 1

Factorial(2).
% 2 * Factorial(1) => 2

..........

Factorial(N).
% N * Factorial(N - 1) => N * N - 1 * N - 2 * ... * 1
```
```ruby
factorial[0]
# 1

factorial[1]
# 1 * factorial[0] => 1

factorial[2]
# 2 * factorial[1] => 2

..........

factorial[n]
# n * factorial[n - 1] => n * n - 1 * n - 2 * ... * 1
```
```javascript
factorial(0);
// 1

factorial(1);
// 1 * factorial(0); => 1

// factorial(2);
// 2 * factorial(1); => 2

..........

factorial(n);
// n * factorial(n - 1); => n * n - 1 * n - 2 * ... * 1
```
{: .multi-code}

That's wonderful! Now we can just define the function!
Like with the functions we defined above, we should be able to define it and invoke it.
Let's define it:

```elixir
fn
  0 -> 1
  n -> n * me.(n - 1)
end

  ** (CompileError) iex:4: undefined function me/0
```
```erlang
fun
  (0) -> 1;
  (N) -> N * Me(N - 1)
end.

* 3: variable 'Me' is unbound
```
```ruby
-> (n) do
  if n == 0
    1
  else
    n * me[n - 1]
  end
end
#<Proc:0x007ffe1cf6ade8@(irb):1 (lambda)>

# but

-> (n) do
  if n == 0
    1
  else
    n * me[n - 1]
  end
end[2]

NameError: undefined local variable or method `me' for main:Object
```
```javascript
(n) => {
  if (n === 0) {
    return 1;
  } else {
    return n * me(n - 1);
  }
};
// function ()

// but

(
  (n) => {
    if (n === 0) {
      return 1;
    } else {
      return n * me(n - 1);
    }
  }
)(2);

ReferenceError: me is not defined
```
{: .multi-code}

Since [OTP 17](http://www.erlang.org/downloads/17.0) for [erlang]() {: .multi-code} it is possible to give a name of the anonymous function
and it will work. For this article we will forget about this language feature as well.

What's happening in the example above is normal. We are defining *factorial*, but it doesn't have a name so we don't know how to recursively call it in its body.
We just put `me` `Me` `me` `me`*multi-code*, but the name doesn't exist and we get an error.

But it is possible to write the factorial function in a way we can pass it to itself as an argument, so the name exists in its definition:

```elixir
fn (me) ->
  fn
    0 -> 1
    n -> n * me.(n - 1)
  end
end
#Function<6.52032458/1 in :erl_eval.expr/5>
```
```erlang
fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * Me(N - 1)
  end
end.
% #Fun<erl_eval.6.99386804>
```
```ruby
-> (me) do
  -> (n) do
    if n == 0
      1
    else
      n * me[n - 1]
    end
  end
end
# => #<Proc:0x007f0c01778f08@(irb):17 (lambda)>
```
```javascript
(me) => {
  return (n) => {
    if (n === 0) {
      return 1;
    } else {
      return n * me(n - 1);
    }
  };
};
// function ()
```
{: .multi-code}

That's working. No errors. But what should we pass as its `me` `Me` `me` `me`*multi-code* argument?
The first thing that comes to mind is to pass the *factorial* function definition to itself.
To ease the code a bit, we now have the permition (and will show the syntax) to assign the above function to a variable:

```elixir
factorial = fn (me) ->
  fn
    0 -> 1
    n -> n * me.(n - 1)
  end
end
#Function<6.52032458/1 in :erl_eval.expr/5>
```
```erlang
Factorial = fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * Me(N - 1)
  end
end.
% #Fun<erl_eval.6.99386804>
```
```ruby
factorial = -> (me) do
  -> (n) do
    if n == 0
      1
    else
      n * me[n - 1]
    end
  end
end
# => #<Proc:0x007f0c01778f08@(irb):17 (lambda)>
```
```javascript
let factorial = (me) => {
  return (n) => {
    if (n === 0) {
      return 1;
    } else {
      return n * me(n - 1);
    }
  };
};
// function ()
```
{: .multi-code}

Now, we can pass it to itself:

```elixir
factorial.(factorial)
#Function<6.52032458/1 in :erl_eval.expr/5>
```
```erlang
Factorial(Factorial).
% #Fun<erl_eval.6.99386804>
```
```ruby
factorial[factorial]
# => #<Proc:0x007f0c01576f48@(irb):26 (lambda)>>
```
```javascript
factorial(factorial);
// function factorial/<()
```
{: .multi-code}

...and that works too! Maybe this function implements factorial? It seems like that.
Let's try it out:

```elixir
factorial.(factorial).(0)
# 1
factorial.(factorial).(1)
** (ArithmeticError) bad argument in arithmetic expression
```
```erlang
(Factorial(Factorial))(0).
% 1
(Factorial(Factorial))(1).
** exception error: an error occurred when evaluating an arithmetic expression
     in operator  */2
             called as 1 * #Fun<erl_eval.6.99386804>
```
```ruby
factorial[factorial][0]
# => 1
factorial[factorial][1]
TypeError: Proc can't be coerced into Fixnum
```
```javascript
factorial(factorial)(0);
// 1
factorial(factorial)(1);
// NaN
```
{: .multi-code}

OK, it works only when we pass `0`, but why? I think the easiest way to understand what is happening is
to write out the actual function we get when we call our `factorial` `Factorial` `factorial` `factorial`*multi-code* function with itself as an argument:

```elixir
(fn (me) ->
  fn
    0 -> 1
    n -> n * me.(n - 1)
  end
end).(fn (me) ->
  fn
    0 -> 1
    n -> n * me.(n - 1)
  end
end)

=>

factorial_factorial = fn
  0 -> 1
  n -> n * (fn (me) ->
    fn
      0 -> 1
      n -> n * me.(n - 1)
    end
  end).(n - 1)
end

=>

factorial_factorial.(0) # Simple -> the 0 -> 1 case is evaluated and it is successful:
1

factorial_factorial.(1) # Here we hit the n -> n * me.(n - 1) case

=>

1 * (fn (me) ->
      fn
        0 -> 1
        n -> n * me.(n - 1)
      end
    end).(0)

=>

1 * (fn
      0 -> 1
      n -> n * 0.(n - 1)
    end)

# This expression can not be reduced any more, so we end up with
# a number multiplied by an anonymous function - ArithmeticError
```

```erlang
(fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * Me(N - 1)
  end
end)(fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * Me(N - 1)
  end
end).

=>

FactorialFactorial = fun
  (0) -> 1;
  (N) -> N * (fun (Me) ->
    fun
      (0) -> 1;
      (N) -> N * Me(N - 1)
    end
  end)(N - 1)
end.

=>

FactorialFactorial(0). % Simple -> the (0) -> 1 case is evaluated and it is successful:
1

FactorialFactorial(1). % Here we hit the (N) -> N * Me(N - 1) case

=>

1 * (fun (Me) ->
      fun
        (0) -> 1;
        (N) -> N * Me(N - 1)
      end
    end)(0).

=>

1 * (fun
      (0) -> 1;
      (N) -> N * 0(N - 1)
    end).

% This expression can not be reduced any more, so we end up with
% a number multiplied by an anonymous function - ArithmeticError
```

```ruby
-> (me) do
  -> (n) do
    if n == 0
      1
    else
      n * me[n - 1]
    end
  end
end[
  -> (me) do
    -> (n) do
      if n == 0
        1
      else
        n * me[n - 1]
      end
    end
  end
]

=>

factorial_factorial = -> (n) do
  if n == 0
    1
  else
    n * -> (me) do
      -> (n) do
        if n == 0
          1
        else
          n * me[n - 1]
        end
      end
    end[n - 1]
  end
end

=>

factorial_factorial[0] # Simple -> the n == 0 case is evaluated and it is successful:
1

factorial_factorial[1] # Here we hit the n * me[n - 1] case

=>

1 * ->(me) do
      -> (n) do
        if n == 0
          1
        else
          n * me[n - 1]
        end
      end
    end[0]

=>

1 * -> (n) do
  if n == 0
    1
  else
    n * 0[n - 1]
  end
end

# This expression can not be reduced any more so we end up with
# a number multiplied by an anonymous function :
# ruby tries to cast the anonymous function to a number - TypeError
```

```javascript
(
  (me) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else {
        return n * me(n - 1);
      }
    };
  }
)(
  (me) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else {
        return n * me(n - 1);
      }
    };
  }
);

=>

let factorialFactorial = (n) => {
  if (n === 0) {
    return 1;
  } else {
    return n * (
      (me) => {
        return (n) => {
          if (n === 0) {
            return 1;
          } else {
            return n * me(n - 1);
          }
        }
      }
    )(n - 1);
  }
};

=>

factorialFactorial(0); // Simple -> the n === 0 case is evaluated and it is successful:
1

factorialFactorial(1); // Here we hit the n * me(n - 1); case

=>

1 * (
  (me) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else {
        return n * me(n - 1);
      }
    };
  }
)(0);

=>

1 * (
  (n) => {
    if (n === 0) {
      return 1;
    } else {
      return n * 0(n - 1);
    }
  }
);

// This expression can not be reduced any more so we end up with
// a number multiplied by an anonymous function :
// javascript casts the function to number and that's NaN. '1 * NaN' is also NaN.
```
{: .multi-code}

That's good. We now know why it worked when we passed `0` to it and why it didn't when we passed `1`.
Now that we know what's the problem, we can start thinking about how we could solve it.

## Factorial II - Forever

How should we augment the `factorial` `Factorial` `factorial` `factorial`*multi-code* function that we have at the moment, so it could work when `1` is passed as an argument?
Let's isolate this case. We know that passing `0` works. Let's try making it work with `1` and not think about all the other infinitely many cases.

It is time to go back to the reduction, we called `factorial_factorial` `FactorialFactorial` `factorial_factorial` `factorialFactorial`*multi-code*:

```elixir
factorial_factorial = fn
  0 -> 1
  n -> n * (fn (me) ->
    fn
      0 -> 1
      n -> n * me.(n - 1)
    end
  end).(n - 1)
end

# And when we pass 1, we have this case:

1 * (fn (me) ->
      fn
        0 -> 1
        n -> n * me.(n - 1)
      end
    end).(0)

```
```erlang
FactorialFactorial = fun
  (0) -> 1;
  (N) -> N * (fun (Me) ->
    fun
      (0) -> 1;
      (N) -> N * Me(N - 1)
    end
  end)(N - 1)
end.

% And when we pass 1, we have this case:

1 * (fun (Me) ->
      fun
        (0) -> 1;
        (N) -> N * Me(N - 1)
      end
    end)(0).
```
```ruby
factorial_factorial = -> (n) do
  if n == 0
    1
  else
    n * -> (me) do
      -> (n) do
        if n == 0
          1
        else
          n * me[n - 1]
        end
      end
    end[n - 1]
  end
end

# And when we pass 1, we have this case:

1 * ->(me) do
      -> (n) do
        if n == 0
          1
        else
          n * me[n - 1]
        end
      end
    end[0]

```
```javascript
let factorialFactorial = (n) => {
  if (n === 0) {
    return 1;
  } else {
    return n * (
      (me) => {
        return (n) => {
          if (n === 0) {
            return 1;
          } else {
            return n * me(n - 1);
          }
        }
      }
    )(n - 1);
  }
};

// And when we pass 1, we have this case:

1 * (
      (me) => {
        return (n) => {
          if (n === 0) {
            return 1;
          } else {
            return n * me(n - 1);
          }
        }
      }
    )(0);
```
{: .multi-code}

This looks like `1 * factorial.(0)` `1 * Factorial(0).` `1 * factorial[0]` `1 * factorial(0);`*multi-code*. And we know that `factorial.(0)` `Factorial(0).` `factorial[0]` `factorial(0);`*multi-code* returns a function.
The working case of `0` is actually `factorial.(factorial).(0)` `(Factorial(Factorial))(0).` `factorial[factorial][0]` `factorial(factorial)(0);`*multi-code*.
That means if we think of a way to get `1 * factorial.(factorial).(0)` `1 * (Factorial(Factorial))(0).` `1 * factorial[factorial][0]` `1 * factorial(factorial)(0);`*multi-code* instead of `1 * factorial.(0)` `Factorial(0).` `1 * factorial[0]` `1 * factorial(0);`*multi-code* in the
above code, the `1` case would work as well.

Let's look at the way we defined the factorial function:

```elixir
factorial = fn (me) ->
  fn
    0 -> 1
    n -> n * me.(n - 1)
  end
end
```
```erlang
Factorial = fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * Me(N - 1)
  end
end.
```
```ruby
factorial = ->(me) do
      -> (n) do
        if n == 0
          1
        else
          n * me[n - 1]
        end
      end
    end
```
```javascript
let factorial = (me) => {
  return (n) => {
    if (n === 0) {
      return 1;
    } else {
      return n * me(n - 1);
    }
  };
};
```
{: .multi-code}

When we try to compute factorial of `1`, the problem we face is that `me.(0)` `Me(0).` `me[0]` `me(0);`*multi-code* returns a function and not
the factorial of `0`. We were thinking that `factorial.(factorial).(0)` `(Factorial(Factorial))(0).` `factorial[factorial][0]` `factorial(factorial)(0);`*multi-code*, which returns the factorial of `0`,
will solve the problem.

This means that if we replace `me.(n - 1)` `Me(N - 1)` `me[n - 1]` `me(n - 1);`*multi-code* with `me.(me).(n - 1)` `(Me(Me))(N - 1)` `me[me][n - 1]` `me(me)(n - 1);`*multi-code* when we call `factorial.(factorial).(1)` `(Factorial(Factorial))(1).` `factorial[factorial][1]` `factorial(factorial)(1);`*multi-code*,
we will get `1 * factorial.(factorial).(0)` `1 * (Factorial(Factorial))(0).` `1 * factorial[factorial][0]` `1 * factorial(factorial)(0);`*multi-code* which is `1 * 0!` or `1!`.
Let's try it:

```elixir
factorial = fn (me) ->
  fn
    0 -> 1
    n -> n * me.(me).(n - 1)
  end
end

factorial.(factorial).(0) # 0!
# 1
factorial.(factorial).(1) # 1!
# 1
```
```erlang
Factorial = fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * (Me(Me))(N - 1)
  end
end.

(Factorial(Factorial))(0). % 0!
% 1
(Factorial(Factorial))(1). % 1!
% 1
```
```ruby
factorial = ->(me) do
  -> (n) do
    if n == 0
      1
    else
      n * me[me][n - 1]
    end
  end
end

factorial[factorial][0] # 0!
# 1
factorial[factorial][1] # 1!
# 1
```
```javascript
let factorial = (me) => {
  return (n) => {
    if (n === 0) {
      return 1;
    } else {
      return n * me(me)(n - 1);
    }
  };
};

factorial(factorial)(0); // 0!
// 1
factorial(factorial)(1); // 1!
// 1
```
{: .multi-code}

Success! It works when we pass `1`! So now we can think of a way to implement `n!`.
Let's try with `factorial.(factorial).(2)` `(Factorial(Factorial))(2).` `factorial[factorial][2]` `factorial(factorial)(2);`*multi-code*, see the possible error and analyse how we got it in the same way we did it with
`factorial.(factorial).(1)` `(Factorial(Factorial))(1).` `factorial[factorial][1]` `factorial(factorial)(1);`*multi-code*:

```elixir
factorial.(factorial).(2)
# 2
# Hmm... That's like 2 * 1 * 0! - 2!

factorial.(factorial).(3)
# 6 or 3!
factorial.(factorial).(5)
# 120 or 5!
factorial.(factorial).(10)
# 3628800 or 10!
```
```erlang
(Factorial(Factorial))(2).
% 2
% Hmm... That's like 2 * 1 * 0! - 2!

(Factorial(Factorial))(3).
% 6 or 3!
(Factorial(Factorial))(5).
% 120 or 5!
(Factorial(Factorial))(10).
% 3628800 or 10!
```
```ruby
factorial[factorial][2]
# 2
# Hmm... That's like 2 * 1 * 0! - 2!

factorial[factorial][3]
# 6 or 3!
factorial[factorial][5]
# 120 or 5!
factorial[factorial][10]
# 3628800 or 10!
```
```javascript
factorial(factorial)(2);
// 2
// Hmm... That's like 2 * 1 * 0! - 2!

factorial(factorial)(3);
// 6 or 3!
factorial(factorial)(5);
// 120 or 5!
factorial(factorial)(10);
// 3628800 or 10!
```
{: .multi-code}

That's a working factorial function and we wrote it only using anonymous functions.
It is important to notice that we can write it without using variables at all (we were using variables only for shortening the code), like this:

```elixir
(fn (me) ->
  fn
    0 -> 1
    n -> n * me.(me).(n - 1)
  end
end).(fn (me) ->
  fn
    0 -> 1
    n -> n * me.(me).(n - 1)
  end
end).(5)
# 120
```
```erlang
((fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * (Me(Me))(N - 1)
  end
end)(fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * (Me(Me))(N - 1)
  end
end))(5).
% 120
```
```ruby
->(me) do
  -> (n) do
    if n == 0
      1
    else
      n * me[me][n - 1]
    end
  end
end[
  ->(me) do
    -> (n) do
      if n == 0
        1
      else
        n * me[me][n - 1]
      end
    end
  end
][5]
# 120

# Actually a shorter version would be:

->(me) { -> (n) { n == 0 ? 1 : n * me[me][n - 1] } } [
  ->(me) { -> (n) { n == 0 ? 1 : n * me[me][n - 1] } }
][5]
# 120
```
```javascript
(
  (me) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else {
        return n * me(me)(n - 1);
      }
    };
  }
)(
  (me) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else {
        return n * me(me)(n - 1);
      }
    };
  }
)(5);
// 120
```
{: .multi-code}

We implemented recursion _ourselves_.
Finally - how is this working?

## Factorial III - Infinity

Let's see what's happening when we invoke `factorial.(factorial).(3)` `(Factorial(Factorial))(3).` `factorial[factorial][3]` `factorial(factorial)(3);`*multi-code*, for example:

```elixir
(fn (me) ->
  fn
    0 -> 1
    n -> n * me.(me).(n - 1)
  end
end).(fn (me) ->
  fn
    0 -> 1
    n -> n * me.(me).(n - 1)
  end
end).(3)

=>

(fn
  0 -> 1
  n -> n * (fn (me) ->
              fn
                0 -> 1
                n -> n * me.(me).(n - 1)
              end
            end).(fn (me) ->
              fn
                0 -> 1
                n -> n * me.(me).(n - 1)
              end
            end).(n - 1)
end).(3)

=>

3 * (fn (me) ->
      fn
        0 -> 1
        n -> n * me.(me).(n - 1)
      end
    end).(fn (me) ->
      fn
        0 -> 1
        n -> n * me.(me).(n - 1)
      end
    end).(2)

=>

3 * (fn
      0 -> 1
      n -> n * (fn (me) ->
                  fn
                    0 -> 1
                    n -> n * me.(me).(n - 1)
                  end
                  end).(fn (me) ->
                    fn
                      0 -> 1
                      n -> n * me.(me).(n - 1)
                    end
                  end).(n - 1)
      end).(2)

# Notice that this is the same function from above, the one we invoked with 3.
# If we call it 'f', this is 3 * f.(2)

=>

3 * 2 * (fn
          0 -> 1
          n -> n * (fn (me) ->
                      fn
                        0 -> 1
                        n -> n * me.(me).(n - 1)
                      end
                      end).(fn (me) ->
                        fn
                          0 -> 1
                          n -> n * me.(me).(n - 1)
                        end
                      end).(n - 1)
        end).(1)
# 3 * 2 * f.(1)

=>

3 * 2 * 1 * (fn
              0 -> 1
              n -> n * (fn (me) ->
                      fn
                        0 -> 1
                        n -> n * me.(me).(n - 1)
                      end
                      end).(fn (me) ->
                        fn
                          0 -> 1
                          n -> n * me.(me).(n - 1)
                        end
                      end).(n - 1)
            end).(0)

# 3 * 2 * 1 * f.(0), but here 0 matches 0 -> 1 and we get:

=>

3 * 2 * 1 * 1 = 6 # 3!
```
```erlang
((fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * (Me(Me))(N - 1)
  end
end)(fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * (Me(Me))(N - 1)
  end
end))(3).

=>

(fun
  (0) -> 1;
  (N) -> N * ((fun (Me) ->
              fun
                (0) -> 1;
                (N) -> N * (Me(Me))(N - 1)
              end
            end)(fun (Me) ->
              fun
                (0) -> 1;
                (N) -> N * (Me(Me))(N - 1)
              end
            end))(N - 1)
end)(3).

=>

3 * ((fun (Me) ->
       fun
         (0) -> 1;
         (N) -> N * (Me(Me))(N - 1)
       end
     end)(fun (Me) ->
       fun
         (0) -> 1;
         (N) -> N * (Me(Me))(N - 1)
       end
     end))(2).

=>

3 * (fun
       (0) -> 1;
       (N) -> N * ((fun (Me) ->
                   fun
                     (0) -> 1;
                     (N) -> N * (Me(Me))(N - 1)
                   end
                 end)(fun (Me) ->
                   fun
                     (0) -> 1;
                     (N) -> N * (Me(Me))(N - 1)
                   end
                 end))(N - 1)
     end)(2).

% Notice that this is the same function from above, the one we invoked with 3.
% If we call it 'F', this is 3 * F(2).

=>

3 * 2 * (fun
          (0) -> 1;
          (N) -> N * ((fun (Me) ->
                      fun
                        (0) -> 1;
                        (N) -> N * (Me(Me))(N - 1)
                      end
                    end)(fun (Me) ->
                      fun
                        (0) -> 1;
                        (N) -> N * (Me(Me))(N - 1)
                      end
                    end))(N - 1)
        end)(1).

% 3 * 2 * F(1).

=>

3 * 2 * 1 * (fun
               (0) -> 1;
               (N) -> N * ((fun (Me) ->
                           fun
                             (0) -> 1;
                             (N) -> N * (Me(Me))(N - 1)
                           end
                         end)(fun (Me) ->
                           fun
                             (0) -> 1;
                             (N) -> N * (Me(Me))(N - 1)
                           end
                         end))(N - 1)
             end)(0).

% 3 * 2 * 1 * F(0).
% But here 0 matches (0) -> 1 and we get:

=>

3 * 2 * 1 * 1 = 6. % 3!
```
```ruby
->(me) do
  -> (n) do
    if n == 0
      1
    else
      n * me[me][n - 1]
    end
  end
end[
  ->(me) do
    -> (n) do
      if n == 0
        1
      else
        n * me[me][n - 1]
      end
    end
  end
][3]

=>

-> (n) do
  if n == 0
    1
  else
    n * ->(me) do
      -> (n) do
        if n == 0
          1
        else
          n * me[me][n - 1]
        end
      end
    end[
      ->(me) do
        -> (n) do
          if n == 0
            1
          else
            n * me[me][n - 1]
          end
        end
      end
    ][n - 1]
  end
end[3]

=>

3 * ->(me) do
      -> (n) do
        if n == 0
          1
        else
          n * me[me][n - 1]
        end
      end
    end[
      ->(me) do
        -> (n) do
          if n == 0
            1
          else
            n * me[me][n - 1]
          end
        end
      end
    ][2]

=>

3 * -> (n) do
      if n == 0
        1
      else
        n * ->(me) do
          -> (n) do
            if n == 0
              1
            else
              n * me[me][n - 1]
            end
          end
        end[
          ->(me) do
            -> (n) do
              if n == 0
                1
              else
                n * me[me][n - 1]
              end
            end
          end
        ][n - 1]
      end
    end[2]

# Notice that this is the same function from above, the one we invoked with 3.
# If we call it 'f', this is 3 * f[2]

=>

3 * 2 * -> (n) do
          if n == 0
            1
          else
            n * ->(me) do
              -> (n) do
                if n == 0
                  1
                else
                  n * me[me][n - 1]
                end
              end
            end[
              ->(me) do
                -> (n) do
                  if n == 0
                    1
                  else
                    n * me[me][n - 1]
                  end
                end
              end
            ][n - 1]
          end
        end[1]

# 3 * 2 * f[1]

=>

3 * 2 * 1 * -> (n) do
              if n == 0
                1
              else
                n * ->(me) do
                  -> (n) do
                    if n == 0
                      1
                    else
                      n * me[me][n - 1]
                    end
                  end
                end[
                  ->(me) do
                    -> (n) do
                      if n == 0
                        1
                      else
                        n * me[me][n - 1]
                      end
                    end
                  end
                ][n - 1]
              end
            end[0]

# 3 * 2 * 1 * f[0]
# But here n is 0, so f[0] returns just 1:

=>

3 * 2 * 1 * 1 == 6 # 3!
```
```javascript
(
  (me) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else {
        return n * me(me)(n - 1);
      }
    };
  }
)(
  (me) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else {
        return n * me(me)(n - 1);
      }
    };
  }
)(3);

=>

(
  (n) => {
    if (n === 0) {
      return 1;
    } else {
      return n * (
                  (me) => {
                    return (n) => {
                      if (n === 0) {
                        return 1;
                      } else {
                        return n * me(me)(n - 1);
                      }
                    };
                  }
                )(
                  (me) => {
                    return (n) => {
                      if (n === 0) {
                        return 1;
                      } else {
                        return n * me(me)(n - 1);
                      }
                    };
                  }
                )(n - 1);
    }
  }
)(3);

// or

factorial(factorial)(3);

=>

(
  (n) => {
    if (n === 0) {
      return 1;
    } else {
      return n * factorial(factorial)(n - 1);
    }
  }
)(3);

// if we reduce evem more =>

3 * (
      (me) => {
        return (n) => {
          if (n === 0) {
            return 1;
          } else {
            return n * me(me)(n - 1);
          }
        };
      }
    )(
      (me) => {
        return (n) => {
          if (n === 0) {
            return 1;
          } else {
            return n * me(me)(n - 1);
          }
        };
      }
    )(2);

// Notice that this is the same function from above, the one we invoked with 3.
// And this function is exactly factorial(factorial).
// If we call it 'f', this is 3 * f(2);

// or

3 * factorial(factorial)(2);

// But we can do the same reductions for factorial(factorial)(2) =>

3 * 2 * (
          (me) => {
            return (n) => {
              if (n === 0) {
                return 1;
              } else {
                return n * me(me)(n - 1);
              }
            };
          }
        )(
          (me) => {
            return (n) => {
              if (n === 0) {
                return 1;
              } else {
                return n * me(me)(n - 1);
              }
            };
          }
        )(1);

// or

3 * 2 * factorial(factorial)(1);

=>

3 * 2 * 1 * (
              (me) => {
                return (n) => {
                  if (n === 0) {
                    return 1;
                  } else {
                    return n * me(me)(n - 1);
                  }
                };
              }
            )(
              (me) => {
                return (n) => {
                  if (n === 0) {
                    return 1;
                  } else {
                    return n * me(me)(n - 1);
                  }
                };
              }
            )(0);

// or

3 * 2 * 1 * factorial(factorial)(0);

// But here n is 0, so factorial(factorial)(0); returns just 1:

=>

3 * 2 * 1 * 1 === 6; // 3!
```
{: .multi-code}

In other words we have a repeating function, which behaves like factorial.
And let's look at the function we called `factorial` `Factorial` `factorial` `factorial`*multi-code* again:

```elixir
factorial = fn (me) ->
  fn
    0 -> 1
    n -> n * me.(me).(n - 1)
  end
end
```
```erlang
Factorial = fun (Me) ->
  fun
    (0) -> 1;
    (N) -> N * (Me(Me))(N - 1)
  end
end.
```
```ruby
factorial = ->(me) do
  -> (n) do
    if n == 0
      1
    else
      n * me[me][n - 1]
    end
  end
end
```
```javascript
let factorial = (me) => {
  return (n) => {
    if (n === 0) {
      return 1;
    } else {
      return n * me(me)(n - 1);
    }
  };
};
```
{: .multi-code}

The name is not right for this function, is it? It is a higher-order function which takes itself to produce the factorial function.
The real definition of the factorial function, using only anonymous functions, would look like this:

```elixir
factorial = (fn (f) ->
  fn
    0 -> 1
    n -> n * f.(f).(n - 1)
  end
end).(fn (f) ->
  fn
    0 -> 1
    n -> n * f.(f).(n - 1)
  end
end)

factorial.(5)
# 120
```
```erlang
Factorial = (fun (F) ->
  fun
    (0) -> 1;
    (N) -> N * (F(F))(N - 1)
  end
end)(fun (F) ->
  fun
    (0) -> 1;
    (N) -> N * (F(F))(N - 1)
  end
end).

Factorial(5).
% 120
```
```ruby
factorial = ->(f) do
  -> (n) do
    if n == 0
      1
    else
      n * f[f][n - 1]
    end
  end
end[
  ->(f) do
    -> (n) do
      if n == 0
        1
      else
        n * f[f][n - 1]
      end
    end
  end
]

factorial[5]
# 120
```
```javascript
let factorial = (
  (f) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else {
        return n * f(f)(n - 1);
      }
    };
  }
)(
  (f) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else {
        return n * f(f)(n - 1);
      }
    };
  }
);

factorial(5);
// 120
```
{: .multi-code}

And with this definition, we did what we wanted to do.

We can play with this a bit more. Let's generalise our recursive function in a way it
could implement another one-argument recursive function. For example, let this be the function which computes the Nth
[Fibonacci number](https://en.wikipedia.org/wiki/Fibonacci_number).
We can do it like this:

```elixir
fib = (fn (f) ->
  fn
    0 -> 1
    1 -> 2
    n -> f.(f).(n - 1) + f.(f).(n - 2)
  end
end).(fn (f) ->
  fn
    0 -> 1
    1 -> 2
    n -> f.(f).(n - 1) + f.(f).(n - 2)
  end
end)

fib.(3)
# 5
```
```erlang
Fib = (fun (F) ->
  fun
    (0) -> 1;
    (1) -> 2;
    (N) -> (F(F))(N - 1) + (F(F))(N - 2)
  end
end)(fun (F) ->
  fun
    (0) -> 1;
    (1) -> 2;
    (N) -> (F(F))(N - 1) + (F(F))(N - 2)
  end
end).

Fib(3).
% 5
```
```ruby
fib = ->(f) do
  -> (n) do
    if n == 0 then 1
    elsif n == 1 then 2
    else f[f][n - 1] + f[f][n - 2]
    end
  end
end[
  ->(f) do
    -> (n) do
      if n == 0 then 1
      elsif n == 1 then 2
      else f[f][n - 1] + f[f][n - 2]
      end
    end
  end
]

fib[3]
# 5
```
```javascript
let fib = (
  (f) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else if (n === 1) {
        return 2;
      } else {
        return f(f)(n - 1) + f(f)(n - 2);
      }
    };
  }
)(
  (f) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else if (n === 1) {
        return 2;
      } else {
        return f(f)(n - 1) + f(f)(n - 2);
      }
    };
  }
);

fib(3);
// 5
```
{: .multi-code}

## Factorial IV - Generalisations

Abstraction is a very important concept in programming.
* It is good to try not to repeat code, when possible.
* It is great to reuse as much code as we can.

We are going to create a function, which will help us define arbitrary recursive functions of one argument.
Let's start with two definitions:

1. A function that has only bound variables is called *combinator*.
These are functions which depend only on the variables, declared as their arguments or in their bodies:

```elixir
id = fn (x) -> x end # This is a combinator - The I combinator or the identity combinator

y = 5
fn (x) -> x + y end
# This is not a combinator.
# It depends on some variable which is not declared in its body or as its arguments.
```
```erlang
I = fun (X) -> X end. % This is a combinator - The I combinator or the identity combinator

Y = 5.
fun (X) -> X + Y end.
% This is not a combinator.
% It depends on some variable which is not declared in its body or as its arguments.
```
```ruby
I = -> (x) { x } # This is a combinator - The I combinator or the identity combinator

-> (x) { x + y }
# This is not a combinator.
# It depends on some variable which is not declared in its body or as its arguments.
```
```javascript
let I = (x) => x; // This is a combinator - The I combinator or the identity combinator

(x) => x + y;
// This is not a combinator.
// It depends on some variable which is not declared in its body or as its arguments.
```
{: .multi-code}

2. The *Omega combinator* or the looping combinator is defined like this:
```elixir
o = fn (f) -> f.(f) end

o.(id) == id
# true
o.(id) == id.(id)
# true
```
```erlang
O = fun (F) -> F(F) end.

O(I) == I.
% true
O(I) == I(I).
% true
```
```ruby
O = -> (f) { f[f] }

O[I] == I
# true
O[I] == I[I]
# true
```
```javascript
let O = (f) => f(f);

O(I) === I;
// true
O(I) === I(I);
// true
```
{: .multi-code}

Returning to our definition of the factorial function, we can try to reuse a piece of code repeated twice:

```elixir
factorial = (fn (f) ->
  fn
    0 -> 1
    n -> n * f.(f).(n - 1)
  end
end).(fn (f) ->
  fn
    0 -> 1
    n -> n * f.(f).(n - 1)
  end
end)

=>

factorial = fn (g) ->
  g.(g)
end.(fn (f) ->
  fn
    0 -> 1
    n -> n * f.(f).(n - 1)
  end
end)
```
```erlang
Factorial = (fun (F) ->
  fun
    (0) -> 1;
    (N) -> N * (F(F))(N - 1)
  end
end)(fun (F) ->
  fun
    (0) -> 1;
    (N) -> N * (F(F))(N - 1)
  end
end).

=>

Factorial = (fun (G) ->
  G(G)
end)(fun (F) ->
  fun
    (0) -> 1;
    (N) -> N * (F(F))(N - 1)
  end
end).
```
```ruby
factorial = ->(f) do
  -> (n) do
    if n == 0
      1
    else
      n * f[f][n - 1]
    end
  end
end[
  ->(f) do
    -> (n) do
      if n == 0
        1
      else
        n * f[f][n - 1]
      end
    end
  end
]

=>

factorial = ->(g) do
  g[g]
end[
  ->(f) do
    -> (n) do
      if n == 0
        1
      else
        n * f[f][n - 1]
      end
    end
  end
]
```
```javascript
let factorial = (
  (f) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else {
        return n * f(f)(n - 1);
      }
    };
  }
)(
  (f) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else {
        return n * f(f)(n - 1);
      }
    };
  }
);

=>

let factorial = ((g) => g(g))(
  (f) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else {
        return n * f(f)(n - 1);
      }
    };
  }
);
```
{: .multi-code}

Yes, we can define the factorial function as just an invocation of the *proto-factorial* function with
one argument - itself. In other words, we are passing the *proto-factorial function* to the *Omega combinator*:

```elixir
factorial = o.(fn (f) ->
  fn
    0 -> 1
    n -> n * f.(f).(n - 1)
  end
end)
```
```erlang
Factorial = O(fun (F) ->
  fun
    (0) -> 1;
    (N) -> N * (F(F))(N - 1)
  end
end).
```
```ruby
factorial = o[
  ->(f) do
    -> (n) do
      if n == 0
        1
      else
        n * f[f][n - 1]
      end
    end
  end
]

# or

factorial = o[->(f) { -> (n) { n == 0 ? 1 : n * f[f][n - 1] } }]
```
```javascript
let factorial = O(
  (f) => {
    return (n) => {
      if (n === 0) {
        return 1;
      } else {
        return n * f(f)(n - 1);
      }
    };
  }
);
```
{: .multi-code}

That's much easier to read.
Now, let's extract the logic, specific for the factorial function, into a separate function:

```elixir
factorial = o.(fn (f) ->
  proto_factorial = fn g ->
    fn
      0 -> 1
      n -> n * g.(n - 1)
    end
  end

  proto_factorial.(fn (x) -> f.(f).(x) end)
end)
```
```erlang
Factorial = O(fun (F) ->
  ProtoFactorial = fun (G) ->
    fun
      (0) -> 1;
      (N) -> N * (G)(N - 1)
    end
  end,

  ProtoFactorial(fun (X) -> (F(F))(X) end)
end).
```
```ruby
factorial = o[
  ->(f) do
    proto_factorial = -> (g) do
      -> (n) do
        if n == 0
          1
        else
          n * g[n - 1]
        end
      end
    end


    proto_factorial[-> (x) { f[f][x] }]
  end
]
```
```javascript
let factorial = O(
  (f) => {
    let protoFactorial = (g) => {
      return (n) => {
        if (n === 0) {
          return 1;
        } else {
          return n * g(n - 1);
        }
      };
    };

    return protoFactorial((x) => f(f)(x));
  }
);
```
{: .multi-code}

It may seem more complex than before, but actually, it is not.
We extracted the essence of the factorial function in a special *proto* factorial function.
But why have we done it?
Because it is a step in the right direction - we want to extract the essence of the factorial from the code that allows us to define recursive anonymous functions.
In fact, we are very close to achieving this. The next step is to extract the *proto* factorial from the body of the `factorial` `Factorial` `factorial` `factorial`*multi-code* function:

```elixir
proto_factorial = fn g ->
  fn
    0 -> 1
    n -> n * g.(n - 1)
  end
end

factorial = (fn (h) ->
  o.(fn (f) ->
    h.(fn (x) -> f.(f).(x) end)
  end)
end).(proto_factorial)
```
```erlang
ProtoFactorial = fun (G) ->
  fun
    (0) -> 1;
    (N) -> N * (G)(N - 1)
  end
end.

Factorial = (fun (H) ->
  O(fun (F) ->
    H(fun (X) -> (F(F))(X) end)
  end)
end)(ProtoFactorial).
```
```ruby
proto_factorial = -> (g) do
  -> (n) do
    if n == 0
      1
    else
      n * g[n - 1]
    end
  end
end

factorial = -> (h) do
  o[-> (f) { h[-> (x) { f[f][x] }] } ]
end[proto_factorial]
```
```javascript
let protoFactorial = (g) => {
  return (n) => {
    if (n === 0) {
      return 1;
    } else {
      return n * g(n - 1);
    }
  };
};

let factorial = (
  (h) => {
    return O(
      (f) => h((x) => f(f)(x))
    );
  }
)(protoFactorial);
```
{: .multi-code}

Extracting a function definition from the body of another function is easy.
We just move its definition outside the defining function and wrap the defining function in another function,
so it can receive the extracted function as an argument and pass the extracted function.

Now we have the piece of code we wanted - a function that lets us define recursive anonymous functions of one argument:

```elixir
fn (h) ->
  o.(fn (f) ->
    h.(fn (x) -> f.(f).(x) end)
  end)
end

=>

y = fn (h) ->
  (fn (f) ->
    f.(f)
  end).(fn (g) ->
    h.(fn (x) -> g.(g).(x) end)
  end)
end
```
```erlang
fun (H) ->
  O(fun (F) ->
    H(fun (X) -> (F(F))(X) end)
  end)
end.

=>

Y = fun (H) ->
  (fun (F) ->
    F(F)
  end)(fun (G) ->
    H(fun (X) -> (G(G))(X) end)
  end)
end.
```
```ruby
-> (h) do
  o[-> (f) { h[-> (x) { f[f][x] }] } ]
end

=>

Y = -> (h) do
  -> (f) { f[f] }[
    -> (g) { h[-> (x) { g[g][x] }] }
  ]
end
```
```javascript
(h) => {
  return O(
    (f) => h((x) => f(f)(x))
  );
};

=>

let Y = (h) => {
  return ((f) => f(f))(
    (g) => h((x) => g(g)(x))
  );
};
```
{: .multi-code}

We can define the *Nth Fibonacci number function* like this:

```elixir
proto_fib = fn (f) ->
  fn
    0 -> 1
    1 -> 2
    n -> f.(n - 1) + f.(n - 2)
  end
end

fib = y.(proto_fib)
```
```erlang
ProtoFib = fun (F) ->
  fun
    (0) -> 1;
    (1) -> 2;
    (N) -> F(N - 1) + F(N - 2)
  end
end.

Fib = Y(ProtoFib).
```
```ruby
proto_fib = -> (f) do
  -> (n) do
    if n == 0 then 1
    elsif n == 1 then 2
    else f[n - 1] + f[n - 2]
    end
  end
end

fib = Y[proto_fib]
```
```javascript
let protoFib = (f) => {
  return (n) => {
    if (n === 0) {
      return 1;
    } else if (n === 1) {
      return 2;
    } else {
      return f(n - 1) + f(n - 2);
    }
  };
};

let fib = Y(protoFib);
```
{: .multi-code}

And that's how we defined the famous **Y combinator**. It's a higher-order function which allows
defining recursion in places where the recursion is not part of the language. We limited our
language to a subset which doesn't support recursion to prove that. There are infinitely many *Y combinators*, the
one we defined above works only with functions of one argument. It is a kind of *Y combinator*, known as **Z combinator**, because
[elixir]() {: .multi-code}, [erlang]() {:.multi-code}, [ruby]() {:.multi-code} and [javascript]() {:.multi-code} are not lazy languages and the arguments of a function are evaluated before passing them to the function.
In lazy languages there are even simpler *Y combinators*.

Another name of the *Y combinator* is _the fixed point combinator_. This means that the combinator returns a fixed point of the function passed to it.
A fixed point `x` for a function `f` is a value for which `f(x) = x`. And we can check that:

```elixir
fib = y.(proto_fib)

proto_fib.(fib).(5)
# 13
fib.(5)
# 13
proto_fib.(proto_fib.(fib)).(5)
# 13
```
```erlang
Fib = Y(ProtoFib).

(ProtoFib(Fib))(5).
% 13
Fib(5).
% 13
(ProtoFib(ProtoFib(Fib)))(5).
% 13
```
```ruby
fib = Y[proto_fib]

proto_fib[fib][5]
# 13
fib[5]
# 13
proto_fib[proto_fib[fib]][5]
# 13
```
```javascript
fib = Y(protoFib);

protoFib(fib)(5);
// 13
fib(5);
// 13
protoFib(protoFib(fib))(5);
// 13
```
{: .multi-code}

Let's create an alias of `proto_fib` `ProtoFib` `proto_fib` `protoFib`*multi-code* - `f` and another one of `y.(proto_fib)` `Y(ProtoFib)` `y[proto_fib]` `Y(protoFib)`*multi-code* - `x`.
We can say that `x` is fixed point of `f`. That's a fact, because the following equation is valid: `f(x) = x = f(f(x)) = f(..(f(..(f(x))..))..)`.

The *Y combinator* won't be of much practical use to you. All the languages, we work with, implement recursion.
Nevertheless, it is a very beautiful construct. It works a bit like magic. Its definition and understanding are not meaningless in the end.
We saw how we can abstract and generalise concepts using ideas and terms from the **lambda calculus**, the grandfather of all the functional programming languages.
And speaking of the *lambda calculus*, I'm thinking of writing more articles related to it and its concepts in the future, so stay tuned.

This post came to you with the help of [Andy](https://github.com/Andreshk) who edited it for grammer and wording mistakes.
Thank you for efford and time, Andy!

Bye for now.
