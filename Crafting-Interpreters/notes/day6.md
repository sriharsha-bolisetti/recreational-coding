#### Dynamic Dispatch
- Dynamic Dispatch is the process of selecting which implementation of a polymorphic operation (method or function) to call at run time. 
- Object-oriented systems model a problem as a set of interacting objects that enact operations referred to by name.
- Polymorphism is the phenomenon wherein somewhat interchangeable objects each expose an operation of the same name but possibly differing in behavior.
- As an example a `File` object and a `Database` object both have a `StoreRecord` method that can be used to write a personnel record to storage.
- Their implementations differ.
- A program holds a reference to an object which may be either a `File` object or a `Database` object. Which it may have been determined by a run-time setting, and at this stage, the program may not know or care which.
- When the program calls `StoreRecord` on the object, something needs to be choose which behavior gets enacted.
- If one thinks of OOP as `sending messages` to objects, then in this example the program sends a `StoreRecord` message to an object of unknown type, leaving it to the `runtime support system to dispatch the message to the right object`. The object enacts whichever behavior it implements.
- Dynamic dispatch contrasts with static dispatch, in which the implementation of a polymorphic operation is selected at compile time.
- The purpose of dynamic dispatch is to defer the selection of an appropriate implementation until the run time type of a parameter (or multiple parameters) is known.
- Dynamic dispatch is different from `late binding`. 
- Name binding `associates a name with an operation`. A polymorphic operation has several implementations, all associated with the same name. Bindings can be made at compile time or (with late binding) at run time.
- With dynamic dispatch, one particular implementation of an operation is chosen at run time. 
- While dynamic dispatch does not imply late binding, late binding does imply dynamic dispatch, since the implementation of a late-boudn operation is not known until run time.
#### Single and Multiple dispatch
- The choice of which version to call may be based on either on a single object or on a combination of objects. 
- The former is called a `single dispatch` and is directly supported by common object-oriented languages such as Smalltalk, C++, Java, C#, Objective-C, Swift, JavaScript, and Python.
- In these and similar languages, one may call a method for division with syntax that resembles
```dividend.divide(divisor) # dividend / divisor ```
where the parameters are optional
- This is thought of as sending a message named `divide` with parameter `divisor` to `dividend`
- An implementation will be chosen based on `dividend's type`(perhaps rational, floating point, matrix), disregarding the type or value of `divisor`
- By contrast, some languages dispatch methods or functions based on the `combination of operands`; in the division case, the types of the `dividend` and `divisor` together determine which `divide` operation will be performed. This is known as `multiple dispatch`. Examples of languages that support multiple dispatch are Common Lisp, Dylan, and Julia.
#### Multiple Dispatch
- Multiple dispatch or multimethods is a feature of some programming languages in which a function or method can be dynamically dispatched based on the run-time (dynamic) type or, in more general case, some other attribute of more than one of its arguments.
- This is a generalization of single-dispatch polymorphism where a function or method call is dynamically dispatched based on the derived type of the object on which the method has been called.
- Multiple dispatch routes the dynamic dispatch to the implementing function or method using the combined characteristics of one or more arguments.
#### Understanding dispatch
- Developers organize source code into named blocks––subroutines, procedures, subprograms, functions or methods.
- The code in the function is executed by `calling it`- executing a piece of code that references its `name`. This transfers control temporarily to the called function; when the function's execution has completed, control is typically transferred back to the instruction in the `caller` that follows the reference.
- Function names are usually selected so as to be descriptive of the function's purpose. It is sometimes desirable to give several functions the same name, often because they perform conceptually similar tasks, but operate on different types of input data. 
-  In such cases, the name reference at the function call site is not sufficient for identifying the block of code to be executed. Instead, the number and type of the arguments to the function call are also used to select among several function implementations.
- In more conventional, i.e., single-dispatch object-oriented programming languages, when invoking a method (sending a message in Smalltalk, calling a member function in C++), one of its arguments is treated specially and used to determine which of the (potentially many) classes of methods of that name is to be applied. In many languages, the special argument is indicated syntactically; for example, a number of programming languages put the special argument before a dot in making a method call: special.method(other, arguments, here), so that lion.sound() would produce a roar, whereas sparrow.sound() would produce a chirp.
- In contrast, in languages with multiple dispatch, the selected method is simply the one whose arguments match the number and type of the function call. There is no special argument that owns the function/method carried out in a particular call.
##### Data types
- When working with languages that can discriminate data types at compile time, selecting among the alternatives can occur then. The act of creating such alternative functions for compile time selection is usually referred to as `overloading` a function.
- In programming languages that defer data type identification until run time (i.e., late binding), selection among alternative functions must occur then, based on the dynamically determined types of function arguments. Functions whose alternative implementations are selected in this manner are referred to most generally as `multimethods`.
- There is some run-time cost associated with dynamically dispatching function calls. In some languages, the distinction between overloading and multimethods can be blurred, with the compiler determining whether compile time selection can be applied to a given function call, or whether slower run time dispatch is needed.
- In a language with only single dispatch, like Java, multiple dispatch can be emulated with multiple levels of single dispatch:
```
interface Collideable {
    void collideWith(final Collideable other);

    /* These methods would need different names in a language without method overloading. */
    void collideWith(final Asteroid asteroid);
    void collideWith(final Spaceship spaceship);
}
class Asteriod implements Collideable {
    public void collideWith(final Collideable other) {
        //Call collideWith on the other object
        other.collideWith(this);
    }
    public void collideWith(final Asteroid asteroid){
        //Handle Asteroid-Asteroid collision.
    }
    public void collideWith(final Spaceship spaceship){
        //Handle Asteroid-Spaceship collision
    }
}
class Spaceship implements Collideable{
    public void collideWith(final Collideable other){
        //Call collideWith on the other object.
        other.collideWith(this);
    }
    public void collideWith(final Asteroid asteroid){
        //Handle Spaceship-Asteroid collision
    }
    public void collideWith(final Spaceship spaceship){
        //Handle Spaceship-Spaceship collision
    }
}
```
- Run time `instanceof` checks at one or both levels can also be used.

