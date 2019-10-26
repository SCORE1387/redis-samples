## Redis cheatsheet

- [Links](#links)
- [Databases](#databases)
- [Run](#run)
- [Basics](#basics)
- [Data Structures](#data-structures)
- [Publish/Subscribe](#publish-subscribe)
- [Transactions](#transactions)
- [Misc](#misc)

### Links

* [Redis documentation](https://redis.io/documentation)
* [Redis commands](https://redis.io/commands)

### Run

Redis uses 6379 port by default.

Run Redis using docker:
```
    docker run --name redis -p 6379:6379 -d redis
```

Connect to Redis using redis-cli in docker:
```
    docker exec redis -it redis-cli
```

### Databases

Redis can have up to 15 __databases__ identified by number.

Switch database (0-15 values are allowed):
```
    SELECT 1
```

Clean database:
```
    FLUSHDB
```
Clean all databases:

```
    FLUSHALL
```

Get the number of keys in the selected database:
```
    DBSIZE
```

### Basics 

Set the value:
```
    SET server:name "fido"
```

Set the value if not exist:
```
    SETNX server:name "fido"
```

Retrieve value:
```
    GET server:name => "fido"
```

Delete value:
```
    DEL server:name
```

---

Redis can be told that a key should only exist for a certain length of time. This is accomplished with the `EXPIRE` and `TTL` commands.

```
    SET resource:lock "Redis Demo"
    EXPIRE resource:lock 120
    TTL resource:lock => 119
```

The -2 for the `TTL` of the key means that the key does not exist (anymore). A -1 for the `TTL` of the key means that it will never expire. Note that if you `SET` a key, its `TTL` will be reset.

---

There are more useful commands:

`INCR key` - increment values 

`APPEND key value` - appends value to the end of string

`GETRANGE key start end` - get substring

`GETSET key value` - get current value and set new in one operation 

`MGET key [key ...]` - multiple get 

`MSET key value [key value ...]` - multiple set

`MSETNX key value [key value ...]` - multiple set if not exists

`SETEX key seconds value` - set a value that expires

`SETRANGE key offset value` - rewrite a substring

`STRLEN key` - get length of string

### Data Structures

Redis also supports several more complex data structures. The first one we'll look at is a list. A list is a series of ordered values. Some of the important commands for interacting with lists are `RPUSH`, `LPUSH`, `LLEN`, `LRANGE`, `LPOP`, and `RPOP`. You can immediately begin working with a key as a list, as long as it doesn't already exist as a different type.

`RPUSH` puts the new value at the end of the list.

```
    RPUSH friends "Alice"
    RPUSH friends "Bob"
```
	
`LPUSH` puts the new value at the start of the list.

```
    LPUSH friends "Sam"
```
	
`LRANGE` gives a subset of the list. It takes the index of the first element you want to retrieve as its first parameter and the index of the last element you want to retrieve as its second parameter. A value of -1 for the second parameter means to retrieve elements until the end of the list.

```
    LRANGE friends 0 -1 => 1) "Sam", 2) "Alice", 3) "Bob"
    LRANGE friends 0 1 => 1) "Sam", 2) "Alice"
    LRANGE friends 1 2 => 1) "Alice", 2) "Bob"
```

---

`LLEN` returns the current length of the list.

```
    LLEN friends => 3
```
	
`LPOP` removes the first element from the list and returns it.

```
    LPOP friends => "Sam"
```

`RPOP` removes the last element from the list and returns it.

```
    RPOP friends => "Bob"
```

Note that the list now only has one element:

```
    LLEN friends => 1
    LRANGE friends 0 -1 => 1) "Alice"
```

---

The next data structure that we'll look at is a set. A set is similar to a list, except it does not have a specific order and each element may only appear once. Some of the important commands in working with sets are `SADD`, `SREM`, `SISMEMBER`, `SMEMBERS` and `SUNION`.

`SADD` adds the given value to the set.

```
    SADD superpowers "flight"
    SADD superpowers "x-ray vision"
    SADD superpowers "reflexes"
```

`SREM` removes the given value from the set.

```
    SREM superpowers "reflexes"
```

---

`SISMEMBER` tests if the given value is in the set. It returns 1 if the value is there and 0 if it is not.

```
    SISMEMBER superpowers "flight" => 1
    SISMEMBER superpowers "reflexes" => 0
```

`SMEMBERS` returns a list of all the members of this set.

```
    SMEMBERS superpowers => 1) "flight", 2) "x-ray vision"
```

`SUNION` combines two or more sets and returns the list of all elements.

```
    SADD birdpowers "pecking"
    SADD birdpowers "flight"
    SUNION superpowers birdpowers => 1) "pecking", 2) "x-ray vision", 3) "flight"
```

---

Sets are a very handy data type, but as they are unsorted they don't work well for a number of problems. This is why Redis 1.2 introduced Sorted Sets.

A sorted set is similar to a regular set, but now each value has an associated score. This score is used to sort the elements in the set.

```
    ZADD hackers 1940 "Alan Kay"
    ZADD hackers 1906 "Grace Hopper"
    ZADD hackers 1953 "Richard Stallman"
    ZADD hackers 1965 "Yukihiro Matsumoto"
    ZADD hackers 1916 "Claude Shannon"
    ZADD hackers 1969 "Linus Torvalds"
    ZADD hackers 1957 "Sophie Wilson"
    ZADD hackers 1912 "Alan Turing"
```

In these examples, the scores are years of birth and the values are the names of famous hackers.

```
    ZRANGE hackers 2 4 => 1) "Claude Shannon", 2) "Alan Kay", 3) "Richard Stallman"
```

---

Simple strings, sets and sorted sets already get a lot done but there is one more data type Redis can handle: Hashes.

Hashes are maps between string fields and string values, so they are the perfect data type to represent objects (eg: A User with a number of fields like name, surname, age, and so forth):

```
    HSET user:1000 name "John Smith"
    HSET user:1000 email "john.smith@example.com"
    HSET user:1000 password "s3cret"
```

To get back the saved data use `HGETALL`:

```
    HGETALL user:1000
```

You can also set multiple fields at once:

```
    HMSET user:1001 name "Mary Jones" password "hidden" email "mjones@example.com"
```

If you only need a single field value that is possible as well:

```
    HGET user:1001 name => "Mary Jones"
```

---

Numerical values in hash fields are handled exactly the same as in simple strings and there are operations to increment this value in an atomic way.

```
    HSET user:1000 visits 10
    HINCRBY user:1000 visits 1 => 11
    HINCRBY user:1000 visits 10 => 21
    HDEL user:1000 visits
    HINCRBY user:1000 visits 1 => 1
```

### Publish/Subscribe

```
PUBSUB <subcommand> ... args ...
PUBSUB CHANNELS [pattern]
PUBSUB NUMSUB [channel-1 ... channel-N]
PSUBSCRIBE pattern [pattern ...]
SUBSCRIBE channel [channel ...]
```

Example:

```
    redis-client1: subscribe channel1
    redis-client2: publish channel1 "test str"
```

### Transactions

`MULTI` - Start a transaction
`EXEC` - Execute a transaction

`DISCARD` - Ditch a transaction
`WATCH` - Start watching a key for changes
`UNWATCH` - Stop watching a key for changes

### Misc

List all matching keys:
```
    KEYS pattern
```

List clients:

```
    CLIENT LIST
```

Asynchronously save the dataset to disk

```
    BGSAVE
```

Listen for all requests received by the server in real time:

```
    MONITOR
```

Disable some command:

```
    rename-command FLUSHALL 
```

### Replication

Redis поддерживает репликацию, которая означает, что все данные, которые попадают
на один узел Redis (который называется master) будут попадать также и на другие узлы
(называются slave). Для конфигурирования slave-узлов можно изменить опцию slaveof
или выполнить аналогичную по написанию команду (узлы, запущенные без подобных
опций являются master-узлами).
Репликация помогает защитить ваши данные, копируя их на другие сервера. Репликация
также может быть использована для увеличения производительности, т.к. запросы на
чтение могут обслуживаться slave-узлами. Эти узлы могут ответить слегка устаревшими
данными, но для большинства приложений это приемлемо.
К сожалению, система репликации Redis еще не поддерживает автоматическую
отказоустойчивость. Если master-узел выходит из строя, необходимо вручную выбрать
новый из slave-узлов. Необходимо использовать традиционные утилиты, использующие
мониторинг и специальные скрипты для переключения master-узлов, если вам необходима
устойчивая к сбоям система.

Configuring slave:

`slaveof 192.168.1.1 6379`

