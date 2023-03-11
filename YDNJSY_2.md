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


Understanding target VS source helps us cover how a variable’s role impacts its' lookup (specifically, if the lookup
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


Shortly speaking `var students = [...]` is processed in 2 steps:
1. Compiler sets up the declaration of the scope variables (it was not previously declared in current scope)
2. While Engine is executing, to process the assignment
part of the statement, Engine asks Scope Manager to look
up the variable, initializes it to undefined so it’s ready
to use, and then assigns the array value to it



Nested Scope 
When it comes time to execute the getStudentName() func-
tion, Engine asks for a Scope Manager instance for that
function’s scope, and it will then proceed to look up the
parameter ( studentID ) to assign the 73 argument value to,
and so on.


- Each scope gets its own Scope Manager instance each time
that scope is executed (one or more times).

- Hoisiting = Each scope automatically has all its identifiers registered at the start of the
scope being executed (this is called “variable hoisting”)

- Look Up = One of the key aspects of lexical scope is that any time an
identifier reference cannot be found in the current scope, the
next outer scope in the nesting is consulted

Look up failure may result in different ways depending on
- whether identifier is called as target or source role
- whether it is strict-mode

code results in a ReferenceError if:
- if variable is source, then unresolved identifier lookup = undeclared variable
- strict-mode + target

```js
var studentName;
typeof studentName; // "undefined"

typeof doesntExist; // "undefined" but should smth like "undeclared" 
```

### Unexpected accidental Global variable

It's legacy behavior for non-strict mode and target variable

```js
function getStudentName() {
	// assignment to an undeclared variable :(
	nextStudent = "Suzy";
}

getStudentName();
console.log(nextStudent);
// "Suzy" -- oops, an accidental-global variable!
```

In strict-mode, the Global Scope Manager would instead have responded:\ 
(Global) Scope Manager: Nope, never heard of it. Sorry, I’ve got to throw a ReferenceError




## Chapter 3: The scope chain

scope chain stands for connections between scopes that are nested within other scopes.

it determines path to access for variables  

it's one-directonal = look up moves upward\outward only



### Runtime `Lookup` is usually conceptual

Suggestion of a `runtime lookup` process works well for
conceptual understanding, but it’s not actually how things
usually work in practice

The color of variable (meta info of origin scope) is actually hard-determined at compilation
stage.
It removes non-essential perfomance trait  
The meta info stored in variable's' entry in the AST


Tho, there may be a reference to variable which is undeclared in any lexicalyy avaialbe scope OF CURRENT FILE. 
if so, the variable scope origin will be uncolored (aka undefined after compilation),
but eventually will be determined at execution process via shared global scope with other files


### Shadowing

Warning: that's bad practice to shadow global variables
```js
var studentName = "Suzy"; // RED SCOPED

function printStudent(studentName) { // BLUE SCOPED `printStudent` FUNCTION + `studentName` function parameter
	studentName = studentName.toUpperCase();
	console.log(studentName);
}

printStudent("Frank"); // FRANK
printStudent(studentName); // SUZY
console.log(studentName); // Suzy
```

The BLUE(2) studentName variable (parameter) shad-
ows the RED(1) studentName . So, the parameter is shadow-
ing the (shadowed) global variable


In this way we force downward scopes
to reference only BLUE(2) scoped studentName if requested  


### Unshadowing trick (only for global declarations)

Warn: Bad practice; confuses the code reader

`var` and `function` declarations expose themselves (with the same name as their lexical identifier) as properties on global object
 
In browser environment `window` is name global object\ 
Bad practice, but still legal
```js
var studentName = "Suzy";

function printStudent(studentName) {
	console.log(studentName);
	console.log(window.studentName);
}

printStudent("Frank");
// "Frank"
// "Suzy"
```


thankfully there are `const` and `let`, which prevent global declarations

```js
var one = 1;
let notOne = 2;
const notTwo = 3;
class notThree {}

console.log(window.one); // 1
console.log(window.notOne); // undefined
console.log(window.notTwo); // undefined
console.log(window.notThree); // undefined
```

