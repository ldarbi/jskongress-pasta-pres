class: center, middle, title-page

# pasta
### pretty (and) accurate stack trace analysis
<img src="images/pasta_128.png"/>

---

# agenda

1. problem statement
2. pasta
3. @bloomberg/pasta-sourcemaps

---
class: center, middle
# 1
# problem statement

---

# production code !== my code

my code

```javascript
function orzo() 
{
    spaghetti();
}

function spaghetti() 
{
    penne();
}

function penne() 
{
    throw Error();
}
orzo();
```

--

production code

```javascript
function n(){o()}function o(){r()}function r(){throw Error()}n();
//# sourceMappingURL=index.js.map
```
---

# there is a crash in production

.pull-left[
```javascript
//**compiled** output

Error
    at r (out.js:1:116)
    at o (out.js:1:93)
    at n (out.js:1:76)
```
]

--

.pull-right[
```javascript
//**original** output

Error
    at penne (index.js:13:11)
    at spaghetti (index.js:8:5)
    at orzo (index.js:3:5)
```    
]

---

# source maps to the rescue

```json
{
    "version" : 3,
    "file": "out.js",
    "sources": ["index.js"],
    "names": 
        ["orzo","spaghetti","penne","Error"],
    "mappings": 
        "<mappings from generated file locations back to original locations>"
}
```

---

# source maps to the rescue

```json
{
    "version" : 3,
    "file": "out.js",
    "sources": ["index.js"],
    "names": 
        ["orzo","spaghetti","penne","Error"],
*   "mappings": 
*       "<mappings from generated file locations back to original locations>"
}
```

--

`mappings` maps *generated* file locations back to:
- *original* file locations
- call site symbol in *original* file

---

# source maps to the rescue

```json
{
    "version" : 3,
    "file": "out.js",
    "sources": ["index.js"],
    "names": 
        ["orzo","spaghetti","penne","Error"],
*   "mappings": 
*       "AAAA,SAASA,IAELC,IAGJ,SAASA,IAELC,IAGJ,SAASA,IAEL,MAAMC,QAEVH"
}
```

`mappings` maps *generated* file locations back to:
- *original* file locations
- call site symbol in *original* file

--

.pull-left[
from
```javascript
Error
    at r (`out.js:1:116`)
    at o (`out.js:1:93`)
    at n (`out.js:1:76`)
```
]

--

.pull-right[
to
```javascript
Error
    at r (`index.js:13:11`)
    at o (`index.js:8:5`)
    at n (`index.js:3:5`)
```
]

---
class: center, middle
# 2
# pasta 

---
# pastafied source maps

```json
{
    "version" : 3,
    "file": "out.js",
    "sources": ["index.js"],
    "names":
        ["orzo","spaghetti","penne","Error"],
    "mappings": 
        "AAAA,SAASA,IAELC,IAGJ,SAASA,IAELC,IAGJ,SAASA,IAEL,MAAMC,QAEVH",
*   "x_com_bloomberg_sourcesFunctionMappings": [
*       "<function name mappings for sources[0]>"
*   ],
}
```
--

- for each source in `sources`, a list of mappings of function names
  - function name index in `names`
  - start line
  - start column
  - end line
  - end column

---
# pastafied source maps

```json
{
    "version" : 3,
    "file": "out.js",
    "sources": ["index.js"],
*   "names":
*       ["orzo","spaghetti","penne","Error"],
    "mappings": 
        "AAAA,SAASA,IAELC,IAGJ,SAASA,IAELC,IAGJ,SAASA,IAEL,MAAMC,QAEVH",
    "x_com_bloomberg_sourcesFunctionMappings": [
        "<function name mappings for sources[0]>"
    ],
}
```

- for each source in `sources`, a list of mappings of function names
  - function name index in `names`
  - start line
  - start column
  - end line
  - end column
- supplementary function names in `names`

---

# example

.pull-left[
```javascript
// barilla.js
//..
function penne() {
   //..
   function orzo() {
      //..
   }
   orzo();
   //..
}
```
]

.pull-right[
```javascript
// muellers.js
//..
function fusilli() {
   //..
}
```
]

.pull-bottom[
vanilla source map

```json
{
    "sources": ["barilla.js", "muellers.js"],
    "names": ["penne","orzo","fusilli"],
    "mappings": "AAmBA,SAASA,IAEL,SAASC,KAGTA,ICtBJ,SAASC,KAIRA",
    ...
}
```
]

---
# example

```javascript
1.    // barilla.js
2.    //..
..    //..
20.   function penne() {
21.       //..
22.       function orzo() {
23.           //..
24.       }
25.       orzo();
..        //..
30.   }
31.   //..
```

