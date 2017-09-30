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

# Да си дефинираме рекурсия

Това е обновен вариант на публикация, която написах за [elixir-lang.bg](https://elixir-lang.bg/posts).
Също, можете да я намерите на [English на този блог](http://themeddle.com/en/posts/y).

Ще направим едно доста интересно упражнение.
Ще използваме под-множество на езика, което не поддържа рекурсия и ще се опитаме да дефинираме рекурсивна функция.
Под език, всъщност разбираме _няколко_ езика. Добавих интересна функционалност към fron-end-а на моя блог engine - [BlogitWeb](https://github.com/meddle0x53/blogit_web),
която ми позволява да пиша примери на повече от един програмен език и по този начин да дам възможност на читателите да изберат на кой от тях искат да ги четат.

В тази публикация ще използваме[elixir]() {: .multi-code}, [erlang]() {:.multi-code}, [ruby]() {:.multi-code} и [javascript]() {:.multi-code}.
Нека започнем.

## Подготовка

За всеки от гореспоменатите езици за програмиране ще използваме само интерпретаторът му и няма да ползваме именовани функции.
Ще забравим, че [elixir]() {: .multi-code}, [erlang]() {:.multi-code}, [ruby]() {:.multi-code} и [javascript]() {:.multi-code} поддържат именовани функции.

И така, нека пуснем избрания интерпретатор:

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
# или някой модерен браузър (Firefox?)
```
{: .multi-code}

Нека се ограничим с използването само на цели числа, прости аритметични операции с тях и сравнения между тях:

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

Ще се опитаме да дефинираме [функцията Факториел](https://bg.wikipedia.org/wiki/%D0%A4%D0%B0%D0%BA%D1%82%D0%BE%D1%80%D0%B8%D0%B5%D0%BB).
Точно поради тази причина ще използваме цели числа и операции с тях.
Нека да видим, как можем да си дефинираме проста анонимна функция. Ще дефинираме функция, която събира своитр два аргумента и връща резултата:

```elixir
fn (a, b) -> a + b end
```
```erlang
fun (A, B) -> A + B end.
```
```ruby
proc { |a, b| a + b }

# или

lambda { |a, b| a + b }

# или

-> (a, b) { a + b }
# Ще използваме синтаксиса '->' за тази публикация.
```
```javascript
(function (a, b) { return a + b; });

// или

(a, b) => { return a + b; };

// Ще използваме ситаксиса '=>' за тази публикация.
```
{: .multi-code}

Сега, можем да извикваме тази функция по следния начин:

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

# или

-> (a, b) { a + b }[4, 5]
# 9

# Ще използваме синтаксиса '[]' за тази публикация.
```
```javascript
(function (a, b) { return a + b; })(4, 5);
// 9

// или

((a, b) => { return a + b; })(4, 5);
// 9
```
{: .multi-code}

Възможно е да се изпълнява различна логика, в зависимост от аргументите, които подаваме на функция:

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

Това всъщност е функция, която връща числото преди това, което сме ѝ подали. За нула просто връща нула, тъй като нулата няма предходно число.
Приемаме, че не знаем нищо за отрицателните числа в нашето малко подмножество на езика.

Добре! Вече знам всичко необходимо да си дефинираме функцията Факториел! Но така ли е наистина?

## Факториел - Началото

Дотук знаем как да си дефинираме функция.
Нека помислим какво трябва да бъде поведението на факториел функцията, ако я имахме дефинирана:

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

Чудесно, нека сега да го дефинираме като анонимна функция!
Подобно на функциите, които дефинирахме в предишната секция, би трябвало да можем да си дефинираме факториел и да я извикаме.
Нека я дефинираме:

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

Нормално. Дефинираме *факториел* без да му даваме име и поради това не знаем как да го извикаме рекурсивно от собственото му тяло.
Поради това, за да сложим нещо, слагаме `me` `Me` `me` `me`*multi-code*, което не съществува като име и имаме грешка.

От друга страна можем да пробваме да направим така, че да имаме такова име, което да е известно
при дефиниране. Как? Ами като се подаде на функция от по-висок ред като аргумент:

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

Това работи и няма грешки. Въпросът е, какво да подадем за `me` `Me` `me` `me`*multi-code*?
Нека пробваме да извикаме тази дефиниция на *факториел* със самата нея.
За да опростим кода, позволяваме (и ще покажем синтаксиса) използването на променливи и присвояване на стойност на променлива:

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

Сега неко го извикаме с него като аргумент:

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

Работи. Върна функция. Може би тази функция е имплементацията на *факториел*?
Така изглежда, да пробваме:

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

Добре. За `0` работи, а за `1` - не. Защо? Най-добре ще си го обясним ако разпишем какво
се случва, когато се изпълня функцията  `factorial` `Factorial` `factorial` `factorial`*multi-code*:

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

factorial_factorial.(0) # Просто -> случаят 0 -> 1 се изпълнява и резултатът е 1:
1

factorial_factorial.(1) # Това е случаят n -> n * me.(n - 1)

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

# Този израз не може да се редуцира повече. Получаваме число, което се умножава
# по анонимна функция - ArithmeticError
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

FactorialFactorial(0). % Просто -> случаят (0) -> 1 се изпълнява и резултатът е 1:
1

FactorialFactorial(1). % Това е случаят (N) -> N * Me(N - 1)

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

% Този израз не може да се редуцира повече. Получаваме число, което се умножава
% по анонимна функция - ArithmeticError
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

factorial_factorial[0] # Просто -> the n == 0 се изпълнява и резултатът е 1:
1

factorial_factorial[1] # Това е случаят n * me[n - 1]

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

# Този израз не може да се редуцира повече. Получаваме число, което се умножава
# по анонимна функция :
# ruby се опитва да cast-не тази функция към число - TypeError
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

factorialFactorial(0); // Просто -> the n === 0 се изпълнява и резултатът е 1:
1

factorialFactorial(1); // Това е случаят n * me(n - 1);

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

// Този израз не може да се редуцира повече.  Получаваме число, което се умножава
//по анонимна функция :
// javascript преобразува функцията в число и това е NaN. '1 * NaN' е също NaN.
```
{: .multi-code}

Добре. Разбрахме защо за `0` работи, а за `1` - не. Сега като знаем какъв е проблемът,
нека помислим как да го решим.

## Факториел II - Завинаги

Как да променим `factorial` `Factorial` `factorial` `factorial`*multi-code* за да работи в случая с `1`?
Нека изолираме само този случай. С `0` работи. Нека проработи с `1`.
Другите безкрайни на брой случаи ще ги оставим за после.

Да се върнем към редукцията която нарекохме `factorial_factorial` `FactorialFactorial` `factorial_factorial` `factorialFactorial`*multi-code*:

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

# И когато подадем 1, стигаме до тук:

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

% И когато подадем 1, стигаме до тук:

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

# И когато подадем 1, стигаме до тук:

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

// И когато подадем 1, стигаме до тук:

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

Това не е ли `1 * factorial.(0)` `1 * Factorial(0).` `1 * factorial[0]` `1 * factorial(0);`*multi-code*?
А знаем, че `factorial.(0)` `Factorial(0).` `factorial[0]` `factorial(0);`*multi-code* връща функция.
От друга страна `factorial.(factorial).(0)` `(Factorial(Factorial))(0).` `factorial[factorial][0]` `factorial(factorial)(0);`*multi-code* е случаят който работи за `0`.
Добре тогава - нека някак направим така че вместо `1 * factorial.(0)` `Factorial(0).` `1 * factorial[0]` `1 * factorial(0);`*multi-code* да имаме `1 * factorial.(factorial).(0)` `1 * (Factorial(Factorial))(0).` `1 * factorial[factorial][0]` `1 * factorial(factorial)(0);`*multi-code*.

Нека отново да погледнем дефиницията:

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

Когато подадем `factorial` `Factorial` `factorial` `factorial`*multi-code* на `factorial` `Factorial` `factorial` `factorial`*multi-code* в случая с `1` виждаме, че проблемът е,
че `me.(0)` `Me(0).` `me[0]` `me(0);`*multi-code* (`me` `Me` `me` `me`*multi-code* при това извикване се заменя с `factorial` `Factorial` `factorial` `factorial`*multi-code*) връща фунцкия,
а не факториел от `0` и си казахме че, `factorial.(factorial).(0)` `(Factorial(Factorial))(0).` `factorial[factorial][0]` `factorial(factorial)(0);`*multi-code*, което
връща факториел от `0` ще помогне.

Това означава, че ако вместо `me.(n - 1)` `Me(N - 1)` `me[n - 1]` `me(n - 1);`*multi-code* имаме  `me.(me).(n - 1)` `(Me(Me))(N - 1)` `me[me][n - 1]` `me(me)(n - 1);`*multi-code*, при извикване
на `factorial.(factorial).(1)` `(Factorial(Factorial))(1).` `factorial[factorial][1]` `factorial(factorial)(1);`*multi-code* ще получим `1 * factorial.(factorial).(0)` `1 * (Factorial(Factorial))(0).` `1 * factorial[factorial][0]` `1 * factorial(factorial)(0);`*multi-code*, което
е `1*0!` или `1!`. Нека да пробваме:

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

Успяхме! Имаме `0!` и `1!`. Сега е време да помислим как да имплементираме `n!`.
Нека да пробваме с `2!`, да видим грешката и да разгърнем извикването, както направихме
с `factorial.(factorial).(1)` `(Factorial(Factorial))(1).` `factorial[factorial][1]` `factorial(factorial)(1);`*multi-code* преди малко:

```elixir
factorial.(factorial).(2)
# 2
# Интересно! Работи и връща верен резултат : 2 * 1 * 0! - 2!

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
% Интересно! Работи и връща верен резултат : 2 * 1 * 0! - 2!

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
# Интересно! Работи и връща верен резултат : 2 * 1 * 0! - 2!

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
// Интересно! Работи и връща верен резултат : 2 * 1 * 0! - 2!

factorial(factorial)(3);
// 6 or 3!
factorial(factorial)(5);
// 120 or 5!
factorial(factorial)(10);
// 3628800 or 10!
```
{: .multi-code}

Това е работещ факториел написан с анонимни функции. Това е рекурсия, написана
само с анонимни функции.
Даже можем да го напишем без да ползваме променливи така:

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

# Ето и по-кратка версия:

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

Имплементирахме *своя* рекурсия.
Но как работи?

## Факториел III - Безкрайност

Нека да видим какво се случва с `factorial.(factorial).(3)` `(Factorial(Factorial))(3).` `factorial[factorial][3]` `factorial(factorial)(3);`*multi-code*.

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

# Забелязваме, че това е същата функция от по-горе, на която подадохме 3.
# Ако я наречем 'f' (factorial.(factorial)), този израз е всъщност 3 * f.(2) (3 * factorial.(factorial).(2))

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

# 3 * 2 * 1 * f.(0), но тук 0 е случая 0 -> 1 и получаваме:

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

% Забелязваме, че това е същата функция от по-горе, на която подадохме 3.
% Ако я наречем 'F' (Factorial(Factorial)), този израз е всъщност 3 * F(2). (3 * (Factorial(Factorial))(2).)

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
% Но тук 0 е случая (0) -> 1 и получаваме:

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

# Забелязваме, че това е същата функция от по-горе, на която подадохме 3.
# Ако я наречем 'f' (factorial[factorial]), този израз е всъщност 3 * f[2] (3 * factorial[factorial][2]).

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
# Но в този случай n е 0, от което следва, че f[0] връща 1:

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

// или

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

// ако редуцираме с още една стъпка =>

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


// Забелязваме, че това е същата функция от по-горе, на която подадохме 3.
// Ако я наречем 'f' (factorial(factorial)), този израз е всъщност 3 * f(2), или:

// or

3 * factorial(factorial)(2);

// Но същите пребразувания могат да се приложат и към factorial(factorial)(2) =>

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

// или

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

// или

3 * 2 * 1 * factorial(factorial)(0);

// Но тук n е 0, от което следва, че factorial(factorial)(0) връща 1:

=>

3 * 2 * 1 * 1 === 6; // 3!
```
{: .multi-code}

В общи линии достигнахме до повтаряща се функция, която се държи като факториел.
В този ред на мисли:

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

Това е грешно име за тази функция.
Ето я истинската дефиниция на `factorial` `Factorial` `factorial` `factorial`*multi-code* като анонимна функция:

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

И тук можем да спрем.

Но защо да не генерализираме малко тази конструкция, така
че да работи и за други функции на един аргумент - да речем *Фибуначи*.
Използвайки тази идея можем да дефинираме функцията намираща `n`-тото число от редицата на Фибуначи така:

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

## Факториел IV - Генерализации

Абстракцията е мого важна част от програмирането.
* Добре е да не повтаряме кода си, ако можем.
* Още по-добре е да преизползваме толкова код, колкото можем.

Ще създадем функция, чрез която ще можем да си дефинираме рекурсивни анонимни функции на един аргумент.
За да направим това, ще ни трябват следните две дефиниции:

1. Ще наричаме функции които имат само свързани променливи комбинатори.
Това са функции които зависят само от променливи декларирани като техни аргументите или в телата им:

```elixir
id = fn (x) -> x end # Това е комбинатор - I комбинаторът или identity комбинаторът.

y = 5
fn (x) -> x + y end
# Това не е комбинатор.
# Зависи от променлива, която не е декларирана в тялото на функцията или не е сред аргументите ѝ.
```
```erlang
I = fun (X) -> X end. % Това е комбинатор - I комбинаторът или identity комбинаторът.

Y = 5.
fun (X) -> X + Y end.
% Това не е комбинатор.
% Зависи от променлива, която не е декларирана в тялото на функцията или не е сред аргументите ѝ.
```
```ruby
I = -> (x) { x } # Това е комбинатор - I комбинаторът или identity комбинаторът.

-> (x) { x + y }
# Това не е комбинатор.
# Зависи от променлива, която не е декларирана в тялото на функцията или не е сред аргументите ѝ.
```
```javascript
let I = (x) => x; // Това е комбинатор - I комбинаторът или identity комбинаторът.

(x) => x + y;
// Това не е комбинатор.
// Зависи от променлива, която не е декларирана в тялото на функцията или не е сред аргументите ѝ.
```
{: .multi-code}

2. Ще дефинираме един комбинатор, наречен Омега комбинатор или looping комбинатор:

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

Да се върнем към нашата *факториел* функция:

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

Да, факториелът е представен като извикване на тази *прото-факториел* функция със
себе си. С други думи:

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

# или

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

Нека да опитаме да извадим логиката специфична за факториела във функция,
която прилича на класическата дефиниция на факториел. Можем ли?

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

Сега изглежда по-сложно, но не е. Извадихме това, което прави факториела - факториел,
есенцията на имплементацията му в обособена *прото* факториел функция.
А защо направихме това? Защото е стъпка към целта - да изкараме есенцията на
факториела от конструкцията, която позволява рекурсия с анонимни функции.
Всъщност ние сме доста близко до тази цел. Следващата стъпка ще е да извадим *прото* факториел функцията навън:

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

Изваждането на една функция е лесно, просто обвиваме тялото на функцията
която я дефинира в друга функция, вадим дефиницията навън и я подаваме на обвиващата функция.
Сега имаме конструкцията, която ни позволява да построим рекурсия за анонимни функции с един аргумент.
Ето я и нея:

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

Ето как с нея можем да си построим функцията за намиране на N-тото число на Фибоначи:

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

И така изведохме небезизвестния **Y комбинатор**. Това е функция от по висок ред,
която позволява функции без имена да поддържат рекурсия. Този изведен от нас го позволява
само за едно-аргументни функции. Освен това е вид *Y комбинатор*, по-известен като
**Z комбинатор**, защото [elixir]() {: .multi-code}, [erlang]() {:.multi-code}, [ruby]() {:.multi-code} и [javascript]() {:.multi-code} не са lazy езици и си изчисляват аргументите преди да ги подадат на функцията.

Друго известно наименование на *Y комбинатора* е _the fixed point combinator_. Тоест връща неподвижната точка
на функцията, която приема като аргумент. Неподвижна точка `x` за функция `f` е такава стойност, за която
`f(x) = x`. Нека да проверим:

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

Ако по горе  `proto_fib` `ProtoFib` `proto_fib` `protoFib`*multi-code* означим с `f`, a `y.(proto_fib)` `Y(ProtoFib)` `y[proto_fib]` `Y(protoFib)`*multi-code* с `x`, то `x` е неподвижна точка на `f`.
Това е така, защото равенството е изпълнено `f(x) = x = f(f(x)) = f(..(f(..(f(x))..))..)`.

Комбинаторът *Y* няма да ви трябва практически, ще работите с именовани функции, все пак, но
е една много красива конструкция. Работи някак магически и все пак няма никаква магия в него.
Разписването и разбирането му, никак не е безмислено упражнение.
Видяхме как абстрахираме и генерализираме концепции чрез функционално програмиране,
също се запознахме с няколко термина идващи от ламбда-смятането.

Това е всичко... засега.
