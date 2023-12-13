# 14. Setup ruby-duckdb 

## Setup duckdb from git source

Build a DB while referring to [the documentation](https://duckdb.org/docs/installation/?version=latest&environment=cli&installer=source)

```
$ git clone  https://github.com/duckdb/duckdb.git ~/.duckdb
$ cd ~/.duckdb
$ git checkout DUCKDB_LATEST_VERSION
$ make -j8
```

Try to connect Database

```
$ ~/.duckdb/build/release/duckdb
v0.9.2 3c695d7ba9
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
D 
```

## Setup ruby-duckdb from git source

Follow the README.md in [this repository](https://github.com/suketa/ruby-duckdb)

```
$ ruby -v  #Install Ruby in advance
ruby 3.2.2 (2023-03-30 revision e51014f9c0) [x86_64-linux]
$ gem install bundler
$ gem install duckdb -- \
  --with-duckdb-include=/home/username/.duckdb/src/include \
  --with-duckdb-lib=/home/username/.duckdb/build/release/src # Specify the pre-installed Duckdb source
```

## Check the connectivity

From irb

```
$ irb
irb(main):001:0> require 'duckdb'
=> true
irb(main):002:0> db = DuckDB::Database.open
=> #<DuckDB::Database:0x00007f45d9fb0e20>
irb(main):003:0> con = db.connect
=> #<DuckDB::Connection:0x00007f45da57d178>
irb(main):004:0> con.query('CREATE TABLE users (id INTEGER, name VARCHAR(30))')
=> #<DuckDB::Result:0x00007f45da497470>
irb(main):005:0> con.query("INSERT into users VALUES(1, 'Alice')")
=> #<DuckDB::Result:0x00007f45da3082a8>
irb(main):006:0> con.query("INSERT into users VALUES(2, 'Bob')")
=> #<DuckDB::Result:0x00007f45d3ed8a30>
irb(main):007:0> con.query("INSERT into users VALUES(3, 'Cathy')")
=> #<DuckDB::Result:0x00007f45d3fff328>
irb(main):008:0> result = con.query('SELECT * from users')
=> #<DuckDB::Result:0x00007f45da496818>
irb(main):009:1* result.each do |row|
irb(main):010:1*   p row
irb(main):011:0> end
this `each` behavior will be deprecated in the future. set `DuckDB::Result.use_chunk_each = true` to use new `each` behavior.
[1, "Alice"]          
[2, "Bob"]
[3, "Cathy"]
=> 3
```

Check process while irb requied duckdb being opened

```
$ ps aux | grep irb | grep -v grep
ezquerro  141651  0.7  0.5 656660 44012 pts/1    Sl+  16:19   0:01 irb
$ lsof | { head -1 ; grep duckdb | grep 141755; }
COMMAND      PID    TID TASKCMD              USER   FD      TYPE             DEVICE SIZE/OFF       NODE NAME
irb       141755                         username  mem       REG               0,35               50842 /home/username/.duckdb/build/release/src/libduckdb.so (path dev=0,40)
irb       141755                         username  mem       REG               0,35               61712 /home/username/.rbenv/versions/3.2.2/lib/ruby/gems/3.2.0/gems/duckdb-0.9.1.2/lib/duckdb/duckdb_native.so (path dev=0,40)
irb       141755 141770 Timeout          username  mem       REG               0,35               50842 /home/username/.duckdb/build/release/src/libduckdb.so (path dev=0,40)
irb       141755 141770 Timeout          username  mem       REG               0,35               61712 /home/username/.rbenv/versions/3.2.2/lib/ruby/gems/3.2.0/gems/duckdb-0.9.1.2/lib/duckdb/duckdb_native.so (path dev=0,40)
```

Make sure irb is referencing the installed Duckdb SO file