---
# example

```javascript
1.    // barilla.js
2.    //..
..    //..
*20.   function penne() {
*21.       //..
*22.       function orzo() {
*23.           //..
*24.       }
*25.       orzo();
*..        //.. 
*30.   }
31.   //..
```

logical encoding

```json
{
    "sources": ["barilla.js", "muellers.js"],
    "names": ["penne","orzo","fusilli"],
    "x_com_bloomberg_sourcesFunctionMappings": [
        [
*           [0,19,0,29,0], // barilla.js (20:1)...(30:1)   => "penne"
            [1,21,4,23,4]  // barilla.js (22:5)...(24:5)   => "orzo"
        ], ...
    ]
}
```

---
# example

```javascript
1.    // barilla.js
2.    //..
..    //..
20.   function penne() {
21.       //..
*22.       function orzo() {
*23.           //..
*24.       }
25.       orzo();
..       //..  
30.   }
31.   //..

```
logical encoding

```json
{
    "sources": ["barilla.js", "muellers.js"],
    "names": ["penne","orzo","fusilli"],
    "x_com_bloomberg_sourcesFunctionMappings": [
        [
            [0,19,0,29,0], // barilla.js (20:1)...(30:1)   => "penne"
*           [1,21,4,23,4]  // barilla.js (22:5)...(24:5)   => "orzo"
        ], ...
    ]
}
```
---

# logical encoding (relative)

- name index relative to previous name index


- .green[start] line relative to **previous** .blue[end] line
- .blue[end] line relative to **current** .green[start]  line


- .green[start]  column relative to **previous** .green[start]  column
- .blue[end] column relative to **previous**  .blue[end] column

---

# logical encoding (relative)
from
```json
{
    "names": ["penne","orzo","fusilli"],
    "x_com_bloomberg_sourcesFunctionMappings": [
        [
            [0,19,0,29,0], // barilla.js (20:1)...(30:1)   => "penne"
*           [1,21,4,23,4]  // barilla.js (22:5)...(24:5)   => "orzo"
        ], ...
    ]
}
```
--

to

```json
{
    "names": ["penne","orzo","fusilli"],
    "x_com_bloomberg_sourcesFunctionMappings": [
        [
            [0,19,0,29,0], // barilla.js (20:1)...(30:1)   => "penne"
*           [1,-8,4,2,4]   // barilla.js (22:5)...(24:5)   => "orzo"
        ], ...
    ]
}
```

---

# VLQ base 64 encoding
from

