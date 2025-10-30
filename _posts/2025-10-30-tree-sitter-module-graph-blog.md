---
title: 'Using tree-sitter queries to generate module graphs'
date: 2025-10-30
permalink: /posts/2025/10/tree-sitter-1
tags:
  - Tree-sitter
  - Language-technology
  - parser-generator
  - graph-data
  - python
---

#  Using tree-sitter queries to generate module graphs

## installation
I'm on a mac so I do:

```sh
brew install tree-sitter-cli
tree-sitter init-config # creates a file under $HOME/.tree-sitter/config.json

```

## a tutorial on building module graphs
Suppose you want to access all the imported libraries or modules in a .py file.

For example this:

```python
# example.py
import collections
from collections import deque
from collections import defaultdict as dd
import numpy as np

def greet(name):
    print(f"Hello, {name}!")

# greet("world")
```

Note I've deliberately added multiple kinds of imports in the file.

Now we want to find the import statements, first we parse the python file into a tree via tree-sitter.

`tree-sitter parse example.py` will now give us the full output of the tree even if there are syntax errors.

It will look something like this:

```sh
(module [0, 0] - [11, 0]
  (comment [0, 0] - [0, 12])
  (import_statement [1, 0] - [1, 18]
    name: (dotted_name [1, 7] - [1, 18]
      (identifier [1, 7] - [1, 18])))
  (import_from_statement [2, 0] - [2, 29]
    module_name: (dotted_name [2, 5] - [2, 16]
      (identifier [2, 5] - [2, 16]))
    name: (dotted_name [2, 24] - [2, 29]
      (identifier [2, 24] - [2, 29])))
  (import_from_statement [3, 0] - [3, 41]
    module_name: (dotted_name [3, 5] - [3, 16]
      (identifier [3, 5] - [3, 16]))
    name: (aliased_import [3, 24] - [3, 41]
      name: (dotted_name [3, 24] - [3, 35]
        (identifier [3, 24] - [3, 35]))
      alias: (identifier [3, 39] - [3, 41])))
  (import_statement [5, 0] - [5, 18]
    name: (aliased_import [5, 7] - [5, 18]
      name: (dotted_name [5, 7] - [5, 12]
        (identifier [5, 7] - [5, 12]))
      alias: (identifier [5, 16] - [5, 18])))
  (function_definition [7, 0] - [8, 28]
    name: (identifier [7, 4] - [7, 9])
    parameters: (parameters [7, 9] - [7, 15]
      (identifier [7, 10] - [7, 14]))
    body: (block [8, 4] - [8, 28]
      (call [8, 4] - [8, 28]
        function: (identifier [8, 4] - [8, 9])
        arguments: (argument_list [8, 9] - [8, 28]
          (string [8, 10] - [8, 27]
            (string_start [8, 10] - [8, 12])
            (string_content [8, 12] - [8, 19])
            (interpolation [8, 19] - [8, 25]
              expression: (identifier [8, 20] - [8, 24]))
            (string_content [8, 25] - [8, 26])
            (string_end [8, 26] - [8, 27]))))))
  (comment [10, 0] - [10, 16]))
```

I say "something like" because the line numbers might be off a bit if you try to reproduce this locally. The 2-tuples are the starting row and column of the text file and the ending row and column of the node. Eg. `module [0, 0] - [11, 0]` means the module has 11 lines starting at the 0th line and ending at the 10th.
In my vim editor the numbering starts at 1 and goes to 11 your mileage may vary.

Ok now let's execute some queries on the node to extract out information that we want.
To do this with the tree-sitter cli we need to use a second file which stores the queries we want to execute.

```sh
# queries.scm
(
  (import_statement) @import.statement
)
```

This will give us the statements and their contents.
Invocation convention is the query file first and then the language file that will be parsed.

```sh
tree-sitter query tsp_queries.scm example.py
```

executing this you'll see something like:

```sh
example.py
  pattern: 0
    capture: 0 - import.statement, start: (1, 0), end: (1, 18), text: `import collections`
  pattern: 0
    capture: 0 - import.statement, start: (5, 0), end: (5, 18), text: `import numpy as np`
```

Ok so we only got two of the imports but in fact we want to get the others too, why weren't they included?
You'll notice that according to the syntax of python those are a separate form of import syntax,
`import_from_statement [2, 0] - [2, 29]`

So for these patterns to be collected we need a second query.
Adding

```sh
(
    (import_from_statement) @import.statement
)
```
to the query file will collect the other form of import. Then executing `tree-sitter query tsp_queries.scm example.py` again we get:

```sh
example.py
  pattern: 0
    capture: 0 - import.statement, start: (1, 0), end: (1, 18), text: `import collections`
  pattern: 1
    capture: 0 - import.statement, start: (2, 0), end: (2, 29), text: `from collections import deque`
  pattern: 1
    capture: 0 - import.statement, start: (3, 0), end: (3, 41), text: `from collections import defaultdict as dd`
  pattern: 0
    capture: 0 - import.statement, start: (5, 0), end: (5, 18), text: `import numpy as np`
```

Ok great but now if we add the import line

```sh
from __future__ import annotations
```

at the top of the example.py file and rerun you'll notice the statement will not be picked up.
Again why not? Here the answer is yet another import statement kind. This kind is called a future import.
You should see something like:

