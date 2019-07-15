# WAT Cheatsheet

### Contents

- [Usefule tooling](#useful-tooling)
- [Mixing linear and S-expressions](#mixing-linear-and-s-expressions)
- [Transforming S-expressions to linear representation](#transforming-s-expressions-to-linear-representation)
- [Switch statements](#switch-statements)
- [Styleguide](#styleguide)
- [For loop](#for-loop)

## Instructions

[List with examples](instructions.md) or full list from [spec](https://webassembly.github.io/spec/core/appendix/index-instructions.html)

## Useful tooling

* Webassembly language: https://github.com/Alhadis/language-webassembly
* Parinfer: https://shaunlebron.github.io/parinfer/
* Bracket Colorize: https://github.com/vn-ki/bracket-colorizer
* wat2wasm: https://github.com/emilbayes/wat2wasm
* wat2js (peer dependency on wat2wasm): https://github.com/mafintosh/wat2js

## Mixing linear and S-expressions

Consider the following code. It is the round function from Murmur3 Hash, and
contain many operator assignments, that make the S-expression code quite terse:

```c
uint32_t h1 = seed; // some other uint32_t
uint32_t k1 = blocks[i];

k1 *= c1;
k1 = ROTL32(k1,15);
k1 *= c2;

h1 ^= k1;
h1 = ROTL32(h1,13);
h1 = h1*5+0xe6546b64;
```

S-expression representation, without the accumulator variable `k1`:

```webassembly
(local $h1 i32)
(set_local $h1 (get_local $seed))

(set_local $h1
  (i32.add
    (i32.mul
      (i32.rotl
        (i32.xor
            (get_local $h1)
            (i32.mul
              (i32.rotl
                (i32.mul
                  (i32.load
                    (get_local $blocks.ptr))
                  (i32.const 0xcc9e2d51))
                (i32.const 15))
              (i32.const 0x1b873593)))
        (i32.const 13))
      (i32.const 5))
    (i32.const 0xe6546b64)))
```

In this case this can be made much shorter and more readable by mixing the
linear and S-expression formats:

```webassembly
(local $h1 i32)
(set_local $h1 (get_local $seed))

(i32.load (get_local $blocks.ptr)) ;; push block[i] on the stack
(i32.mul (i32.const 0xcc9e2d51))   ;; pop one value from the stack and use one arg
(i32.rotl (i32.const 15))          ;; pop one value and use one arg
(i32.mul (i32.const 0x1b873593))   ;; pop one value and use one arg
(get_local $h1)                    ;; push another value on the stack
(i32.xor)                          ;; pop two values from the stack
(i32.rotl (i32.const 13))          ;; pop one value and use one arg
(i32.mul (i32.const 5))            ;;  pop one value and use one arg
(i32.add (i32.const 0xe6546b64))   ;; pop one value and use one arg
(set_local $h1)                    ;; pop one value from the stack
```

As seen in the above case, it can be beneficial to exploit the stack while using
S-expressions where it makes the code readable.

## Transforming S-expressions to linear representation

Consider the following code:

```webassembly
(i32.sub (i32.const 5)
         (i32.const 4))
```

```webassembly
(i32.const 5)
(i32.sub (i32.const 4))
```

```webassembly
i32.const 5
i32.const 4
i32.sub
```

## Switch statements

```c
switch (branch) {
  case 3: a = 3;
  case 2: a = 2;
  case 1: a = 1; break;
  case 0: a = 0;
}
```

```webassembly
(block $break
  (block $0
    (block $1
      (block $2
        (block $3
          (block $switch
            ;; exprs running before any branching
            (br_table $0 $1 $2 $3
                      (get_local $branch)))
            ;; exprs running only if branching to $switch
          (set_local $a (i32.const 3))) ;; These these branches have fall through
        (set_local $a (i32.const 2)))
      (set_local $a (i32.const 1))
      (br $break)) ;; except this branch, which breaks
    (set_local $a (i32.const 0)))
  (nop)) ;; break (nop added for readability)
```

## Styleguide

```webassembly
;; alignment
;; if it fits within 80 chars, it can stay on a single line
(i32.add (i32.const 1) (i32.const 1))
;; align arguments
(i32.add (i32.const 1)
         (i32.const 1))

;; Or two space indentation
(i32.add
  (i32.const 1)
  (i32.const 1))

;; mimic mnemonics for custom functions
(func $i32.spy
  (param $x i32)
  (result i32))

  (call $i32.log (get_local $x))
  (get_local $x))

(func $i32x2.spy
  (param $x i64)
  (result i64))

  (call $i32.log (get_local $x))
  (get_local $x))
```

## For loop

The following will loop from `$data.ptr` until `$data.end`, one byte at a time.
Even though the convention in C is often to do `(unsigned char * ptr, size_t len)`
in WASM it can often be easier to work with `($data.ptr i32, $data.end i32)`, as
the bounds check becomes easier that way:

```webassembly
;; Maybe locals or params
(local $data.ptr i32)
(local $data.end i32)

(loop $continue
  ;; Do something here one byte at a time
  (drop (i32.load8_u (get_local $data.ptr)))

  (br_if $continue
         (i32.lt_u (tee_local $data.ptr (i32.add (get_local $data.ptr)
                                                 (i32.const 1)))
                   (get_local $data.end))))
```
