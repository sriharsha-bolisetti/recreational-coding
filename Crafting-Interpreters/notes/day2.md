- A JIT compiler doesn't complile code `Ahead-Of-Time (AOT)`, but still compiles source code to machine code and therefore is not an interpreter.
- JITs `compile code at runtime, while your program is executing.`
- This gives JITs flexibility for dynamic language features, wile maintaining speed from optimized machine code output.
- JIT-compiling C would make it slower as we'd just be adding the compilation time to the execution time.
- JIT-compiling Python would be fast, as `compilation + executing machine code can often be faster than interpreting`
- JITs improve implemnetations in speed by being able to optimize on information that is only available at runtime.
- A common theme between compiled languages is that they're statically typed. That means when the programmer creates or uses a value, they’re telling the computer what type it is and that information is guaranteed at compile time.
- Julia is dynamically typed, but internally Julia is much closer to being statically typed.
- Most JITs are not about compiling code `just-in-time`, but compiling `optimal code` at an optimal time.
- In some cases that time is never.
- In other cases, `compilation occurs more than once`
- In many cases where code is compiled, it doesn't occur until after source code has been executed numerous times, and the JIT will stay in an interpreter as the overhead to compilation is too high to be valuable.
- The other aspect at play is generating `optimal code`
- Assembly instructions are not created equal, and compilers will put a lot of effort into generating well-optimized machine code.
- The other aspect at play is generating `optimal code`
- Assembly instructions are not created equal, and compilers will put a lot of effort into generating well-optimized machine code.
- Usually, its possible for a human to write better assembly than a compiler, because the compiler cannot dynamically analyze your code. Eg. things like knowing possible range of integers or what keys are in the map, as these are things that a computer could only know after (partially) executing your program.
- A JIT compiler can do those things because it interprets your code first and gathers data from the execution. Thus, JITs are `expensive in that they interpret, and add compilation time to execution time,` but they make it up in highly optimised compiled code. With that, the timing of compilation is also dependent on whether the JIT has gathered enough valuable information.
- `JIT implementations of C could not be faster than existing compiled implementations.`
- If C is `JIT compiled` similar to Julia, it would be impossible to make it faster than compiled-C as the `compiled-time is non-negative and the generated machine code is essentially the same`
- JITs has a concept of `warming up.` Because interpretation and profiling time is expensive, JITs will start by executing a program slowly and then work towards `peak performance`
- For JITs with interpreted counterparts like Pypy, the JIT without warmup performs much worse at the beginning of execution due to the overhead of profiling. It's also the reason that JITs will consume significantly more memory.
- Warmup adds complexity to measuring efficiency of a JIT!
- It's fine if you're measuring the performance of generating the mandelbrot set, but becomes painful if you're serving a web application and the first N requests are painfully slow. It means that Javascript is relatively less performant as a command line tool than it is for a webserver. It’s complicated by the fact that the performance doesn’t strictly increase.
- If Pypy decides it needs to compile many things all at once after JITs compiling some functions, then you might have a slow-down in the middle.
- It also makes benchmark results more ambiguous, as you have to check if the jitted languages were given time to warmup, but you'd also want to know if it took an unseemly amount of time to warmup.
- Optimizing compiled code and warmup speed is unfortuantely zero-sum by nature.
- If you try to get your code to compile sooner, less data will be available, the compiled code will not be as efficient and peak performance will be lower. Aiming for higher peak performance of course, often means higher profiling costs.
- Java and Javascript engines are examples of JITs that have put really good care into warmup time, but you may find that languages built for academic uses have monstrous warmup times in favour of snazzy peak performances.

