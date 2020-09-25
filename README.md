# Overview
![Lego Kids](lego-kid.jpeg) Based on my earlier work on [Functional Thinking](https://github.com/van001/lesscode), I propose the following [functions](https://github.com/van001/lesscode-fp/blob/master/lesscode/src/index.js) (subject to change). I will be using these functions and the FP principles to build other/ future projects. So stay tuned...

In pure functional programming languages, you are either writing functions with no side effects (pure functions) or functions with 
side effects (Monads). 

- **Pure functions** : have no side-effects, are time independent & have referential integrity, which means you can replace the function with the value it produces, anytime.

- **Monads** : functions with side-effects, let you write functions that can separate concerns (decorator pattern), allow side effect (IO), introduce sequence (one after another).

- **Fewer categories** :  e.g list, tuple, map (non mutable). 

- **Immutable** : data is immutable. 

- **Single input/ output** : the origin of functional programming, ***lambda calculus***, only allowed single input/ ouput. While it may not be always practical, try to adhere by it as much.

- **Data Last** : functions that take more than one parameter,  should accept data as the last parameter.

- **Currying** : function that takes more than one parameter, curry  them (f(a, b) => f(a) -> f(b)). 
Currying allows you to partially apply other options (initialize) and dependencies (injection) that you might need for multi parameter functions.
Currying also allows you to create your own DSL (domain specifi language) by partially applying many generic functions and creating a new domain specfic one.

- **Composition** : in FP, since there is no assignment you just compose functions to produce more specific functions/ solutions.
We will be using $ / $M for pure functions / monadic composition (see [examples](https://github.com/van001/lesscode-fp#examples) below)

# Features
- Fewer category / data structures - string, list/tuple n map (read only).
- Built using point free style n currying.
- Composable function ($(...)/SM(...)) for both pure (no side effect) and monads (side effect).
- Functions to manipulate a given category.
- Functions to expand, collapse a given category. 
- Functions to transform one category to another.
- Built-in Monads to handle side-effects (http, file etc)

# Examples

## Algorithms
Coming soon...

## Real-world 
**[Image Download](https://github.com/van001/lesscode-fp/tree/master/lesscode/examples/image-download)**

Download list of images specified in a file and write metadata(url, size, hash) to the specified output file.

**Parallel** : doing bunch of things in parallel, waiting for the result and then doing something else. 
Also tolerating the failures instead of aborting on any error (if a file download fails it is ok, just write the error).

Main pipeline : read bottom to top, right to left
```
                    
File Write <output>  <<=      // Write to file
List 2 String        <<=     // Convert List to String
Wait                 <<=     // Wait till everything is done.
Map (sub-pipeline)   <<=     // Parallelly, transform to List containing results.
String 2 List        <<=     // Convert to List.
File Read <input>            // Read the file
```
**'<<='**  indicate bind / join (feed output of one monad to another)
```

// Lesscode-fp
const { 
    $, $M, Hint, Trace, print, hash, Lmap, Wait, mget, exit, mgettwo, 
    linebreak, utf8, newline,  
    L2String, S2List, 
    FileRead, FileWrite,
    HttpGET } = require('lesscode-fp')


const inputFile = process.argv[2]
const outputFile = process.argv[3]

// processFile :: String -> String
const processURL = name => {
    const computeHash = $(hash('sha256'), mget('data'))
    const contentLen = mgettwo('headers')('content-length')
    const LogData = name => async data => `${name} ${contentLen(data)} ${computeHash(data)}`
    const LogErorr = name =>  async err => `${name} 0  ${escape(err)}`
    return $M(
        Trace(`Extracted metadata...............`), LogData(name),      // Success
        Hint(`Downloaded ${name}................`), HttpGET)(name)
        .catch($(Trace(`[Fail] : ${name}........`), LogErorr(name)))    // Failure
}

// Pipeline
$M(
    Trace('Write to output file................'), FileWrite(utf8)(outputFile), 
    Trace('Convert List 2 String...............'), L2String(newline), 
    Trace('Wait................................'), Wait, 
    Trace('Processed URLs......................'), Lmap(processURL), 
    Trace('Converted to List...................'), S2List(linebreak), 
    Trace('Read input file.....................'), FileRead(utf8))(inputFile)
.catch($(exit, print))

```

**[File Streaming](https://github.com/van001/lesscode-fp/tree/master/lesscode/examples/file-streaming)**

Streams content of a text file, converts to uppercase then write back to another stream (output file).

**Streaming** : sometimes, input is a **stream** or too big / time consuming to handle in parallel.
```
 File Stream In ( File Stream Out <<= 2 UpperCase ) <input>
```



