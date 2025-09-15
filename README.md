# dbdriver

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-0.13.0-green.svg)](#)

A database driver library written in Nature programming language (>0.6.0), providing unified access interface for MySQL, PostgreSQL, and Redis.

## 📦 Installation

Add the dependency to your `package.toml` file:

```toml
[dependencies]
db = { type = "git", version = "v0.13.0", url = "github.com/weiwenhao/dbdriver" }
```

Run `npkg sync` in the directory where package.toml is located

## 🔧 Quick Start

### MySQL Connection Example

```nature
import db.mysql
import db.result

fn main():void! {
    // Establish connection
    var c = mysql.connect("127.0.0.1:3306", "root", "root", "test_db")
    var r = c.execute("CREATE TABLE IF NOT EXISTS users (id INT PRIMARY KEY AUTO_INCREMENT, name VARCHAR(50), email VARCHAR(100), age INT)")

    r2 = c.query("SELECT * FROM users")
    r2.dump()

    fmt.printf("\n=== Scan Result Set ===\n")
    type user_t = struct{
        int id
        string name
        string? email
        int age
    }
    var list = result.scan<user_t>(r2) 
    for item in list {
        fmt.printf("ID: %d, Name: %s, Email: %v, Age: %d\n", item.id, item.name, item.email, item.age)
    }

    // Prepared statement
    var insert_stmt = c.prepare("INSERT INTO users (name, email, age) VALUES (?, ?, ?)")

    // Use prepared statement to insert data
    r = insert_stmt.execute("Zhao Liu", "zhaoliu@example.com", 35)

    var select_stmt = c.prepare("SELECT * FROM users WHERE id > ?")

    // Use prepared statement to query data
    var r3 = select_stmt.query(1)
    r3.dump()
    insert_stmt.close()
    select_stmt.close()

}
```

### Redis Connection Example

```nature

import db.redis

fn main():void! {
    var conn = redis.connect('127.0.0.1:6379', 'hello')
    
    // Basic string operations
    var resp = conn.execute('SET', 'test_key', 'Hello Redis!')
    println('SET test_key:', resp)

    resp = conn.execute('GET', 'test_key')
    println('GET test_key:', resp)

    resp = conn.execute('PING')
    println('PING:', resp)

    // String operation tests
    resp = conn.execute('APPEND', 'test_key', ' World!')
    println('APPEND test_key:', resp)

    resp = conn.execute('GET', 'test_key')
    println('GET test_key after append:', resp)

    resp = conn.execute('STRLEN', 'test_key')
    println('STRLEN test_key:', resp)

    // Number operations
    resp = conn.execute('SET', 'counter', '10')
    println('SET counter:', resp)

    resp = conn.execute('INCR', 'counter')
    println('INCR counter:', resp)

    resp = conn.execute('DECR', 'counter')
    println('DECR counter:', resp)

    resp = conn.execute('INCRBY', 'counter', '5')
    println('INCRBY counter 5:', resp)

    // Key operations
    resp = conn.execute('EXISTS', 'test_key')
    println('EXISTS test_key:', resp)

    resp = conn.execute('EXPIRE', 'test_key', '60')
    println('EXPIRE test_key 60:', resp)

    resp = conn.execute('TTL', 'test_key')
    println('TTL test_key:', resp)

    // List operations
    resp = conn.execute('LPUSH', 'test_list', 'item1', 'item2', 'item3')
    println('LPUSH test_list:', resp)

    resp = conn.execute('LLEN', 'test_list')
    println('LLEN test_list:', resp)

    resp = conn.execute('LPOP', 'test_list')
    println('LPOP test_list:', resp)

    resp = conn.execute('RPUSH', 'test_list', 'item4')
    println('RPUSH test_list:', resp)

    // Hash operations
    resp = conn.execute('HSET', 'test_hash', 'field1', 'value1')
    println('HSET test_hash:', resp)

    resp = conn.execute('HGET', 'test_hash', 'field1')
    println('HGET test_hash field1:', resp)

    resp = conn.execute('HMSET', 'test_hash', 'field2', 'value2', 'field3', 'value3')
    println('HMSET test_hash:', resp)

    resp = conn.execute('HLEN', 'test_hash')
    println('HLEN test_hash:', resp)

    // Set operations
    resp = conn.execute('SADD', 'test_set', 'member1', 'member2', 'member3')
    println('SADD test_set:', resp)

    resp = conn.execute('SCARD', 'test_set')
    println('SCARD test_set:', resp)

    resp = conn.execute('SISMEMBER', 'test_set', 'member1')
    println('SISMEMBER test_set member1:', resp)

    // Sorted set operations
    resp = conn.execute('ZADD', 'test_zset', '1', 'one', '2', 'two', '3', 'three')
    println('ZADD test_zset:', resp)

    resp = conn.execute('ZCARD', 'test_zset')
    println('ZCARD test_zset:', resp)

    resp = conn.execute('ZSCORE', 'test_zset', 'two')
    println('ZSCORE test_zset two:', resp)

    // Clean up test data
    resp = conn.execute('DEL', 'test_key', 'counter', 'test_list', 'test_hash', 'test_set', 'test_zset')
    println('DEL cleanup:', resp)
}
```

### Result Mapping

#### Mapping to Structs

```nature
import db.result as db_result

fn main() {
    type user_t = struct {
        int id
        string name
        int? age 
        string email
    }

    var r = conn.query("SELECT id, name, age, email FROM users")
    var users = db_result.scan<user_t>(r)
    for user in users {
        println(user.id, user.name, user.age, user.email)
    }
}
```

#### Mapping to Map

```nature
import db.result as db_result

fn main() {
    var r = conn.query("SELECT name, value FROM settings")
    var settings = db_result.scan<{string:string}>(r)
    for setting in settings {
        println(setting['name'], setting['value'])
    }
}
```


## 📄 License

This project is licensed under the MIT License