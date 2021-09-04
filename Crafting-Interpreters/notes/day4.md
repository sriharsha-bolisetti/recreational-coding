##### The lexical grammars of Python and Haskell are not regular. What does that mean, and why aren’t they?

- Python and Haskell both use `indentation-sensitive syntax`, so they have to recognize changes in the indentation level as tokens.
- Doing that requires comparing amount of whitespace at the start of the consecutive lines, which can't be done using a regular grammar.
### Representing Code

- Before converting tokens into complex representation, we need to define it.
- The main goal is `a representation for code`
- It should be simple for parser to produce and easy for interpreter to consume.
##### Formal Language
- In logic, mathematics, computer science, and linguistics, a formal language consists of words whose letters are taken from an alphabet and are well-formed according to a specific set of rules.
- Well-formedness is the quality of a clause, word, or other linguistic element that conforms to the grammar of the language of which it is a part. Well-formed words or phrases are grammatical, meaning they obey all relevant rules of grammar. In contrast, a form that violates some grammar rule is ill-formed and does not constitute part of the language.
- The alphabet of a formal language consists of symbols, letters, or tokens that concatenate into strings of the language. Each string concatenated from symbols of this alphabet is called a word, and the words that belong to a particular formal language are sometimes called well-formed words or well-formed formulas. 
- A formal language is often defined by means of a formal grammar such as a regular grammar or context-free grammar, which consists of its formation rules.
- A formal grammar describes how to form strings from a language's alphabet that are valid according to language's syntax. A grammar does not describe the meaning of the strings or what can be done with them in whatever context––only their form. 
- A formal grammar is defined as a set of production rules for strings in a formal language.
- Rewriting covers a wide range of methods of replacing subterms of a formula with other terms. The objects of focus for this article include `rewriting systems` In their most basic form, they consist of set of objects, plus relations on how to transform these objects.
- Rewriting can be non-deterministic. One rul to rewrite a term could be applied in many different ways to that term, or more than one rule could be applicable.
- Rewriting systems then do not provide an algorithm for changing one term to another, but a set of possible rule applications. 
- When combined with an appropriate algorithm, however, rewrite systems can be viewed as computer programs, and several theorem provers and declarative programming languages are based on term rewriting.
- A production or production rule in computer science is a rewrite rule specifying a symbol substitution that can be recursively performed to generate new symbol sequences.
- A formal grammar is a set of rules for rewriting strings, along with a `start symbol` for which rewriting starts. Therefore, a grammar is usually thought of as a language generator. However, it can also sometimes be used the basis for a `recognizer`––a function in computing that determines whether a given string belongs to the language or is grammatically incorrect.
- To describe such recognizers, formal language theory uses seperate formalisms, known as `automata theory`
- One of the interesting results of automata theory is that it is not possible to design a recognizer for certain formal languages.
-  Parsing is the process of recognizing an utterance (a string in natural languages) by breaking it down to a set of symbols and analyzing each one against the grammar of the language. 
- Most languages have the meanings of their utterances structured according to their syntax—a practice known as compositional semantics. 
- As a result, the first step to describing the meaning of an utterance in language is to break it down part by part and look at its analyzed form (known as its parse tree in computer science, and as its deep structure in generative grammar).

##### Context-Free Grammars
- The formalism used for defining the lexical grammar——the rules for how characters get grouped into tokens——was called a `regular language`
- That was fine for scanner, which emits a flat sequence of tokens. But regular languages aren't powerful enough to handle expressions which can nest arbitrarily deeply.
- We need a bigger hammer, and that hammer is a `context-free grammar (CFG)`
- It's next heaviest tool in the toolbosx of `formal grammars.` 
- A formal grammar takes a set of atomic pieces it calls its `alphabet`. Then it defines a (usually infinite) set of `strings` that are `in` the grammar. Each string is a sequence of `letters` in the alphabet.
- In the scanner's grammar, the alphabet consists of individual characters and the strings are valid lexemes——roughly `words`. In the syntactic grammer, we are talking about was at a different level of granularity. Now each `letter` in the alphabet is an entire token and a `string` is a sequence of `tokens`——an entire expression.

| Terminology	|	Lexical grammar	| Syntactic grammar |
|---|---|---|
| The “alphabet” is . . .	→ |	Characters	| Tokens |
| A “string” is . . .	→ |	Lexeme or token |	Expression |
| It’s implemented by the . . .	→ |	Scanner |	Parser |
- A formal grammar's job is to specify which strings are valid and which aren't.
- If we were defining a grammar for English sentences, “eggs are tasty for breakfast” would be in the grammar, but “tasty breakfast for are eggs” would probably not.
#### Rules for Grammars
- How do we write down a grammar that contains an infinite number of valid strings? We obviously can't list them all out. Instead we create a finite set of rules. 
- This can be thought of as a game to be played in one of two directions.
- If you start with the rules, you can use them to `generate` strings that are in the grammar.
- Strings created this way are called `derivations` because each is `derived` from the rules of the grammar.
- In each step of the game, you pick a rule and follow what it tells you to do. Most of the lingo around formal grammars comes from playing them in this direction.
- Rules are called `productions` because they `produce` strings in the grammar.
- Each production in a context-free grammar has a `head`——its name––and a `body`, which describes what it generates.
- Restricting heads to a single symbol is a defining feature of context-free grammars. More powerful formalisms like unrestricted grammars allow a sequence of symbols in the head as well as in the body.
- In its pure form, the body is simply a list of symbols.
- Symbols come in two delectable flavors:
1. A `terminal` is a letter from the grammar's alphabet. You can think of it like a literal value. In the syntactic grammar we're defining, the terminals are individual lexemes––tokens coming from the scanner like `if` or `1234`