#### Late Binding
- Late binding is a programming mechanism in which the method being called upon an object, or the function being called with arguemtns is looked up by the name at runtime. In other words, a name is associated with a particular operation or object at runtime, rather than during compilation.
- With early binding or static binding, in an object-oriented language, the compilation phase fixes all types of variables and expressions. This is stored in the compliled program as an offset in a virtual method table. In contrast, with late binding, the compiler does not read enough information to verify the method exists or bind its slot on the v-table. Instead, the method is looked up by name at runtime.
- Java can use late binding using its reflection APIs and type introspection much in the same way it is done in COM and .NET programming. Generally speaking those who only program in Java do not call this late binding. Likewise the use of "duck typing" techniques is frowned upon in Java programming, with abstract interfaces used instead.

#### Type Introspection
- In computing, type introspection is the ability of a program to examine the type or properties of an object at runtime. Some programming languages possess this capability.
- Introspection should not be confused with reflection, which goes a step further and is the ability for a program to manipulate the values, metadata, properties, and functions of an object at runtime. Some programming languages also possess that capability (e.g., Java, Python, Julia, and Go).
#### Double Dispatch
- Double dispatch is a special form of multiple dispatch, and a mechanism that dispatches a function call to different concrete functions depending on the runtime types of two objects involved in the call.
- In most object-oriented systems, the concrete function that is called from a function call in the code depends on the dynamic type of a single object and therefore they are known as single dispatch calls, or simply virtual function calls.
- The general problem addressed is how to dispatch a message to different methods depending not only on the receiver but also on the arguments.
- To that end, systems like Common Lisp Object System implement multiple dispatch. Double dispatch is another solution that gradually reduces the polymorphism on systems that do not support multiple dispatch.
##### Use Cases
- Double dispatch is useful in situations where the choice of computation depends on the runtime types of its arguments. 
- For example, a programmer could use double dispatch in the following situations:
- Sorting a mixed set of objects: algorithms require that a list of objects be sorted into some canonical order. Deciding if one element comes before another element requires knowledge of both types and possibly some subset of the fields.
- Adaptive collision algorithms usually require that collisions between different objects be handled in different ways. A typical example is in a game environment where the collision between a spaceship and an asteroid is computed differently from the collision between a spaceship and a spacestation.
- Painting algorithms that require the intersection points of overlapping sprites to be rendered in a different manner.
- Personnel management systems may dispatch different types of jobs to different personnel. A `schedule` algorithm that is give a person object typed as an accountant and a job object typed as engineering rejects the scheduling of that person for that job.
- Event handling systems that use both the event type and the type of the receptor object in order to call the correct event handling routine.
- Lock and key systems where there are many types of locks and many types of keys and every type of key opens multiple types of locks. Not only do you need to know the types of the objects involved, but the subset of `information about a particular key that are relevant to seeing if a particular key opens a particular lock` is different between different lock types.
##### Common Idiom
- The common idiom, as in the examples presented above, is that the selection of the appropriate algorithm is based on the` call's argument types at runtime`. 
- The call is therefore subject to all the usual additional performance costs that are associated with dynamic resolution of calls, usually more than in a language supporting only single method dispatch. 
- In C++, for example, a dynamic function call is usually resolved by a single offset calculation - which is possible because the compiler knows the location of the function in the object's method table and so can statically calculate the offset. 
- In a language supporting double dispatch, this is slightly more costly, because the compiler must generate code to calculate the method's offset in the method table at runtime, thereby increasing the overall instruction path length (by an amount that is likely to be no more than the total number of calls to the function, which may not be very significant).
#### Visitor Pattern
- The visitor pattern's `visit`/`accept` constructs are a necessary evil due to C-like languages' (C#, Java, etc.) semantics. 
- The goal of the visitor pattern is to `use double-dispatch to route your call`
- Normally when the visitor pattern is used, an object hierarchy is involved where all the nodes are derived from a base Node type, referred to henceforth as `Node`
```
Node root = GetTreeRoot();
new MyVisitor().visit(root);
```
- Herein lies the problem. If our MyVisitor class was defined like the following:
```
class MyVisitor implements IVisitor {
  void visit(CarNode node);
  void visit(TrainNode node);
  void visit(PlaneNode node);
  void visit(Node node);
}
```
- If, `at runtime`, regardless of the `actual` type that `root` is, our call would go into the overload `visit(Node node)`
- This would be true for all variables declared of type Node. Why is this? 
- Because Java and other C-like languages `only consider the static type`, or the type that the variable is declared as, `of the parameter when deciding which overload to call`
- Java doesn't take the extra step to ask, for every method call, at runtime, "Okay, what is the dynamic type of root? Oh, I see. It's a TrainNode. Let's see if there's any method in MyVisitor which accepts a parameter of type TrainNode...".
- The compiler, at compile-time, determines which is the method that will be called.
- Java does give us one tool for taking into account the runtime (i.e. dynamic) type of an object when a method is called -- virtual method dispatch. 
- When we call a virtual method, the call actually goes to a table in memory that consists of function pointers.
-  Each type has a table. If a particular method is overridden by a class, that class' function table entry will contain the address of the overridden function. If the class doesn't override a method, it will contain a pointer to the base class' implementation. 
- This still incurs a performance overhead (each method call will basically be dereferencing two pointers: one pointing to the type's function table, and another of function itself), but it's still faster than having to inspect parameter types.
- `The goal of the visitor pattern is to accomplish double-dispatch` –– not only is the type of the call target considered (MyVisitor, via virtual methods), but also the type of the parameter (what type of Node are we looking at)?
- The Visitor pattern allows us to do this by the visit/accept combination.
- By changing our line to this:
`root.accept(new MyVisitor());`
- We can get what we want: via virtual method dispatch, we enter the correct accept() call as implemented by the subclass ––  in our example with TrainElement, we'll enter TrainElement's implementation of accept():
```
class TrainNode extends Node implements IVisitable {
  void accept(IVisitor v) {
    v.visit(this);
  }
}
```
- What does the compiler know at this point, inside the scope of TrainNode's accept? `It knows that the static type of this is a TrainNode`.
- This is an important additional shred of information that `the compiler was not aware of in our caller's scope`: there, all it knew about root was that it was a Node.
- Now the compiler knows that this (root) is not just a Node, but it's actually a TrainNode.
- In consequence, the one line found inside `accept(): v.visit(this)`, means something else entirely.
- The compiler will now look for an overload of visit() that takes a TrainNode
- If it can't find one, it'll then compile the call to an overload that takes a Node.
-  If neither exist, you'll get a compilation error (unless you have an overload that takes object)
- `Execution will thus enter what we had intended all along: MyVisitor's implementation of visit(TrainNode e)`
-  No casts were needed, and, most importantly, no reflection was needed. Thus, the overhead of this mechanism is rather low: it only consists of pointer references and nothing else.
-  We can use a cast and get the correct behavior. However, often, we don't even know what type Node is. Take the case of the following hierarchy:
```
abstract class Node { ... }
abstract class BinaryNode extends Node { Node left, right; }
abstract class AdditionNode extends BinaryNode { }
abstract class MultiplicationNode extends BinaryNode { }
abstract class LiteralNode { int value; }
```
- And we were writing a simple compiler which parses a source file and produces a object hierarchy that conforms to the specification above. 
- If we were writing an interpreter for the hierarchy implemented as a Visitor:
```
class Interpreter implements IVisitor<int> {
  int visit(AdditionNode n) {
    int left = n.left.accept(this);
    int right = n.right.accept(this); 
    return left + right;
  }
  int visit(MultiplicationNode n) {
    int left = n.left.accept(this);
    int right = n.right.accept(this);
    return left * right;
  }
  int visit(LiteralNode n) {
    return n.value;
  }
}
```
- Casting wouldn't get us very far, since we don't know the types of `left` or `right` in the `visit()` methods.
- Our parser would most likely also just return an object of type `Node` which pointed at the root of the hierarchy as well, so we can't cast that safely either. So our simple interpreter can look like:
```
Node program = parse(args[0]);
int result = program.accept(new Interpreter());
System.out.println("Output: " + result);
```
- The visitor pattern allows us to do something very powerful: `given an object hierarchy, it allows us to create modular operations that operate over the hierarchy without needing requiring to put the code in the hierarchy's class itself.`
- The visitor pattern is used widely, for example, in compiler construction. Given the syntax tree of a particular program, many visitors are written that operate on that tree: type checking, optimizations, machine code emission are all usually implemented as different visitors.
- In the case of the optimization visitor, it can even output a new syntax tree given the input tree.
- It has its drawbacks, of course: if we add a new type into the hierarchy, we need to also add a visit() method for that new type into the IVisitor interface, and create stub (or full) implementations in all of our visitors. We also need to add the accept() method too, for the reasons described above. If performance doesn't mean that much to you, there are solutions for writing visitors without needing the accept(), but they normally involve reflection and thus can incur quite a large overhead.

TODO read -> http://codebetter.com/jeremymiller/2007/10/31/be-not-afraid-of-the-visitor-the-big-bad-composite-or-their-little-friend-double-dispatch/