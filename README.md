## MiniCRT

### How to compile MiniCRT
on `Linux`:
```
gcc -c -m32 -fno-builtin -nostdlib -fno-stack-protector entry.c malloc.c stdio.c string.c printf.c
ar -rs minicrt.a malloc.o printf.o stdio.o string.o
```
`-fnobuiltin` 关闭 gcc 的内置函数功能，默认情况下 GCC 会把 strlen、strcmp 等函数展开成它内部的实现
`-nostdlib` 不使用来自 Glibc、GCC 的库文件和启动文件，它包含了 `-nostartfile` 这个参数
`-fno-stack-protector` 关闭堆栈保护功能，较新版本的 GCC 会在 vprintf 这样的变长参数函数中插入堆栈保护函数，如果不关闭，我们使用 MiniCRT 的时候会发生 `__stack_chk_fail` 函数未定义的错误

on `Windows`:
```
cl /c /DWIN32 /GS- entry.c malloc.c printf.c stdio.c string.c
lib entry.obj malloc.obj printf.obj stdio.obj string.obj /OUT:minicrt.lib
```
`/DWIN32` 表示定义 WIN32 这个宏
`/GS-` 表示关闭堆栈保护功能。MSVC 和 GCC 一样也会在不定参数的函数中插入堆栈保护功能。

### Compile & Run test.c
on `Linux`:
```
gcc -c -m32 -ggdb -fno-builtin -nostdlib -fno-stack-protector test.c
ld -m elf_i386 -static -e mini_crt_entry entry.o test.o minicrt.a -o test
```
`-e mini_crt_entry` 指定入口函数

on `Windows`:
```
cl /c /DWIN32 test.c
link test.obj minicrt.lib kernel32.lib /NODEFAULTLIB /entry:mini_crt_entry
```


## MiniCRT++

### How to compile MiniCRT++
on `Windows`:
```
cl /c /DWIN32 /GS- entry.c malloc.c printf.c string.c atexit.c
cl /c /DWIN32 /GS- /GR- crtbegin.cpp crtend.cpp ctor.cpp new_delete.cpp iostream.cpp
lib entry.obj malloc.obj printf.obj stdio.obj string.obj ctor.obj new_delete.obj atexit.obj iostream.obj /OUT:mincrt.lib
```


on `Linux`:
In order to run MiniCRT++ in Linux, we must add `sysdep.cpp`
```
gcc -c -m32 -fno-builtin -nostdlib -fno-stack-protector entry.c malloc.c stdio.o cstring.c printf.c atexit.c
g++ -c -m32 -nostdlib -fno-rtti -fno-exceptions -fno-builtin -nostdlib -fno-stack-protector crtbegin.cpp crtend.cpp ctor.cpp new_delete.cpp sysdep.cpp iostream.cpp sysdep.cpp
ar -rs minicrt.a malloc.o printf.o stdio.o string.o ctor.o atexit.o iostream.o new_delete.o sysdep.o
```
`-fno-rtti` 关闭 RTTI
`-fno-exceptions` 关闭异常支持

### Compile & Run test.cpp
on `Windows`:
```
cl /c /DWIN32 /GR- test.cpp
link test.obj minicrt.lib kernel32.lib /NODEFAULTLIB /entry:mini_crt_entry
```

on `Linux`:
```
g++ -c -m32 -nostdinc++ -fno-rtti -fno-exceptions -fno-builtin -nostdlib -fno-stack-protector test.cpp
ld -m elf_i386 -static -e mini_crt_entry entry.o crtbegin.o test.o minicrt.a crtend.o -o test
```