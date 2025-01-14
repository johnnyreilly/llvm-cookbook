# Constant Global Variable

A constant global variable comes in multiple flavours:

- A literal constant which can be assigned when declaring the variable, and
- A calculated constant which can only be assigned through executing some code.

## Literal Constant

So the following code:

```java
const i32 MAGIC_NUMBER = 123;

fn main() {
    _print_i32(MAGIC_NUMBER);
    _print_newline();
}
```

can be translated into

```llvm
; ModuleID = 'literal-constant.ll'
target triple = "x86_64-pc-linux-gnu"
target datalayout = "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-f80:128-n8:16:32:64-S128"

declare void @_initialise_lib()
declare void @_print_i32(i32)
declare void @_print_newline()

@MAGIC_NUMBER = constant i32 123

define i32 @main() {
  call void @_initialise_lib()
  %1 = load i32, i32* @MAGIC_NUMBER
  call void @_print_i32(i32 %1)
  call void @_print_newline()
  ret i32 0
}
```

Running this gives the expected result

```
123
```

## Calculated Constant

The following code is an example of a calculated constant.

```java
const i32 MAGIC_NUMBER = 123 + 654;

fn main() {
    _print_i32(MAGIC_NUMBER);
    _print_newline();
}
```

I realise that `MAGIC_NUMBER` can be replaced with the literal constant `777` however, for the sake of illustration, let's not do that but rather treat it is an arbitrary expression.

```llvm
; ModuleID = 'calculated-constant.ll'
target triple = "x86_64-pc-linux-gnu"
target datalayout = "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-f80:128-n8:16:32:64-S128"

declare void @_initialise_lib()
declare void @_print_i32(i32)
declare void @_print_newline()

@MAGIC_NUMBER = global i32 0

define i32 @main() {
  call void @_initialise_lib()                 ; Initialise runtime system

  ; Initialise @MAGIC_NUMBER
  %1 = add i32 123, 654
  store i32 %1, i32* @MAGIC_NUMBER             ; Save the caculated result to @MAGIC_NUMBER

  ; Now let's print @MAGIC_NUMBER
  %2 = load i32, i32* @MAGIC_NUMBER
  call void @_print_i32(i32 %2)
  call void @_print_newline()
  ret i32 0
}
```

which, when run, returns the following result.

```
777
```

Returning to the calculated constant, the calculation can contain an arbitrary expression calling IR functions.  For example, using the technique above

```java
const i32 SOME_NUMBER = 123 + arb_function(23);

fn arb_function(i32 a) -> i32 {
    return a * a;
}

fn main() {
    _print_i32(SOME_NUMBER);
    _print_newline();
}
```

is expressed as

```llvm
; ModuleID = 'calculated-constant-2.ll'
target triple = "x86_64-pc-linux-gnu"
target datalayout = "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-f80:128-n8:16:32:64-S128"

declare void @_initialise_lib()
declare void @_print_i32(i32)
declare void @_print_newline()

@SOME_NUMBER = global i32 0

define i32 @arb_function(i32 %0) {
  %2 = mul i32 %0, %0
  ret i32 %2
}

define i32 @main() {
  call void @_initialise_lib()                 ; Initialise runtime system

  ; Initialise @SOME_NUMBER
  %1 = call i32 @arb_function(i32 23)
  %2 = add i32 123, %1
  store i32 %2, i32* @SOME_NUMBER             ; Save the caculated result to @SOME_NUMBER

  ; Now let's print @SOME_NUMBER
  %3 = load i32, i32* @SOME_NUMBER
  call void @_print_i32(i32 %3)
  call void @_print_newline()
  ret i32 0
}
```

which, when run, returns the following result.

```
652
```
