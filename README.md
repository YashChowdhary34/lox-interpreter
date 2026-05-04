# Lox Interpreter (jlox)

A complete tree-walk interpreter for the **Lox** programming language, implemented in Java by following *[Crafting Interpreters](https://craftinginterpreters.com/)* by Robert Nystrom.

Lox is a dynamically-typed, garbage-collected scripting language with first-class functions, closures, and class-based object orientation. This implementation covers **every chapter of Part II** of the book — from scanning through classes and inheritance.

---

## Table of Contents

- [Features](#features)
- [Language Overview](#language-overview)
- [Project Structure](#project-structure)
- [Architecture](#architecture)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Build](#build)
  - [Run](#run)
- [Usage Examples](#usage-examples)
- [Implementation Chapters](#implementation-chapters)
- [Error Handling](#error-handling)

---

## Features

- **Scanning** — full lexical analysis of all Lox tokens
- **Parsing** — recursive-descent parser with error recovery and synchronisation
- **Expressions** — arithmetic, string concatenation, comparison, logical (`and`/`or`), unary
- **Statements** — `print`, expression statements, `var` declarations, blocks
- **Control flow** — `if`/`else`, `while`, `for` (desugared to `while`)
- **Functions** — first-class functions, closures, recursion, up to 255 arguments
- **Classes** — fields, methods, constructors (`init`), `this`
- **Inheritance** — single inheritance via `<`, `super` access
- **Variable resolution** — static scope analysis pass that computes exact variable depths before execution
- **Runtime error reporting** — line-accurate messages for type errors, undefined variables, division by zero
- **Native functions** — built-in `clock()` for benchmarking
- **REPL** — interactive prompt for quick experimentation

---

## Language Overview

```lox
// Variables and types
var name = "Lox";
var version = 1.0;
var active = true;
var nothing = nil;

// Arithmetic and string concatenation
print "Hello, " + name + "!";   // Hello, Lox!
print 1 + 2 * 3;                // 7

// Control flow
if (version >= 1.0) {
  print "stable";
} else {
  print "beta";
}

// While loop
var i = 0;
while (i < 3) {
  print i;
  i = i + 1;
}

// For loop
for (var j = 0; j < 3; j = j + 1) {
  print j;
}

// Functions and closures
fun makeCounter() {
  var count = 0;
  fun increment() {
    count = count + 1;
    return count;
  }
  return increment;
}

var counter = makeCounter();
print counter();  // 1
print counter();  // 2

// Classes
class Animal {
  init(name) {
    this.name = name;
  }
  speak() {
    print this.name + " makes a sound.";
  }
}

class Dog < Animal {
  speak() {
    print this.name + " barks.";
  }
}

var d = Dog("Rex");
d.speak();  // Rex barks.
```

---

## Project Structure

```
lox-interpreter/
├── src/
│   ├── App.java                          # Entry point
│   └── interpreter/
│       ├── lox/
│       │   ├── Lox.java                  # Main class — wires the pipeline
│       │   ├── Scanner.java              # Lexical analyser
│       │   ├── Token.java                # Token data class
│       │   ├── TokenType.java            # Token type enum (30+ types)
│       │   ├── Expr.java                 # Expression AST nodes (12 types)
│       │   ├── Stmt.java                 # Statement AST nodes (9 types)
│       │   ├── AstPrinter.java           # Debug utility — prints AST as S-expressions
│       │   ├── Parser.java               # Recursive-descent parser
│       │   ├── Resolver.java             # Static variable resolution pass
│       │   ├── Interpreter.java          # Tree-walk interpreter
│       │   ├── Environment.java          # Lexical scope chain
│       │   ├── RuntimeError.java         # Runtime error with token context
│       │   ├── Return.java               # Control-flow exception for return
│       │   ├── LoxCallable.java          # Interface for callable values
│       │   ├── LoxFunction.java          # User-defined function with closure
│       │   ├── LoxClass.java             # Class object (also callable as constructor)
│       │   └── LoxInstance.java          # Runtime instance of a class
│       └── tool/
│           └── GenerateAst.java          # Code-gen tool that produced Expr/Stmt skeletons
├── bin/                                  # Compiled .class files (generated)
└── README.md
```

---

## Architecture

The interpreter follows a classic **pipeline** of four stages:

```
Source code
    │
    ▼
┌──────────┐
│  Scanner │  Converts raw characters → Token list
└──────────┘
    │
    ▼
┌──────────┐
│  Parser  │  Converts Token list → AST (Expr / Stmt tree)
└──────────┘
    │
    ▼
┌──────────────┐
│   Resolver   │  Walks AST, resolves variable scope depths
└──────────────┘
    │
    ▼
┌─────────────┐
│ Interpreter │  Walks AST, evaluates nodes, executes side effects
└─────────────┘
```

### Design Patterns

| Pattern | Where Used |
|---|---|
| **Visitor** | `Expr.Visitor` / `Stmt.Visitor` — decouples operations (interpret, resolve, print) from node types |
| **Chain of Responsibility** | `Environment` scope chain — variable lookup walks parent scopes |
| **Exception as control flow** | `Return` exception — unwinds the call stack on `return` statements |
| **Closure** | `LoxFunction` captures its declaring `Environment` at creation time |

---

## Getting Started

### Prerequisites

- Java Development Kit (JDK) **11 or higher**

```bash
java -version
javac -version
```

### Build

Compile all sources from the project root:

```bash
# macOS / Linux
mkdir -p bin
find src -name "*.java" | xargs javac -d bin

# Windows (PowerShell)
mkdir bin
Get-ChildItem -Recurse -Filter "*.java" src | ForEach-Object { $_.FullName } | xargs javac -d bin
```

### Run

**REPL (interactive prompt):**

```bash
java -cp bin interpreter.lox.Lox
```

**Run a Lox source file:**

```bash
java -cp bin interpreter.lox.Lox path/to/script.lox
```

**Exit codes:**

| Code | Meaning |
|---|---|
| `0` | Success |
| `64` | Usage error (wrong number of arguments) |
| `65` | Syntax / compile-time error |
| `70` | Runtime error |

---

## Usage Examples

### Fibonacci

```lox
fun fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}

for (var i = 0; i < 10; i = i + 1) {
  print fib(i);
}
```

### Closures

```lox
fun makeAdder(x) {
  fun adder(y) { return x + y; }
  return adder;
}

var add5 = makeAdder(5);
print add5(3);   // 8
print add5(10);  // 15
```

### Classes with Inheritance

```lox
class Shape {
  init(colour) {
    this.colour = colour;
  }
  describe() {
    print "A " + this.colour + " shape.";
  }
}

class Circle < Shape {
  init(colour, radius) {
    super.init(colour);
    this.radius = radius;
  }
  area() {
    return 3.14159 * this.radius * this.radius;
  }
}

var c = Circle("red", 5);
c.describe();       // A red shape.
print c.area();     // 78.53975
```

### Benchmarking with `clock()`

```lox
var start = clock();
// ... code to benchmark ...
var elapsed = clock() - start;
print "Elapsed: " + elapsed + "s";
```

---

## Implementation Chapters

Each file corresponds to one or more chapters in *Crafting Interpreters*:

| Chapter | Topic | Files |
|---|---|---|
| 4 | Scanning | `Scanner.java`, `Token.java`, `TokenType.java` |
| 5 | Representing Code | `Expr.java`, `AstPrinter.java`, `GenerateAst.java` |
| 6 | Parsing Expressions | `Parser.java` |
| 7 | Evaluating Expressions | `Interpreter.java`, `RuntimeError.java` |
| 8 | Statements & State | `Stmt.java`, `Environment.java`, `Lox.java` |
| 9 | Control Flow | `Parser.java` (if/while/for), `Interpreter.java` |
| 10 | Functions | `LoxCallable.java`, `LoxFunction.java`, `Return.java` |
| 11 | Resolving & Binding | `Resolver.java` |
| 12 | Classes | `LoxClass.java`, `LoxInstance.java` |
| 13 | Inheritance | `LoxClass.java` (super), `Resolver.java` (super/this) |

---

## Error Handling

The interpreter distinguishes three error categories:

**Scan/Parse errors** — reported with line numbers, parsing continues to find further errors:
```
[line 3] Error at 'foo': Expect ';' after expression.
```

**Resolution errors** — static semantic errors caught before execution:
```
[line 7] Error at 'return': Can't return from top-level code.
[line 12] Error at 'x': Can't read local variable in its own initializer.
```

**Runtime errors** — execution halts and reports the offending line:
```
Operands must be numbers.
[line 5]
```

---

## Reference

- Book: [*Crafting Interpreters* — Robert Nystrom](https://craftinginterpreters.com/)
- The complete jlox reference implementation is available in the book's [GitHub repository](https://github.com/munificent/craftinginterpreters)
