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

You might be wondering why you would care about this information so here is some motivation.

In source code (static) analysis it's common to build graphs of various data produced by parsing code.
The data from the code parsing is used to construct a graph and from the graph you can make certain analyses.
One example you could imagine is the "unused import" notice inside a file in your IDE, or across an entire
project. How would you determine that the import is unused? First you'd need to confirm that the import is not used
in this file. If you also care if it's used in the project but outside of the file itself you'd check other files too.

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

Now let's move on and see how we can get this data from a collection of source code.

## installation
I'm on a mac so I do:

```sh
brew install tree-sitter-cli
tree-sitter init-config # creates a file under $HOME/.tree-sitter/config.json
```

Other package managers (linux/windows) should support treee sitter too.

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

I say "something like" because the line numbers might be off a bit if you try to reproduce this locally.
The 2-tuples are the starting row and column of the text file and the ending row and column of the node.
For example,  `module [0, 0] - [11, 0]` means the module has 11 lines starting at the 0th line and ending
at the 10th. In my vim editor the numbering starts at 1 and goes to 11-your mileage may vary.

One important thing to study in the parse tree above is that each node has a particular type.
For example there are `function_definition` nodes which are the root of a tree that contains a python
function. The subtree of a `function_definition` node contains things related to that function, like
the body of the function, the parameters, the function name, etc. Similarly the import statements
are nodes in the tree as well. The indentation in the listing is for visual appeal and readability.

Ok now let's execute some queries on the tree to extract out information that we want.
To do this with the tree-sitter cli we need to use a second file which stores the queries we want to execute.

The queries are in scheme so we write a `queries.scm` file for what we want to query.

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
Again, why not? Here the answer is yet another import statement kind. This kind is called a future import.
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
on how you put the future import statement into the python file.
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

Now this gives us what we want which is for each python file we want to be able to determine which other packages or files
are imported. The module graph is then constructed as a graph of the python files (the nodes in the graph) and the import
statements represent edges to other files (modules is the python nomenclature for a `.py` file). When you build a module
graph you can add further logic to restrict the nodes and edges to only the files in the package or repository you are studying
via walking the root of the current package directory and building a set out of those files and some additional logic to filter
known packages, both builtins like `collections` and 3rd party ones like `numpy`. There would be some work to get that filtering logic
to a reasonable state but this is the general approach.

To generate a short list of the imports on any given python file you can then run

```sh
tree-sitter query tsp_queries.scm example.py | grep capture | awk -F: '{sub("start", "|", $2) sub(", end", "|", $3) sub(", text", "|", $4) $5; print}' | python3 handler.py
```
replacing the `example.py` with the python file you want to parse. The `handler.py` is a simple cli tool I wrote to make this
bit of parsing simpler.

Executing the line above will put the stuff into a pipe splittable format that can be sent into
use by a downstream application via a pipe delimited file.

For building a change impact tool you'd want some fully fledged programming language like python
to implement further processing and to connect it to a database.

The contents of the `handler.py` file are

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

These can be stored somewhere like a database or they can be processed by an application.
Basically your data would be the collection of these rows of values, one collection per file for the project.
The next thing is to build the graph. Once the graph is built then the question is which files changed and
of those files that changed which are potentially impacted? Using this information you can reduce the number
of files that need testing and thereby save costs, reduce noise from flaky tests, and accelerate testing.

If you have a set of marked files for changes then you can do a reachability analysis across the module graph.
Which modules import the changed modules either directly or transitively.
Those files-perhaps a subset of them-are the set to scan to determine if they are impacted by the changes and how.

Then a short python script to read in the pipe delimited file into a database is all you need.
You'd parse each file in your project similar to the way we've done here with a single file and you'd
have another file to connect the various files via their import relations.

For something like a change impact analysis, the approach here allows you to eliminate whole files but not individual
classes or functions within the files. With more queries you could get much more granular than a file level but the
general approach would be the same, parse the files into trees, run the queries to get the data you need then
match these entries to those in the existing tests.

I'll also note that for some programming language test harnesses
this logic is already built in but for most it is not. Before embarking on implementing something like this in a system
with consequences, it is worthwhile to see if the programming language and available testing harnesses will support a
change impact analysis. Also check your devops tooling, it is also common for change impact analysis to be a supported
feature there too. If you're already using a tool which supports this then use the tool rather than reinventing the
wheel. Unless your goal is to understand how a change impact tool works. Our goal here was explanation at a conceptual
level which is why we did not build out a full demo. Also with this-and subsequent-tree sitter blog posts my personal
goal is to demonstrate the tool and its' capabilities. Parser generators and language technologies in general are very
handy tools to have and to understand, even if you're not building them yourself, it is still worthwhile to understand
the basics of the technology.

## One last helpful note for debugging

For more on debugging a file's parse tree you can use the playground.

Ok so we've gone through basic setup of a parser of an existing grammar in tree-sitter.
There are lots of other tips and tricks you can pick up as you start working on tree-sitter grammars.
For example the playground is a helpful tool for debugging why a file did not extract what you wanted.
Playground is also handy for building out custom grammars and debugging `grammar.js` files.

```sh
tree-sitter build --wasm
tree-sitter playground
```

provided you have docker running on your machine you will see something open at `http://127.0.0.1:8000/`.
This is basically a local playground instance for you to write an example file and see what the parse tree
of that file looks like.

This is especially helpful when your building out custom grammars and parsers for a language. It can also
be helpful if you have some malformatted source code and you're trying to understand why the parse tree
that is being generated but is not generating the parse tree what you want or expect.

A fuller list of the useful
`npm` commands you'll find [here](https://github.com/Inferara/tree-sitter-inference/blob/639510ac55d561a0f81717d3e27e476d00422485/package.json#L30).

## Future posts

A future post I'm working on will take a look at writing a custom grammar and parser generator using
tree-sitter. In that post knowing the basics from this post are both useful from a capability perspective
as well as from a developing perspective.
