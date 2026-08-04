# Lua Code Golf Cheatsheet

Notes for code golfing in modern Lua (v5.3+)


## Standard Tips

Posts and references on Lua code golfing typically mention these:

1. **Omit Whitespace:** You can omit whitespace between a number, identifier,
   or keyword and an adjacent punctuation character. For example, these are all
   valid:
   - `do return{1,2,3}end`
   - `print(3)a=4`
   - `a=3("foo"):gsub('o','O')`

   Note that, although some sources insist otherwise, *it is not valid* to omit
   the space between a number and an identifier or keyword.

2. **Omit Parentheses:** Function calls with a single string or table argument
   do not need parentheses.

   These are equivalent:
   - `print"foo"`
   - `print("foo")`

   These are equivalent:
   - `table.concat{1,2,3}`
   - `table.concat({1,2,3})`

3. **Global Variables:** You can often use `a=1` instead of `local a=1`.

4. **Numeric Loops:** Using a `for` loop with a numeric index is shorter than
   using `ipairs()`.

   These are equivalent:
   - `for i=1,9 do print(t[i])end` (27 bytes)
   - `for _,v in ipairs(t)do print(v)end` (34 bytes)

5. **Functions with load:** You can declare functions using `load()` instead of
   `function() end`. The runtime efficiency of loaded functions is lower, but
   source code size matters more for code golf.

   These are equivalent:
   - `f=load"return...+1"` (19 bytes)
   - `f=function(n)return n+1 end` (27 bytes)

   Note that the arguments to the load function are available as the special
   `...` *vararg expression* which behaves like multi-valued expressions
   created with commas. If you use `...` in a `print()`, a `return`, or a
   multiple assignment, you may get multi-valued behavior. If you use `...` in
   place that expects a single-valued expression, you'll get a single value
   (the first function argument). Read Lua language docs for details.

   **CAUTION**: Functions created with `load()` can see globals (contents of
   `_ENV`) but not locals. You may be used to using lexical closures with
   `function() end` declarations, but `load` works differently (it does not use
   lexical scope). Loaded functions can't see anything outside of the global
   scope unless you burn extra bytes passing an upvalue table to `load()`.

   Consequences of `i` being a declared as a local var within `for` loop:
   - Raises error: `   for i=1,3 do          f=load"print(i*i)" f() end`
   - This works: `j=0 for i=1,3 do j=i      f=load"print(j*j)" f() end`
   - This works: `    for i=1,3 do _ENV.i=i f=load"print(i*i)" f() end`

6. **Multiple Assignment for Multi-Valued Functions:** To swap variables or
   otherwise compute a multi-valued function of multiple variables, using
   multiple assignment lets you avoid declaring temporary variables.

   These are equivalent:
   - `a,b=b,a+b` (9 bytes)
   - `t=a+b a=b b=t` (13 bytes)

7. **String Method Calls:** You can call string functions with the method call
   syntax (colon) for either a string literal or a string variable. Be sure to
   put parentheses around literals. Method calls work for other things, but in
   code golf they're most commonly useful for string functions.

   These are equivalent:
   - `print(('foo'):upper())` (22 bytes)
   - `s='foo'print(s:upper())` (23 bytes)
   - `print(string.upper'foo')` (24 bytes)

8. **Ternary Conditional:** It's usually possible to use the and/or ternary
   idiom for conditional assignments or control flow rather than using an
   if/then/else/end statement. By carefully ordering the elements of the
   condition, it's sometimes possible to omit whitespace.

   **CAUTION**: The value after the `and` must be truthy (not `nil`, not
   `false`).

   These are equivalent:
   - `a=3 b=5 c=a>b and'A'or'B'print(c)` (33 bytes)
   - `a=3 b=5 if a>b then c='A'else c='B'end print(c)` (47 bytes)
   - `a=3 b=5 if a>b then print'A'else print'B'end` (44 bytes)

   These are equivalent:
   - `print(string.byte('A')>95 and 1 or 2)` (37 bytes)
   - `print(95<string.byte('A')and 1 or 2)` (36 bytes)
   - `print(95<('').byte('A')and 1 or 2)` (34 bytes)
   - `print(95<('A'):byte()and 1 or 2)` (32 bytes)