```json
```json
{
    "names": ["penne","orzo","fusilli"],
    "x_com_bloomberg_sourcesFunctionMappings": [
        [
*           [0,19,0,29,0], // barilla.js (20:1)...(30:1)   => "penne"
*           [1,-8,4,2,4]   // barilla.js (22:5)...(24:5)   => "orzo"
        ], ...
    ]
}
```
--

to 

```json
{
    ...
    "names": ["penne","orzo","fusilli"],
    "x_com_bloomberg_sourcesFunctionMappings": [
*       "AmBA6BA,CRIEI",
        ...
    ]
}
```
---

# about base 64 VLQ

<table>
    <thead>
        <tr>
            <th>B5</th>
            <th>B4</th>
            <th>B3</th>
            <th>B2</th>
            <th>B1</th>
            <th>B0</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>C</td>
            <td colspan=4>value</td>
            <td>S</td>
        </tr>
    </tbody>
</table>

---

# about base 64 VLQ

<table>
    <thead>
        <tr>
            <th>B5</th>
            <th>B4</th>
            <th>B3</th>
            <th>B2</th>
            <th>B1</th>
            <th>B0</th>
            <th>&nbsp;&nbsp;&nbsp;&nbsp;</th>
            <th>B5</th>
            <th>B4</th>
            <th>B3</th>
            <th>B2</th>
            <th>B1</th>
            <th>B0</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>C</td>
            <td colspan=4>value</td>
            <td>S</td>
            <td>&nbsp;&nbsp;&nbsp;&nbsp;</td>
            <td>C</td>
            <td colspan=5>value</td>
        </tr>
    </tbody>
</table>

--

- use sequences of 6 bit groups
- base 64 encode each group
- values in the range [-15, 15] will be encoded as a single character

---

# VLQ examples

<table>
    <thead>
        <tr>
            <th>B5</th>
            <th>B4</th>
            <th>B3</th>
            <th>B2</th>
            <th>B1</th>
            <th>B0</th>
            <th>&nbsp;&nbsp;&nbsp;&nbsp;</th>
            <th>B5</th>
            <th>B4</th>
            <th>B3</th>
            <th>B2</th>
            <th>B1</th>
            <th>B0</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>C</td>
            <td colspan=4>value</td>
            <td>S</td>
            <td>&nbsp;&nbsp;&nbsp;&nbsp;</td>
            <td>C</td>
            <td colspan=5>value</td>
        </tr>
    </tbody>
</table>

--

14 in binary: .blue[1110]

<table>
    <thead>
        <tr>
            <th>B5</th>
            <th>B4</th>
            <th>B3</th>
            <th>B2</th>
            <th>B1</th>
            <th>B0</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>0</td>
            <td class="blue">1</td>
            <td class="blue">1</td>
            <td class="blue">1</td>
            <td class="blue">0</td>
            <td>0</td>
        </tr>
    </tbody>
</table> 

--

011100 => 28 => (base 64) .green[**c**] 

---

# VLQ examples

<table>
    <thead>
        <tr>
            <th>B5</th>
            <th>B4</th>
            <th>B3</th>
            <th>B2</th>
            <th>B1</th>
            <th>B0</th>
            <th>&nbsp;&nbsp;&nbsp;&nbsp;</th>
            <th>B5</th>
            <th>B4</th>
            <th>B3</th>
            <th>B2</th>
            <th>B1</th>
            <th>B0</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>C</td>
            <td colspan=4>value</td>
            <td>S</td>
            <td>&nbsp;&nbsp;&nbsp;&nbsp;</td>
            <td>C</td>
            <td colspan=5>value</td>
        </tr>
    </tbody>
</table>

109 in binary: .red[110].blue[1101]

--

<table>
    <thead>
        <tr>
            <th>B5</th>
            <th>B4</th>
            <th>B3</th>
            <th>B2</th>
            <th>B1</th>
            <th>B0</th>
            <th>&nbsp;&nbsp;&nbsp;&nbsp;</th>
            <th>B5</th>
            <th>B4</th>
            <th>B3</th>
            <th>B2</th>
            <th>B1</th>
            <th>B0</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
            <td class="blue">1</td>
            <td class="blue">1</td>
            <td class="blue">0</td>
            <td class="blue">1</td>
            <td>0</td>
            <td>&nbsp;&nbsp;&nbsp;&nbsp;</td>
            <td>0</td>
            <td class="red">0</td>
            <td class="red">0</td>
            <td class="red">1</td>
            <td class="red">1</td>
            <td class="red">0</td>
        </tr>
    </tbody>
</table>

--

111010, 000110 => 58,6 => (base 64) .green[**6G**] 

---
# decoding

crash happened in file `out.js`, `line` = 1, `column` = 27

--

- use the `mappings` field in source map to decode to original file, line and column

--

```json
{
    "version" : 3,
    "file": "out.js",
    "sources": ["barilla.js", "muellers.js"],
    "names": ["penne", "orzo"],
*   "mappings": 
*       "AAmBA,SAASA,IAEL,SAASC,KAGTA,ICtBJ,SAASC,KAIRA",
    "x_com_bloomberg_sourcesFunctionMappings": [
        "AmBA6BA,CRIEI",
        "EEAEA"
    ],
}
```

- original file = `barilla.js`, `line` = 23, `column` = 5

---
# decoding

crash happened in file `out.js`, `line` = 1, `column` = 27

- use the `mappings` field in source map to decode to original file, line and column

```json
{
    "version" : 3,
    "file": "out.js",
    "sources": ["barilla.js", "muellers.js"],
    "names": ["penne", "orzo"],
    "mappings":  
        "AAmBA,SAASA,IAEL,SAASC,KAGTA,ICtBJ,SAASC,KAIRA",
*   "x_com_bloomberg_sourcesFunctionMappings": [
*       "AmBA6BA,CRIEI",
        "EEAEA"
    ],
}
```

- original file = `barilla.js`, `line` = 23, `column` = 5
- .green[**NEW**] use pasta metadata to decode original function name 
    - function name = `orzo`

---

# result

.pull-left[
from
```javascript
Error
    at r (`out.js:1:116`)
    at o (`out.js:1:93`)
    at n (`out.js:1:76`)
```
]

.pull-right[
to 
```javascript
Error
    at r (`index.js:13:11`)
    at o (`index.js:8:5`)
    at n (`index.js:3:5`)
```
]

--

.pull-left[
    <span style="color:green">with pasta</span>
]

.pull-right[
    &nbsp;
]


.pull-left[
from
```javascript
Error
    at `r` (index.js:13:11)
    at `o` (index.js:8:5)
    at `n` (index.js:3:5)
