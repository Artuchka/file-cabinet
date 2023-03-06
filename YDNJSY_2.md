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


page 15 Cheating: Runtime Scope
Modifications










































