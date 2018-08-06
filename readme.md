naturalJS
=========
Naturaljs is a natural language programming extension to javascript. Our goal is to provide a programming language as close as possible to natural language. We will do that by gradually adding natural language features to javascript code and providing a parser that will identify the natural language islands in javascript code and transpile them into javascript.

# Status
At this stage, naturaljs is a proposal. Before we implement it, we want to make sure the project is of interest to other programmers. Implementation will start once this GitHub project reaches **100 stars**. If you see merit in natural language programming and in this project, **please star this project**. The text in this file is not formalized. Formalizing naturalJS syntax and behavior will be part of the implementation process.

# Initial Naturaljs Features
Here are the initial natural language features we intend to implement.

## Inline Argument Notation
The classic c style notation is function style like this:

```javascript
strlen(str);
getElementById('button1').focus();
Node2.parentElement.insertBefore(Node1,Node2);
```

It borrows from the mathematical notation of function. It is not the way most humans communicate. In naturaljs, expression of the same functionality would look like this:

```javascript
length of str
focus on element "button1"
insert Node1 before Node2
```
Notice that arguments are inlined in the flow of the expression.

### Defining natural function
Here are some examples of defining functions in naturaljs:

```javascript
DEFINE length of Str AS return Str.length; END
DEFINE show X AS console.log(X); END
DEFINE move element E to point P AS ... END
```
The syntax elements are:
* Function definition starts with keyword 'DEFINE'.
* Followed by a sequence of one or more function name elements separated by space. A function name element can either be a name token or an argument name.
* function name tokens must start with lower case character and may be followed by a sequence of alphanumeric characters. Special characters such as '$' and '_' are not allowed.
* Function name must not start with a token that is a Javascript keyword. This limitation might be relaxed in the future to allow tokens such as "delete" and "switch" that might be essential for some natural language function expressions.
* Argument names must start with an upper-case character and may be followed by a sequence of alphanumeric characters. Special characters are not allowed.
* Argument names may not be one of the following: IN, A, AN, AS, THE.
* An argument name may not be followed immediately by another argument name. 
* The function name is followed by a function body. Function body is enclosed by an opening keyword "AS" and an ending keyword "END"

The Javascript function name is derived from the natural function name, replacing spaces with '_' symbol and argument names with '$' symbol. The function `move element E to point P` is transpiled to `move_element_$_to_point_$`. This is the function signature. 

### Calling Functions
Natural function can be called anywhere in the code, including Javascript code. The transpiler first identifies the natural function calls and transpiles them into their Javascript counterpart (The traspiling process is described later in this document).
```javascript
function someFunction(SomeStr){
    console.log(length of SomeStr);
}
```
The natural function call is comprised of a sequence of function call elements separated by spaces. A function call element can be a function name token or function argument value. Function argument values can be one of the followings:
* Any variable name that starts with an upper-case character - `Name` but not `name`
* Array literals `[1,3,5]`
* Integer literals `4, -5`
* Floating point literals `.45, -3.1E+12`
* String literals using double quotes only `"hello"` but not `'world'`
* Any expression enclosed in parentheses

Boolean literals and non-capitalized variable name are not allowed to avoid ambiguity.

The sequence of calling function elements must match an existing natural function signature where spaces are replaced by the '_' symbol and argument values are replaced by the '$' symbol. The parser should emit an error if a natural function name is used that was not declared in the file being parsed or in any of the used packages.

## Referencing Context
In natural language, expressions are evaluated in the context of the text. It is common in programming to pass context arguments, that provide the context of the function call. In object-oriented programming, the object is the context of the member function. However, in natural language, context is assumed and not stated explicitly. Passing context objects requires a special mechanism.

Defining natural functions with context is done like this:

```javascript
DEFINE move to next token CONTEXT context AS
    context.currentToken = text.substr(context.currentPosition).match(/s*[\S]+);
    if(context.currentToken !== null){
        move the current position by (length of (context.currentToken));
    }
END
//calling the function
let context = {
    text : "hello world",
    currentPosition : 0,
    currentToken : null
};
move to next token;
show (context.currentToken);//hello
move to next token;
show (context.currentToken);//world
```
This code is transpile as follows:

```javascript
//defining the function
function move_to_next_token(context){...};
//calling the function
move_to_next_token(context);
```
`CONTEXT` keyword must be followed by a comma separated list of identifiers to pass as additional arguments to the function. Identifiers named "AS" are not allowed.

## Using Naturaljs Functions From Other Modules
Using natural functions that are defined in other modules, requires the following syntax:
```javascript
USE './other-module.njs'
USE 'path/to/other/module', 'path/to/another/module';
```
The paths are derived similarly to Javascript modules.

