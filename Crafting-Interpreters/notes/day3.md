- It's good engineering practice to separate the code that `generates` the errors from the code that `reports` them.
- Various phases of the front end will detect errors, but it's not really their job to know how to present that to a user.
- In a full-featured language implementation, you will likely have multiple ways errors get displayed: on stderr, in an IDE's error window, logged to a file, etc. You don't want that code smeared all over scanner and parser.
- Ideally, there would be an actual abstraction, some kind of "ErrorReporter" interface that gets passed to the scanner and parser so that we can swap out different reporting strategies. 
#### Lexemes and Tokens
- Lexical Analysis -> scans through the list of characters and group them together into the smallest sequences that still represent something. 
- Each of these blobs of characters is called a `lexeme`
Eg. `var language = "lox";`
Lexemes -> var, language, =, "lox", ;
- The lexemes are only the raw substrings of the source code. 
- However, in the process of grouping character sequences into lexemes, we also stumble upon some other useful information. 
- When we take the lexeme and bundle it together with that other data, the result is a `token`. It includes useful stuff like
##### Token Type
- Keywords are part of the shape of language's grammar, so the parser often has code like - "if the next token is `while` then do..." That means the parser wants to know not just that it has a lexeme for some identifier, but that it has a `reserved` word, and `which` keyword it is.
- The parser could catagorize tokens from the raw lexeme by comparing the strings, but that's slow and kind of ugly.
- Instead, at the point that we recognize a lexeme, we also remember which `kind` of lexeme it represents. 
- We have a different type for each keyword, operator, bit of punctuation, and literal type.
##### Literal Value
- There are lexemes for literal values––numbers and strings and the like. Since the scanner has to walk each character in the literal to correctly identify it, it can also convert that textual representation of a value to the living runtime object that will be used by the interpreter later.

##### Location Information
##### Location Information
- To tell users `where` errors occurred, tracking that starts here. 
##### Regualr Languages and Expressions
- The core of the scanner is a loop. 
- Starting at the first character of the source code, the scanner figures out what lexeme the character belongs to, and consumes it and any following characters that are part of that lexeme. When it reaches the end of that lexme, it emits a `token`
- Then it loops back and does it again, starting from the very next character in the source code. It keeps doing that, eating characters and occasionally, uh, excreting tokens, until it reaches the end of the input.
##### Maximal munch
- When two lexical grammar rules can both match a chunk of code that the scanner is looking at, `whichever one matches the most characters wins`
- That rule states that if we can match `orchid` as identifier and `or` as a keyword, then the former wins. This is also why we tacitly assumed, previously, that `<=` should be scanned as a single `<=` token and not `<` followed by `=`
- Maximal munch means we can't easily detect a reserved word until we've reached the end of what might instead be an identifier. After all, a reserved word `is` an identifier, it's just one that has been claimed by the language for its own use. That's where the term `reserved word` somes from.
- So we begin by assuming any lexeme starting with a letter or underscore is an identifier.