proof, trick works only for global
```js
var special = 42;
function lookingFor(special) {
	// The identifier `special` (parameter) in this
	// scope is shadowed inside keepLooking(), and
	// is thus inaccessible from that scope.
	function keepLooking() {
		var special = 3.141592;
		console.log(special);
		console.log(window.special);
	}
	keepLooking();
}
lookingFor(123);
// 3.141592
// 42
```
RED(1) `special` = 42 is shadowed by BLUE(2) `special` = 123
BLUE(2) `special` is shadowed by GREEN(3) `special` = 3.141592 

But yeah, we can stil access special from RED(1) scope via global `window` object


### Illegal shadowing

1. only `let` can shadow `var`(which is ?kinda? on ?global function? object)
2. but `var` cannot shadow `let`
3. `var` can shadow `let` only if there's function boundary 

1. 
```js
function something() {
	var special = "JavaScript";
	{
		let special = 42;
		// totally fine shadowing
		// ..
	}
}
```
2. 
```js
function another() {
	// ..
	{
		let special = "JavaScript";
		{
			var special = "JavaScript";
			// ^^^ Syntax Error
			// ..
		}
	}
}

```
In the another() function, the inner var `special`
declaration is attempting to declare a function-wide special ,
which in and of itself is fine

reason it’s raised as a SyntaxError is because the
var is basically trying to “cross the boundary” of (or hop over)
the let declaration of the same name, which is not allowed.

3. 
That boundary-crossing prohibition effectively stops at each
function boundary, so this variant raises no exception
```js

function another() {
	// ..
	{
		let special = "JavaScript";
		ajax("https://some.url",function callback(){
			// totally fine shadowing
			var special = "JavaScript";
		});
	}
}
```


### 'Function Name' scope

1. Function declaration:
```js
function askQuestion() {
	// ..
}
```
we know, such a function declaration will create an identifier in the enclosing scope (in
this case, the global scope) named askQuestion.


2. the same goes for function expression:
```js
var askQuestion = function(){
	// ..	
};
```
the `askQuestion` variable will be hoisted, but not its' function definition, so yeah, the function will not hoist

3. Named Function expressin = function expression with named function definition

```js
var askQuestion = function ofTheTeacher() {
	console.log(ofTheTeacher);
};

askQuestion();
// function ofTheTeacher()...

console.log(ofTheTeacher);
// ReferenceError: ofTheTeacher is not defined
```

as we can see, `ofTheTeacher` is declared as identifier in the scope of function itself
that's why we are not able to call ofTheTeacher() by it's name in outer scope



3.1 

also it's read-only
tho in non-strict-mode it would slip through with no assignment 

```js
var askQuestion = function ofTheTeacher() {
	"use strict";
	ofTheTeacher = 42;
	// TypeError
	//..
};

askQuestion();
// TypeError

```


4. Arrow functions

they are lexically anonymous = no directly related identifier that references the function.

```js
var askQuestion = () => {
	// ..
};
askQuestion.name; // askQuestion
```


Summirize:
- New Function defined = New scope created
- Scope hierarchy of nested scopes is called `Scope chain`  
- ScopeChain controls variables access(shadowing), oriented upward
- Shadowing of outer scoped variable occurs when variable name is repeated in inner scope



## Chaper 4: Global scope

- What is it
- Why it's useful and relevant to writing JS
- How to access it
- How it differs depending on environment


### Why we gotta understand global 
It's essential to understand global scope\
for using lexical scope to structure programs 

