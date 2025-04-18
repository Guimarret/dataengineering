---
title: "Compilers"
date: 2025-03-21T16:14:08-03:00
draft: true
---

So, after a long time, I'm back here to show what i studied these days.
This time i read the [crafting intepreters](https://craftinginterpreters.com/) and [Tiger book (modern compiler implementation)](http://www.infouem.com.br/wp-content/uploads/2011/03/Modern-Compiler-Implementation-in-C.pdf) to get a better understanding of how compilers and languages work.

## Introduction
So far so good, started with the crafting interpreters which was really noob friendly (as clearly intended), i liked it that way because i knew close to nothing about the theme and read one chapter a day and after finishing a did the same with the Tiger book which was immensely harder and much more "academic" but even then it was delightful, there is still a LOT that i don't know but i have a slight idea of how to search about at least haha.

Also, my goal with these posts is to synthesize the points and get an easier way to remember in later days.

The first part of the crafting interpreters is good for visualization as Nystrom says, the focus is on the concepts. The creation of the tree-walk interpreter which evaluates to the AST is funny if i can say, because it shows that there is nothing incredible going on, it's just what it is.

# Thoughs

## Lexical analysis
Jumping into the chapters we get the first contact with lexical analysis, which converts the raw code into "something"... There are some parts for this stage, and starting with the parser the one i liked the most is the implementation in the *Tiger* (gonna refer this way to be less repetitive), this one uses a parser generator (really like bison btw) Yacc, that just need a config file which makes everything direct to the point and simple, it converts the source code into [deterministic finite automata(DFA)](https://en.wikipedia.org/wiki/Deterministic_finite_automaton), also it's really important to read the theoretical part of this one because it shows step by step the idea of deterministic *finite* executions.
- Important to say that in *CI* (crafting interpreters) this part is built by "hand" with recursive descent and the difference to the second part is the simplicity, the second integrates into a single-pass compiler (good [source](https://www.geeksforgeeks.org/single-pass-two-pass-and-multi-pass-compilers/) if you don't know what it is) and produces the tokens lazily (Like... on demand, we could say).

## Parser

In *Tiger* the parser is made LR (It's important to say here that the book goes deeper into this topic, it shows detailed information about LL, LR, SLR, LALR, GLR, and the implications of each one, in the end there is this representation)

![](/img/compilers/formal_grammar_correct.svg)

After the LR parsing, the ASTs separate the syntactic structure which also enables multiple passes for optimization as shown in the image with subexpression elimination:

![](/img/compilers/LR_parsing_optmization.svg)


The *CI* part.1 does the parsing using recursive descent and directly integrates to the AST (directly from the functions), the main point of this implementation is the error recovery via sync token consumption (That would be like a panic mode in which the program begin to discard tokens until reaching the *statement terminator*, the advantage is avoiding cascaded errors and continue parsing to reach more errors).

Part. 2 is a bit complex, it compiles directly to bytecode as i said before, in a single pass and the parser emite bytecode instructions while parsing, the advantage is that enables immediate optimization like *[constant folding](https://en.wikipedia.org/wiki/Constant_folding#:~:text=Constant%20folding%20is%20the%20process,are%20known%20at%20compile%20time.)*, and *constant propagation*.

## Error Handling

The *Tiger* error handling in parsers is the worst because they are runtime stack traces, but at the end of the day is all about trade-offs in efficiency at this point, etc, the error recovery is bytecode validation which i'm not gonna talk much because it links to control flow analysis that's gonna be talked later.

The *CI part.1* has some nice error handling with some context like "Unexpected token" which makes it easier to debug and fix errors also the debug support is line-number tracking which i particularly appreciate (it can be implemented in other compilers' points and other ways).

The *CI part.2* are generic syntax errors and the debugger uses symbol tables (gonna talk about later).

Not gonna extend much here because it's kind of boring to be honest, if you are actually looking forward to this part i recommend reading the chapters which are fast and are better for getting the details of the implementations but this point is gonna be back in other points, because in some way error handling/recovery will be present in all points of the compilers.

## Semantical analysis

So here it starts to get trickier, and the topic scope starts to get heavier. In semantical analysis, there are some important points like type system, scope resolution, error reporting (relating to the last topic), etc.
> Type system, this point is a bit too big to get handled in just some little observation in the sub-topic but i will try to make it clarifying.

### Scope resolution

This topic is really important and interesting because the scope of some functions can get shade, the way we treat and store information about the scope may change the extension of possibilities, with this i mean the scope is complex.

<details> <summary>Scope Examples in C (Click here)</summary> <!-- Static vs. Dynamic Scoping --> <details> <summary>Static vs. Dynamic Scoping</summary>

  ```C
  int x = 10;

  int f() {
      return x;
  }

  int g() {
      int x = 20;
      return f();
  }

  int main() {
      printf("%d\n", g());
      return 0;
  }
  ```

  </details> <!-- Variable Shadowing --> <details> <summary>Variable Shadowing</summary>

  ```C
  void complex_scoping() {
      int a = 5;
      {
          int b = a + 1;  // b = 6
          printf("Outer block: a=%d, b=%d\n", a, b);
          {
              int a = b + 1;  // New a = 7, shadows outer a
              printf("Inner block: a=%d, b=%d\n", a, b);
          }
          printf("Back to outer: a=%d, b=%d\n", a, b);
      }
  }
  ```

  </details> <!-- Function Scope --> <details> <summary>Inside Function Scope</summary>

  ```C
  int calculate(int x, int y) {
      int result = 0;
      {
          int temp = x * 2;
          if (temp > y) {
              int y = temp;  // Shadows parameter y
              result = x + y;
          } else {
              result = x - y;  // Uses parameter y
          }
      }
      return result;
  }
  ```

  </details> <!-- Variable Initialization --> <details> <summary>Variable Initialization</summary>

  ```C
  int tricky_init() {
      int x = 10;
      {
          int y = x + 5;  // y = 15
          int x = 20;     // New x shadows outer x
          int z = x + 5;  // z = 25 (uses inner x)
          return y + z;   // Returns 40
      }
  }
  ```

  </details> <!-- Recursive Functions --> <details> <summary>Recursive Functions</summary>

  ```C
  int factorial(int n) {
      if (n <= 1) return 1;
      return n * factorial(n - 1);
  }
  ```
</details> </details>

So considering the examples let's dig into the important points. If we think that depending on the situation the scope changes, we have some options:
- Limit the programmer options with some declarations, like:
  - *let*
  - *var*

That will create different things, for example, the let in javascript will create a block declaration, so it preserves the variable just in this block state and is not imported for insider functions or outside the main function, while the var is usually accessible for all scope depths of the function that is deeper than the original function, and in case of declaration outside functions it becomes global to access technically anywhere in the code, example:

```javascript
function varScoping() {
  var x = 1;
  if (true) {
    var x = 2;
    console.log(x); // will print 2
  }
  console.log(x); // will print 2
}

function letScoping() {
  let x = 1;
  if (true) {
    let x = 2;
    console.log(x); // will print 2
  }
  console.log(x); // will print 1
}
```

There are also other differences between the let and var but i will talk more in the type checker part.
