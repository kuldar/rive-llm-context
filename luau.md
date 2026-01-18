---
slug: getting-started
title: An introduction to Luau
sidebar:
  order: 1
---

Luau is a fast, small, safe, gradually typed embeddable scripting language derived from Lua 5.1. Luau ships as a command line tool for running, analyzing, and linting your Luau scripts, and is also integrated with Roblox Studio. Roblox developers should also visit our [Creator Docs Luau Section](https://create.roblox.com/docs/luau).

To get started with Luau, you can use the `luau` command line binary to run your code and `luau-analyze` to run static analysis (including type checking and linting). You can download these from [a recent release](https://github.com/luau-lang/luau/releases).

## Creating a script

To create your own testing script, create a new file with `.luau` as the extension:

```luau
--!hidden mode=nonstrict
function ispositive(x)
    return x > 0
end

print(ispositive(1))
print(ispositive("2"))

function isfoo(a)
    return a == "foo"
end

print(isfoo("bar"))
print(isfoo(1))
```

You can now run the file using `luau test.luau` and analyze it using `luau-analyze test.luau`.

Note that there are no warnings about calling ``ispositive()`` with a string, or calling ``isfoo()`` with a number. This is because Luau's type checking uses non-strict mode by default, which only reports errors if it's certain a program will error at runtime.

## Type checking

Now modify the script to include ``--!strict`` at the top:

```luau
--!strict

function ispositive(x)
    return x > 0
end

print(ispositive(1))
print(ispositive("2"))
```

We just mentioned ``nonstrict`` mode, where we only report errors if we can prove that a program will error at runtime. We also have ``strict`` mode, which will report errors if a program might error at runtime.

In this case, Luau will use the ``return x > 0`` statement to infer that ``ispositive()`` is a function taking a number and returning a boolean. Note that in this case, we were able to determine the type of `ispositive` without the presence of any explicit type annotations.

Based on Luau's type inference, the analysis tool will now flag the incorrect call to ``ispositive()``:

```
$ luau-analyze test.luau
test.luau(7,18): TypeError: Type 'string' could not be converted into 'number'
```

## Annotations

You can add annotations to locals, arguments, and function return types. Among other things, annotations can help enforce that you don't accidentally do something silly. Here's how we would add annotations to ``ispositive()``:

```luau
--!strict

function ispositive(x : number) : boolean
    return x > 0
end

local result : boolean
result = ispositive(1)

```

Now we've explicitly told Luau that ``ispositive()`` accepts a number and returns a boolean. This wasn't strictly (pun intended) necessary in this case, because Luau's inference was able to deduce this already. But even in this case, there are advantages to explicit annotations. Imagine that later we decide to change ``ispositive()`` to return a string value:

```luau
--!strict

function ispositive(x : number) : boolean
    if x > 0 then
        return "yes"
    else
        return "no"
    end
end

local result : boolean
result = ispositive(1)
```

Oops -- we're returning string values, but we forgot to update the function return type. Since we've told Luau that ``ispositive()`` returns a boolean (and that's how we're using it), the call site isn't flagged as an error. But because the annotation doesn't match our code, we get a warning in the function body itself:

```
$ luau-analyze test.luau
test.luau(5,9): TypeError: Type 'string' could not be converted into 'boolean'
test.luau(7,9): TypeError: Type 'string' could not be converted into 'boolean'
```

The fix is simple; just change the annotation to declare the return type as a string:

```luau
--!strict

function ispositive(x : number) : string
    if x > 0 then
        return "yes"
    else
        return "no"
    end
end

local result : boolean
result = ispositive(1)
```

Well, almost - since we declared ``result`` as a boolean, the call site is now flagged:

```
$ luau-analyze test.luau
test.luau(12,10): TypeError: Type 'string' could not be converted into 'boolean'
```

If we update the type of the local variable, everything is good. Note that we could also just let Luau infer the type of ``result`` by changing it to the single line version ``local result = ispositive(1)``.

```luau
--!strict

function ispositive(x : number) : string
    if x > 0 then
        return "yes"
    else
        return "no"
    end
end

local result : string
result = ispositive(1)
```

## Conclusions

---

---
slug: syntax
title: Luau syntax by example
description: An informal reference for Luau's syntax by example.
sidebar:
  order: 3
---

Luau uses the baseline [syntax of Lua 5.1](https://www.lua.org/manual/5.1/manual.html#2). For detailed documentation, please refer to the Lua manual, this is an example:

```luau
local function tree_insert(tree, x)
    local lower, equal, greater = split(tree.root, x)
    if not equal then
        equal = {
            x = x,
            y = math.random(0, 2^31-1),
            left = nil,
            right = nil
        }
    end
    tree.root = merge3(lower, equal, greater)
end
```

Note that future versions of Lua extend the Lua 5.1 syntax with more  features; Luau does  support string literal extensions but does not support other 5.x additions; for details please refer to [compatibility section](../compatibility).

The rest of this document documents additional syntax used in Luau.

## String literals

Luau implements support for hexadecimal (`\x`), Unicode (`\u`) and `\z` escapes for string literals. This syntax follows [Lua 5.3 syntax](https://www.lua.org/manual/5.3/manual.html#3.1):

- `\xAB` inserts a character with the code 0xAB into the string
- `\u{ABC}` inserts a UTF8 byte sequence that encodes U+0ABC character into the string (note that braces are mandatory)
- `\z` at the end of the line inside a string literal ignores all following whitespace including newlines, which can be helpful for breaking long literals into multiple lines.

## Number literals

In addition to basic integer and floating-point decimal numbers, Luau supports:

- Hexadecimal integer literals, `0xABC` or `0XABC`
- Binary integer literals, `0b01010101` or `0B01010101`
- Decimal separators in all integer literals, using `_` for readability: `1_048_576`, `0xFFFF_FFFF`, `0b_0101_0101`

Note that Luau only has a single number type, a 64-bit IEEE754 double precision number (which can represent integers up to 2^53 exactly), and larger integer literals are stored with precision loss.

## Continue statement

In addition to `break` in all loops, Luau supports `continue` statement. Similar to `break`, `continue` must be the last statement in the block.

Note that unlike `break`, `continue` is not a keyword. This is required to preserve backwards compatibility with existing code; so this is a `continue` statement:

```luau
--!hidden mode=nocheck
if x < 0 then
    continue
end
```

Whereas this is a function call:

```luau
--!hidden mode=nocheck
if x < 0 then
    continue()
end
```

When used in `repeat..until` loops, `continue` can not skip the declaration of a local variable if that local variable is used in the loop condition; code like this is invalid and won't compile:

```luau
repeat
    do continue end
    local a = 5
until a > 0
```

## Compound assignments

Luau supports compound assignments with the following operators: `+=`, `-=`, `*=`, `/=`, `//=`, `%=`, `^=`, `..=`. Just like regular assignments, compound assignments are statements, not expressions:

```luau
local a = 5

-- this works
a += 1

-- this doesn't work
print(a += 1)
```

Compound assignments only support a single value on the left and right hand side; additionally, the function calls on the left hand side are only evaluated once:

```luau
function foo() return 2 end
local a = { 1, 2, 3 }

-- calls foo() twice
a[foo()] = a[foo()] + 1

-- calls foo() once
a[foo()] += 1
```

Compound assignments call the arithmetic metamethods (`__add` et al) and table indexing metamethods (`__index` and `__newindex`) as needed - for custom types no extra effort is necessary to support them.

## Type annotations

To support gradual typing, Luau supports optional type annotations for variables and functions, as well as declaring type aliases.

Types can be declared for local variables, function arguments and function return types using `:` as a separator:

```luau
function foo(x: number, y: string): boolean
    local k: string = y:rep(x)
    return k == "a"
end
```

In addition, the type of any expression can be overridden using a type cast `::`:

```luau
function foo(x: number, y: unknown): boolean
    local k = (y :: string):rep(x)
    return k == "a"
end
```

There are several simple builtin types: `any` (represents inability of the type checker to reason about the type), `nil`, `boolean`, `number`, `string` and `thread`.

Function types are specified using the arguments and return types, separated with `->`:

```luau
local foo: (number, string) -> boolean
```

To return no values or more than one, you need to wrap the return type position with parentheses, and then list your types there.

```luau
local no_returns: (number, string) -> ()
local returns_boolean_and_string: (number, string) -> (boolean, string)

function foo(x: number, y: number): (number, string)
    return x + y, tostring(x) .. tostring(y)
end
```

Note that function types are specified without the argument names in the examples above, but it's also possible to specify the names (that are not semantically significant but can show up in documentation and autocomplete):

```luau
local callback: (errorCode: number, errorText: string) -> ()
```

Table types are specified using the table literal syntax, using `:` to separate keys from values:

```luau
local array: { [number] : string }
local object: { x: number, y: string }
```

When the table consists of values keyed by numbers, it's called an array-like table and has a special short-hand syntax, `{T}` (e.g. `{string}`).

Additionally, the type syntax supports type intersections (`((number) -> string) & ((boolean) -> string)`) and unions (`(number | boolean) -> string`). An intersection represents a type with values that conform to both sides at the same time, which is useful for overloaded functions; a union represents a type that can store values of either type - `any` is technically a union of all possible types.

It's common in Lua for function arguments or other values to store either a value of a given type or `nil`; this is represented as a union (`number | nil`), but can be specified using `?` as a shorthand syntax (`number?`).

In addition to declaring types for a given value, Luau supports declaring type aliases via `type` syntax:

```luau
type Point = { x: number, y: number }
type Array<T> = { [number]: T }
type Something = typeof(string.gmatch("", "%d"))
```

The right hand side of the type alias can be a type definition or a `typeof` expression; `typeof` expression doesn't evaluate its argument at runtime.

By default type aliases are local to the file they are declared in. To be able to use type aliases in other modules using `require`, they need to be exported:

```luau
export type Point = { x: number, y: number }
```

An exported type can be used in another module by prefixing its name with the require alias that you used to import the module.

```luau
--!file main.luau
local M = require("./other")

local a: M.Point = {x=5, y=6}

--!file other.luau
export type Point = { x: number, y: number }

return {}
```

For more information please refer to [typechecking documentation](../types).

## If-then-else expressions

In addition to supporting standard if *statements*, Luau adds support for if *expressions*.  Syntactically, `if-then-else` expressions look very similar to if statements.  However instead of conditionally executing blocks of code, if expressions conditionally evaluate expressions and return the value produced as a result. Also, unlike if statements, if expressions do not terminate with the `end` keyword.

Here is a simple example of an `if-then-else` expression:
```luau
local a, b = 6, 7
local maxValue = if a > b then a else b
```

`if-then-else` expressions may occur in any place a regular expression is used.  The `if-then-else` expression must match `if <expr> then <expr> else <expr>`; it can also contain an arbitrary number of `elseif` clauses, like `if <expr> then <expr> elseif <expr> then <expr> else <expr>`. Note that in either case, `else` is mandatory.  

Here's is an example demonstrating `elseif`:
```luau
local x = -21
local sign = if x < 0 then -1 elseif x > 0 then 1 else 0
```

**Note:** In Luau, the `if-then-else` expression is preferred vs the standard Lua idiom of writing `a and b or c` (which roughly simulates a ternary operator).  However, the Lua idiom may return an unexpected result if `b` evaluates to false.  The `if-then-else` expression will behave as expected in all situations.

## Generalized iteration

Luau uses the standard Lua syntax for iterating through containers, `for vars in values`, but extends the semantics with support for generalized iteration. In Lua, to iterate over a table you need to use an iterator like `next` or a function that returns one like `pairs` or `ipairs`. In Luau, you can simply iterate over a table:

```luau
for k, v in {1, 4, 9} do
    assert(k * k == v)
end
```

Further, iteration can be extended for tables or userdata by implementing the `__iter` metamethod which is called before the iteration begins, and should return an iterator function like `next` (or a custom one):

```luau
local obj = { items = {1, 4, 9} }
setmetatable(obj, { __iter = function(o) return next, o.items end })

for k, v in obj do
    assert(k * k == v)
end
```

The default iteration order for tables is specified to be consecutive for elements `1..#t` and unordered after that, visiting every element; similarly to iteration using `pairs`, modifying the table entries for keys other than the current one results in unspecified behavior.

## String interpolation

Luau adds an additional way to define string values that allows you to place runtime expressions directly inside specific spots of the literal.

This is a more ergonomic alternative over using `string.format` or `("literal"):format`.

To use string interpolation, use a backtick string literal:

```luau
local count = 3
print(`Bob has {count} apple(s)!`)
--> Bob has 3 apple(s)!
```

Any expression can be used inside `{}`:

```luau
local combos = {2, 7, 1, 8, 5}
print(`The lock combination is {table.concat(combos)}.`)
--> The lock combination is 27185.
```

Inside backtick string literal, `\` is used to escape `` ` ``, `{`, `\` itself and a newline:

```luau
print(`Some example escaping the braces \{like so}`)
--> Some example escaping the braces {like so}

print(`Backslash \ that escapes the space is not a part of the string...`)
--> Backslash  that escapes the space is not a part of the string...

print(`Backslash \\ will escape the second backslash...`)
--> Backslash \ will escape the second backslash...

print(`Some text that also includes \`...`)
--> Some text that also includes `...

local name = "Luau"

print(`Welcome to {
    name
}!`)
--> Welcome to Luau!
```

### Restrictions and limitations

The sequence of two opening braces {{"`{{`"}} is rejected with a parse error.
This restriction is made to prevent developers using other programming languages with a similar feature from trying to attempt that as a way to escape a single `{` and getting unexpected results in Luau.

Luau currently does not support backtick string literals in type annotations, therefore `` type Foo = `Foo` `` is invalid syntax.

Unlike single and double-quoted string literals, backtick string literals must always be wrapped in parentheses for function calls:
```luau
print "hello" -- valid 
print`hello` -- invalid syntax
print(`hello`) -- valid
```

## Floor division (`//`)

Luau supports floor division, including its operator (`//`), its compound assignment operator (`//=`), and overloading metamethod (`__idiv`), as an ergonomic alternative to `math.floor`.

For numbers, `a // b` is equal to `math.floor(a / b)`:
```luau
local n = 6.72
print(n // 2) --> 3
n //= 3
print(n) --> 2
```

Note that it's possible to get `inf`, `-inf`, or `NaN` with floor division; when `b` is `0`, `a // b` results in positive or negative infinity, and when both `a` and `b` are `0`, `a // b` results in NaN.

For native vectors, `c // d` applies `math.floor` to each component of the vector `c`. Therefore `c // d` is equivalent to `vector.create(math.floor(c.x / d), math.floor(c.y / b), math.floor(c.z / b))`.

Floor division syntax and semantics follow from [Lua 5.3](https://www.lua.org/manual/5.3/manual.html#3.4.1) where applicable.

---

---
slug: library
title: Standard Library
description: The official reference for Luau's standard library.
sidebar:
  order: 1
---

Luau comes equipped with a standard library of functions designed to manipulate the built-in data types. Note that the library is relatively minimal and doesn't expose ways for
scripts to interact with the host environment - it's expected that embedding applications provide extra functionality on top of this and limit or sandbox the system access
appropriately, if necessary. For example, Roblox provides [a rich API to interact with the 3D environment and limited APIs to interact with external services](https://developer.roblox.com/en-us/api-reference).

This page documents the available builtin libraries and functions. All of these are accessible by default by any script, assuming the host environment exposes them (which is usually a safe assumption outside of extremely constrained environments).

## Global functions

While most library functions are provided as part of a library like `table`, a few global functions are exposed without extra namespacing.

```
function assert<T>(value: T, message: string?): T
```

`assert` checks if the value is truthy; if it's not (which means it's `false` or `nil`), it raises an error. The error message can be customized with an optional parameter.
Upon success the function returns the `value` argument.

```
function error(obj: any, level: number?)
```

`error` raises an error with the specified object. Note that errors don't have to be strings, although they often are by convention; various error handling mechanisms like `pcall`
preserve the error type. When `level` is specified, the error raised is turned into a string that contains call frame information for the caller at level `level`, where `1` refers
to the function that called `error`. This can be useful to attribute the errors to callers, for example `error("Expected a valid object", 2)` highlights the caller of the function
that called `error` instead of the function itself in the callstack.

```
function gcinfo(): number
```

`gcinfo` returns the total heap size in kilobytes, which includes bytecode objects, global tables as well as the script-allocated objects. Note that Luau uses an incremental
garbage collector, and as such at any given point in time the heap may contain both reachable and unreachable objects. The number returned by `gcinfo` reflects the current heap
consumption from the operating system perspective and can fluctuate over time as garbage collector frees objects.

```
function getfenv(target: (function | number)?): table
```

Returns the environment table for target function; when `target` is not a function, it must be a number corresponding to the caller stack index, where 1 means the function that calls `getfenv`, and the environment table is returned for the corresponding function from the call stack. When `target` is omitted it defaults to `1`, so `getfenv()` returns the environment table for the calling function.

```
function getmetatable(obj: any): table?
```

Returns the metatable for the specified object; when object is not a table or a userdata, the returned metatable is shared between all objects of the same type. Note that when metatable is protected (has a `__metatable` key), the value corresponding to that key is returned instead and may not be a table.

```
function next<K, V>(t: { [K]: V }, i: K?): (K, V)?
```

Given the table `t`, returns the next key-value pair after `i` in the table traversal order, or nothing if `i` is the last key. When `i` is `nil`, returns the first key-value pair instead.

```
function newproxy(mt: boolean?): userdata
```

Creates a new untyped userdata object; when `mt` is true, the new object has an empty metatable that can be modified using `getmetatable`.

```
function print(args: ...any)
```

Prints all arguments to the standard output, using Tab as a separator.

```
function rawequal(a: any, b: any): boolean
```

Returns true iff `a` and `b` have the same type and point to the same object (for garbage collected types) or are equal (for value types).

```
function rawget<K, V>(t: { [K]: V }, k: K): V?
```

Performs a table lookup with index `k` and returns the resulting value, if present in the table, or nil. This operation bypasses metatables/`__index`.

```
function rawlen<K, V>(t: { [K]: V } | string): number
```

Returns the raw length of the table or string. If it is a string, this operation is identical to `#str` or `string.len(str)`. This operation bypasses metatables/`__len`.

```
function rawset<K, V>(t: { [K] : V }, k: K, v: V)
```

Assigns table field `k` to the value `v`. This operation bypasses metatables/`__newindex`.

```
function select<T>(i: string, args: ...T): number
function select<T>(i: number, args: ...T): ...T
```

When called with `'#'` as the first argument, returns the number of remaining parameters passed. Otherwise, returns the subset of parameters starting with the specified index.
Index can be specified from the start of the arguments (using 1 as the first argument), or from the end (using -1 as the last argument).

```
function setfenv(target: function | number, env: table)
```

Changes the environment table for target function to `env`; when `target` is not a function, it must be a number corresponding to the caller stack index, where 1 means the function that calls `setfenv`, and the environment table is returned for the corresponding function from the call stack.

```
function setmetatable(t: table, mt: table?)
```

Changes metatable for the given table. Note that unlike `getmetatable`, this function only works on tables. If the table already has a protected metatable (has a `__metatable` field), this function errors.

```
function tonumber(s: string, base: number?): number?
```

Converts the input string to the number in base `base` (default 10) and returns the resulting number. If the conversion fails (that is, if the input string doesn't represent a valid number in the specified base), returns `nil` instead.

```
function tostring(obj: any): string
```

Converts the input object to string and returns the result. If the object has a metatable with `__tostring` field, that method is called to perform the conversion.

```
function type(obj: any): string
```

Returns the type of the object, which is one of `"nil"`, `"boolean"`, `"number"`, `"vector"`, `"string"`, `"table"`, `"function"`, `"userdata"`, `"thread"`, or `"buffer"`.

```
function typeof(obj: any): string
```

Returns the type of the object; for userdata objects that have a metatable with the `__type` field *and* are defined by the host (not `newproxy`), returns the value for that key.
For custom userdata objects, such as ones returned by `newproxy`, this function returns `"userdata"` to make sure host-defined types can not be spoofed.

```
function ipairs(t: table): <iterator>
```

Returns the triple (generator, state, nil) that can be used to traverse the table using a `for` loop. The traversal results in key-value pairs for the numeric portion of the table; key starts from 1 and increases by 1 on each iteration. The traversal terminates when reaching the first `nil` value (so `ipairs` can't be used to traverse array-like tables with holes).

```
function pairs(t: table): <iterator>
```

Returns the triple (generator, state, nil) that can be used to traverse the table using a `for` loop. The traversal results in key-value pairs for all keys in the table, numeric and otherwise, but doesn't have a defined order.

```
function pcall(f: function, args: ...any): (boolean, ...any)
```

Calls function `f` with parameters `args`. If the function succeeds, returns `true` followed by all return values of `f`. If the function raises an error, returns `false` followed by the error object.
Note that `f` can yield, which results in the entire coroutine yielding as well.

```
function xpcall(f: function, e: function, args: ...any): (boolean, ...any)
```

Calls function `f` with parameters `args`. If the function succeeds,  returns `true` followed by all return values of `f`. If the function raises an error, calls `e` with the error object as an argument, and returns `false` followed by the first return value of `e`.
Note that `f` can yield, which results in the entire coroutine yielding as well.
`e` can neither yield nor error - if it does raise an error, `xpcall` returns with `false` followed by a special error message.

```
function unpack<V>(a: {V}, f: number?, t: number?): ...V
```

Returns all values of `a` with indices in `[f..t]` range. `f` defaults to 1 and `t` defaults to `#a`. Note that this is equivalent to `table.unpack`.

## math library

```
function math.abs(n: number): number
```

Returns the absolute value of `n`. Returns NaN if the input is NaN. 

```
function math.acos(n: number): number
```

Returns the arc cosine of `n`, expressed in radians. Returns a value in `[0, pi]` range. Returns NaN if the input is not in `[-1, +1]` range.

```
function math.asin(n: number): number
```

Returns the arc sine of `n`, expressed in radians. Returns a value in `[-pi/2, +pi/2]` range. Returns NaN if the input is not in `[-1, +1]` range.

```
function math.atan2(y: number, x: number): number
```

Returns the arc tangent of `y/x`, expressed in radians. The function takes into account the sign of both arguments in order to determine the quadrant. Returns a value in `[-pi, pi]` range.

```
function math.atan(n: number): number
```

Returns the arc tangent of `n`, expressed in radians. Returns a value in `[-pi/2, pi-2]` range.

```
function math.ceil(n: number): number
```

Rounds `n` upwards to the next integer boundary.

```
function math.cosh(n: number): number
```

Returns the hyperbolic cosine of `n`.

```
function math.cos(n: number): number
```

Returns the cosine of `n`, which is an angle in radians. Returns a value in `[0, 1]` range.

```
function math.deg(n: number): number
```

Converts `n` from radians to degrees and returns the result.

```
function math.exp(n: number): number
```

Returns the base-e exponent of `n`, that is `e^n`.

```
function math.floor(n: number): number
```

Rounds `n` downwards to previous integer boundary.

```
function math.fmod(x: number, y: number): number
```

Returns the remainder of `x` modulo `y`, rounded towards zero. Returns NaN if `y` is zero.

```
function math.frexp(n: number): (number, number)
```

Splits the number into a significand (a number in `[-1, +1]` range) and binary exponent such that `n = s * 2^e`, and returns `s, e`.

```
function math.ldexp(s: number, e: number): number
```

Given the significand and a binary exponent, returns a number `s * 2^e`.

```
function math.lerp(a: number, b: number, t: number): number
```

Linearly interpolated between number value `a` and `b` using factor `t`, generally returning the result of `a + (b - a) * t`.
When `t` is exactly `1`, the value of `b` will be returned instead to ensure that when `t` is on the interval `[0, 1]`, the result of `lerp` will be on the interval `[a, b]`.

```
math.map(x: number, inmin: number, inmax: number, outmin: number, outmax: number): number
```

Returns a value that represents `x` mapped linearly from the input range (`inmin` to `inmax`) to the output range (`outmin` to `outmax`).

```
function math.log10(n: number): number
```

Returns base-10 logarithm of the input number. Returns NaN if the input is negative, and negative infinity if the input is 0.
Equivalent to `math.log(n, 10)`.

```
function math.log(n: number, base: number?): number
```

Returns logarithm of the input number in the specified base; base defaults to `e`. Returns NaN if the input is negative, and negative infinity if the input is 0.

```
function math.max(list: ...number): number
```

Returns the maximum number of the input arguments. The function requires at least one input and will error if zero parameters are passed. If one of the inputs is a NaN, the result may or may not be a NaN.

```
function math.min(list: ...number): number
```

Returns the minimum number of the input arguments. The function requires at least one input and will error if zero parameters are passed. If one of the inputs is a NaN, the result may or may not be a NaN.

```
function math.modf(n: number): (number, number)
```

Returns the integer and fractional part of the input number. Both the integer and fractional part have the same sign as the input number, e.g. `math.modf(-1.5)` returns `-1, -0.5`.

```
function math.pow(x: number, y: number): number
```

Returns `x` raised to the power of `y`.

```
function math.rad(n: number): number
```

Converts `n` from degrees to radians and returns the result.

```
function math.random(): number
function math.random(n: number): number
function math.random(min: number, max: number): number
```

Returns a random number using the global random number generator. A zero-argument version returns a number in `[0, 1]` range. A one-argument version returns a number in `[1, n]` range. A two-argument version returns a number in `[min, max]` range. The input arguments are truncated to integers, so `math.random(1.5)` always returns 1.

```
function math.randomseed(seed: number)
```

Reseeds the global random number generator; subsequent calls to `math.random` will generate a deterministic sequence of numbers that only depends on `seed`.

```
function math.sinh(n: number): number
```

Returns a hyperbolic sine of `n`.

```
function math.sin(n: number): number
```

Returns the sine of `n`, which is an angle in radians. Returns a value in `[0, 1]` range.

```
function math.sqrt(n: number): number
```

Returns the square root of `n`. Returns NaN if the input is negative.

```
function math.tanh(n: number): number
```

Returns the hyperbolic tangent of `n`.

```
function math.tan(n: number): number
```

Returns the tangent of `n`, which is an angle in radians.

```
function math.noise(x: number, y: number?, z: number?): number
```

Returns 3D Perlin noise value for the point `(x, y, z)` (`y` and `z` default to zero if absent). Returns a value in `[-1, 1]` range.

```
function math.clamp(n: number, min: number, max: number): number
```

Returns `n` if the number is in `[min, max]` range; otherwise, returns `min` when `n < min`, and `max` otherwise. If `n` is NaN, may or may not return NaN.
The function errors if `min > max`.

```
function math.sign(n: number): number
```

Returns `-1` if `n` is negative, `1` if `n` is positive, and `0` if `n` is zero or NaN.

```
function math.round(n: number): number
```

Rounds `n` to the nearest integer boundary. If `n` is exactly halfway between two integers, rounds `n` away from 0.

## table library

```
function table.concat(a: {string}, sep: string?, f: number?, t: number?): string
```

Concatenate all elements of `a` with indices in range `[f..t]` together, using `sep` as a separator if present. `f` defaults to 1 and `t` defaults to `#a`.

```
function table.foreach<K, V, R>(t: { [K]: V }, f: (K, V) -> R?): R?
```

Iterates over all elements of the table in unspecified order; for each key-value pair, calls `f` and returns the result of `f` if it's non-nil. If all invocations of `f` returned `nil`, returns no values. This function has been deprecated and is not recommended for use in new code; use `for` loop instead.

```
function table.foreachi<V, R>(t: {V}, f: (number, V) -> R?): R?
```

Iterates over numeric keys of the table in `[1..#t]` range in order; for each key-value pair, calls `f` and returns the result of `f` if it's non-nil. If all invocations of `f` returned `nil`, returns no values. This function has been deprecated and is not recommended for use in new code; use `for` loop instead.

```
function table.getn<V>(t: {V}): number
```

Returns the length of table `t`. This function has been deprecated and is not recommended for use in new code; use `#t` instead.

```
function table.maxn<V>(t: {V}): number
```

Returns the maximum numeric key of table `t`, or zero if the table doesn't have numeric keys.

```
function table.insert<V>(t: {V}, v: V)
function table.insert<V>(t: {V}, i: number, v: V)
```

When using a two-argument version, appends the value to the array portion of the table (equivalent to `t[#t+1] = v`).
When using a three-argument version, inserts the value at index `i` and shifts values at indices after that by 1. `i` should be in `[1..#t]` range.

```
function table.remove<V>(t: {V}, i: number?): V?
```

Removes element `i` from the table and shifts values at indices after that by 1. If `i` is not specified, removes the last element of the table.
`i` should be in `[1..#t]` range.
Returns the value of the removed element, or `nil` if no element was removed (e.g. table was empty).

```
function table.sort<V>(t: {V}, f: ((V, V) -> boolean)?)
```

Sorts the table `t` in ascending order, using `f` as a comparison predicate: `f` should return `true` iff the first parameter should be before the second parameter in the resulting table. When `f` is not specified, builtin less-than comparison is used instead.
The comparison predicate must establish a strict weak ordering - sort results are undefined otherwise.

```
function table.pack<V>(args: ...V): { [number]: V, n: number }
```

Returns a table that consists of all input arguments as array elements, and `n` field that is set to the number of inputs.

```
function table.unpack<V>(a: {V}, f: number?, t: number?): ...V
```

Returns all values of `a` with indices in `[f..t]` range. `f` defaults to 1 and `t` defaults to `#a`.
Note that if you want to unpack varargs packed with `table.pack` you have to specify the index fields because `table.unpack` doesn't automatically use the `n` field that `table.pack` creates. Example usage for packed varargs: `table.unpack(args, 1, args.n)`

```
function table.move<V>(a: {V}, f: number, t: number, d: number, tt: {V}?)
```

Copies elements in range `[f..t]` from table `a` to table `tt` if specified and `a` otherwise, starting from the index `d`.

```
function table.create<V>(n: number, v: V?): {V}
```

Creates a table with `n` elements; all of them (range `[1..n]`) are set to `v`. When `v` is nil or omitted, the returned table is empty but has preallocated space for `n` elements which can make subsequent insertions faster.
Note that preallocation is only performed for the array portion of the table - using `table.create` on dictionaries is counter-productive.

```
function table.find<V>(t: {V}, v: V, init: number?): number?
```

Find the first element in the table that is equal to `v` and returns its index; the traversal stops at the first `nil`. If the element is not found, `nil` is returned instead. The traversal starts at index `init` if specified, otherwise 1.

```
function table.clear(t: table)
```

Removes all elements from the table while preserving the table capacity, so future assignments don't need to reallocate space.

```
function table.freeze(t: table): table
```

Given a non-frozen table, freezes it such that all subsequent attempts to modify the table or assign its metatable raise an error. If the input table is already frozen or has a protected metatable, the function raises an error; otherwise it returns the input table.
Note that the table is frozen in-place and is not being copied. Additionally, only `t` is frozen, and keys/values/metatable of `t` don't change their state and need to be frozen separately if desired.

```
function table.isfrozen(t: table): boolean
```

Returns `true` iff the input table is frozen.

```
function table.clone(t: table): table
```

Returns a copy of the input table that has the same metatable, same keys and values, and is not frozen even if `t` was.
The copy is shallow: implementing a deep recursive copy automatically is challenging, and often only certain keys need to be cloned recursively which can be done after the initial clone by modifying the resulting table.

## string library

```
function string.byte(s: string, f: number?, t: number?): ...number
```

Returns the numeric code of every byte in the input string with indices in range `[f..t]`. `f` defaults to 1 and `t` defaults to `f`, so a two-argument version of this function returns a single number. If the function is called with a single argument and the argument is out of range, the function returns no values.

```
function string.char(args: ...number): string
```

Returns the string that contains a byte for every input number; all inputs must be integers in `[0..255]` range.

```
function string.find(s: string, p: string, init: number?, plain: boolean?): (number?, number?, ...string)
```

Tries to find an instance of pattern `p` in the string `s`, starting from position `init` (defaults to 1). When `plain` is true, the search is using raw (case-sensitive) string equality, otherwise `p` should be a [string pattern](https://www.lua.org/manual/5.3/manual.html#6.4.1). If a match is found, returns the position of the match and the length of the match, followed by the pattern captures; otherwise returns `nil`.

```
function string.format(s: string, args: ...any): string
```

Returns a formatted version of the input arguments using a [printf-style format string](https://en.cppreference.com/w/c/io/fprintf) `s`. The following format characters are supported:

- `c`: expects an integer number and produces a character with the corresponding character code
- `d`, `i`, `u`: expects an integer number and produces the decimal representation of that number
- `o`: expects an integer number and produces the octal representation of that number
- `x`, `X`: expects an integer number and produces the hexadecimal representation of that number, using lower case or upper case hexadecimal characters
- `e`, `E`, `f`, `g`, `G`: expects a number and produces the floating point representation of that number, using scientific or decimal representation
- `q`: expects a string and produces the same string quoted using double quotation marks, with escaped special characters if necessary
- `s`: expects a string and produces the same string verbatim

The formats support modifiers `-`, `+`, space, `#` and `0`, as well as field width and precision modifiers - with the exception of `*`.

```
function string.gmatch(s: string, p: string): <iterator>
```

Produces an iterator function that, when called repeatedly explicitly or via `for` loop, produces matches of string `s` with [string pattern](https://www.lua.org/manual/5.3/manual.html#6.4.1) `p`. For every match, the captures within the pattern are returned if present (if a pattern has no captures, the entire matching substring is returned instead).

```
function string.gsub(s: string, p: string, f: function | table | string, maxs: number?): (string, number)
```

For every match of [string pattern](https://www.lua.org/manual/5.3/manual.html#6.4.1) `p` in `s`, replace the match according to `f`. The substitutions stop after the limit of `maxs`, and the function returns the resulting string followed by the number of substitutions.

When `f` is a string, the substitution uses the string as a replacement. When `f` is a table, the substitution uses the table element with key corresponding to the first pattern capture, if present, and entire match otherwise. Finally, when `f` is a function, the substitution uses the result of calling `f` with call pattern captures, or entire matching substring if no captures are present.

```
function string.len(s: string): number
```

Returns the number of bytes in the string (equivalent to `#s`).

```
function string.lower(s: string): string
```

Returns a string where each byte corresponds to the lower-case ASCII version of the input byte in the source string.

```
function string.match(s: string, p: string, init: number?): ...string?
```

Tries to find an instance of pattern `p` in the string `s`, starting from position `init` (defaults to 1). `p` should be a [string pattern](https://www.lua.org/manual/5.3/manual.html#6.4.1). If a match is found, returns all pattern captures, or entire matching substring if no captures are present, otherwise returns `nil`.

```
function string.rep(s: string, n: number): string
```

Returns the input string `s` repeated `n` times. Returns an empty string if `n` is zero or negative.

```
function string.reverse(s: string): string
```

Returns the string with the order of bytes reversed compared to the original. Note that this only works if the input is a binary or ASCII string.

```
function string.sub(s: string, f: number, t: number?): string
```

Returns a substring of the input string with the byte range `[f..t]`; `t` defaults to `#s`, so a two-argument version returns a string suffix.

```
function string.upper(s: string): string
```

Returns a string where each byte corresponds to the upper-case ASCII version of the input byte in the source string.

```
function string.split(s: string, sep: string?): {string}
```

Splits the input string using `sep` as a separator (defaults to `","`) and returns the resulting substrings. If separator is empty, the input string is split into separate one-byte strings.

```
function string.pack(f: string, args: ...any): string
```

Given a [pack format string](https://www.lua.org/manual/5.3/manual.html#6.4.2), encodes all input parameters according to the packing format and returns the resulting string. Note that Luau uses fixed sizes for all types that have platform-dependent size in Lua 5.x: short is 16 bit, long is 64 bit, integer is 32-bit and size_t is 32 bit for the purpose of string packing.

```
function string.packsize(f: string): number
```

Given a [pack format string](https://www.lua.org/manual/5.3/manual.html#6.4.2), returns the size of the resulting packed representation. The pack format can't use variable-length format specifiers. Note that Luau uses fixed sizes for all types that have platform-dependent size in Lua 5.x: short is 16 bit, long is 64 bit, integer is 32-bit and size_t is 32 bit for the purpose of string packing.

```
function string.unpack(f: string, s: string): ...any
```

Given a [pack format string](https://www.lua.org/manual/5.3/manual.html#6.4.2), decodes the input string according to the packing format and returns all resulting values. Note that Luau uses fixed sizes for all types that have platform-dependent size in Lua 5.x: short is 16 bit, long is 64 bit, integer is 32-bit and size_t is 32 bit for the purpose of string packing.

## coroutine library

```
function coroutine.create(f: function): thread
```

Returns a new coroutine that, when resumed, will run function `f`.

```
function coroutine.running(): thread?
```

Returns the currently running coroutine, or `nil` if the code is running in the main coroutine (depending on the host environment setup, main coroutine may never be used for running code).

```
function coroutine.status(co: thread): string
```

Returns the status of the coroutine, which can be `"running"`, `"suspended"`, `"normal"` or `"dead"`. Dead coroutines have finished their execution and can not be resumed, but their state can still be inspected as they are not dead from the garbage collector point of view.

```
function coroutine.wrap(f: function): function
```

Creates a new coroutine and returns a function that, when called, resumes the coroutine and passes all arguments along to the suspension point. When the coroutine yields or finishes, the wrapped function returns with all values returned at the suspension point.

```
function coroutine.yield(args: ...any): ...any
```

Yields the currently running coroutine and passes all arguments along to the code that resumed the coroutine. The coroutine becomes suspended; when the coroutine is resumed again, the resumption arguments will be forwarded to `yield` which will behave as if it returned all of them.

```
function coroutine.isyieldable(): boolean
```

Returns `true` iff the currently running coroutine can yield. Yielding is prohibited when running inside metamethods like `__index` or C functions like `table.foreach` callback, with the exception of `pcall`/`xpcall`.

```
function coroutine.resume(co: thread, args: ...any): (boolean, ...any)
```

Resumes the coroutine and passes the arguments along to the suspension point. When the coroutine yields or finishes, returns `true` and all values returned at the suspension point. If an error is raised during coroutine resumption, this function returns `false` and the error object, similarly to `pcall`.

```
function coroutine.close(co: thread): (boolean, any?)
```

Closes the coroutine which puts coroutine in the dead state. The coroutine must be dead or suspended - in particular it can't be currently running. If the coroutine that's being closed was in an error state, returns `false` along with an error object; otherwise returns `true`. After closing, the coroutine can't be resumed and the coroutine stack becomes empty.

## bit32 library

All functions in the `bit32` library treat input numbers as 32-bit unsigned integers in `[0..4294967295]` range. The bit positions start at 0 where 0 corresponds to the least significant bit.

```
function bit32.arshift(n: number, i: number): number
```

Shifts `n` by `i` bits to the right (if `i` is negative, a left shift is performed instead). The most significant bit of `n` is propagated during the shift. When `i` is larger than 31, returns an integer with all bits set to the sign bit of `n`. When `i` is smaller than `-31`, 0 is returned.

```
function bit32.band(args: ...number): number
```

Performs a bitwise `and` of all input numbers and returns the result. If the function is called with no arguments, an integer with all bits set to 1 is returned.

```
function bit32.bnot(n: number): number
```

Returns a bitwise negation of the input number.

```
function bit32.bor(args: ...number): number
```

Performs a bitwise `or` of all input numbers and returns the result. If the function is called with no arguments, zero is returned.

```
function bit32.bxor(args: ...number): number
```

Performs a bitwise `xor` (exclusive or) of all input numbers and returns the result. If the function is called with no arguments, zero is returned.

```
function bit32.btest(args: ...number): boolean
```

Perform a bitwise `and` of all input numbers, and return `true` iff the result is not 0. If the function is called with no arguments, `true` is returned.

```
function bit32.extract(n: number, f: number, w: number?): number
```

Extracts bits of `n` at position `f` with a width of `w`, and returns the resulting integer. `w` defaults to `1`, so a two-argument version of `extract` returns the bit value at position `f`. Bits are indexed starting at 0. Errors if `f` and `f+w-1` are not between 0 and 31.

```
function bit32.lrotate(n: number, i: number): number
```

Rotates `n` to the left by `i` bits (if `i` is negative, a right rotate is performed instead); the bits that are shifted past the bit width are shifted back from the right.

```
function bit32.lshift(n: number, i: number): number
```

Shifts `n` to the left by `i` bits (if `i` is negative, a right shift is performed instead). When `i` is outside of `[-31..31]` range, returns 0.

```
function bit32.replace(n: number, r: number, f: number, w: number?): number
```

Replaces bits of `n` at position `f` and width `w` with `r`, and returns the resulting integer. `w` defaults to `1`, so a three-argument version of `replace` changes one bit at position `f` to `r` (which should be 0 or 1) and returns the result. Bits are indexed starting at 0. Errors if `f` and `f+w-1` are not between 0 and 31.

```
function bit32.rrotate(n: number, i: number): number
```

Rotates `n` to the right by `i` bits (if `i` is negative, a left rotate is performed instead); the bits that are shifted past the bit width are shifted back from the left.

```
function bit32.rshift(n: number, i: number): number
```

Shifts `n` to the right by `i` bits (if `i` is negative, a left shift is performed instead). When `i` is outside of `[-31..31]` range, returns 0.

```
function bit32.countlz(n: number): number
```

Returns the number of consecutive zero bits in the 32-bit representation of `n` starting from the left-most (most significant) bit. Returns 32 if `n` is zero.

```
function bit32.countrz(n: number): number
```

Returns the number of consecutive zero bits in the 32-bit representation of `n` starting from the right-most (least significant) bit. Returns 32 if `n` is zero.

```
function bit32.byteswap(n: number): number
```

Returns `n` with the order of the bytes swapped.

## utf8 library

Strings in Luau can contain arbitrary bytes; however, in many applications strings representing text contain UTF8 encoded data by convention, that can be inspected and manipulated using `utf8` library.

```
function utf8.offset(s: string, n: number, i: number?): number?
```

Returns the byte offset of the Unicode codepoint number `n` in the string, starting from the byte position `i`. When the character is not found, returns `nil` instead.

```
function utf8.codepoint(s: string, i: number?, j: number?): ...number
```

Returns a number for each Unicode codepoint in the string with the starting byte offset in `[i..j]` range. `i` defaults to 1 and `j` defaults to `i`, so a two-argument version of this function returns the Unicode codepoint that starts at byte offset `i`.

```
function utf8.char(args: ...number): string
```

Creates a string by concatenating Unicode codepoints for each input number.

```
function utf8.len(s: string, i: number?, j: number?): number?
```

Returns the number of Unicode codepoints with the starting byte offset in `[i..j]` range, or `nil` followed by the first invalid byte position if the input string is malformed.
`i` defaults to 1 and `j` defaults to `#s`, so `utf8.len(s)` returns the number of Unicode codepoints in string `s` or `nil` if the string is malformed.

```
function utf8.codes(s: string): <iterator>
```

Returns an iterator that, when used in `for` loop, produces the byte offset and the codepoint for each Unicode codepoints that `s` consists of.

## os library

```
function os.clock(): number
```

Returns a high-precision timestamp (in seconds) that doesn't have a defined baseline, but can be used to measure duration with sub-microsecond precision.

```
function os.date(s: string?, t: number?): table | string
```

Returns the table or string representation of the time specified as `t` (defaults to current time) according to `s` format string.

When `s` starts with `!`, the result uses UTC, otherwise it uses the current timezone.

If `s` is equal to `*t` (or `!*t`), a table representation of the date is returned, with keys `sec`/`min`/`hour` for the time (using 24-hour clock), `day`/`month`/`year` for the date, `wday` for week day (1..7), `yday` for year day (1..366) and `isdst` indicating whether the timezone is currently using daylight savings.

Otherwise, `s` is interpreted as a [date format string](https://www.cplusplus.com/reference/ctime/strftime/), with the valid specifiers including any of `aAbBcdHIjmMpSUwWxXyYzZ` or `%`. `s` defaults to `"%c"` so `os.date()` returns the human-readable representation of the current date in local timezone.

```
function os.difftime(a: number, b: number): number
```

Calculates the difference in seconds between `a` and `b`; provided for compatibility only. Please use `a - b` instead.

```
function os.time(t: table?): number
```

When called without arguments, returns the current date/time as a Unix timestamp. When called with an argument, expects it to be a table that contains `sec`/`min`/`hour`/`day`/`month`/`year` keys and returns the Unix timestamp of the specified date/time in UTC.

## debug library

```
function debug.info(co: thread, level: number, s: string): ...any
function debug.info(level: number, s: string): ...any
function debug.info(f: function, s: string): ...any
```

Given a stack frame or a function, and a string that specifies the requested information, returns the information about the stack frame or function.

Each character of `s` results in additional values being returned in the same order as the characters appear in the string:

- `s` returns source path for the function
- `l` returns the line number for the stack frame or the line where the function is defined when inspecting a function object
- `n` returns the name of the function, or an empty string if the name is not known
- `f` returns the function object itself
- `a` returns the number of arguments that the function expects followed by a boolean indicating whether the function is variadic or not

For example, `debug.info(2, "sln")` returns source file, current line and function name for the caller of the current function.

```
function debug.traceback(co: thread, msg: string?, level: number?): string
function debug.traceback(msg: string?, level: number?): string
```

Produces a stringified callstack of the given thread, or the current thread, starting with level `level`. If `msg` is specified, then the resulting callstack includes the string before the callstack output, separated with a newline. The format of the callstack is human-readable and subject to change.

## buffer library

Buffer is an object that represents a fixed-size mutable block of memory.

All operations on a buffer are provided using the 'buffer' library functions.

Many of the functions accept an offset in bytes from the start of the buffer.
Offset of 0 from the start of the buffer memory block accesses the first byte.

All offsets, counts and sizes should be non-negative integer numbers.

If the bytes that are accessed by any read or write operation are outside the buffer memory, an error is thrown.

```
function buffer.create(size: number): buffer
```

Creates a buffer of the requested size with all bytes initialized to 0.

Size limit is 1GB or 1,073,741,824 bytes.

```
function buffer.fromstring(str: string): buffer
```

Creates a buffer initialized to the contents of the string.

The size of the buffer equals to the length of the string.


```
function buffer.tostring(b: buffer): string
```

Returns the buffer data as a string.

```
function buffer.len(b: buffer): number
```

Returns the size of the buffer in bytes.

```
function buffer.readi8(b: buffer, offset: number): number
function buffer.readu8(b: buffer, offset: number): number
function buffer.readi16(b: buffer, offset: number): number
function buffer.readu16(b: buffer, offset: number): number
function buffer.readi32(b: buffer, offset: number): number
function buffer.readu32(b: buffer, offset: number): number
function buffer.readf32(b: buffer, offset: number): number
function buffer.readf64(b: buffer, offset: number): number
```

Used to read the data from the buffer by reinterpreting bytes at the offset as the type in the argument and converting it into a number.

Available types:

Function | Type | Range |
---------|------|-------|
readi8 | signed 8-bit integer | [-128, 127]
readu8 | unsigned 8-bit integer | [0, 255]
readi16 | signed 16-bit integer | [-32,768, 32,767]
readu16 | unsigned 16-bit integer | [0, 65,535]
readi32 | signed 32-bit integer | [-2,147,483,648, 2,147,483,647]
readu32 | unsigned 32-bit integer | [0, 4,294,967,295]
readf32 | 32-bit floating-point number | Single-precision IEEE 754 number
readf64 | 64-bit floating-point number | Double-precision IEEE 754 number

Floating-point numbers are read and written using a format specified by IEEE 754.

If a floating-point value matches any of bit patterns that represent a NaN (not a number), returned value might be converted to a different quiet NaN representation.

Read and write operations use the little endian byte order.

Integer numbers are read and written using two's complement representation.

```
function buffer.writei8(b: buffer, offset: number, value: number): ()
function buffer.writeu8(b: buffer, offset: number, value: number): ()
function buffer.writei16(b: buffer, offset: number, value: number): ()
function buffer.writeu16(b: buffer, offset: number, value: number): ()
function buffer.writei32(b: buffer, offset: number, value: number): ()
function buffer.writeu32(b: buffer, offset: number, value: number): ()
function buffer.writef32(b: buffer, offset: number, value: number): ()
function buffer.writef64(b: buffer, offset: number, value: number): ()
```

Used to write data to the buffer by converting the number into the type in the argument and reinterpreting it as individual bytes.

Ranges of acceptable values can be seen in the table above.

When writing integers, the number is converted using `bit32` library rules.

Values that are out-of-range will take less significant bits of the full number.
For example, writing 43,981 (0xabcd) using writei8 function will take 0xcd and interpret it as an 8-bit signed number -51.
It is still recommended to keep all numbers in range of the target type.

Results of converting special number values (inf/nan) to integers are platform-specific.

```
function buffer.readstring(b: buffer, offset: number, count: number): string
```

Used to read a string of length 'count' from the buffer at specified offset.

```
function buffer.writestring(b: buffer, offset: number, value: string, count: number?): ()
```

Used to write data from a string into the buffer at a specified offset.

If an optional 'count' is specified, only 'count' bytes are taken from the string.

Count cannot be larger than the string length.

```
buffer.readbits(b: buffer, bitOffset: number, bitCount: number): number
```

Used to read a range of `bitCount` bits from the buffer, at specified offset `bitOffset`, into an unsigned integer.

`bitCount` must be in `[0, 32]` range.

```
buffer.writebits(b: buffer, bitOffset: number, bitCount: number, value: number): ()
```

Used to write `bitCount` bits from `value` into the buffer at specified offset `bitOffset`.

`bitCount` must be in `[0, 32]` range.

```
function buffer.copy(target: buffer, targetOffset: number, source: buffer, sourceOffset: number?, count: number?): ()
```

Copy 'count' bytes from 'source' starting at offset 'sourceOffset' into the 'target' at 'targetOffset'.

It is possible for 'source' and 'target' to be the same. Copying an overlapping region inside the same buffer acts as if the source region is copied into a temporary buffer and then that buffer is copied over to the target.

If 'sourceOffset' is nil or is omitted, it defaults to 0.

If 'count' is 'nil' or is omitted, the whole 'source' data starting from 'sourceOffset' is copied.

```
function buffer.fill(b: buffer, offset: number, value: number, count: number?): ()
```

Sets the 'count' bytes in the buffer starting at the specified 'offset' to the 'value'.

If 'count' is 'nil' or is omitted, all bytes from the specified offset until the end of the buffer are set.

## vector library

This library implements functionality for the vector type in addition to the built-in primitive operator support. 
Default configuration uses vectors with 3 components (`x`, `y`, and `z`). 
If the _4-wide mode_ is enabled by setting the `LUA_VECTOR_SIZE` VM configuration to 4, vectors get an additional `w` component. 

Individual vector components can be accessed using the fields `x` or `X`, `y` or `Y`, `z` or `Z`, and `w` or `W` in 4-wide mode.
Since vector values are immutable, writes to individual components are not supported.

```
vector.zero
vector.one
```

Constant vectors with all components set to 0 and 1 respectively. Includes the fourth component in _4-wide mode_.

```
vector.create(x: number, y: number, z: number): vector
vector.create(x: number, y: number, z: number, w: number): vector
```

Creates a new vector with the given component values. The first constructor sets the fourth (`w`) component to 0.0 in _4-wide mode_.

```
vector.magnitude(vec: vector): number
```

Calculates the magnitude of a given vector. Includes the fourth component in _4-wide mode_.

```
vector.normalize(vec: vector): vector
```

Computes the normalized version (unit vector) of a given vector. Includes the fourth component in _4-wide mode_.

```
vector.cross(vec1: vector, vec2: vector): vector
```

Computes the cross product of two vectors. Ignores the fourth component in _4-wide mode_ and returns the 3-dimensional cross product. 

```
vector.dot(vec1: vector, vec2: vector): number
```

Computes the dot product of two vectors. Includes the fourth component in _4-wide mode_.

```
vector.angle(vec1: vector, vec2: vector, axis: vector?): number
```

Computes the angle between two vectors in radians. The axis, if specified, is used to determine the sign of the angle. Ignores the fourth component in _4-wide mode_ and returns the 3-dimensional angle.

```
vector.floor(vec: vector): vector
```

Applies `math.floor` to every component of the input vector. Includes the fourth component in _4-wide mode_.

```
vector.ceil(vec: vector): vector
```

Applies `math.ceil` to every component of the input vector. Includes the fourth component in _4-wide mode_.

```
vector.abs(vec: vector): vector
```

Applies `math.abs` to every component of the input vector. Includes the fourth component in _4-wide mode_.

```
vector.sign(vec: vector): vector
```

Applies `math.sign` to every component of the input vector. Includes the fourth component in _4-wide mode_.

```
vector.clamp(vec: vector, min: vector, max: vector): vector
```

Applies `math.clamp` to every component of the input vector. Includes the fourth component in _4-wide mode_.

```
vector.max(...: vector): vector
```

Applies `math.max` to the corresponding components of the input vectors. Includes the fourth component in _4-wide mode_. Equivalent to `vector.create(math.max((...).x), math.max((...).y), math.max((...).z), math.max((...).w))`.

```
vector.min(...: vector): vector
```

Applies `math.min` to the corresponding components of the input vectors. Includes the fourth component in _4-wide mode_. Equivalent to `vector.create(math.min((...).x), math.min((...).y), math.min((...).z), math.min((...).w))`.

---

---
slug: grammar
title: Syntax Grammar
sidebar:
  order: 3
---

This is the complete syntax grammar for Luau in EBNF. More information about the terminal nodes STRING and NUMBER
is available in the [syntax section](../syntax).

```ebnf
chunk ::= block
block ::= {stat [';']} [laststat [';']]
stat ::= varlist '=' explist |
    var compoundop exp |
    functioncall |
    'do' block 'end' |
    'while' exp 'do' block 'end' |
    'repeat' block 'until' exp |
    'if' exp 'then' block {'elseif' exp 'then' block} ['else' block] 'end' |
    'for' binding '=' exp ',' exp [',' exp] 'do' block 'end' |
    'for' bindinglist 'in' explist 'do' block 'end' |
    attributes 'function' funcname funcbody |
    attributes 'local' 'function' NAME funcbody |
    'local' bindinglist ['=' explist] |
    ['export'] 'type' NAME ['<' GenericTypeListWithDefaults '>'] '=' Type |
    ['export'] 'type' 'function' NAME funcbody

laststat ::= 'return' [explist] | 'break' | 'continue'

funcname ::= NAME {'.' NAME} [':' NAME]
funcbody ::= ['<' GenericTypeList '>'] '(' [parlist] ')' [':' ReturnType] block 'end'
parlist ::= bindinglist [',' '...' [':' (GenericTypePack | Type)]] | '...' [':' (GenericTypePack | Type)]

explist ::= {exp ','} exp

binding ::= NAME [':' Type]
bindinglist ::= binding [',' bindinglist] (* equivalent of Lua 5.1 'namelist', except with optional type annotations *)

var ::= NAME | prefixexp '[' exp ']' | prefixexp '.' NAME
varlist ::= var {',' var}
prefixexp ::= var | functioncall | '(' exp ')'
functioncall ::= prefixexp funcargs | prefixexp ':' NAME funcargs

exp ::= asexp { binop exp } | unop exp { binop exp }
ifelseexp ::= 'if' exp 'then' exp {'elseif' exp 'then' exp} 'else' exp
asexp ::= simpleexp ['::' Type]
stringinterp ::= INTERP_BEGIN exp { INTERP_MID exp } INTERP_END
simpleexp ::= NUMBER | STRING | 'nil' | 'true' | 'false' | '...' | tableconstructor | attributes 'function' funcbody | prefixexp | ifelseexp | stringinterp
funcargs ::=  '(' [explist] ')' | tableconstructor | STRING

tableconstructor ::= '{' [fieldlist] '}'
fieldlist ::= field {fieldsep field} [fieldsep]
field ::= '[' exp ']' '=' exp | NAME '=' exp | exp
fieldsep ::= ',' | ';'

compoundop ::= '+=' | '-=' | '*=' | '/=' | '//=' | '%=' | '^=' | '..='
binop ::= '+' | '-' | '*' | '/' | '//' | '^' | '%' | '..' | '<' | '<=' | '>' | '>=' | '==' | '~=' | 'and' | 'or'
unop ::= '-' | 'not' | '#'

littable ::= '{' [litfieldlist] '}'
litfieldlist ::= litfield {fieldsep litfield} [fieldsep]
litfield ::= [NAME '='] literal

literal ::= 'nil' | 'false' | 'true' | NUMBER | STRING | littable
litlist ::= literal {',' literal}

pars ::= '(' [litlist] ')' | littable | STRING 
parattr ::= NAME [pars]
attribute ::= '@' NAME | '@[' parattr {',' parattr} ']'
attributes ::= {attribute}

SimpleType ::=
    'nil' |
    SingletonType |
    NAME ['.' NAME] [ '<' [TypeParams] '>' ] |
    'typeof' '(' exp ')' |
    TableType |
    FunctionType |
    '(' Type ')'

SingletonType ::= STRING | 'true' | 'false'

Union ::= [SimpleType {'?'}] {'|' SimpleType {'?'}}
Intersection ::= [SimpleType] {'&' SimpleType}
Type ::= Union | Intersection

GenericTypePackParameter ::= NAME '...'
GenericTypeList ::= NAME [',' GenericTypeList] | GenericTypePackParameter {',' GenericTypePackParameter}

GenericTypePackParameterWithDefault ::= NAME '...' '=' (TypePack | VariadicTypePack | GenericTypePack)
GenericTypeListWithDefaults ::=
    NAME ['=' Type] [',' GenericTypeListWithDefaults] |
    GenericTypePackParameterWithDefault {',' GenericTypePackParameterWithDefault}

TypeList ::= Type [',' TypeList] | '...' Type
BoundTypeList ::= [NAME ':'] Type [',' BoundTypeList] | GenericTypePack | VariadicTypePack
TypeParams ::= (Type | TypePack | VariadicTypePack | GenericTypePack) [',' TypeParams]
TypePack ::= '(' [TypeList] ')'
GenericTypePack ::= NAME '...'
VariadicTypePack ::= '...' Type
ReturnType ::= Type | TypePack | GenericTypePack | VariadicTypePack
TableIndexer ::= ['read' | 'write'] '[' Type ']' ':' Type
TableProp ::= ['read' | 'write'] NAME ':' Type
PropList ::= TableProp [fieldsep PropList] | TableIndexer {fieldsep TableProp} 

TableType ::= '{' Type '}' | '{' [PropList] '}'
FunctionType ::= ['<' GenericTypeList '>'] '(' [BoundTypeList] ')' '->' ReturnType
```
