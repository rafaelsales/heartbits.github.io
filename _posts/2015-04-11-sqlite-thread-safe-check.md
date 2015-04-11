---
layout: post
title:  "SQLite thread-safety check"
date:   2015-04-11 02:50:00
categories: ruby sqlite threadsafe concurrency
comments: true
---

Before start using an [RDMS](http://en.wikipedia.org/wiki/Relational_database_management_system)
we should always evaluate what are its constraints and how we are planning to use it.

We often implement multithread processes that access database. One simple example
is acessing a database on a multithreaded application server, such as
[Puma](http://puma.io), or on a multithreaded background processing framework, such as
[Sidekiq](http://sidekiq.org).

When running code that access database on such multithreaded environments, we should
always evaluate if the database is ready to work with accept connections from multiple
threads and, sometimes, if a single connection can be shared with multiple threads.

### A real scenario

Recently I was working on an application that uses in-memory SQLite and the team decided
to migration from [Resque](https://github.com/resque/resque) - *a process-based background
processing framework* - to Sidekiq, which is thread-based. This change begs the question:
question: **Is SQLite thread-safe?** Well, it depends.

### Is SQLite thread-safe?

SQLite was build to work on three diferent modes in this regard and depends on
a compilation flag named `THREADSAFE`. It can have three values:

* `THREADSAFE=0`: It's unsafe to use SQLite in a multithreaded application - it ommits all
[mutexing](http://en.wikipedia.org/wiki/Mutual_exclusion) code on the compiled code;
* `THREADSAFE=1`: It's safe to use SQLite in a multithreaded application and multiple threads
can use the same connection. **This is the default option when compiling SQLite**;
* `THREADSAFE=2`: It's safe to use SQLite in a multithreaded application but each connection
must be used by a single thread at a time.

### Checking thread-safety level of SQLite installation

#### **In Ruby**

Open interactive Ruby console: `$ irb` and execute:

```ruby
SQLite3::Database.new(":memory:").
  execute("PRAGMA compile_options").
  map(&:first).find { |option| option =~ /THREADSAFE/ }

# This will produce an output like "THREADSAFE=2"
```

#### **On a C program**

Create a file named `sqlite_threadsafe.c` with following content:

```c
#include <sqlite3.h>

int main() {
  return sqlite3_threadsafe();
}
```

Then compile with:

```
$ gcc sqlite3_threadsafe.c -lsqlite3 -o sqlite3_threadsafe
```

And finally see the thread-safety level executing:

```
$ ./sqlite_threadsafe
2 # this is the value used on THREADSAFE compilation flag
```

#### **On SQLite console**

Execute `PRAGMA compile_options` on SQLite console to see all the compile options,
including the `THREADSAFE` option.
E.g.:

```
$ sqlite3

SQLite version 3.8.5 2014-08-15 22:37:57
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.

sqlite> PRAGMA compile_options;
ENABLE_FTS3
ENABLE_FTS3_PARENTHESIS
ENABLE_LOCKING_STYLE=1
ENABLE_RTREE
OMIT_AUTORESET
OMIT_BUILTIN_TEST
OMIT_LOAD_EXTENSION
SYSTEM_MALLOC
THREADSAFE=2
```

**References:**

* https://www.sqlite.org/compile.html#threadsafe
* http://www.sqlite.org/cvstrac/wiki?p=MultiThreading
