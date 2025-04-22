---
title: "Compilers"
date: 2025-03-21T16:14:08-03:00
draft: false
---

So, after a long time, I'm back to show you what I've been studying these past few days. This time, I read [Crafting Interpreters](https://craftinginterpreters.com/) and the [Tiger book (Modern Compiler Implementation)](http://www.infouem.com.br/wp-content/uploads/2011/03/Modern-Compiler-Implementation-in-C.pdf) to get a better understanding of how compilers and programming languages work.

## Introduction
So far so good, I started with reading one chapter a day of the Crafting Interpreters, which was really noob friendly (as clearly intended), I thought that was good, since I knew close to nothing about the topic. After finishing, I did the same with the Tiger book, which was delightful, even though it was immensely harder and much more theoretical. There is still a LOT that I need to learn about, but at least I have a slight idea of how to research it haha.

Also, my goal with these posts is to synthesize the points I have learned and make it easier for me to remember them later, as the days go by.

The first part of the Crafting Interpreters is good for visualization because, as Nystrom says, the focus is on the concepts. The creation of the tree-walk interpreter, which evaluates to the AST, is funny (if I can put it that way), because it shows that there is nothing incredible going on, it’s just what it is.

# Thoughts

## Lexical analysis

Jumping into the chapters, we get the first contact with lexical analysis, which converts raw code into “something” more structured. This section has a few parts, and once we get to the parser, the one I liked the most was the implementation in *Tiger* (I’ll refer to it this way to avoid being repetitive), in which they use a parser generator (that is really similar to bison btw) Yacc that just needs a config file, which makes everything super direct and simple. It converts the source code into a [Deterministic Finite Automata (DFA)](https://en.wikipedia.org/wiki/Deterministic_finite_automaton).

I also want to point out that reading the theoretical part of this section is really important, because it walks you through the concept of deterministic *finite* executions step by step.
- It's important to say that in *CI* (crafting interpreters) this part is built “by hand” using recursive descent. The difference to the second part is the simplicity, the second integrates into a single-pass compiler (good source if you don’t know what it is) and produces the tokens lazily (Like, on demand, we could say).

## Parser

In *Tiger* the parser is made using LR (It’s important to say here that the book goes deeper into this topic, providing detailed information about LL, LR, SLR, LALR, GLR, and the implications of each one) in the end there is this representation:

![](/img/compilers/formal_grammar_correct.svg)

After the LR parsing, the ASTs is separated from the syntactic structure, which also enables multiple passes for optimization as shown in the image with subexpression elimination:

![](/img/compilers/LR_parsing_optmization.svg)


The *CI* part.1 does the parsing using recursive descent and directly integrates to the AST (directly from the functions), the main point of this implementation is the error recovery via sync token consumption (That works sort of like a “panic mode”, in which the program begins to discard tokens until the *statement terminator. The advantage it provides is that it avoids cascaded errors and continues parsing to help identify more errors).

Part. 2 is a little more complex, it compiles directly to bytecode as i said before, in a single pass and the parser emite bytecode instructions while parsing, the advantage is that enables immediate optimization like *[constant folding](https://en.wikipedia.org/wiki/Constant_folding#:~:text=Constant%20folding%20is%20the%20process,are%20known%20at%20compile%20time.)*, and *constant propagation*.

## Error Handling

The *Tiger* error handling in parsers is the worst because they are runtime stack traces, but at the end of the day is all about trade-offs in efficiency at this point, etc, the error recovery is bytecode validation which I won't talk much about because it links to control flow analysis which I will cover in a later segment.

The *CI part.1* has some nice error handling with some context like "Unexpected token" which makes it easier to debug and fix errors also the debug support is line-number tracking which i particularly appreciate (since it can be implemented in other compilers' points and other ways).

The *CI part.2* are generic syntax errors and the debugger uses symbol tables (gonna talk about later).

I won’t extend the subject too much here because it's kind of boring to be honest, if you are actually looking forward to this part i recommend reading the chapters. They are short and will provide better implementations but this point will be talked about be back in other points, because in some way error handling/recovery will be present in all points of the compilers.

## Semantical analysis

So by this point it starts to get trickier, and the topic scope starts to get more complex. In semantical analysis, there are some important points like scope resolution, type system, error reporting (relating to the last topic), etc.
> The Type system point is a bit too complex to cover briefly, but I’ll try to break it down clearly.

### Scope resolution

This topic is both really important and interesting, because the scope of some functions can get shaded. The way we treat and store information about the scope may change the extension of possibilities, and with this I mean the scope is complex.

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

So considering the examples let’s dig into the important points. If we assume that depending on the situation the scope changes, we have some options:
- Limit the programmer options with some declarations, like:
  - *let*
  - *var*

That will create different scenarios. For example, the let in javascript will create a block declaration, so it preserves the variable just in this block state and is not imported for insider functions or outside the main function, while the var is usually accessible for all scope depths of the function that is deeper than the original function, and in case of declaration outside functions it becomes a global variable and can technically be accessed from anywhere in the code. For example:

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

There are also other differences between let and var, but I’ll talk more about them later on.

Also, we can use the const keyword to declare constants that cannot be reassigned. This can help prevent bugs and make the code more predictable. If you are interested I would recommend that you look for separate articles in mutability and sharing (because won't be talking enough about them), but for context: immutable variables can not be reassigned but could be pointed or copied depending on the type and so on.

There are some strategies to handle scope and variable declaration in functions for different programming languages. And there are some functional programming languages that use lazy evaluation (I’ll explain more about this in the Optimization section), Haskell is an example, it creates a thunk to catch the environment and then, later on in the scope resolution, it determines the variable based on this thunk. Of course, this is a massive simplification of the actual process but it’s important for you to know that it exists.

In sum, there are multiple techniques and ways to solve different problems, and for the sake of the blog post I’m gonna move on to the comparison between the implementations in the books:

The *Tiger* book takes the approach of creating hierarchical symbol tables per scope for data structures, assigning type, scope and memory location for variable storage and it doesn't support closures.

*CI part.1* Uses a stack of hash maps for data structure, annotations for scope depth on the AST nodes and it catches the variables in heap environments for the closure.

*CI part.2* Uses a stack of local slots and upvalue tracking for data structure. For variable storage it uses slot indices encoded in bytecode and upvalues linked to heap or outer stack frames for closures.

### Type System

The type system is a rule based mechanism that determines the type of something based on the types of its components and the rules of its language. It can be static or dynamic and can be used to catch errors at compile time or runtime. It might look like a simple mechanism but it is complex and powerful because it can prevent errors and optimize code (If you are interested in this topic, I would recommend searching for “type theory and type soundness” material, that will give you a whole new world to explore). There is also type inference that consists of automatically deducing the type that should be in some place if there is no declaration (I will also not be elaborating on this topic).

*Tiger* creates a type system for statically typed languages. It Formalizes static rules that are enforced during AST transversal, flagging mismatches before the code generation. Also, the type metadata is embedded in IR (intermediate representation), enabling optimization such as register allocation and dead code elimination. In the second part, the book also shows the Hindley–Milner type system inference and parametric polymorphism (I didn’t go deeper into these points because I found them too difficult for the time being, but in the future, I intend to write another blog post that will focus in talking about type system as a separate topic).

*CI part.1* performs runtime type checks that are really simple. The values are Java objects from a base Value class and it uses a visitor pattern for dynamic dispatch, to evaluate expression. When it encounters binary operations it dynamically checks types and throws errors if they are mismatched.

*CI part.2* values are encoded as a struct with a type tag and a union of possible data (for numbers and strings, for example) and that’s basically it.

## Garbage collector

The Garbage collector purpose is to free memory that is no longer being used by the program. What we need to think about is when and how to use the Garbage collector. Knowing how to use it is less abstract, at least for me, because there are some simple ways to implement it, which I’ll cover soon.

Now, the task of knowing when to use it becomes more difficult, because there are many more variables to take into consideration. Since the implementations I know are not possible to do in a side thread (afaik), it depends directly on the language and the implementation. To make it simple, if the program has to run the *GC*, the rest of the program is stopped and this can slow down the whole execution, so it’s better to avoid it if possible. At the same time, if we take too long to run the *GC*, the execution itself it’s gonna take longer because there will, naturally, be more garbage to be *collected*, so we have to find a middle point. In the *CI Part.2* it is implemented to run up until a certain threshold is reached, and that threshold is that the allocated bytes are bigger than an arbitrary value (_1025*1025_). There is also a growing factor to the heap in this implementation, to increase the memory based on the program execution.

As to the “how”, both the *Tiger* and the *CI Part.2* use the same implementation of the mark and sweep algorithm, this algorithm is simple to demonstrate and it follows two main steps: marking and sweeping. The process involves marking the reachable objects from the roots, and then sweeping the unreachable ones, creating a scenario with 3 colours, white, gray and black. Every time the *GC* runs, it reaches all nodes that are initially white with a depth-first search algorithm and mark as gray. Then, it runs again to check if there are any nodes associated with the gray nodes. If so, it marks those as gray and changes the gray parent to black. This process is repeated recursively until no more gray nodes remain. Once all nodes have been processed, only the black and white nodes are left (those that were never reached and didn’t change to gray), the white ones are _collected_.

- Mark and sweep process demonstration (one *GC* run):
<p>
  <img src="/img/compilers/mark_sweep_1.svg" width="45%" style="display:inline-block; margin-right:4%;" />
  <img src="/img/compilers/mark_sweep_2.svg" width="45%" style="display:inline-block;" />
</p>
<p>
  <img src="/img/compilers/mark_sweep_3.svg" width="45%" style="display:inline-block; margin-right:4%;" />
  <img src="/img/compilers/mark_sweep_4.svg" width="45%" style="display:inline-block;" />
</p>
<p align="center">
  <img src="/img/compilers/mark_sweep_final.svg" width="40%" />
</p>

> It's Important to say that, this presupposes that the language variables are connected by some kind of pointer. In these two cases there is a direct pointer from one variable to another, so it’s simple to reach the nodes sequentially.

There are other GC types that are usually more complex. In the Tiger book several are introduced, including: Reference counting, generational collection, incremental collection, Baker’s and Cheney algorithm. The most interesting ones, for me, were the generational collection and the Baker’s and Cheney algorithm, which I would recommend you guys to look into and learn more about them.

## Optimizations
