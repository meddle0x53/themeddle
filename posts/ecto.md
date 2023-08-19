---
category: Elixir
tags:
  - elixir
  - dbs
  - erlang
  - ets
  - dets
---

# ETS

If we believe that the processes in Erlang/Elixir can communicate with each other **only** through message passing,
we wouldn't be entirely correct.

Erlang/OTP also brings *Erlang Term Storage* or ETS.
ETS represents an in-memory key-value database that is part of the BEAM virtual machine.
It's not implemented in Erlang; instead, it's built into the virtual machine itself.
This means it's written in C and optimized for concurrent reading and writing.
Internally, it stores data that is *mutable*.

## The need of ETS

Sometimes, we need to have a process (let's say a `GenServer` or an `Agent`) that holds a state which should be accessible to multiple processes.

Let's imagine that we have a service (let's say an HTTP Server) and we want to keep track of statistics such as how many requests each user has made.
We could use this information to limit the number of requests a user can make within a certain time frame or to reward our most active users.
How could we achieve this?

The simplest approach is to maintain this statistics in memory using a specialized `GenServer`:


```elixir
defmodule RequestsPerUser do
  use GenServer

  def start_link do
    GenServer.start_link(__MODULE__, nil, name: __MODULE__)
  end

  def init(_), do: {:ok, %{}}

  def update(user), do: GenServer.cast(__MODULE__, {:update, user})
  def get(user), do: GenServer.call(__MODULE__, {:get, user})

  def handle_call({:get, user}, _, state) do
    {:reply, Map.get(state, user, 0), state}
  end

  def handle_cast({:update, user}, state) do
    {:noreply, Map.update(state, user, 1, & &1 + 1)}
  end
end
```

In this example, `RequestStatistics` is a `GenServer` that maintains a map of user IDs to the number of requests they have made.
You can increment the request count for a user and retrieve the request count for a user using the provided functions.

For each request to our service, a given process will perform an action (such as reading from a remote database or another service,
performing calculations, reading/writing files) and then return a result to the user.
Before returning this result, it will invoke `RequestsPerUser.update/1` to increment the request count for the given user:

```elixir
RequestsPerUser.update("peter60")
#=> :ok
```

If we have **1000+** requests per second, which is not uncommon for a heavily visited website or API,
it means that the `RequestsPerUser` process will receive **1000+** messages every second.
Each of these messages is processed sequentially.
For now, this isn't a problem (although `RequestsPerUser` might be slow) because the request processes simply perform casts.
However, let's imagine that our service provides access to this statistic.
Calling `RequestsPerUser.get/1` could take some time, as there might be thousands of other messages to be processed before the message sent by it.

In this way, we encounter a *bottleneck* - numerous processes are utilizing a single process to manage and modify a particular state.
Even if a program uses multiple threads/processes to achieve concurrent reading and writing, if there's one thread/process that all others rely on, it's as if we have a sequential program.

And here arises the question of how to implement shared state between processes in the BEAM with concurrent access.
The answer is ETS (*Erlang Term Storage*)!

## What is ETS?

As we mentioned, ETS stands for *Erlang Term Storage*.
We know that each process has its own stack and heap.
We also know that there is shared memory for larger binaries.
However, there is another form of shared memory - ETS.

ETS is not implemented in Erlang and is not built using Erlang processes or familiar persistent data structures like `Map` or `List`.
ETS is a component of the virtual machine, written in *C*, and the structures it employs are mutable.
Its interface is exposed in Erlang, and it might be perceived from the outside that access to its tables is achieved through messages, but this is not the case.
Accessing an ETS table is quite fast (*O(1)* or *O(log(n))*), and it supports concurrent operations.

ETS consists of a multitude of tables (up to **1400** by default, which can be altered using `iex --erl '-env ERL_MAX_ETS_TABLES N'`).
Each of these tables can be created by any process and is owned by a process (initially by the one that created it).

There are **4** types of tables: `:set`, `:ordered_set`, `:bag`, and `:duplicate_bag`.
Each table contains multiple entries, always represented by a `tuple`.
One element of each of these tuples is a key, and quick access can be achieved by searching based on it.

