# WASM instructions

List of WASM instructions in WAT format

### `(unreachable)`

Break execution

### `(nop)`

No operation

### `(block $label type? expr*)`

```webassembly
block $label

end
```

### `(loop $label type? expr*)`

```webassembly
loop $label

end
```

### `(if $label (result type)? i32.condition (then expr*) (else expr*)?)`

```webassembly
(if (i32.gt_u (local.get $data.len) (i32.const 3))
  (then (; do something ;)))
```

```webassembly
(if (i32.gt_s (local.get $x) (i32.const 0))
  (then (; do something ;))
  (else (; do something else ;)))
```

```webassembly
;; pushes a i32 on the stack. If $test.value is zero, then 1, otherwise 2
(if $is.zero
  (result i32)
  (i32.eqz (local.get $test.value))

  (then (i32.const 1))
  (else (i32.const 2)))
```

In linear format

```webassembly
i32.const 1 ;; test condition
if (result i32) ;; define type
  i32.const 1 ;; then branch
else
  i32.const 2 ;; else branch
end
```

### `(br $label)`

Unconditionally go to `$label`. Remember that you can only jump to previous
labels that are in scope

### `(br $label i32.condition)`

### `(br_table $labels* i32.index)`

### `(return expr?)`

### `(call $label arg-exprs*)`

### `(call-indirect ...)`

TODO

### `(drop expr)`

### `(select i32.condition then-expr else-expr)`

### `(local.get $label)`

Formerly known as `get_local`

### `(local.set $label value-expr)`

Formerly known as `set_local`

### `(local.tee $label value-expr)`

Formerly known as `tee_local`

### `(global.get $label)`

Formerly known as `get_global`

### `(global.set $label value-expr)`

Formerly known as `set_global`

## Working with linear memory

### `(type.load [offset=0] [align=0] index-expr)`

Example:

```webassembly
(i32.load offset=4 (i32.const 0)) ;; load from main memory at 0 + 8
(i32.load offset=4 (local.get $ptr)) ;; load from main memory at $ptr + 8
(i32.load (local.get $ptr)) ;; load from main memory at $ptr
```

| Mnemonic      | Range                                               | Bit pattern<br/>(*Key*: `0`-bit value, `X` from value, `S` sign, `E` exponent, `F` fraction) |
|:------------- |:--------------------------------------------------- |:-------------------------------------------------------------------- |
| `i32.load`    | `[0, 2**32)`/`[-2**31, 2**31)`                      | `0bXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`                                 |
| `i32.load8_u` | `[0, 255]`                                          | `0b000000000000000000000000XXXXXXXX`                                 |
| `i32.load8_s` | `[-128, 127]`                                       | `0bSSSSSSSSSSSSSSSSSSSSSSSSXXXXXXXX`                                 |
| `i32.load16_u` | `[0, 2**16)`                                          | `0b0000000000000000XXXXXXXXXXXXXXXX`                                 |
| `i32.load16_s` | `[-2**15, 2**15)`                                     | `0bSSSSSSSSSSSSSSSSXXXXXXXXXXXXXXXX`                                 |
| `i64.load`    | `[0, 2**64)`/`[-2**63, 2**63)`                      | `0bXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`                                 |
| `i64.load8_u` | `[0, 255]`                                          | `0b00000000000000000000000000000000000000000000000000000000XXXXXXXX`                                 |
| `i64.load8_s` | `[-128, 127]`                                       | `0bSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSXXXXXXXX`                                 |
| `i64.load16_u` | `[0, 2**16)`                                          | `0b000000000000000000000000000000000000000000000000XXXXXXXXXXXXXXXX`                                 |
| `i64.load16_s` | `[-2**15, 2**15)`                                     | `0bSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSXXXXXXXXXXXXXXXX`                                 |
| `i64.load32_u` | `[0, 2**32)`                                          | `0b00000000000000000000000000000000XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`                                 |
| `i64.load32_s` | `[-2**31, 2**31)`                                     | `0bSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`                                 |
| `i64.load`    | `[0, 2**64)`/`[-2**63, 2**63)`                      | `0bXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX` |
| `f32.load`    | IEEE 754 Single-Precision                           | `0bSEEEEEEEEFFFFFFFFFFFFFFFFFFFFFFF`                                 |
| `f64.load`    | IEEE 754 Double-Precision<br/>(JavaScript `Number`) | `0bSEEEEEEEEEEEFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF` |
