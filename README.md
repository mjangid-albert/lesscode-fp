# Motivation

![Lego Kids](lego-kid.jpeg) Going back in time, when Alan Turing built 
[Turing machine](https://en.wikipedia.org/wiki/Turing_machine) to perform computation, Alanzo Chruch designed 
[Lambda Calculus](https://en.wikipedia.org/wiki/Lambda_calculus) to mathematically compute anything by composing lambda functions (world's smallest programming langauage). 
Soon world got filled with programming languages in both the paradigms and many languages start borrowing each other's concept (sounds familiar?) to only become bloated and un-manageable. 

Fast forward 1977...John Bakus's paper, ["Can programming be liberated from von nuemann style?"](https://github.com/van001/lesscode/blob/master/can-programming-be-liberated.pdf) put it very rightly.

The above article influenced me to dive deep into programming paradigm concepts, specially functional programming and really understand it from the work of those 2 greats.
So begin the journey of [Functional Thinking](https://github.com/van001/lesscode), with few very simple goals in mind - 

- Seek deep understanding of motivation and concepts behind the functional programming, 
without being caught in the programming lanugage nuances, syntax or even nomenclature. 

- Come up with a language agnostic list of real-world functions (library), which can be implemented in any multi-paradigm language that supports functional programming. 

# Overview

This repo is about my learning of functional programming and coming up with a language agnostic library of functions, which
can be applied to solve many real-world problems.

In functional programming language :

- you either write pure functions (no [side-effects](https://en.wikipedia.org/wiki/Side_effect_(computer_science))) or functions with side-effects ( writing to logs / screen, reading from file, making a database call , etc). 

- you compose functions to produce a desired outcome.

- you treat functions as a 1st class citizen and pass / return a function to / from another function.


[lesscode-fp](https://github.com/van001/lesscode-fp/blob/master/lesscode/src/index.js), is a language agnostic javascript functional library, which define functions to manipulate  String, List & Map / Object categories.
You can implement these functions in any multi-paradigm language that supports functional programming.
It's built using the following functional programming principles :

### Pure functions ### 
Pure functions have no side-effects, are time independent & maintain [referential transparency](https://en.wikipedia.org/wiki/Referential_transparency), which means, for a given input, you can replace the function with it's output, anytime. 
Think of them as a mathematical function, which for a given input will awlays return the same output.
This also means, a pure function can easily cache ([memoize](https://en.wikipedia.org/wiki/Memoization)) the value it returns after the execution. This is huge, now that memory / storage is cheap and computation is expensive.

All the functions in lesscode library are pure, unless they are not. Pure function name starts with lowercase,  
impure with uppercase letters.

```
// appends String to another String and returns a new String
const sappend = str1 => str2 => str1 + str2 

// checks if a value is null or not?
const eqNull = val => (val == null || undefined) ? true : false 

// Find the max of 2 munmers
const max = a => b => Math.max(a,b)
```

### Immutable ###
In functional programming you do not mutate data, instead you compute a new. Also, most FP languages support lazy evaluation,
 which means your function is executed only when it's needed. Isn't it sweet...

 Lesscode adheres to the principle, unless it's a Monad ([see below]https://github.com/van001/lesscode-fp#Monad)

```
// appends String to another String and returns a new String
const sappend = str1 => str2 => str1 + str2 
```

### Single input / output ### 
The origin of functional programming, ***[lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus)***, only allowed single input/ ouput. While it may not seem practical, currying ([see below](https://github.com/van001/lesscode-fp#Currying))  allows you to do so. Functional programming treats functions as a 1st class citizen, so you can pass a function as a parameter and return a function as a result.

Lesscode functions accept single parameter; multi-paramter functions are curried.

```
// reverses a List, point-free (see [Data-Last](https://github.com/van001/lesscode-fp#Data-Last)).
// same as : lreverse =>  lst => lfold(([])(lappend)(lst)
const lreverse => lfold(([])(lappend)

// sappend takes a String as 1st parameter and using currying , 
// returns a function that takes another String as a parameter
// sappend = str1 => ( (str2) => str1 + str2 )
const sappend = str1 => str2 => str1 + str2 
```

### Currying ### 
[Currying](https://en.wikipedia.org/wiki/Currying) (f(a, b) => f(a)(b)), allows function with multiple parameters to be written as sequence of functions that each take a single parameter. 
Currying allows you to partially apply other options (initializion) and dependencies (injection) on multi-parameter functions.
Currying also allows you to create your own DSL (domain specific language) by partially applying many generic functions and creating a new domain specfic one.

Every function in lesscode library is curried and [point-free](https://en.wikipedia.org/wiki/Tacit_programming) (where possible).

```
// it splits a String into List based on pattern matching.
// s2List is a curried function. 
const s2List = ptrn => async str => str.split(ptrn)

// suppose your domain breaks Sting into List based on space.
// partially apply s2List, with space and create a new function.
const space2List = s2List(space)

// will break String into List on every whitespace.
space2List('This is cool') // ['This','is','cool']
```

### Data-Last ###
Functions that take more than one parameter,  should accept data as a last parameter. If you are coming from imperative programming paradigm, this might be new to you.
In imperative programming you pass data 1st & options (default values - ring the bell?) later.  Or in the case of object oriented programming (still imperative), 
you manipulate the encapsulated data with function(s), which take additional options. 

In FP, data and functions are separate, hence you build library of functions to work with your data.  

Data last principle makes function [composition](https://github.com/van001/lesscode-fp#Composition), easier (builder pattern). Also, it allows for a point-free style of composition.

```
// l2String converts, List to String. 
// it takes List (which it will convert to String) as a last parameter.
const l2String = sep => lst => lst.join(sep)
```

### Composition ### 
Crux of any programming paradigm is composition. Composition, allows you to re-use the code (less code ;-))

In functional programming there is no assignment, you just compose functions to produce more specific functions/ solutions. Haskell
has infix composition operator like '.' (for pure function) and '>>= / >>' (for [monadic composition](https://github.com/van001/lesscode-fp#Monad)). Other multi-paradigm languages like javascript, java etc do not have any such 
operators nor they support infix styling.

Lesscode library provide :
-  **$(...)** for pure function composition.
-  **$M(...)** for [monadic](https://github.com/van001/lesscode-fp#Monad) composition. 

```
// coconut machine will take List of coconuts, then slice the top & put a straw.
// same as  : coconutMachine = DropStraw . SliceFromTop 
const coconutMachine = $(DropStraw ,SliceFromTop)

// to copy the content of one file to another (point-free)
const FileCopy = to => $M(FileWrite(utf8)(to), FileRead(utf8))
```
***'$' has a very small foot-print and can be easily spotted to show the composition.***

### Fewer Categories ### 
Unlike, Object Oriented programming, where every class is a new category, functional programming benefits from fewer categories and coming up with domain specific abstractions to solve them generically.

Lesscode library provide funtions to manipulate / transform the following categories : String, List, Map / Object (non mutable). These categories are quiet popular in many programming languages. List,  allows easier work distribution; Map / Object,  allows faster retrieval; String, is a preffered way of storing data.

Lesscode libraray prefixes the name of category for all the category specific functions. This design is by choice.

```
// slices String at the specified position
const sslice  = start => end => str => str.slice(start,end)

// converts String to List, by breaking it with supplied pattern
const s2List = ptrn => str => str.split(ptrn)
```

### Functor ###
Since functional programming is already about abstraction ( what to do, not, how to do), the proliferation of  [Category Theory](https://en.wikipedia.org/wiki/Category_theory) took it to the next level. Since many of the real-world problems are about transformation of one Group / Set of data into another, it was but natural.

In category theory, Functor is a structure preserving transformation from one category to another. Since functor always produces another functor, composition is trivial.

For e.g : Transforming List of lowercase text to List of uppercase. Transforming List of Strings to List of Hash of Strings.

Lesscode library provide 'lmap' function for the List functor. Using lmap you can transform one List into another. **lmap**, essentially lifts each item from the List, hands it to the function for transformation, then puts it back into a new List.

```
// Map over List to tranform into another List, while preserving the structure (Functor - borrowed from category theory). 
const lmap = func => lst => lst.map(func)

// convert the List of Strings to uppercase
const str = ['neelesh' , 'vaikhary']
const strUpper = lmap(supper)(str)
print(strUpper) // ['NEELESH' , 'VAIKHARY']

// Applying the cocununt machine logic to slice & straw, list of coconuts
const readyCoconut = lmap(coconutMachine)['full coconut', 'full coconut']
// or                lmap($(DropStraw ,SliceFromTop))['full coconut', 'full coconut'] 

print(readyCoconut) // ['strawed sliced coconut', 'strawed sliced coconut']
```

### Monad (endofunctor) ### 
Uptill now we only talked about pure functions but real-world functions have side-effects (hidden data, wrapped context), are in-predictable (fail or different result), or even need sequential execution (one after another). 

Monads, let you do all that. Monads let you write functions that can separate concerns (decorator pattern), allow side effect (IO), introduce sequence. 

Think of monad as a function that wraps all the side-effects and make it a pure and allow composition by passing the value from one to another. In category
theroy Monads jargon, Monads are just a [Monoid](https://en.wikipedia.org/wiki/Monoid_(category_theory)) (Category with single element that can be joined). 

As Saunders Mac Lane said, ***Monad is just a monoid in the category of endofunctors***

Lesscode  provide some built-in Monads using javascript async (promise). It also provide a function **M(...)** to convery any pure function to Monad.

**$M(...)** supports [monadic composition](https://github.com/van001/lesscode-fp#Composition) (instead of a >==> operator or kliesli arrow , as it's called in category theory)

```
// Read content of a file. 
const FileRead = option =>  name => fs.promises.readFile(name, option);

// mondaic composition : copy the content of one file to another (point-free)
const FileCopy = to => $M(FileWrite(utf8)(to), FileRead(utf8))
```

### Applicative ###
Applicative is also about functions with side-effects, but the name doesn't originate form Category therory, instead it was coined to solve something 
between functor and monad. When you have to write a function with side-effects that accept more than one paramter you use applicative functor. 
Normal functor composition only works with functions with single parameter. Also applicative allows for the concurrent execution of functions with side-effects
without any dependecies.

Lesscode implements applicative using **$A(...)** and accept List as a parameter. Functions is appplicative composition should accept the same number of parameters.
The result is again a List. 

```
```

# Examples

## Algorithms
Coming soon...

## Real-world 

### Concurrent ###

Doing bunch of things concurrently, waiting for the result and then doing something else. 
Also, tolerating failures instead of aborting on error (if a file download fails it is ok, just write the error).

**[Image Download](https://github.com/van001/lesscode-fp/tree/master/lesscode/examples/image-download)**

Download list of images specified in a file and write metadata(url, size, hash) to the specified output file.

input.txt
```
http://i.imgur.com/c2z0yhtx.jpg
http://i.imgur.com/KxyEGOn.jpg
http://i.imgur.com/vPae8qL.jpg
http://i.imgur.com/cz0yhtx.jpg
```

output.txt
```
http://i.imgur.com/c2z0yhtx.jpg 0  Error%3A%20Request%20failed%20with%20status%20code%20404
http://i.imgur.com/KxyEGOn.jpg 470489 cc042826ed386d4247aca63cb7aff54b37acb1f89ce8f4549ac96a9e8683360c
http://i.imgur.com/vPae8qL.jpg 69085 bde0a62bb35dffe66cebf6cfd9ca3841ca1419fb323281bd67c86145f2173207
http://i.imgur.com/cz0yhtx.jpg 2464218 ed9f8c1e95d58e02fcf576f64ec064a64bc628852bc3b298cda15f3e47dfe251
```

```
// Lesscode-fp
const {
    $M, $, hint, trace, print, hash, lmap, Wait, mget, exit, mgettwo,
    linebreak, utf8, newline,
    l2String, s2List,
    FileRead, FileWrite,
    HttpGET } = require('lesscode-fp')

const inputFile = process.argv[2]
const outputFile = process.argv[3]

const LogData = name => async data => `${name} ${mgettwo('headers')('content-length')} ${hash('sha256')(mget('data'))}`
const LogErorr = name => async err => `${name} 0  ${escape(err)}`

// processFile :: String -> String
const ProcessURL = name => $M(LogData(name), HttpGET)(name).catch(LogErorr(name))

// ProcessContent :: String -> String
const ProcessContent = $M(l2String(newline), Wait, lmap(ProcessURL), s2List(linebreak))

// Main pipeline.
$M(FileWrite(utf8)(outputFile), ProcessContent, FileRead(utf8))(inputFile).catch(print)
```
### Stream ###

Sometimes, input is a **stream** or needs to be handled in chunk.

**[File Streaming](https://github.com/van001/lesscode-fp/tree/master/lesscode/examples/file-streaming)**

Streams content of a text file, converts to uppercase then write back to another stream (output file).



input.txt (the real file may be huge...)
```
this text is all lowercase. please turn it into to uppercase.
```

output.txt
```
THIS TEXT IS ALL LOWERCASE. PLEASE TURN IT INTO TO UPPERCASE.
```

```
const { $M, print, utf8, FileStreamIn, FileStreamOut} = require('lesscode-fp')

const Uppercase = async str => str.toUpperCase()
const SaveFile = name => FileStreamOut(utf8)(name)

// Process Stream
const ProcessStream = $M(SaveFile(process.argv[3]),Uppercase)

// Pipeline
FileStreamIn(utf8)( ProcessStream )(process.argv[2]).catch(print)
```