```sh
(module [0, 0] - [12, 0]
  (comment [0, 0] - [0, 12])
  (future_import_statement [1, 0] - [1, 34]
    name: (dotted_name [1, 23] - [1, 34]
      (identifier [1, 23] - [1, 34])))
  (import_statement [2, 0] - [2, 18]
... other output similar to previous tree is elided ...
```
But subsequent nodes in the tree will have different line start and end rows, they'll be offset by 1-2 lines or more depending
on how you put future import statement into the python file.
Ok for the third type of import we'll add yet another query, it looks like this:

```sh
(
    (future_import_statement) @import.statement
)
```

Nothing too crazy and now if you execute again you'll pick up the `__future__` using import statement.

run `tree-sitter query tsp_queries.scm example.py` and see:

```sh
example.py
  pattern: 2
    capture: 0 - import.statement, start: (1, 0), end: (1, 34), text: `from __future__ import annotations`
  pattern: 0
    capture: 0 - import.statement, start: (2, 0), end: (2, 18), text: `import collections`
  pattern: 1
    capture: 0 - import.statement, start: (3, 0), end: (3, 29), text: `from collections import deque`
  pattern: 1
    capture: 0 - import.statement, start: (4, 0), end: (4, 41), text: `from collections import defaultdict as dd`
  pattern: 0
    capture: 0 - import.statement, start: (6, 0), end: (6, 18), text: `import numpy as np`
```

You might be wondering why you would care about this information.

In source code (static) analysis it's common to build graphs of various data in a bunch of code and then using
these graphs you can make certain determinations, e.g. think the "unused import" inside a file, or in an entire
project. How would you determine that the import is unused? Firsy you'd need to confirm that the import is not used
in this file. Then you'd want to check the other files as well.

By building a graph where each node is a file and each node has some properties like imported items as done here,
we can do a BFS or a DFS of the source files of the project (the graph) and determine whether the import is used transitively.
If it is then we might want to push the import to the particular module which uses the import directly.
Doing so makes our code more readable and also more efficient because we eschew unnecessary imports.


Some other applications:

1) Change impact analysis tooling
2) Library splitting/partitioning


For  (1) a change impact analysis is an answer to the question, "if I change this in X where else in library Y will be impacted?"
Applications include reducing the test harness runtime and load by removing tests which would not be reachable by the given change.
Also for maintainenance of code you might want to know which other places in the code would be impacted if you (say) delete an argument
in a function's signature. Many other examples are possible.

For (2) you could build the full graph and notice that there are isolated components or weakly connected components by visually inspecting
the graph after building. Perhaps immediately or perhaps with some small refactor you can simplify the functionality and reduce your package
complexity by using 2 packages rather than 1. It might seem counter intuitive but this is sometimes helpful to reduce loads-especially if
coupled with something like the change impact analysis in (1).

```sh
tree-sitter query tsp_queries.scm example.py | grep capture | awk -F: '{sub("start", "|", $2) sub(", end", "|", $3) sub(", text", "|", $4) $5; print}' | python3 handler.py
```

will puth the stuff into a pipe splittable format that can be sent into use by a downstream application via a pipe delimited file.
For building a change impact tool you'd want some fully fledged programming language like python.

The contents of `handler.py` are simply

```python
#!/usr/bin/env python3
import sys

def process_data():
    """Reads lines from stdin, processes them, and writes to stdout."""
    for line in sys.stdin:
        # Example processing: convert to uppercase and strip whitespace
        processed_line = line.strip().split("|")
        pline = [e.strip() for e in processed_line]
        sys.stdout.write("|".join(pline[:]) + '\n')

if __name__ == "__main__":
    process_data()
```

and the output looks like:

```sh
(1, 0)|(1, 34)|`from __future__ import annotations`
(2, 0)|(2, 18)|`import collections`
(3, 0)|(3, 9)|`import os`
(4, 0)|(4, 29)|`from collections import deque`
(5, 0)|(5, 41)|`from collections import defaultdict as dd`
(6, 0)|(6, 56)|`from collections import (UserDict, UserList, UserString)`
(8, 0)|(8, 18)|`import numpy as np`
```

These can be stored somewhere like a database easily or they can be processed by an application.
Basically your data would be the collection of these rows of values, one collection per file for the project.
The next thing is to build the graph. Once the graph is built then question is which files changed and of those files
that changed what is impacted?

If you have a set of marked files for changes then you can do a reachability analysis across the module graph.
Which modules import the changed modules either directly or transitively.
All those files are the limited set to scan to determine if they are impacted by the changes and how.

The a short python script to read in the pipe delimited file into a database is all you need.

## One last helpful note for debugging

For more on debugging a file's parse tree you can use the playground.

Ok so we've gone through basic setup of a grammar in tree-sitter using a simplified version of markdown.
There are lots of other tips and tricks you can pick up as you start working on tree-sitter grammars.
For example the playground is a helpful tool for debugging `grammr.js` files.

```sh
tree-sitter build --wasm
tree-sitter playground
```

provided you have docker running on your machine you will see something open at `http://127.0.0.1:8000/`.
This is basically a playground instance for you to write an example file and see what the parse of that
looks like using the latest built grammar.

A fuller list of the useful
`npm` commands you'll find [here](https://github.com/Inferara/tree-sitter-inference/blob/639510ac55d561a0f81717d3e27e476d00422485/package.json#L30).