Ways to separate files, which are stitched together in a single runtime context by JS engine:
*in browser environment*
1. import ES module
2. bundler (files' contents wrapped in a single enclosing scope Universal Module = UMD)
```js
function wrappingOuterScope(){
	var moduleOne = (function one(){
		// ..
	})();
	var moduleTwo = (function two(){
		// ..
		function callModuleOne() {
			moduleOne.someMethod();
		}
		// ..
	})();
})();
```
wrappingOuterScope() is a function and
not the full environment global scope, it does act as a sort
of “application-wide scope,” a bucket where all the top-level
identifiers can be stored, though not in the real global scope

3. via `<script>` tags or other dynamic JS resource loading
```js
var moduleOne = (function one(){
	// ..
	}
)();
var moduleTwo = (function two(){
	// ..
	function callModuleOne() {
		moduleOne.someMethod();
		}
		// ..
	}
)();
```
since there is no surrounding function scope, these
moduleOne and moduleTwo declarations are simply dropped
into the global scope


so yeah, global store accounts where application's  code resided during runtime
also global store is place where:
- JS built-ins are exposed: 
	- primitives: undefined , null , Infinity , NaN
	- natives: Date() , Object() , String() , etc.
	- global functions: eval() , parseInt() , etc.
	- namespaces: Math , Atomics , JSON
	- friends of JS: Intl , WebAssembly
- The environment hosting the JS engine exposes its own
built-ins:
	- console
	- DOM's properties and methods
	- timers
	- navigator, WebRTC etc


Node exposed elements: 
	- require()
	- __dirname
	- module
	- URL, etc


### Where Global scope is located?  

depends on environment, actually

1. Browser "Window"

- the most pure JS environment (with least extension of expected behavior)
- is declared within any script tag: 
	- inline script
	- src script tag
	- dynamically created script tag as DOM element (from js probably)

Real behavior is Expected one:
```js
var studentName = "Kyle";
function hello() {
	console.log(`Hello, ${ window.studentName }!`);
}
window.hello();
// Hello, Kyle!
```

#### Tricky shadowing: 
within just the global scope itself, a global object property can be
shadowed by a global variable

```js
window.something = 42;
let something = "Kyle";

console.log(something); // Kyle
console.log(window.something); // 42
```

`let` declaration adds a `something` global variable but not a global object property

want to use global properites? (ugh, you, wierdo)\
so just use ur old-fashioned-legacy-dinosour-australopitek `var` declaration, you dumb***

#### Unexpected Global Extension in browser

1. DOM elements with `id` attr
a DOM element with an id attribute automatically creates a global variable
that references it

consider you have this markup:
```html
<ul id="my-todo-list">
	<li id="first">Write a book</li>
	..
</ul>
```

so your js would work like in awesome reliable way:  (it's legacy browser behavior, actually)
```js
first; // <li id="first">..</li>

window["my-todo-list"]; // <ul id="my-todo-list">..</ul>
```


2. `window` has a pre-defined global `name` property with typeof === 'string'
```js
var name = 42; // `var` redeclaration is ignored  
console.log(name, typeof name); // "42" string
// auto setter converted 42 number to "42" string
```

#### Purest enviroment in wild production

it's WebWorkers!
Web platform extension on top of browser js behavior,which allows a JS file to run in a completely
separate thread (operating system wise) from the thread that’s running the main JS program

- separated from main thread
- does not have access to DOM
- may have `navigator` access 
- not sharing global scope with main JS program


global is referenced by `this`: 
```js
var studentName = "Kyle";
let studentID = 42;

function hello() {
	console.log(`Hello, ${ self.studentName }!`); // because defined globally by `var` declaration
}

self.hello(); // Hello, Kyle! // because defined globally by `function` declaration
self.studentID; // undefined // because defined locally by `let` declaration
```

#### DevTools enviroment

it's adherent js enviroment, because leant in favor to DeveloperExperience:
- certain errors may be relaxed or not displayed
- difference in:
	- global scope behavior
	- hoisting
	- let\const (aka block-scoped declarations) behavior, when used in outermost scope

#### ES Modules (aka ESM)

core feature for first-class support for module pattern in ES6

ESM changes behavior of observably top-level scope in a file:


```js
var studentName = "Kyle";

function hello() {
console.log(`Hello, ${ studentName }!`);
}

hello(); // Hello, Kyle!

export hello; // `export` keywoard used to adjust to ESM format 
```
these `studentName` and `hello` are not global, they are that called `module-wide`





















 