## Using async
The introduction of async/await to Javascript made writing asyncrhonous code much easier. It is possible to define natural functions as `async` by preceding the function definition with the keyword.
```
async DEFINE get url Url AS ... END
```
The keyword `await` can be used within the natural function body.

# Transpiling NaturalJS files
Transpiling naturalJS files is the process of converting a naturalJS format file into a valid Javascript file. Transpiling take the following steps:
1. Derive the available natural functions from natural function definitions in the processed file
1. Add to the pool of available natural functions, all functions defined in USE modules.
1. Replace natural function definitions with Javascript code. 
1. Replace all USE statements with Javascript import
1. Parse the file identifying all possible natural function calls and replacing them with their Javascript parallels

## Identifying natural function calls
Natural function calls are evaluated as types of expressions. They get precedence over any other Javascript expressions, i.e. whenever a sequence of characters can either be parsed as a natural function call or any other Javascript expression, it will be parsed as a natural function call. It is important to note that a legal natural function call is a call that was defined in the parsed file or included modules using the `USE` keyword. 

# Future Features
Here is a list of the next-in-line features to implement. They might change significantly before implementation.

## Argument Phrases
Typically actions may have many optional arguments. It is impossible to define a function for each permutation and the natural syntax does not allow for undefined values. We suggest defining optional argument phrases that are optional parts of the natural function call. Each argument phrase is preceded by a comma.
```javascript
DEFINE create a meeting, with Person, at Time, for Duration, on Date, in Location AS ... END

//calling the natural function
create a meeting  with "John Doe"  at "5 PM" for "2 hours" in "my office"
create a meeting with "John Doe" at "4 pm"//not all argument phrases are present
```

## Context References
While, using `CONTEXT` keyword in function definition offers a solution to passing context to functions, its use is a bit clumsy. Developer needs to set a context object containing the different values that might change during function processing. It would be useful to have a mechanism for referencing context variables defined from the calling function. In this suggested feature, context variables can be referenced using the `the` keyword as follows:

```javascript
DEFINE move to next token CONTEXT the text, the currentPosition, the currentToken AS
    the currentToken = text.substr(currentPosition).match(/s*[\S]+);
    if(the currentToken !== null){
        move the current position by (length of (the currentToken));
    }
END
//calling the function
the text = "hello world";
the currentPosition = 0;
the currentToken = null;
move to next token;
show (token);//hello
move to next token;
show (token);//world
```

## Handling Types and Tags
Types are an important part of how people think of concepts. Moving a cursor on the screen is different from moving a spreadsheet row. Typing can be used to correctly associate the natural function call with the relevant natural function, similarly to typed languages such as C++. The syntax of typed arguments is as follows:

```javascript
DEFINE move A row Row to A row number Y AS ... END
```

Classes or variables can be typed using the TAG keyword
```
TAG class Row AS row
TAG name AS string
```
TAG statements are processed at compile time only.

An alternative syntax for typing functions using capitalized class names would be as follows:
```
class Row{...};
class RowNumber{...};
DEFINE move A Row to A RowNumber AS 
    Row.rowNumber = the RowNumber; 
END 
```

Tagged entities can then be referenced using `the` keyword:
```
TAG X as name;
console.log(the name);
```

## Handling Execution Flow
We would like to provide a natural language notation for defining execution flow such as if ...then, while, for ... etc.

## Event Handlers
Event Emitters are an important part of the way Javascript is used. We would like to provide a simple, natural language way of defining event handlers

# Benefits of Natural Language Programming
- Easier to understand - human beings need to make an effort to understand computer language while understanding natural language is, well, more natural.
- Easier to maintain - the challenge of maintaining other programmers code or even one's own code written a while ago is not trivial. Having the code easier to understand makes it easier to maintain.
- Easier to integrate - integrating third-party libraries presents the challenge of learning new APIs and understanding the correct way of using them. Having the API in natural language makes it a lot easier to learn new APIs
- Easier to communicate to non-programmers - with current programming languages there is an iron wall between programmer's world and user world (users, product managers, managers). Having near natural language programming allows the user side understand the logic implemented in programmers code.

# Contact Us
We would love to receive your feedback. Send us an email to elshor@gmail.com or DM us at @elshor. For updates, follow the twitter account @naturaljslang. If you want to raise an issue that may open a discussion, send us an issue report on the GitHub project. Once again, if you see merit in this project, please **star it**. Once the project reaches **100 starts**, we will start implementing it.
