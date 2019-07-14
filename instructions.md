#WASM instructions

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

### `(i32.load [offset=0] [align=0] index-expr)`
