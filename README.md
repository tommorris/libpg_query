# libpg_query [![Build Status](https://travis-ci.org/lfittl/libpg_query.svg?branch=master)](https://travis-ci.org/lfittl/libpg_query)

C library for accessing the PostgreSQL parser outside of the server.

This library uses the actual PostgreSQL server source to parse SQL queries and return the internal PostgreSQL parse tree.

Note that this is mostly intended as a base library for https://github.com/lfittl/pg_query (Ruby), https://github.com/lfittl/pg_query.go (Go), https://github.com/zhm/pg-query-parser (Node), https://github.com/alculquicondor/psqlparse (Python) and https://pypi.org/project/pg_query/ (Python 3.6).

You can find further background to why a query's parse tree is useful here: https://pganalyze.com/blog/parse-postgresql-queries-in-ruby.html


## Installation

```sh
git clone -b 10-latest git://github.com/lfittl/libpg_query
cd libpg_query
make
```

Due to compiling parts of PostgreSQL, running `make` will take a bit. Expect up to 3 minutes.

For a production build, its best to use a specific git tag (see CHANGELOG).


## Usage: Parsing a query

A [full example](https://github.com/lfittl/libpg_query/blob/master/examples/simple.c) that parses a query looks like this:

```c
#include <pg_query.h>
#include <stdio.h>

int main() {
  PgQueryParseResult result;

  result = pg_query_parse("SELECT 1");

  printf("%s\n", result.parse_tree);

  pg_query_free_parse_result(result);
}
```

Compile it like this:

```
cc -Ilibpg_query -Llibpg_query example.c -lpg_query
```

This will output:

```json
[{"SelectStmt": {"targetList": [{"ResTarget": {"val": {"A_Const": {"val": {"Integer": {"ival": 1}}, "location": 7}}, "location": 7}}], "op": 0}}]
```


## Usage: Fingerprinting a query

Fingerprinting allows you to identify similar queries that are different only because
of the specific object that is being queried for (i.e. different object ids in the WHERE clause),
or because of formatting.

Example:

```c
#include <pg_query.h>
#include <stdio.h>

int main() {
  PgQueryFingerprintResult result;

  result = pg_query_fingerprint("SELECT 1");

  printf("%s\n", result.hexdigest);

  pg_query_free_fingerprint_result(result);
}
```

This will output:

```
8e1acac181c6d28f4a923392cf1c4eda49ee4cd2
```

See https://github.com/lfittl/libpg_query/wiki/Fingerprinting for the full fingerprinting rules.

## Usage: Parsing a PL/pgSQL function (Experimental)

A [full example](https://github.com/lfittl/libpg_query/blob/master/examples/simple_plpgsql.c) that parses a [PL/pgSQL](https://www.postgresql.org/docs/current/static/plpgsql.html) method looks like this:

```c
#include <pg_query.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
  PgQueryPlpgsqlParseResult result;

  result = pg_query_parse_plpgsql(" \
  CREATE OR REPLACE FUNCTION cs_fmt_browser_version(v_name varchar, \
                                                  v_version varchar) \
RETURNS varchar AS $$ \
BEGIN \
    IF v_version IS NULL THEN \
        RETURN v_name; \
    END IF; \
    RETURN v_name || '/' || v_version; \
END; \
$$ LANGUAGE plpgsql;");

  if (result.error) {
    printf("error: %s at %d\n", result.error->message, result.error->cursorpos);
  } else {
    printf("%s\n", result.plpgsql_funcs);
  }

  pg_query_free_plpgsql_parse_result(result);

  return 0;
}
```

This will output:

```json
[
{"PLpgSQL_function": {"datums": [{"PLpgSQL_var": {"refname": "found", "datatype": {"PLpgSQL_type": {"typname": "UNKNOWN"}}}}], "action": {"PLpgSQL_stmt_block": {"lineno": 1, "body": [{"PLpgSQL_stmt_if": {"lineno": 1, "cond": {"PLpgSQL_expr": {"query": "SELECT v_version IS NULL"}}, "then_body": [{"PLpgSQL_stmt_return": {"lineno": 1, "expr": {"PLpgSQL_expr": {"query": "SELECT v_name"}}}}]}}, {"PLpgSQL_stmt_return": {"lineno": 1, "expr": {"PLpgSQL_expr": {"query": "SELECT v_name || '/' || v_version"}}}}]}}}}
]
```

## Versions

For stability, it is recommended you use individual tagged git versions, see CHANGELOG.

`master` reflects a PostgreSQL base version of 9.4, with a legacy output format.

New development is happening on `10-latest`, which reflects a base version of Postgres 10.


## Resources

pg_query wrappers in other languages:

* Ruby: [pg_query](https://github.com/lfittl/pg_query)
* Go: [pg_query_go](https://github.com/lfittl/pg_query_go)
* Javascript (Node): [pg-query-parser](https://github.com/zhm/pg-query-parser)
* Javascript (Browser): [pg-query-emscripten](https://github.com/lfittl/pg-query-emscripten)
* Python: [psqlparse](https://github.com/alculquicondor/psqlparse), [pglast](https://github.com/lelit/pglast)

Products, tools and libraries built on pg_query:

* [pganalyze](https://pganalyze.com/)
* [hsql](https://github.com/JackDanger/hsql)
* [sqlint](https://github.com/purcell/sqlint)
* [pghero](https://github.com/ankane/pghero)
* [dexter](https://github.com/ankane/dexter)
* [pgscope](https://github.com/gjalves/pgscope)
* [pg_materialize](https://github.com/aanari/pg-materialize)
* [DuckDB](https://github.com/cwida/duckdb) ([details](https://github.com/cwida/duckdb/tree/master/third_party/libpg_query))


Please feel free to [open a PR](https://github.com/lfittl/libpg_query/pull/new/master) to add yours! :)


## Authors

- [Lukas Fittl](mailto:lukas@fittl.com)


## License

PostgreSQL server source code, used under the [PostgreSQL license](https://www.postgresql.org/about/licence/).<br>
Portions Copyright (c) 1996-2017, The PostgreSQL Global Development Group<br>
Portions Copyright (c) 1994, The Regents of the University of California

All other parts are licensed under the 3-clause BSD license, see LICENSE file for details.<br>
Copyright (c) 2017, Lukas Fittl <lukas@fittl.com>
