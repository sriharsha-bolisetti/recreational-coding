- Given a string –– a series of tokens –– we map those tokens to terminals in the grammar to figure out `which rules could have generated that string`.
- The `could have` part is interesting. It's entirely possible to create a grammar that is `ambiguous`, where different choices of productions can lead to the same string. When you're using the grammar to `generate` strings, that doesn't matter much.
- When parsing, ambiguity means the parser may misunderstand the user's code.
- As we parse, we aren’t just determining if the string is valid Lox code, we’re also tracking which rules match which parts of it so that we know what part of the language each token belongs to.
- `Precedence` determines which operator is evaluated first in an expression containing a mixture of different operators. Precedence rules tell us that we evaluate the / before the -. Operators with higher precedence are evaluated before operators with lower precedence. Equivalently, higher precedence operators are said to “bind tighter”.
- Associativity determines which operator is `evaluated first in a series of the same operator`. When an operator is left-associative (think “left-to-right”), operators on the left evaluate before those on the right. Since - is left-associative, this expression:
5 - 3 - 1
is equivalent to:

(5 - 3) - 1
Assignment, on the other hand, is right-associative. This:

a = b = c
is equivalent to:

a = (b = c)
- Without well-defined precedence and associativity, an expression that uses multiple operators is `ambiguous`—it can be parsed into `different syntax trees`, which could in turn evaluate to `different results`.
- Right now, the grammar stuffs all expression types into a single expression rule. That same rule is used as the non-terminal for operands, which lets the grammar accept any kind of expression as a subexpression, regardless of whether the precedence rules allow it.
- We fix that by stratifying the grammar. We define a separate rule for each precedence level.
- Instead of baking precedence right into the grammar rules, some parser generators let you keep the same ambiguous-but-simple grammar and then add in a little explicit operator precedence metadata on the side in order to disambiguate.
- Each rule here only matches expressions at its precedence level or higher. For example, unary matches a unary expression like !negated or a primary expression like 1234. And term can match 1 + 2 but also 3 * 4 / 5. The final primary rule covers the highest-precedence forms—literals and parenthesized expressions.
- The top expression rule matches any expression at any precedence level. Since equality has the lowest precedence, if we match that, then it covers everything.

`expression     → equality`
- Over at the other end of the precedence table, a primary expression contains all the literals and grouping expressions.
```
primary        → NUMBER | STRING | "true" | "false" | "nil" | "(" expression ")" 
```
- A unary expression starts with a unary operator followed by the operand. Since unary operators can nest—!!true is a valid if weird expression—the operand can itself be a unary operator. A recursive rule handles that nicely.
`unary          → ( "!" | "-" ) unary ;`
- But this rule has a problem. It never terminates.
- `unary          → ( "!" | "-" ) unary
               | primary ;`
- `Remember, each rule needs to match expressions at that precedence level or higher, so we also need to let this match a primary expression.`
- `unary  → ( "!" | "-") unary  | primary ;`
- The remaining rules are all binary operators. We’ll start with the rule for multiplication and division. Here’s a first try:
`factor   → factor ( "/" | "*" ) unary   | unary ;`
- The rule recurses to match the left operand. That enables the rule to match a series of multiplication and division expressions like 1 * 2 / 3. `Putting the recursive production on the left side and unary on the right makes the rule left-associative and unambiguous`.
- All of this is correct, but the fact that the first symbol in the body of the rule is the same as the head of the rule means this production is left-recursive. Some parsing techniques, including the one we’re going to use, have trouble with left recursion
- Recursion elsewhere, like we have in unary and the indirect recursion for grouping in primary are not a problem.
- There are many grammars you can define that match the same language. The choice for how to model a particular language is partially a matter of taste and partially a pragmatic one. This rule is correct, but not optimal for how we intend to parse it. Instead of a left recursive rule, we’ll use a different one.
```
factor         → unary ( ( "/" | "*" ) unary )* ;
```
- We define a factor expression as a flat sequence of multiplications and divisions. This matches the same syntax as the previous rule, but better mirrors the code we’ll write to parse Lox. We use the same structure for all of the other binary operator precedence levels, giving us this complete expression grammar:

```
expression     → equality ;
equality       → comparison ( ( "!=" | "==" ) comparison )* ;
comparison     → term ( ( ">" | ">=" | "<" | "<=" ) term )* ;
term           → factor ( ( "-" | "+" ) factor )* ;
factor         → unary ( ( "/" | "*" ) unary )* ;
unary          → ( "!" | "-" ) unary
               | primary ;
primary        → NUMBER | STRING | "true" | "false" | "nil"
               | "(" expression ")" ;
```
#### Recursive Descent Parsing
- Recursive descent is considered a `top-down parser` because it starts from the top or outermost grammar rule (here expression) and works its way down into the nested subexpressions before finally reaching the leaves of the syntax tree. 
- This is in contrast with bottom-up parsers like LR that start with primary expressions and compose them into larger and larger chunks of syntax.
- A recursive descent parser is a literal translation of the grammar’s rules straight into imperative code. Each rule becomes a function. The body of the rule translates to code roughly like:

```

Grammar notation	Code representation
Terminal =	Code to match and consume a token
Nonterminal	= Call to that rule’s function
|	= if or switch statement
* or + =	while or for loop
?	= if statement
```
- The descent is described as “recursive” because when a grammar rule refers to itself—directly or indirectly—that translates to a recursive function call.
- Like the scanner, the parser consumes a flat input sequence, only now we’re reading tokens instead of characters. We store the list of tokens and use current to point to the next token eagerly waiting to be parsed.
- We’re going to run straight through the expression grammar now and translate each rule to Java code. The first rule, expression, simply expands to the equality rule, so that’s straightforward.
```
 private Expr expression() {
    return equality();
  }

```
- Each method for parsing a grammar rule produces a syntax tree for that rule and returns it to the caller. When the body of the rule contains a nonterminal—a reference to another rule—we call that other rule’s method.
- This is why left recursion is problematic for recursive descent. The function for a left-recursive rule immediately calls itself, which calls itself again, and so on, until the parser hits a stack overflow and dies.
- The fact that the parser looks ahead at upcoming tokens to decide how to parse puts recursive descent into the category of predictive parsers.
#### Syntax Errors
A parser really has two jobs:
1. Given a valid sequence of tokens, produce a corresponding syntax tree.
2. Given an invalid sequence of tokens, detect any errors and tell the user about their mistakes.
- There are a couple of hard requirements for when the parser runs into a syntax error. A parser must:
1. `Detect and report the error`. If it doesn’t detect the error and passes the resulting malformed syntax tree on to the interpreter, all manner of horrors may be summoned.
2. `Avoid crashing or hanging`. Syntax errors are a fact of life, and language tools have to be robust in the face of them. Segfaulting or getting stuck in an infinite loop isn’t allowed. While the source may not be valid code, it’s still a valid input to the parser because users use the parser to learn what syntax is allowed.
- A decent parser should:
- `Be fast`. Computers are thousands of times faster than they were when parser technology was first invented. The days of needing to optimize your parser so that it could get through an entire source file during a coffee break are over. But programmer expectations have risen as quickly, if not faster. They expect their editors to reparse files in milliseconds after every keystroke.
-`Report as many distinct errors as there are`. Aborting after the first error is easy to implement, but it’s annoying for users if every time they fix what they think is the one error in a file, a new one appears. They want to see them all.
- `Minimize cascaded errors`. Once a single error is found, the parser no longer really knows what’s going on. It tries to get itself back on track and keep going, but if it gets confused, it may report a slew of ghost errors that don’t indicate other real problems in the code. When the first error is fixed, those phantoms disappear, because they reflect only the parser’s own confusion. Cascaded errors are annoying because they can scare the user into thinking their code is in a worse state than it is.
#### Panic mode error recovery
- Of all the recovery techniques devised in yesteryear, the one that best stood the test of time is called—somewhat alarmingly—panic mode. As soon as the parser detects an error, it enters panic mode. It knows at least one token doesn’t make sense given its current state in the middle of some stack of grammar productions.
- Before it can get back to parsing, it needs to get its state and the sequence of forthcoming tokens aligned such that the next token does match the rule being parsed. This process is called `synchronization`.
- To do that, `we select some rule in the grammar that will mark the synchronization point`. 
- The parser fixes its parsing state by jumping out of any nested productions until it gets back to that rule. 
- Then it synchronizes the token stream by discarding tokens until it reaches one that can appear at that point in the rule.
- Any additional real syntax errors hiding in those discarded tokens aren’t reported, but it also means that any mistaken cascaded errors that are side effects of the initial error aren’t falsely reported either, which is a decent trade-off.
####  Synchronizing a recursive descent parser
- With recursive descent, the parser’s state—which rules it is in the middle of recognizing—is not stored explicitly in fields. Instead, we use Java’s own call stack to track what the parser is doing. Each rule in the middle of being parsed is a call frame on the stack. In order to reset that state, we need to clear out those call frames.
- The natural way to do that in Java is exceptions. When we want to synchronize, we throw that ParseError object. Higher up in the method for the grammar rule we are synchronizing to, we’ll catch it. Since we synchronize on statement boundaries, we’ll catch the exception there. After the exception is caught, the parser is in the right state. All that’s left is to synchronize the tokens.
- We want to discard tokens until we’re right at the beginning of the next statement. That boundary is pretty easy to spot—it’s one of the main reasons we picked it. After a semicolon, we’re probably finished with a statement. Most statements start with a keyword—for, if, return, var, etc. When the next token is any of those, we’re probably about to start a statement.
```
 private void synchronize() {
    advance();

    while (!isAtEnd()) {
      if (previous().type == SEMICOLON) return;

      switch (peek().type) {
        case CLASS:
        case FUN:
        case VAR:
        case FOR:
        case IF:
        case WHILE:
        case PRINT:
        case RETURN:
          return;
      }

      advance();
    }
  }
```
- It discards tokens until it thinks it has found a statement boundary. After catching a ParseError, we’ll call this and then we are hopefully back in sync. When it works well, we have discarded tokens that would have likely caused cascaded errors anyway, and now we can parse the rest of the file starting at the next statement.