### The Lox Language
- Lox is `dynamically typed`
- High level languages exist to eliminate error-prone, low-level drudgery, an.
- There are two main techniques for managing memory - `reference counting` and `tracing garbage collection`
- Ref counting is much simpler to implement.
- Lox has only one kind of number——double-precision floating point.
- `nil`
- `Strings`
- Expressions
- Statements -> Where an expression's main job is to produce a `value`, a statement's job is to produce an `effect`
- Since, by definition, statements don't evaluate to a value, to be useful they have to otherwise change the world in some way —— usually modifying some state, reading input, or producing output
- An expression followed by a semicolon (;) promotes the expression to statement-hood. This is called (imaginatively enough), an `expression statement`.
- If you want to pack a series of statements where a single one is expected, you can wrap them up in a `block`.
- Control Flow
- Function
- An `argument` is an actual value you pass to a function when you call it. So a function `call` has an `argument` list. 
- A `parameter` is a variable that holds the value of the argument `inside the body of the function`.  Thus, a function `declaration` has a `parameter` list. 
##### Closures
- Functions are `first class` in Lox, which just means they are real values that you can get a reference to, store in variables, pass around, etc.
- Since function declarations are statements, you can declare local functions inside another functions.
- If you combine local functions, first-class functions, and block scope, you run into this interesting situation
```
fun returnFunction() {
  var outside = "outside";

  fun inner() {
    print outside;
  }

  return inner;
}

var fn = returnFunction();
fn();
```
- Here  `inner()` accesses a local variable declared outside of its body in the surrounding function. 
- For this to work, `inner()` has to `hold on` to references to any surrounding variables that it uses so that they stay around even after the outer function has returned. We call functions that do this `closures`
- For a dynamically typed language, objects are pretty handy.
- We need `some` way of defining compound data types to bundle blobs of stuff together.
- If we can also hang methods off of those, then we avoid the need to prefix all of our functions with the name of the data type they operate on to avoid colliding with similar functions for different types.
- In, say, Racket, you end up having to name your functions like hash-copy (to copy a hash table) and vector-copy (to copy a vector) so that they don’t step on each other. Methods are scoped to the object, so that problem goes away.
- In class-based languages, there are two core concepts: instances and classes.
- Instances store the state for each object and have a reference to the instance's class.
- Classes contain the methods and inheritance chain.
- To call a method on an instance, there is always a level of indirection. You look up the instance's class and then you find the method `there`
- Prototype-based languages merge these two concepts. There are only objects—no classes—and each individual object may contain state and methods. Objects can directly inherit from each other (or “delegate to” in prototypal lingo):
- This means that in some ways prototypal languages are more fundamental than classes. They are really neat to implement because they’re so simple. Also, they can express lots of unusual patterns that classes steer you away from.

You declare a class and its methods like so:
```
class Breakfast {
  cook() {
    print "Eggs a-fryin'!";
  }

  serve(who) {
    print "Enjoy your breakfast, " + who + ".";
  }
}
```
The body of a class contains its methods. They look like function declarations but without the fun keyword. When the class declaration is executed, Lox creates a class object and stores that in a variable named after the class. Just like functions, classes are first class in Lox.

They are still just as fun, though.
```
// Store it in variables.
var someVariable = Breakfast;
```
// Pass it to functions.
someFunction(Breakfast);
Next, we need a way to create instances. We could add some sort of `new` keyword, but to keep things simple, in Lox the class itself is a factory function for instances. Call a class like a function, and it produces a new instance of itself.
```
var breakfast = Breakfast();
print breakfast; 
```
- Classes that only have behavior aren't super useful. The idea behind object-oriented programming is `encapsulating behavior and state together` To do that, you need `fields`

```
breakfast.meat = "sausage";
breakfast.bread = "sourdough";
```
Assigning to a field creates it if it doesn't already exist.
- If you want to access a field or method on the current object from within a method, you use good old this.
- Part of encapsulating data within an object is ensuring that object is in a valid state when it's created. To do that, you can define an initializer. If you class has a method named init(), it is called automatically when the object is constructed. Any parameters passed to this class are forwarded to its initializer.
- `<` is the inheritance operator
- `init()` methods gets inherited.
- In practice, the subclass usually wants to define its own `init()` method too, but the original one also needs to be called so that the superclass can maintain its state. 
- We need some way to call a method on our own `instance` without hitting our own `methods` - you can use `super` for that
- Standard library——the set of functionality that is implemented directly in the interpreter and that all user-defined behavior is built on top of.

#### Resources:
1. https://carolchen.me/blog/technical/jits-intro/
2. http://craftinginterpreters.com/the-lox-language.html