## Chapter 1:

how does js program is compiled? 
1. Tokenization \ Lexing = breaking up words into 'tokens'
	`var a = 2;` turns into following tokens `var` `a` `=` `2` `;`
2. Parsing = creating AST (Abstract Syntax Tree)
	takes stream (array) of tokens, turns them into a tree of nested elements, which represent the grammatical structure of program.  
3. Code generation
	transforms AST into executable code


Page 7.

### Syntax errors
 js is not executing programm line-by-line
it firstly, parses the whole programm

proof:
```
var greeting = "Hello";
console.log(greeting);
greeting = ."Hi";
// SyntaxError: unexpected token .
```
here, you see right? `Tokenization` part comes out <= no "Hello", though it's well-written error-free line.


page 8.

### Early Errors
```
console.log("Howdy");
saySomething("Hello","Hi");
// Uncaught SyntaxError: Duplicate parameter name not
// allowed in this context
function saySomething(greeting,greeting) {
	"use strict";
	console.log(greeting);
}
```

yeah, ofc, there would be no error without strict-mode, but anyways
how does js knows whether to throw an (as called `early`) error, because of function being in strict-mode, if js haven't yet seen the "use strict" pragma, which appears later after function declaration?
The only way may be found - the programm is FULLY parsed before execution  


3. Hoisting


```javascript
function saySomething() {
var greeting = "Hello";
	{
		greeting = "Howdy"; // error comes from here
		let greeting = "Hi";
		console.log(greeting);
	}
}
saySomething();
// ReferenceError: Cannot access 'greeting' before
// initialization
```
The only way the JS engine could know, at the line where
the error is thrown, that the next statement would declare
a block-scoped variable of the same name ( greeting ) is if
the JS engine had already processed this code in an earlier
pass, and already set up all the scopes and their variable
associations. This processing of scopes and declarations can
only accurately be accomplished by parsing the program
before execution

The ReferenceError here technically comes from greeting
= "Howdy" accessing the greeting variable too early, a con-
flict referred to as the Temporal Dead Zone (TDZ).


===> js programms are parsed before execution begins

so they are compiled, right? 

actually, it's possible to parse and interprete results of AST
but unlikely because of inefficient perfomance.  

usually AST is converted(aka compiled) into the most efficient
(binary) representation for the engine to then execute.


LHS = RHS
Target = Source
How are targets determined?

1. clear assignment
```js
students = [...]
```

2. less obvious assignments
```js
for (let student of students) {
//assigns a value to student for each iteration of the loop
```
3. 
```js
getStudentName(73)
//the argument 73 is assigned to the parameter studentID
```
4. 
```js
function getStudentName(studentID) {
//A function declaration is a special case of a target reference
```
An identifier getStudentName is declared (at compile time)
but the = function(studentID) part is also handled at
compilation; the association between getStudentName and
the function is automatically set up at the beginning of the
scope rather than waiting for an = assignment statement to
be executed
This automatic association of function and variable is referred to as “function hoisting”



Sources are:
```js
if (student.id == studentID)
```

getStudentName(73), getStudentName is a source reference (which we hope resolves to a function reference value)


Understanding targetVSsource helps us cover how a variable’s role impacts its lookup (specifically, if the lookup
fails)


page 15 Cheating: Runtime Scope Modifications\
you can craete a runtime scope, not only compiled one

1. use eval('')

```js
function badIdea() {
	eval("var oops = 'Ugh!';");
	console.log(oops);
}
badIdea();
// Ugh!
```
eval(..) modifies the scope of the
badIdea() function at runtime. This is bad for many rea-
sons, including the performance hit of modifying the already
compiled and optimized scope, every time badIdea() runs.


2. umm, anyone heard of `with` keywoard?
```js
var badIdea = { oops: "Ugh!" };
with (badIdea) {
	console.log(oops); // Ugh!
}
```
The global scope was not modified here, but badIdea was
turned into a scope at runtime rather than compile time, and
its property oops becomes a variable in that scope. Again, this
is a terrible idea, for performance and readability reasons


Fortunately, neither of these cheats is available in strict-mode


### lexical scope
the key
idea of “lexical scope” is that it’s controlled entirely by the
placement of functions, blocks, and variable declarations, in
relation to one another.

- compilation doesn’t actually do anything in terms of reserving memory for scopes and variables. None of the program has been executed yet.

- instead, compilation creates a map of all the lexical scopes
that lays out what the program will need while it executes.
You can think of this plan as inserted code for use at runtime,
which defines all the scopes (aka, “lexical environments”) and
registers all the identifiers (variables) for each scope.

- while scopes are identified during compilation, they’re not actually created until runtime, each time a scope needs to run



## Chapter 2: Illustrating Lexical Scope

- Variables are declared in specific scopes, which can
be thought of as colored marbles from matching-color
buckets

- Any variable reference that appears in the scope where
it was declared, or appears in any deeper nested scopes,
will be labeled a marble of that same color—unless an
intervening scope “shadows” the variable declaration;
see “Shadowing” in Chapter 3.

- The determination of colored buckets, and the marbles
they contain, happens during compilation. This infor-
mation is used for variable (marble color) “lookups”
during code execution


### Conversations amount friends

How does JS engine process following program?

```js
var students = [
	{ id: 14, name: "Kyle" },
	{ id: 73, name: "Suzy" },
	{ id: 112, name: "Frank" },
	{ id: 6, name: "Sarah" }
];
```

A reasonable assumption would be that Compiler will produce code for the first statement such as:\
“Allocate memory for a variable,\
label it students\
, then stick a reference to the array into that variable.”

But that’s not the whole story

We should see what's inside `js engine`:
- Engine: does start-to-finish (?as just manager?) compilation + execution of out JS program
- Compiler: handles dirty work = parsing and code-generation
- Scope Manager: collects + maintains a lookup list of all declared variables(identifiers); enforces rules as to how these [variables] are accesible to currently executing code   

Steps Compiler will follow to handle that statement:


So actually the story follows as: 
1. Compiler tokenizes the code
2. Compiler creates AST
3. Long story begins...

1. Compiler encounters `var studnets` 
=> asks Scope Manager to look up a variable named `students` in particaular scope
==> if exists, then compiler ignores declaration
==> if does not exist, then compiler will produce code (at execution time) that asks ScopeManager to create new variable called `students` in that scope

2. Compiler produces code for Engine to later execute = how to handle `students = []` assignment. 
=> Later Enginge will execute this code and again ask ScopeManager if there's a `students` in current scope.
==> if not in current scope, then Engine keeps digging up to Nested Scope
=> when it finds variable, Engine assigns the reference of the [{...},{...},...] array to it

 

-> 2. Later, when it comes to execution of the program, the con-
versation will shift to Engine and Scope Manager














