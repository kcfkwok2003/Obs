## activate tool path
```
export PATH=$PATH:/.../gcc-arm-none-eabi-10.3-2021.10/bin
```

## create new 
```
$ cargo new --edition 2018 --bin app
```

## size tool
```
$ cargo size --target thumbv7m-none-eabi --bin app

text data bss dec hex filename 
0 0 0 0 0 app
```

## set config to avoid pass --target flag and flag
```
.cargo/config
---
[target.thumbv7m-none-eabi] rustflags = ["-C", "link-arg=-Tlink.x"]

[build]
target = "thumbv7-none-eabi"

```

## compile and link
```
$ cargo rustc -- --emit=obj
$ cargo rustc -- -C link-arg=-Tlink.x
```

## nm tool (but I got error)
```
$ cargo nm -- target/thumbv7m-none-eabi/debug/deps/app-*.o | grep '[0-9]* [^N] '

00000000 T rust_begin_unwind
```

## inspecting with disassembly
```
$ cargo objdump --bin app -- -d --no-show-raw-insn
```

```
app:	file format elf32-littlearm
Disassembly of section .text:  
<Reset>:     
    sub	sp, #4
    movs	r0, #42
    str	r0, [sp]
	b	0x10 <Reset+0x8>        @ imm = #-2
	b	0x10 <Reset+0x8>        @ imm = #-4`
```

## inspecting section
```
$ cargo objdump --bin app -- -s --section .vector_table
```

```
app: file format elf32-littlearm
Contents of section .vector_table: 
0000 00000120 09000000 ... ....
```

## testing
```
$ # this program will block
$ qemu-system-arm \
	-cpu cortex-m3 \
	-machine lm3s6965evb \
	-gdb tcp::3333 \
	-S \
	-nographic \ 
	-kernel target/thumbv7m-none-eabi/debug/app
```

```
$ # on a different terminal 
$ arm-none-eabi-gdb -q target/thumbv7m-none-eabi/debug/app

Reading symbols from target/thumbv7m-none-eabi/debug/app...done.
```

## debug examples
```
(gdb) target remote :3333 
Remote debugging using :3333
Reset () at src/main.rs:8 
8 pub unsafe extern "C" fn Reset() -> ! {
```

```
(gdb) # the SP has the initial value we programmed in the vector table 
(gdb) print/x $sp 
$1 = 0x20010000
```

```
(gdb) step 
9 let _x = 42; 
(gdb) step
12 loop {}
```

```  
(gdb) # next we inspect the stack variable `_x` (gdb) print _x 
$2 = 42 
(gdb) print &_x
$3 = (i32 *) 0x2000fffc
```

```
(gdb) quit
```