### Tables of type :set

These tables can have a record with a given key only once.
If we try to add a record with an existing key, we will overwrite the current one in the table.
They are implemented with a hash table, which promises read and write access in *O(1)* time complexity.
This hash table is *mutable*, as mentioned.

This is the default type of tables in ETS.

### Tables of type :ordered_set

These tables are managed similarly to `:set` tables, but their records are sorted by their keys.
It's possible to traverse these records using their order.
They are implemented using a balanced tree and offer read and write access with an average time complexity of O(log(n)).
Depending on the concurrency option the tree implementation can vary (for example for `write_concurrency` tables CA tree is used, but if that option is not specified AVL tree is used, [more](https://www.erlang.org/blog/the-new-scalable-ets-ordered_set)).

### Tables of type :bag/:duplicate_bag

Tables of type `:bag` allow multiple entries with the same key, but these entries cannot be identical.
For example, we can have:

```elixir
{1, "peter60", 16}
{1, "teddy84", 16}
```

In the above example, the key is the first element. However, with `:bag`, we cannot have:

```elixir
{1, "peter60", 16}
{1, "peter60", 16}
```

If we want to have fully repeating keys, we will use `:duplicate_bag`.

These types of tables are implemented using the same type of hash table as `:set` tables.


## Table access

When an ETS table is created, it is owned by a process.
When this process ceases to exist, the table either gets transferred to another process or is cleared from memory.
There are three possible types of access to ETS tables, which are related to the processes that own them, as well as the other processes in the BEAM.

### protected

The process that owns a table can both read from and write to it.
Other processes can only read from the given table.
This is the default access behavior.

### private

The process that owns a table can both read from and write to it.
Other processes do not have access to the table.

### public

All processes have both read and write access to the table.

## Table creation

Tables can be created by any process.
This process becomes the owner of the table.
To work with ETS tables, we use the Erlang module `:ets`.
To create a table, we use the function `:ets.new/2`:


```elixir
:ets.new(:users, [])
#=> #Reference<0.4003328283.155844609.148120>
```

We've created a table named :users, to which we have access using the reference value returned by `:ets.new/2`.
The list of options we provided is empty, so we are using the default options:


```elixir
table = :ets.new(:users, [])
#=> #Reference<0.4003328283.155844609.148120>

:ets.info(table)
#=>  [
#=>    id: #Reference<0.1206420823.2459828225.3724>,
#=>    decentralized_counters: false,
#=>    read_concurrency: false,
#=>    write_concurrency: false,
#=>    compressed: false,
#=>    memory: 305,
#=>    owner: #PID<0.110.0>,
#=>    heir: :none,
#=>    name: :users,
#=>    size: 0,
#=>    node: :nonode@nohost,
#=>    named_table: false,
#=>    type: :set,
#=>    keypos: 1,
#=>    protection: :protected
#=>  ]
```

As we can see, the access is `:protected`, and the table type is `:set`.
The option `:keypos` indicates the position in a record where the key will be.

Above, we see that the table is not named (`named_table: false`), which means the following won't work:

```elixir
:ets.insert(:users, {"meddle", 39, "Sofia", [:elixir, :programming, :gaming, :meddling, :music]})
#=>  ** (ArgumentError) errors were found at the given arguments:
#=>
#=>    * 1st argument: the table identifier does not refer to an existing ETS table
#=>
#=>      (stdlib 3.15) :ets.insert(:users, {"meddle", 33, "Sofia", [:elixir, :programming, :gaming, :meddling]})
```

We're attempting to add a new entry to the `:users` table, but we can't refer to it by its name (`name: :users`).
We can use its reference, and this will work:

```elixir
# From before:
table = :ets.new(:users, [])
#=> #Reference<0.4003328283.155844609.148120>


:ets.insert(table, {"meddle", 39, "Sofia", [:elixir, :programming, :gaming, :meddling, :music]})
#=> true

:ets.info(table, :size)
#=> 1
```

We can give the table a unique name - an *atom*, and every process will have access to this table (*for reading*, as it's `:protected`) through the name.
This is done as follows:

```elixir
:ets.new(:users, [:named_table])
#=> :users

:ets.info(:users, :named_table)
#=> true
```

We can notice a few things.
The function `:ets.new/2` doesn't return a reference; instead, it returns the *atom* `:users`, which is the table's name.
This name is unique to the current node, so we can reference the table by it.
If we try to create a table with the same name, which is `:named_table`, we will receive an `ArgumentError`.

Now, `:ets.info/1` contains `named_table: true`, and the good news is that there's also `:ets.info/2`, which can take any keyword key from the list returned by `:ets.info/1` as its second parameter.
This version of the function will return the value only for that key.
We used it with `:size` above to see how many records the table contains.
We will explore the other options more in detail while working with ETS tables.

## Writing to ETS tables

Adding a new entry to an ETS table is done using `:ets.insert/2` or `:ets.insert_new/2`, with the difference being that in `:ets.insert/2`,
it overwrites existing entries if the key already exists (in the context of `:set` and `:ordered_set tables`), while `:ets.insert_new/2` simply returns `false` in such a situation without writing anything.

Let's have a table that contains a set of people. In it, we'll have:

|Name|Family|Unique nick (key)|City|Age|Hobbies|
|:--:|:--:|:--:|:--:|:--:|:--:|
|Nikolay|Tsvetinov|meddle|Sofia|39|elixir, gaming, music, reading, writing|
|Simeon|Alexandrov|faxel|Sofia|38|javascript, vue.js, music, cleaning|
{: .table .table-striped .table-bordered .table-hover}


We have two sample rows.
We want the rows to be sorted by *nicknames*, and the table to be accessible for both reading and writing from anywhere:

```elixir
:ets.new(:people, [:named_table, :public, :ordered_set, keypos: 3])
#=> :people
```

This way, we achieve what we want.

Now we can add the two entries for Nikolay and Simeon:

```elixir
:ets.insert(
  :people,
  [
    {"Nikolay", "Tsvetinov", "meddle", 39, ["elixir", "gaming", "music", "reading", "writing"]},
    {"Simeon", "Alexandrov", "faxel", 38, ["javascript", "vue.js", "music", "cleaning"]}
  ]
)
#=> true

:ets.info(:people, :size)
#=> 2
```

We notice that the `insert` functions can also take a list of tuples, creating multiple entries at once in this way.


```elixir
:ets.insert_new(
  :people,
  [
    {"Simeon", "Alexandrov", "faxel", 38, ["javascript", "vue.js", "music", "cleaning"]},
    {"Dalia", "Tsvetinova", "dali", 9, ["gaming", "pokemon", "drawing", "reading"]}
  ]
)
#=> false

:ets.info(:people, :size)
#=> 2
```

If we try to create multiple entries with `insert_new`, and even just one of them already exists, the whole operation fails.
Creating multiple entries is an atomic operation.

We can add larger or smaller tuples as well.
The crucial thing is that they have at least as many elements as the `:keypos`.
In our case, it's three. If we try to add a tuple with two elements:

```elixir
:ets.insert(:people, {"Slavi", "Boyanov"})
#=> ** (ArgumentError) argument error
#=>    (stdlib) :ets.insert(:people, {"Slavi", "Boyanov"})
```

We get the familiar ArgumentError.

## Reading from ETS tables

There are many ways to read from ETS tables. The simplest way is to search by key:

```elixir
:ets.lookup(:people, "meddle")
#=> [
#=>   {"Nikolay", "Tsvetinov", "meddle", 39,
#=>    ["elixir", "gaming", "music", "reading"
#=>     "writing"]}
#=> ]


:ets.lookup(:people, "meddle0x53")
#=> []
```

The function `:ets.lookup/2` always returns a list.
This is because for *bag* and *duplicate bag* tables, there's a possibility of having more than one result.

If we need only a specific column for a given key, we can:

```elixir
age = :ets.lookup_element(:people, "meddle", 4)
#=> 39

age = :ets.lookup_element(:people, "meddle0x53", 4)
#=> ** (ArgumentError) errors were found at the given arguments:
#=>
#=>   * 2nd argument: not a key that exists in the table
#=>
#=>     (stdlib 3.15) :ets.lookup_element(:people, "meddle0x53", 4)
```

You'll notice that in ETS, positions start from **1**.
It can be confusing, but it's assumed that the table is numbered with columns from **1** to **N**.

### Selection through match specifications

Erlang has a special 'language' for data retrieval.
The language is a bit peculiar and archaic, and its complete description can be found [here](http://erlang.org/doc/apps/erts/match_spec.html).
Several functions in ETS use it for data retrieval.

A *match* specification consists of:
* **Head**: Describes the rows we want to match or select.
* **Guard**: Additional filters applied to these rows-entries.
* **Result**: How the result should appear. It can involve transformations of some kind.

Example:

```elixir
match_spec = [
  {
    {:"$1", :"$2", :_, :_, :"$3"}, # Head
    [{:andalso, {:is_list, :"$3"}, {:>, {:length, :"$3"}, 3}}], # Guard
    [{{:"$1", :"$2"}}] # Result
  }
]

:ets.select(:people, match_spec)
#=> [{"Nikolay", "Tsvetinov"}, {"Simeon", "Alexandrov"}]
```

The `:ets.select/2` function is a powerful tool for selecting, filtering, and transforming various results.
It uses the aforementioned match specification.
We won't delve into these specifications in detail, as that's the job of the documentation, but we'll explain what happens in the example.

1. The *Head* part describes the entries we want to select. We have `tuples` with **5** elements in the table, which is why we have it here. If we had users without hobbies (i.e., entries with 4 elements), they would automatically be filtered out by the Head part. Everything is described using atoms. The atom `:"_"` signifies something that we won't reference in other parts of the specification, whereas `:"$N"` is a variable that we'll reference, and the numbers of the variables don't matter. If we weren't interested in how the entry looks, instead of a `tuple`, we could simply put `:"_"`. What we've described is something like `fn ({v1, v2, _, _, v3}) -> ... end`.
2. The *Guard* part represents filters. They're described in `tuples` of `atoms`. The first element is the operation, and the others are its parameters. We have the `:andalso` operation with two other tuples as parameters. The first one is the `:is_list` operation with the parameter `:"$3"`, which is a reference to the *Head* part. We're checking if the fifth element of the entries is a list. With the next operation, we ensure that this list has a length greater than **3**. What we've described so far is something like `fn ({v1, v2, _, _, v3}) when is_list(v3) and length(v3) > 3 -> ... end`.
3. The *Result* part describes the result. A list of `tuples` where we have the values of the first and second columns, as described in the *Head* part, or something like `fn ({v1, v2, _, _, v3}) when is_list(v3) and length(v3) > 3 -> {v1, v2} end`.

We can think of the *match specification* as an anonymous function that pattern-matches the rows, can have guards, and a body that transforms the result.
It's just a special syntax. In fact, there's an `:ets` function that converts regular functions into match specifications:

```elixir
match_func =
  fn {v1, v2, _, _, v3} when is_list(v3) and length(v3) > 3 ->
    {v1, v2}
  end
#=> #Function<6.99386804/1 in :erl_eval.expr/5>

:ets.fun2ms(match_func)
#=> [
#=>   {{:"$1", :"$2", :_, :_, :"$3"},
#=>    [{:andalso, {:is_list, :"$3"}, {:>, {:length, :"$3"}, 3}}],
#=>    [{{:"$1", :"$2"}}]}
#=> ]
```

This way, we can build and extract as *module attributes* the *match specifications* that we will use.
Keep in mind that while the `:ets.fun2ms/1` function can be convenient for converting regular functions into match specifications, it has some limitations:
1. *Complexity*: `:ets.fun2ms/1` works well for simple cases, but as the complexity of the function increases, the translation might become more convoluted and less intuitive. It may not handle all cases perfectly.
2. *Unsupported Constructs*: Not all Erlang constructs and functions can be accurately translated into match specifications. Some advanced language features or constructs might not have a straightforward representation in the match specification language.
3. *Performance*: While *match specifications* can be efficient for certain tasks, using the `:ets.fun2ms/1` function to create match specifications from arbitrary functions might lead to less optimized query execution compared to hand-crafted match specifications. Manually optimizing the match specification might yield better performance.
4. *Limited Documentation*: The documentation for `:ets.fun2ms/1` is not as extensive as one might hope, and there might be certain nuances or edge cases that are not clearly documented.
5. *Maintenance*: If you heavily rely on automatically generated match specifications from `:ets.fun2ms/1`, changes to the original Erlang function might not be automatically reflected in the match specification. This could lead to unintended behavior or outdated queries.

Overall, while `:ets.fun2ms/1` can be useful for quickly converting simple functions into match specifications, it's important to be aware of its limitations and carefully evaluate its usage for more complex scenarios.
For complex or performance-critical queries, manually crafting match specifications might be a more reliable approach. We recommend using it while developing the code to generate the *match specification*, then to test what was generated and if needed to modify it.
Don't use it in runtime.

There's also a version of `:ets.select/2` with a third parameter, limit, which restricts the number of returned results.
For `:ordered_set`, the results will always be sorted by the key. For other table types, the order of results is not deterministic; it will be in the order they are stored in the table.

We've explored the most complex but also the most powerful way to read from ETS tables. There are other methods that build upon `:ets.select/2` and can be used for simpler (and more common) cases.

### Reading through simple matching

Simplified pattern matching over the rows of the table can be achieved with `:ets.match/2`:

```elixir
:ets.match(:people, {:"$1", :"$2", :"_", :"_", :"_"})
#=> [["Nikolay", "Tsvetinov"], ["Simeon", "Alexandrov"]]
```

Rows are matched, and then the 'variables' are used for selection.
If the key is specified, the matching process is quite fast, which is important for `:bag` and `:duplicate_bag` tables.
If we want to select entire rows without specifying 'variables', we can use `:ets.match_object/2`, which always returns the whole records and doesn't pay attention to variables except for pattern matching.

The *match*-related functions also have variants with a third parameter - `limit`.
Matching follows the usual Elixir/Erlang pattern matching, meaning we can do things like:

```elixir
:ets.match(:people, {:"_", :"_", :"$2", :"_", [:"$1" | :"_"]})
[["javascript", "faxel"], ["elixir", "meddle"]]
```

## Traversing ETS tables

ETS contains functions for traversing the rows of a table as a list.

```elixir
:ets.first(:people)
#=> "faxel"
```

In `:ordered_set` tables, the order is based on the keys, which are sorted.
For other table types, the order depends on how the records are internally stored.
The function `:ets.first/1` will return the key of the first row. Similarly, `:ets.last/1` will return the key of the last row.

Keys can be used for traversal:

```elixir
:ets.next(:people, :ets.first(:people))
#=> "meddle"

:ets.prev(:people, "meddle")
#=> "faxel"

:ets.next(:people, "meddle")
#=> :"$end_of_table"

:ets.prev(:people, "faxel")
#=> :"$end_of_table"
```

When the atom `:"$end_of_table"` is reached, it means there is no smaller (prev) or larger (next) key in the table compared to the given value.

Values can be specified for which there are no keys in the table. They will be compared, and the nearest key will be found:

```elixir
:ets.prev(:people, "g")
#=> "faxel"

:ets.next(:people, "n")
#=> :"$end_of_table"
```

## Modifications

An easy way to modify a record is using `:ets.update_element/3`:

```elixir
:ets.update_element(:people, "meddle", {1, "Meddle"})
#=> true

:ets.lookup(:people, "meddle")
#=> [
#=>   {"Meddle", "Tsvetinov", "meddle", 39,
#=>    ["elixir", "gaming", "music", "reading"
#=>     "writing"]}
#=> ]
```

You provide the table, key, and a `tuple` of the position for the change and the new value.

Another easy way is to use `:ets.insert/2` for an existing key, which will overwrite the row.

There are specialized `update_counter` functions for atomic increment or decrement of a counter stored in the table.
An example is the following implementation of `RequestsPerUser`:

```elixir
defmodule RequestsPerUser do
  use GenServer

  def start_link do
    GenServer.start_link(__MODULE__, nil, name: __MODULE__)
  end

  def init(_) do
    {:ok, :ets.new(__MODULE__, [:named_table, :public])}
  end

  def update(user) do
    :ets.update_counter(__MODULE__, user, {2, 1}, {user, 0})
  end
  def get(user) do
    case :ets.match(__MODULE__, {user, :"$1"}) do
      [[n]] -> n
      [] -> 0
    end
  end
end

RequestsPerUser.start_link()
#=> {:ok, #PID<0.115.0>}

RequestsPerUser.get("meddle")
#=> 0

RequestsPerUser.update("meddle")
#=> 1
RequestsPerUser.get("meddle")
#=> 1
```

Here, the table is publicly accessible for reading and writing by all processes, but it is created by a `GenServer` process, which is the initial owner.
We can add various operations that are infrequently executed on it and need to be performed in a single process, such as `handle_call` and `handle_cast` callback functions.

## Deleting records

We can easily delete a record by its key:


```elixir
:ets.delete(:people, "meddle")
#=> true

:ets.info(:people, :size)
#=> 1
```

Or by pattern matching:

```elixir
:ets.match_delete(:people, {"Simeon", :"_", :"_", :"_", :"_"})
#=> true

:ets.info(:people, :size)
#=> 0
```

We can also free the entire table from memory when needed:

```elixir
:ets.delete(:people)
#=> true
```

Public tables can be deleted by any process. If a table is not public, it can only be deleted by the process that owns it.

## When to use ETS?

If multiple processes need to use shared state and require concurrent and fast access to it, the solution is ETS.
It can be used to store global state or configuration that needs to be accessed across multiple processes.
This allows processes to share information without the need for explicit message passing.

Another use of ETS is to store the current state of a process, which can be restored if the process dies.
However, this approach is not typical for the "let it crash" principle, so think carefully before implementing it.
It's often cleaner for a restarted process to start with its initial state.

ETS tables can serve as lookup tables for mapping keys to values. This is useful for scenarios where you need fast and efficient lookups based on a key.
They also can be used to transform or manipulate data. For instance, you could store raw data in one ETS table and perform various transformations on it using ETS functions.
You can use ETS to maintain shared counters and statistics across processes. This is useful for scenarios where multiple processes need to keep track of counts, sums, averages, or other metrics.

ETS can also be used to register known processes.
In fact, in Elixir, this functionality is implemented through the `Registry` module.
This module internally uses ETS to store data related to a given process or a name associated with a process.
Related to that, ETS tables can be used as a central hub for a publish-subscribe messaging system.
Processes can subscribe to specific events or topics and receive updates through ETS.
You can also use them to coordinate activities between processes in a distributed system.
Processes can use ETS tables to communicate, share status, and coordinate actions.
In scenarios where messages from multiple processes need to be aggregated or summarized, ETS tables can be used to accumulate and process these messages efficiently.
ETS tables can be used to manage concurrency control mechanisms such as semaphores or locks. Processes can coordinate access to shared resources using ETS-based locks.

ETS tables can be used as a caching mechanism to store frequently accessed data and avoid costly computations or database queries.
This can improve the performance of applications by reducing the need to recalculate or fetch data repeatedly.

ETS tables can be used to implement rate limiting mechanisms.
For example, you can use ETS to track the number of requests made by an IP address within a certain time frame and apply rate limits accordingly.

ETS tables can be used to store dynamic configuration settings that can be updated at runtime without the need to restart the application.

## When to not use ETS?

When we want to work with data that can be sent from one node to another, ETS cannot be copied between nodes.

When we need a distributed database across multiple nodes, there's actually an abstraction over ETS that serves this purpose — *Mnesia*.

When we want to persist our data between restarts of our node, it's best to use *DETS*, although there are functions in the `:ets` module that can save the state of the database to a file and read state from a file.

In most cases, simple communication between processes suffices.
For testing, if temporary in-memory state is needed, it's better to use an `Agent`.
For memoizing a result while building data that will be used later within a process, it's better to use a structure in the process's heap.

ETS is optimized for fast read and write operations, but if your use case involves extremely frequent updates and lookups, especially in a concurrent environment, the contention for locks in ETS might become a performance bottleneck.

ETS provides basic querying and indexing capabilities through its match specifications, but for more complex querying needs, you might need a dedicated database system that supports more advanced query languages.
If your data has complex relationships that require referential integrity checks or complex joins, ETS might not provide the necessary tools to efficiently handle these scenarios. A database system with relational capabilities might be more suitable.

ETS primarily addresses the single-process bottleneck problem. If our data lives only in the process that uses it or if only one process reads/writes from the current one, ETS is not necessary.

## More ETS

### Heirs

As we mentioned, if the owning process stops existing, the ETS table owned by it gets cleared from memory.
If we don't want this to happen, we can set a *heir* process.
This can be done in `:ets.new/2` using the option `{:heir, <pid>, <data>}`.

Let's have an example:

```elixir
defmodule Heir do
  use GenServer

  def start_link do
    GenServer.start_link(__MODULE__, nil, name: __MODULE__)
  end

  def transfer_data(table) do
    GenServer.call(__MODULE__, {:transfer_data, table})
  end

  def init(_), do: {:ok, %{}}

  def handle_info({:"ETS-TRANSFER", table, from, data}, state) do
    IO.puts("#{inspect table} transfered from #{inspect from} to #{inspect self()}")

    {:noreply, Map.put(state, table, data)}
  end

  def handle_call({:transfer_data, table}, _, state) do
    {:reply, Map.get(state, table), state}
  end
end
```

Now if we start this process:

```elixir
{:ok, heir_pid} = Heir.start_link()
#=> {:ok, #PID<0.115.0>}
```

We can specify it as an heir to the table:

```elixir
spawn(fn -> :ets.new(:some_table, [:named_table, {:heir, heir_pid, "woo"}]) end)
#=> #PID<0.118.0>

#output: :some_table transfered from #PID<0.118.0> to #PID<0.115.0>

:ets.info(:some_table, :owner)
#=> #PID<0.115.0>
```

What actually happens is that when a `:heir` is specified, if the owner process of a table dies, the table isn't immediately cleared from memory.
Instead, the *heir process* receives a message of the form `{:"ETS-TRANSFER", <table name or reference>, <previous owner pid>, <inheritance information>}`.
As soon as the original owner process ceases to exist, the table becomes owned by the *heir process*.
The information provided during the heir assignment can be used for debugging or tracking the inheritance history.

If we don't specify a heir during table creation, we can do so later using `:ets.setopts(<table>, heir_tuple)`.
It's also possible to manually change the owner of a table even when the original owner process is still alive.
This can be done with `:ets.give_away/3` (`:ets.give_away(<table>, <heir_pid>, <transfer_data>)`).

By default, the heir value is set to `:none`. We can always remove a heir by setting this value:

```elixir
:ets.setopts(:some_table, {:heir, :none})
```

This should be done from the owner process, otherwise an `ArgumentError` will be raised.

### Options for concurrent writing and reading

There are two optimization options for common scenarios, both of which are disabled (`false`) by default.
* If a table will have infrequent writes on multi-core environment we can mark it as `read_concurrency: true`. If we do this and frequently alternate between read and write operations, access to the table will degrade.
* If a table will experience multiple non-overlapping changes that do not affect the same records—such as insert, delete, and similar functions—we can mark it as `write_concurrency: true`. Again, if we alternate between these write operations and frequent reads, it's not a good idea. The faster concurrent writes come at the cost of using more memory.
* We can combine both options if we have periods of heavy reading and periods of frequent writing.

These options are related to the way locking and synchronization for accessing records in the tables are managed.
They have been introduced to contribute to optimizing specific cases and should be used only in those cases.
It's best to determine the values of these options through observation and testing.
For some types of tables on some types of environment (single-core, for example) they may do nothing. For some types of tables they can change the whole implementation of the table (as we mentioned when we talked about `:ordered_set`).

Additionally to these two options from *OTP 22* there is a new option `decentralized_counters`. It is an option that helps to scale wrties (remove/update) wtith the number of utilized cores. More on it [here](https://www.erlang.org/blog/scalable-ets-counters/)

### Compression

We can set the `:compressed` option on a table.
This means that everything except the key of a given record will be compressed, which will optimize the memory used by the table.
The trade-off of this optimization is that reading entire rows will become slower. Adding new records will also become slower.

## Alternatives to and abstractions on ETS

### DETS

DETS (Disk Erlang Term Storage) is slower than ETS for most tasks, with nearly the same interface.
However, it writes to disk, which is why it's slower.
It has an `:auto_save` option that flushes it to disk every **3** seconds by default.
It's used when we don't want to lose state. If our program exits without crashing, DETS will be flushed.
It doesn't support `:ordered_set` tables.

Its largest use-case is Mnesia, but even there, we can replace it with other solutions like [RocksDB](https://github.com/aeternity/mnesia_rocksdb) or [LevelDB](https://github.com/klarna/mnesia_eleveldb).
DETS has a limitation of **2GB** per file, which leads Mnesia to distribute data across files, making it slower with larger databases.

### Mnesia

Mnesia is a collection of ETS tables. If disk persistence is required, DETS is used.
It employs a relational/object hybrid model. The relationships between ETS tables are implemented using other ETS tables.
It also utilizes ETS's spec language for querying.

Mnesia supports ACID (atomicity, consistency, isolation, durability) transactions.
It allows backend plugins for storage (we mentioned two), enabling DETS to be swapped as the disk storage option.
It is distributed, meaning multiple nodes can have shared data.
An Elixir wrapper for Mnesia is [Amnesia](https://github.com/meh/amnesia).

In summary, Mnesia extends the capabilities of ETS by providing a distributed and fault-tolerant storage solution with features like
data replication, synchronization, schema evolution, and transaction support across distributed nodes.
It's a powerful choice for applications that require both scalability and high availability.

### Persistent Term

An alternative to ETS is `:persistent_term`, which shares similarities with ETS in its concept of shared memory.
However, the difference lies in its further optimization for reading, while being less efficient for writing and updating operations.
Reading (`get/1`) is always *O(1)*. There is no locking, and data copying to the process is avoided.

On the other hand, writing (`put/2`) is proportional to the number of all Persistent Terms because everything is copied – this is how this memory works.
Writes and deletions often lead to *garbage collection* (GC) across all processes referencing the Persistent Term table.
If a term is deleted, which is still in use by a process, it is saved in the heap of that process.
This can result in unexpectedly high memory usage.

It's better to have a few large Persistent Terms rather than many small ones.
They are beneficial for shared configuration among processes and for holding references to long-lived resources like NIF resources.
They are also useful for maintaining references to ETS tables and counters.

This is another optimization for various special cases.

### Process dictionary

A local map with fast access in each process.
It is often used to store process-related metadata.
Do not use it if you don't understand why.

Inside a process, it behaves as a global value!
`Logger` uses it for metadata, `IEX` uses it for history.
A process starting a `Task` uses it with the `"$ancestors"` key.

Learn more about it [here](https://brainlid.org/elixir/2017/09/24/elixir-processes-and-state-abuse.html).

## Conclusion

This post is something I wrote years ago (and updated and translated to English now) for the Sofia University ["Functional Programming With Elixir"](https://elixir-lang.bg/en/posts) course.
I hope it can be used as a quick reference and show what you can do with ETS in a simplified human-readable way.

This is available as a [presentation in Bulgarian](https://slides.elixir-lang.bg/slides/ecto.html) too.