These are called `terminals`, in the sense of an `end point` because they don't lead to any further `moves` in the game. You simply produce that one symbol.

2. A `nonterminal` is a named reference to another rule in the grammar. It means `play that rule and insert whatever it produces here`. In this way, `the grammar composes`
- There is one last refinement: you may have multiple rules with the same name.
- When you reach a nonterminal with that name, you are allowed to pick any of the rules for it, whichever floats your boat.
 Each rule is a name, followed by an arrow (→), followed by a sequence of symbols, and finally ending with a semicolon (;). Terminals are quoted strings, and nonterminals are lowercase words.

 Using that, here's a grammar for breakfast menus:

breakfast  → protein `"with"` breakfast `"on the side"` ;
breakfast  → protein ;
breakfast  → bread ;

protein    → crispiness `"crispy" "bacon"` ;
protein    → `"sausage"` ;
protein    → cooked `"eggs"`;

crispiness → `"really"` ;
crispiness → `"really"` crispiness ;

cooked     → `"scrambled"` ;
cooked     → `"poached"` ;
cooked     → `"fried"` ;

bread      → `"toast"` ;
bread      → `"biscuits"` ;
bread      → `"English muffin"` ;

- We can use this grammar to generate random breakfasts. 
- The game starts with the first rule in the grammar, here `breakfast`. There are three productions for that, and we randomly pick the first one. 
 - Our resulting string looks like

 `protein "with" breakfast "on the side"`
 - We need to expand that first nonterminal, `protien`, so we pick a production for that. Let's pick:

protein → cooked `"eggs"`;

- Next, we need a production for `cooked`, and so we pick `"poached"`. That’s a terminal, so we add that. Now our string looks like:

`"poached" "eggs" "with" breakfast "on the side"`
- The next non-terminal is breakfast again. The first breakfast production we chose recursively refers back to the breakfast rule.
- Recursion in the grammar is a good sign that the language being defined is `context-free` instead of `regular`
- In particual, recursion where the recrusive nonterminal has productions on both sides implies that the language is not regular.
- We could keep picking the first production for breakfast over and over again yielding all manner of breakfasts like “bacon with sausage with scrambled eggs with bacon . . . ” We won’t though. This time we’ll pick bread. There are three rules for that, each of which contains only a terminal. We’ll pick “English muffin”.
- With that, every nonterminal in the string has been expanded until it finally contains only terminals and we’re left with:
`Pached eggs with English muffin on the side`
- Imagine that we’ve recursively expanded the breakfast rule here several times, like “bacon with bacon with bacon with . . . ” In order to complete the string correctly, we need to add an equal number of “on the side” bits to the end. Tracking the number of required trailing parts is beyond the capabilities of a regular grammar. Regular grammars can express repetition, but they can’t keep count of how many repetitions there are, which is necessary to ensure that the string has the same number of with and on the side parts.
- Any time we hit a rule that had multiple productions, we just picked one arbitrarily.
- It is this flexibility that allows a short number of grammar rules to encode a combinatorially larger set of strings. 
- The fact that a rule can refer to itself—directly or indirectly—kicks it up even more, letting us pack an infinite number of strings into a finite grammar.

#### Enahancing notation
- Stuffing an infinite set of strings in a handful of rules is pretty fantastic, but let’s take it further. Our notation works, but it’s tedious. So, like any good language designer, we’ll sprinkle a little syntactic sugar on top—some extra convenience notation. In addition to terminals and nonterminals, we’ll allow a few other kinds of expressions in the body of a rule:
1. Instead of repeating the rule name each time we want to add another production for it, we’ll allow a series of productions separated by a pipe (|).
`bread → "toast" | "biscuits" | "English muffin" ;`
- Further, we’ll allow parentheses for grouping and then allow | within that to select one from a series of options within the middle of a production.
` protein → ( "scrambled" | "poached" | "fried" ) "eggs" ;`

- Using recursion to support repeated sequences of symbols has a certain appealing purity, but it's kind of a chore to make a seperate named sub-rule each time we want to loop. So we use a `postfix` `*` `to allow the previous symbol or group to be repeated zero or more times.`

`crispiness → "really" "really"* ;`
- A postfix `+` is similar, but requires `the preceding production to appear at least once.`
`crispiness → "really"+ ;`
- A postfix `?` is for an `optional production`. The thing before it can `appear zero or one time`, but not more.
`breakfast → protein ( "with" breakfast "on the side" )? ;`
- With all of those syntactic niceties, our breakfast grammar condenses down to:
```
breakfast → protein ( "with" breakfast "on the side" )?
          | bread ;

protein   → "really"+ "crispy" "bacon"
          | "sausage"
          | ( "scrambled" | "poached" | "fried" ) "eggs" ;

bread     → "toast" | "biscuits" | "English muffin" ;
```
- If you’re used to grep or using regular expressions in your text editor, most of the punctuation should be familiar. The main difference is that symbols here represent entire tokens, not single characters.