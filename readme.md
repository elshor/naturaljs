naturaljs
======================================================
Naturaljs is a natural language programming addition to javascript. Our goal is to provide a programming language as close as possible to natural language. Our method will be gradually adding natural language features to javascript code and providing a parser that will identify the natural language islands in javascript code and transpile them into javascript. Unlike typical programming language parsers that are deterministic and only have a single possible parse, the natural language parser is a non-deterministic, best-effort parser, trying to provide a transpiled code closest as possible to the programmers intentions (read [here][Ambiguity] about handling ambiguity)

# Status
At this stage, naturaljs is a proposal. Before we implement it, we want to make sure the project is of interest to other programmers. Implementation will start once this GitHub project reaches **100 stars**. If you see merit in natural language programming and in this project, **please star this project**.

# Initial Naturaljs Features
Here are the initial natural language features we intend to implement:

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

Naturaljs functions  are defined like this:
```javascript
nfunction length of Str {return Str.length;}
nfunction show X {console.log(X);}
```
The definition starts with the keyword `nfunction`, followed by a sequence of lowercase expression tokens and capitalized argument names.

Calling the function would look like this:
```javascript
show length of "hello world"//11
```
The parser will transpile it to the following javascript code:
```javascript
show_$(length_of_$("hello world"));
```
Inlined arguments can be identifiers, natural expressions or any javascript expression or naturaljs expressions enclosed in parentheses.

As you can see, the parsing of the expression can be ambiguous. However, it is not too complicated to create a parser that will generate the above code.

## Referencing Context
In natural language, expressions are evaluated in the context of the text. It is common in programming to pass context arguments, that provide the context of the function call. In object-oriented programming, the object is the context of the member function. However, in natural language, context is assumed and not stated explicitly. Defining naturaljs functions with context would look like this:

```javascript
nfunction move to next token CONTEXT the text, the current position,the current token{
    the current token = the text.substr(the current position).match(/s*[\S]+);
    if(the current token !== null){
        move the current position by length of the token;
    }
}
//calling the function
the text = "hello world";
the current position = 0;
the current token = null;
move to next token;
show the token;//hello
move to next token;
show the token;//world
```

We are using here a mix of natural expressions and javascript expressions. Of course, the javascript functions and operators can be replaced by [natural functions](inline argument notation).

Context arguments must start with the keyword `the`. They are called by reference, not value. When context arguments reference primitive values, they can be implemented as a stand-for object that has a valueOf function evaluating to the primitive value and a set function for setting its value.

## Managing asynchronous calls
ECMAScript 2017 made async programming a lot easier with `await`/`async`. We would expect natural language programming to hide the technical details of handling asynchronicity. When a function is defined as `async`, arguments and function calls within the function are preceded with the `await` keyword.
```javascript
aysnc nfunction show X{
    await console.log(await x);
}
```
of course, a smart parser would recognize that `console.log` is not an asynchronous function and will *not* precede `console.log`  with `await`

## Using Natural Functions From Other Modules
Using natural functions that are defined in other modules, requires the following syntax:
```javascript
use 'path/to/other/module';
console.log(this function is defined in another module);
```
# Future Features
Here is a list of next features to implement:
## Argument Phrases
Typically actions may have many optional arguments. It is impossible to define a function for each permutation and the natural syntax does not allow for null values. We suggest defining optional argument phrases that are optional parts of the natural function call. Each argument phrase is preceded by a comma.
```javascript
nfunction create a meeting, with Person, at Time, for Duration, on Date, in Location{...}
//calling the natural function
create a meeting with "John Doe" at "5 PM" for "2 hours" in "my office"
create a meeting with "John Doe" at "4 pm"//not all argument phrases are present
```
## Handling types
Types are an important part of how people think of concepts. Moving a cursor on the screen is a different from moving a spreadsheet row. We would expect natural functions to be typed. The syntax would probably look something like this:
```javascript
nfunction move [a row Row] to [a row number Y]{...}
```

## Handling Execution Flow
Provide a more natural way of managing execution flow, if ...then, while, for etc.

# Design Considerations

## Ambiguity
As any programmer knows, computer code must be unambiguous. It should have only one interpretation. This contradicts the notion of natural language parsing which is inherently ambiguous. There are several ways to handle this issue.

First, it is possible to write unambiguous code using parentheses. The unambiguous version of the above example of calling functions would look like this:
```javascript
show (length of "hello world");

```

Second, when the programmer writes a code that may be interpreted in several ways, the parser can warn the programmer that the expression is ambiguous. The parser can also show the possible parses in unambiguous notation using parentheses, so the programmer can choose the correct parse.

## Comprehensibility
We believe that complete natural language programming is too complex to handle in a single go. As stated, we prefer that gradual approach of incrementally adding natural language functionality to an existing programming language, javascript in our case. The optimal programming style would be layered:
* lower level - javascript functions
* medium level - natural functions defined with javascript body
* high level - natural functions defined where the body of the function is defined using naturaljs.

The high-level functions are expected to be written by programmers, but comprehensible to non-programmers such as product manager and business owners.

Another aspect of natural language programming is hiding the technical design details of programming, in effect, making programming closer to requirement writing. Achieving that requires a lot of work that is not related in any way to natural language processing. We do think, tackling natural language programming should consider these aspects of programming. Hiding asynchronicity is one of them but many other aspects are yet to be dealt with.


# Benefits of Natural Language Programming
- Easier to understand - human beings need to make an effort to understand computer language while understanding natural language is, well, more natural.
- Easier to maintain - the challenge of maintaining other programmers code or even one's own code written a while ago is not trivial. Having the code easier to understand makes it easier to maintain.
- Easier to integrate - integrating third-party libraries presents the challenge of learning new APIs and understanding the correct way of using them. Having the API in natural language makes it a lot easier to learn new APIs
- Easier to communicate to non-programmers - with current programming languages there is an iron wall between programmer's world and user world (users, product managers, managers). Having near natural language programming allows the user side understand the logic implemented in programmers code.

# Contact Us
We would love to receive your feedback. Send us an email to elshor@gmail.com or DM us at @naturaljs. For updates, follow the twitter account @naturaljs. If you want to raise an issue that may open a discussion, send us an issue report on the GitHub project. Once again, if you see merit in this project, please star it. Once this git project reaches 100 starts, we will start with implementation.

