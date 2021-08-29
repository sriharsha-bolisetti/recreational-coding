#### Scanning 
- Lexical Analysis
- A scanner takes in the linear stream of characters and chunks them together into a series of something akin to words.
- In programming languages, each of these words is called a `token`
- Some tokens are single characters, like ( and , .Others may be several characters long, like numbers (123), string literals ("hi!"), and identifiers(min)
- Some characters in a source file don't actually mean anything.
- Whitespace is often insignificant, and comments, by definition are ignored by the language.
- The scanner usually discards these, leaving a clean sequence of meaningul tokens.

#### Parsing
- The next step is `parsing`
- This is where our syntax gets a `grammar` - the ability to compose larger expressions and statements out of smaller parts.
- A `parser` takes the flat sequence of tokens and builds a tree structure that mirros the nested nature of the grammar.
- These trees have a couple of different names - parse tree or abstract syntax tree - depending on how close to the bare syntactic structure of the source language they are.
- In practice, language hackers usually call them `syntax trees, ASTs,` or often just `trees`

#### Static Analysis
- The first bit of analysis that most languages do is called `binding` or `resolution`
- For each `identifier`, we find out where that name is defined and wire the two together
- This is where `scope` comes into play - the region of source code where a certain name can be used to refer to a certain declaration.
- If the language is statically typed, this is when we type check.
- Once we know where a and b are declared, we can also figure out their types.
- Then if those types don't support being added to each other, we report a `type error`
- All this semantic insight that is visible to us from analysis needs to be stored somewhere. There are few places we can squirrel it away -
1. Often, it gets stored right back as `attributes` on the syntax tree itself——extra fields in the nodes that aren't initialized during parsing but get filled in later.
2. Other times, we may store data in a lookup table off to the side. Typically, the keys to this table are identifiers——names of variables and declarations. In that case, we call it a `symbol table` and the values it associates with each key tell us what the identifier referes to.
- The most powerful bookkeeping tool is to transform the tree into an entirely new data structure that more directly expresses the semantics of the code.
#### Intermediate Representations
- Complier can be thought of as a pipeline where each stage's job is to organize the data representing the user's code in a way that makes the next stage simpler to implement.
- The `front end of the pipeline is specific to the source language the program is written in`.
- The `back end is concerned with final architecture where the program will run`
- In the middle, the code may be stored in some `intermediate representation` that isn't tightly tied to either the source or destination forms. Instead, the IR acts as an interface between these two languages.
- This lets you support multiple source languages and target platforms with less effort.
- Say you want to implement Pascal, C and Fortran compilers, and you want to target x86, ARM and SPARC. Normally, that means you’re signing up to write nine full compilers: Pascal→x86, C→ARM, and every other combination.
- A shared intermediate representation reduces that dramatically. You write one front end for each source language that produces the IR. Then one back end for each target architecture.

There's another big reason we might want to transform the code into a form that makes the semantics more apparent...

#### Optimization
- Once we understand what the user's program means, we are free to swap it out with a different program that has the `same semantics` but implements them more efficiently - we can `optimize` it.
- A simple example is `constant folding` - if some expression always evaluates to the exact same value, we can do the evaluation at compile time and replace the code for the expression with its result.
#### Code Generation
- We have applied all of the optimizations we can think of to the user's program.
- The last step is converting it to a from the machine can actually run.
- In other words, `generating code (or code gen)` where code here usually refers to the kind of primitive assembly like instructions a CPU runs and not the kind of "source code" a human want to read.
- We have a decision to make. Do we generate instructions for a real CPU or a virtual one? If we generate real machine code, we get an executable that the OS can load directly onto the chip. Native code is lightning fast, but generating it is a lot of work. 
- Instead of instructions for some real chip, they produced code for a hypothetical, idealized machine. Wirth called this `p-code` for `portable`, but today this is generally called `bytecode` because each instruction is often a single byte long.
- These synthetic instructions are designed to map a little more closely to the language's semantics, and not to be so tied to the peculiarities of any one computer architecture and its accumulated historical cruft.
- You can think of it like a dense, binary encoding of the language's low-level operations.
#### Virtual Machine
- If your compiler produces bytecode, your work isn't over once that's done. 
- Since there is no chip that speaks that bytecode, it's our job to translate.
- We have two options.
1. You can write a little mini-compiler for each target architecture that converts the bytecode to native code for that machine. You still have to do work for each chip you support, but this last stage is pretty simple and you get to reuse the rest of the compiler pipeline across all the machines you support. Here, you're basically using your bytecode as an intermediate representation.
- The basic principle here is that the farther down the pipeline you push the architecture-specific work, the more of the earlier phases you can share across architectures.
- There is tension though. Many opitmizations, like register allocation and instruction selection, work best when they know the strengths and capabilities of a specific chip. Figuring out which parts of your compiler can be shared and which should be target-specific is an art.
2. Or you can write a `virtual machine (VM)`, a program that emulates a hypothetical chip supporting your virtual architecture at runtime. Running bytecode in a VM is slower than translating it to native code ahead of time because every instruction must be simulated at runtime each time it executes. In return, you get simplicity and portability. Implement your VM in say C, and you can run your language on any platform that has a C compiler. 
#### Runtime
- Once we finally hammered the user's program into a form that we can execute. The last step is running it.
- If we compliled it to machine code, we can simply tell operating system to load the executable and off it goes.
- If we complied it to bytecode, we need to start up the VM and load the program into that.
- In both cases, for all but the basest of low-level languages, we usually `need some services that our language provides while the program is running.` 
- For example, if the language automatically manages memory, we need a garbage collector going in order to reclaim unused bits.
- If our language supports `instance of` tests so you can see what kind of object you have, then we need some representation to keep track of the type of each object during execution.
- All of this stuff is going at runtime, so it's called, appropriately, the `runtime`
- In a fully compiled language, the code implementing the runtime gets inserted directly into the resulting executable.
- In Go, each compiled application has its own copy of Go's runtime directly embedded in it.
- If the language is run inside an interpreter or VM, then runtime lives there. This is how most implementations of languages like Java, Python and JavaScript work.

### Shortcuts and Alternate Routes
- That's the long path covering every possible phase you might implement.
- Many languages do walk the entire route, but there are a few shortcuts and alternate paths

#### Single-pass compilers
- Some simple compilers interleave parsing, analysis, and code generation so that they produce output code directly in the parser, without ever allocating any syntax trees or other IRs. These `single-pass compilers` restrict the design of the language.
- You have no intermediate data structures to store global information about the program, and you don't revisit any previously parsed part of the code. This means as soon as you see some expression, you need to know enough to correctly compile it.
- Pascal and C were designed around this limitation. At the time, memory was so precious that a compiler  
#### Tree-walk Interpreters
- Some programming languages begin executing code right after parsing it to an AST.
- To run the program, the interpreter traverses the syntax tree one branch and leaf at a time, evaluating each node as it goes.
- This implementation style is common for student projects and little languages but is not widely used for general-purpose languages since it tends to be slow.
#### Transpilers
- Writing a complete back end for a languatge can be lot of work.
- If you have some existing generic IR to target, you could bolt your front end onto that.
- Otherwise, it seems like you're stuck. But what if you treated some other `source language` as if it were an intermediate representation.
- You write a front end for your language. Then, in the back end, instead of doing all the work to `lower` the semantics to some primitive target language, you produce a string of valid source code for some other language that's about as high level as yours. Then, you can use the existing compilation tools for `that` language as your escape route off the mountain and down to something you can execute.
- They used to call this as a `source-to-source compiler` or a `transcompiler`. After the rise of languages that compile to Javascript in order to run in browser, they’ve affected the hipster sobriquet transpiler.
- The front end - scanner and parser - of a transpiler looks like other compilers. Then, if the source language is only a simply syntactic skin over the target language, it may skip analysis entirely and go straight to outputting the analogous syntax in the destination language.
- If the two languages are more semantically different, you’ll see more of the typical phases of a full compiler including analysis and possibly even optimization. Then, when it comes to code generation, instead of outputting some binary language like machine code, you produce a string of grammatically correct source (well, destination) code in the target language.
- Either way, you then run that resulting code through the output language’s existing compilation pipeline, and you’re good to go.

#### Just-in-time compilation
- The fastest way to execute code is by compiling it to machine code.
- To work around the problem of not knowing the architecture of end user's machine - the same strategy employed by Hotspot JVM or Microsoft's Common Language runtime and most Javascript interpretors can be used. 
- On the end user's machine, when the program is loaded - either from source in the case of JS, or platform-independent bytecode for the JVM and CLR - you compile it to native code for the architecture their computer supports.
- This is called `just-in-time-compilation`

#### Compilers and Interpreters
- `Compiling` is an `implementation technique` that involves translating a source language to some other - usually lower-level form. When you generate bytecode or machine code you are compiling. When you transpile to another higher level language, you are compiling too.
- When we say a language implementation `is a compiler`, we mean it translates source code to some other form but doesn't execute it. The user has to take the resulting output and run it themselves.
- Conversely, when we say an implementation `is an interpreter`, we mean it takes in source code and executes it immediately. It runs programs `from source`