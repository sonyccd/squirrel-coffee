[![Codacy Badge](https://api.codacy.com/project/badge/Grade/d4d4ee1f737c4c338d27906ac2432873)](https://www.codacy.com/app/snakes-in-the-box/squirrel-coffee?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=sonyccd/squirrel-coffee&amp;utm_campaign=Badge_Grade)

# Squirrel Coffee

A compiler for a simplified Java-like programming language that compiles to JVM bytecode.

## Overview

Squirrel Coffee is an educational compiler project that demonstrates classic compiler construction techniques. It takes source code written in the Squirrel Coffee language (a subset of Java) and compiles it to JVM bytecode that can run on any Java Virtual Machine.

## Compiler Architecture

The Squirrel Coffee compiler follows a traditional multi-pass architecture:

### 1. **Lexical Analysis** (Frontend)
   - **Tool**: Flex lexer generator
   - **File**: `tjc.l`
   - Tokenizes source code into meaningful symbols (keywords, identifiers, operators, literals)

### 2. **Syntax Analysis** (Parser)
   - **Tool**: Bison parser generator  
   - **File**: `tjc.y`
   - Builds an Abstract Syntax Tree (AST) from tokens
   - Enforces language grammar rules

### 3. **Semantic Analysis** (Middle-end)
   - **Symbol Table Construction** (`SymbolTable.cpp`)
     - First AST traversal
     - Builds hierarchical symbol table
     - Tracks variable/method declarations and scopes
   
   - **Type Checking** (`SemanticCheck.cpp`)
     - Second AST traversal
     - Validates type compatibility
     - Enforces semantic rules
     - Performs automatic type conversions

### 4. **Code Generation** (Backend)
   - **File**: `CodeGenerator.cpp`
   - Third AST traversal using visitor pattern
   - Generates Jasmin assembly code (.jsm files)
   - Handles JVM stack management and instruction selection

### 5. **Assembly**
   - **Tool**: Jasmin assembler (v2.4)
   - Converts Jasmin assembly to JVM bytecode (.class files)
   - Final executable output

## Language Features

### Data Types
- **Primitives**: `int`, `float`
- **Reference**: `String` (limited support)
- **Automatic Conversions**: int → float when needed
- **Explicit Casting**: `(int)` and `(float)` casts supported

### Operators

#### Arithmetic
- Binary: `+`, `-`, `*`, `/`
- Unary: `+`, `-`
- Precedence follows standard mathematical rules

#### Relational
- Comparison: `<`, `<=`, `>`, `>=`
- Equality: `==`, `!=`
- Returns boolean values for control flow

### Statements
- **Assignment**: `variable = expression;`
- **Control Flow**: `if`, `while`
- **Method Calls**: Static method invocations
- **Return**: Required at end of every method
- **Blocks**: `{ }` for statement grouping

### Program Structure
- Single public class per file
- All methods must be static
- All fields must be static with initializers
- Entry point: `public static void main(String[] args)`
- No packages or imports allowed

### Built-in I/O
The `SimpleIO` class provides console I/O:
- **Input**: `readInt()`, `readFloat()`, `readString()`
- **Output**: `printInt()`, `printFloat()`, `printString()`, `println()`

## Building the Compiler

### Prerequisites
- C++11 compatible compiler (g++)
- Flex (lexical analyzer generator)
- Bison (parser generator)
- Java Runtime Environment (for Jasmin)
- CMake (3.10 or higher) or Make

### Build Commands

#### Using CMake (Recommended - Cross-platform)
```bash
# Create build directory
mkdir build
cd build

# Configure and build
cmake ..
make

# Or for parallel build
make -j4
```

#### Using Make (Legacy)
```bash
# Build the compiler
make all

# Clean build artifacts
make clean
```

## Using the Compiler

### Compile a Program
```bash
./tjc MyProgram.java
```

This generates:
- `MyProgram.jsm` - Jasmin assembly code
- `MyProgram.class` - JVM bytecode

### Run Compiled Program
```bash
java MyProgram
```

## Example Program

```java
public class HelloWorld {
    static int count = 0;
    
    public static void main(String[] args) {
        count = 5;
        while (count > 0) {
            SimpleIO.printString("Hello ");
            SimpleIO.printInt(count);
            SimpleIO.println();
            count = count - 1;
        }
    }
}
```

## Project Structure

```
squirrel-coffee/
├── tjc.l              # Lexer specification
├── tjc.y              # Parser grammar
├── ast/               # AST node definitions
│   ├── Declaration/   # Class, method, variable declarations
│   ├── Expression/    # Expression nodes
│   └── Statement/     # Statement nodes
├── SymbolTable.cpp    # Symbol table construction
├── SemanticCheck.cpp  # Type checking and validation
├── CodeGenerator.cpp  # Jasmin code generation
├── jasmin-2.4/        # Bundled Jasmin assembler
└── Sample*.java       # Example programs
```

## Implementation Details

### Visitor Pattern
The compiler uses the visitor pattern for AST traversal, allowing clean separation of concerns:
- Each compilation phase is a separate visitor
- Visitors traverse the same AST structure
- Easy to add new analysis passes

### Type System
- Strong static typing
- Automatic widening conversions (int → float)
- Explicit narrowing casts required (float → int)
- Type checking at compile time

### Error Handling
- Lexical errors detected during tokenization
- Syntax errors caught by parser
- Semantic errors (type mismatches, undefined variables) reported with line numbers
- Runtime errors handled by JVM

## Limitations

- No object-oriented features (inheritance, instances)
- No arrays (except String[] in main)
- No method overloading
- No access modifiers (except public for main class)
- No exception handling
- Limited string operations

## Testing

Test programs are provided in `Sample1.java` and `Sample2.java`. To test:

```bash
./tjc Sample1.java
java Sample1
```

## Academic Note

This project was developed as part of a compiler construction course. The language design and project structure credit goes to the course instructor.