```
]


.pull-right[
to
```javascript
Error
    at `penne` (index.js:13:11)
    at `spaghetti` (index.js:8:5)
    at `orzo` (index.js:3:5)
```
]

---

class: center, middle
# 3
# @bloomberg/pasta-sourcemaps

---
# @bloomberg/pasta-sourcemaps

```bash
> npm install @bloomberg/pasta-sourcemaps
```

--

- components
  - parser
  - encoder
  - decoder

---
# parser

.pull-left[
input
```javascript
0.    // barilla.js
1.    const p = 42;
2.    function penne() {
3.        
4.        const o = 43;
5.        function orzo() {
6.           console.log("yum");
7.        }
8.        orzo();
9.    }
10.   penne();
```
]

.pull-right[
output
```json
[
    {
        "name": "<top-level>",
        "startLine": 0,
        "startColumn": 0,
        "endLine": 10,
        "endColumn": 8
    },
    {
        "name": "penne",
        "startLine": 1,
        "startColumn": 13,
        "endLine": 9,
        "endColumn": 1
    },
    {
        "name": "orzo",
        "startLine": 4,
        "startColumn": 17,
        "endLine": 7,
        "endColumn": 5
    }
]
```
]

---

# parser

.pull-left[
input
```javascript
0.    // barilla.js
1.    const p = 42;
*2.    function penne() {
*3.  
*4.        const o = 43;
*5.        function orzo() {
*6.           console.log("yum");
*7.        }
*8.        orzo();
*9.    }
10.   penne();
```
]

.pull-right[
output
```json
[
    {
        "name": "<top-level>",
        "startLine": 0,
        "startColumn": 0,
        "endLine": 10,
        "endColumn": 8
    },
*   {
*       "name": "penne",
*       "startLine": 1,
*       "startColumn": 13,
*       "endLine": 9,
*       "endColumn": 1
*   },
    {
        "name": "orzo",
        "startLine": 4,
        "startColumn": 17,
        "endLine": 7,
        "endColumn": 5
    }
]
```
]

---
# parser

.pull-left[
input
```javascript
0.    // barilla.js
1.    const p = 42;
2.    function penne() {
3.       
4.        const o = 43;
*5.        function orzo() {
*6.           console.log("yum");
*7.        }
8.         orzo();
9.    }
10.   penne();
```
]

.pull-right[
output
```json
[
    {
        "name": "<top-level>",
        "startLine": 0,
        "startColumn": 0,
        "endLine": 10,
        "endColumn": 8
    },
    {
        "name": "penne",
        "startLine": 1,
        "startColumn": 13,
        "endLine": 9,
        "endColumn": 1
    },
*   {
*       "name": "orzo",
*       "startLine": 4,
*       "startColumn": 17,
*       "endLine": 7,
*       "endColumn": 5
*   }
]
```
]

---

# parser

.pull-left[
input
```javascript
*0.    // barilla.js
*1.    const p = 42;
*2.    function penne() {
*3.   
*4.        const o = 43;
*5.        function orzo() {
*6.           console.log("yum");
*7.        }
*8.        orzo();
*9.    }
*10.   penne();
```
]

.pull-right[
output
```json
[
*   {
*       "name": "<top-level>",
*       "startLine": 0,
*       "startColumn": 0,
*       "endLine": 10,
*       "endColumn": 8
*   },
    {
        "name": "penne",
        "startLine": 1,
        "startColumn": 13,
        "endLine": 9,
        "endColumn": 1
    },
    {
        "name": "orzo",
        "startLine": 4,
        "startColumn": 17,
        "endLine": 7,
        "endColumn": 5
    }
]
```
]

---

# parser

- supports JavaScript, TypeScript, JSX and TSX
- uses the TypeScript parser to build the AST
- outputs a `<top-level>` element that captures the whole file
- outputs `<anonymous>` for anonymous functions
- handles a variety of function declaration and expression syntaxes

---

# function declarations and expressions

```javascript
// function declaration

function foo() {} // name = "foo"
```

--

```javascript
// anonymous function expressions

(function(){})();             // name = "<anonymous>"
array.map(x=>{return x*x;});  // name = "<anonymous>"
```

---

# function declarations and expressions

```javascript
// anonymous function expressions assigned to variables

let foo = function() {};            // name = "foo"
let foo = () => {};                 // name = "foo"
let foo = function localName() {};  // name = "localName"
```

--

```javascript
// anonymous function expressions assigned to properties