9. **Iterate with gsub or gmatch:** To iterate over each character of a string,
   you can pass a function argument to `string.gsub()` *without* needing to use
   a `for` loop. It's also possible to use `string.gmatch()` with `for in`
   loops, but the `gmatch()` option tends to be longer.

   These are equivalent (`gsub` wins):
   - `('abcdefg'):gsub('.',load'print(string.byte(...))')` (51 bytes)
   - `('abcdefg'):gsub('.',load'print((...):byte())')` (47 bytes)
   - `for c in ('abcdefg'):gmatch'.'do print(c:byte())end` (51 bytes)
   - `s='abcdefg'for c in s:gmatch'.'do print(c:byte())end` (52 bytes)
   - `s='abcdefg'for i=1,#s do print(s:sub(i,i):byte())end` (52 bytes)

   These are equivalent (`gsub` wins):
   - `s=('123456'):gsub('(%d)(%d)',load"a,b=...return b..a")print(s)`
   - `s=''for a,b in('123456'):gmatch'(%d)(%d)'do s=s..b..a end print(s)`

   These are equivalent (`gsub` wins):
   - `('abc d#efg'):gsub('%a',load"print((...):byte())")`
   - `for c in('abc d#efg'):gmatch'%a'do print(c:byte())end`


## Special Tricks

These are less common ways of shortening special tasks.

1. **Conditional Sum with Multiply:** Sometimes a conditional addition can be
   replaced by a carefully crafted multiply-add expression.

   These are equivalent:
   - `s=i%j<1 and s+j or 0` (20 bytes)
   - `s=s+(1>>i%j)*j` (14 bytes)

   In the second example, right shifting 1 by anything greater than 0 yields 0.
   So, the second term evaluates to `1*j` when `i` is a multiple of `j`, or
   `0*j` otherwise.

2. **Equivalent Conditionals:** When you know the possible values a variable
   might hold, you can often save a byte by using `<` or `>` instead of `==` or
   `~=`.

   When you know `i` will be 0 or greater, these are equivalent:
   - `i<1`
   - `i==0`

3. **Short-Circuit Conditional:** An assignment statement with a short-circuit
   boolean expression can take the place of an if/then/end statement.

   These are equivalent:
   - `_=i&1<1 or print(i)` (19 bytes)
   - `if i&1>0 then print(i)end` (25 bytes)

4. **Packed Bitfields:** This only works in special cases. But, for short
   series where the values can be scaled or translated to fit in the range 1 to
   62, you can pack the values into a bitfield. The unpacking code can be very
   short:<br>
   `for i=1,62 do _=0x10203040506>>i&1<1 or print(i)end`

5. **Algebric Equivalence:** Particularly for problems like calculating the sum
   of a series, it may be possible to do sneaky algebra tricks. You may be able
   to get an equivalent result from a simpler formula, perhaps combined with a
   different range for the loop index variable. Do not assume that wikipedia or
   other similar sources will give you the most efficient formula for a series
   or the most efficient implementation for an algorithm.


## Non-Helpful Advice

Some sources avocate techniques that are out of date, misleading, or otherwise
not helpful for code golfing with modern Lua (v5.3+).

1. **Omit whitespace between number and identifier:** This does not work. The
   Lexer won't allow stuff like `for i=1,3do`. You'll get an error about a
   `malformed number near '3do'`.

2. **Multiple Assignment Declaration:** For declaring globals, multiple
   assignment does not add or save any bytes. For example: `a,b=1,2` and
   `a=1 b=2` are both 7 bytes.

3. **Use pair() to Iterate Over Array:** Some sources advocate using `pair()`
   to iterate over an array table (rather than `ipair()`). That's bad advice
   for two reasons: First, `pair()` only works correctly for arrays in special
   cases. Second, indexed loops are shorter.

   These are equivalent (requires special case of a trivial array literal):
   - `for i,v in ipairs({1,2,3})do print(i,v)end`
   - `for k,v in pairs({1,2,3})do print(k,v)end`
   - `t={1,2,3}for i=1,#t do print(i,t[i])end` (note indexed is shorter!)

   **BUT**, the `pairs()` method will give wrong results if you invoke the
   interpreter to use the `arg` table (not a trivial array):
   - `lua -e 'for i,v in ipairs(arg)do print(i,v)end' /dev/null 1 2 3`
   - `lua -e 'for k,v in pairs(arg)do print(k,v)end' /dev/null 1 2 3`

   The problem is that `arg` has additional values in the table (at negative
   and 0 indexes) which `pairs()` will include but `ipairs()` will omit.