a.foo = function(){};               // name = "a.foo"
a["foo"] = function(){};            // name = "a.foo"
a[foo] = function(){};              // name = "a.<computed: foo>"
a[foo + bar] = function(){};        // name = "a.<computed>"
a["foo"][42].bar = function(){};    // name = "a.foo.42.bar"
```

--

- v8 has adopted the `a.<computed>` notation (thanks Sathya!)
  - https://bugs.chromium.org/p/v8/issues/detail?id=8823


---


# class methods

```javascript
// class methods

class MyClass {
    constructor(){}     // name = "MyClass"
    foo(){}             // name = "MyClass.prototype.foo"
    static foo(){}      // name = "MyClass.foo"
    get foo(){}         // name = "MyClass.prototype.get foo"
    set foo(val){}      // name = "MyClass.prototype.set foo"
    "foo"(){}           // name = "MyClass.prototype.foo"
    ["foo"](){}         // name = "MyClass.prototype.foo"
    [foo](){}           // name = "MyClass.prototype.<computed: foo>"
    [foo + bar]() {}    // name = "MyClass.prototype.<computed>"
}
```

--

```javascript
let MyClass = class {
    foo(){} // name = "MyClass.prototype.foo"
}
```

--

```javascript
class {
    foo(){} // name = "<anonymous>.prototype.foo"
}
```

---
# class properties (TypeScript)

```javascript
// class properties (TypeScript)

class MyClass {
    foo = function(){};         // name = "foo"
    static foo = function(){};  // name = "MyClass.foo"
}
```

---

# object literal properties

```javascript
// object literal properties

let a =
{ 
    b: {
        foo: function(){} 
        // name = "a.b.foo"
    }
}
```

--

```javascript
func( { b: {foo: function(){} } } );
            // name = "<Object>.b.foo"

```


---
# encoder

--

.pull-left-small[
input
```json
{ 
    "barilla.js": [
    {
        "name": "<top-level>",
        "startLine": 0,
        ...
    },
    {
        "name": "penne",
        "startLine": 1,
        ...
    },
    {
        "name": "orzo",
        "startLine": 4,
        ...
    }
],
    "muellers.js": [
    ...
    ]
}
```
]

.pull-right-big[
input
```json
{
    ...
    "sources": ["barilla.js", "muellers.js"],
    "names": [
        "penne", 
        "orzo", 
        "fusilli"
    ],
    "mappings": "..."
}
```
]

---
# encoder

.pull-left-small[
input
```json
{ 
    "barilla.js": [
    {
        "name": "<top-level>",
        "startLine": 0,
        ...
    },
    {
        "name": "penne",
        "startLine": 1,
        ...
    },
    {
        "name": "orzo",
        "startLine": 4,
        ...
    }
],
    "muellers.js": [
    ...
    ]
}
```
]

.pull-right-big[
input / output
```json
{
    ...
    "sources": ["barilla.js", "muellers.js"],
    "names": [
        "penne", 
        "orzo",
        "fusilli",
*       "<top-level>",
    ],
    "mappings": "...",
*   "x_com_bloomberg_sourcesFunctionMappings": [
*       "AAA6BE,C7BA6BA,CVkBII", 
*       "AAAQE,CRAQA"
*   ]
}
```
]


---
# encoder

- converts absolute locations to relative
- performs VLQ encoding

---
# decoder

input
```json
{
    ...
    "sources": ["barilla.js", "muellers.js"],
    "names": [
        "penne", 
        "orzo",
        "fusilli",
        "<top-level>",
    ],
    "mappings": "...",
    "x_com_bloomberg_sourcesFunctionMappings": [
        "AAA6BE,C7BA6BA,CVkBII", 
        "AAAQE,CRAQA"
    ]
}
```
<div>&nbsp;</div>

```javascript
decode("barilla.js", 6, 10)
```

---
# decoder

input
```json
{
    ...
    "sources": ["barilla.js", "muellers.js"],
    "names": [
        "penne", 
        "orzo",
        "fusilli",
        "<top-level>",
    ],
    "mappings": "...",
    "x_com_bloomberg_sourcesFunctionMappings": [
        "AAA6BE,C7BA6BA,CVkBII", 
        "AAAQE,CRAQA"
    ]
}
```
output

```javascript
decode("barilla.js", 6, 10) // returns "orzo"
```

---
class: center, middle

# recap

---

# recap 

.pull-left[
from
```javascript
Error
    at `r` (out.js:1:116)
    at `o` (out.js:1:93)
    at `n` (out.js:1:76)
```
]


.pull-right[
to
```javascript
Error
    at `penne` (index.js:13:11)
    at `spaghetti` (index.js:8:5)
    at `orzo` (index.js:3:5)
```
]

---

class: center, middle

# pasta
### https://github.com/bloomberg/pasta-sourcemaps
<img src="images/pasta_128.png"/>

