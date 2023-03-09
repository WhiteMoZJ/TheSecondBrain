全称：GNU Project Debugger
运行环境 **wsl2 with Ubuntu 18.04**

调试示例 `basic1.cpp`
```cpp
#include <iostream>

int main() {
	int j = 3;
	int k = 7;

	j += k;
	k = j * 2;
	std:cout << "Hello there" << std::endl;
}
```
默认编译质量 `g++ basic1.cpp` 得出来的 `a.out` 文件不适合gdb ，应使用指令编译及参数
```shell
g++ -g -std=c++14 basic1.cpp -o basic1
```

# 基础运行
## 执行 `run/r`
从头运行函数，如果在没有断点或错误的情况下会运行结束
```
(gdb) run
Starting program: /home/xxx/basic1
Hello there
[Inferior 1 (process 19565) exited normally]
```

## 断点 `break/b`
断点的位置可以是文件具体行数，也可以是函数名
比如
```gdb
(gdb) break main
Breakpoint 1 at 0x5555555551a9: file basic1.cpp, line 4.
```
此时执行 `run`
```gdb
(gdb) run
Starting program: /home/xxx/basic1

Breakpoint 1, main () at basic1.cpp:4
4           int j = 3;
```

## 运行到下一栈堆栈 `next/n`
此命令所调用的是程序执行时所处的位置，如果有函数调用，则会移到函数所定义的位置
```gdb
(gdb) next
5           int k = 7;
```

## 显示运行到某一行的附近文本 `list/l`
会显示该行前后几行的内容
```gdb
(gdb) list
1       #include <iostream>
2
3       int main(){
4           int j = 3;
5           int k = 7;     # current line #
6
7           j += k;
8           k = j * 2;
9           std::cout << "Hello there" << std::endl;
10      }
```

## 输出变量值 `print/p`
```gdb
(gdb) print j
$1 = 3
```
输出的可以是变量，也可以是由变量组成的表达式
```gdb
(gdb) n
7           j += k;
(gdb) n
8           k = j * 2;
(gdb) print ((j*3)+k*2)-2
$2 = 42
(gdb) p &j
$3 = (int *) 0x7fffffffdba8
```

## 退出调试 `quit/q`
```gdb
(gdb) quit
A debugging session is active.

        Inferior 1 [process 19569] will be killed.

Quit anyway? (y or n)
```

# 错误调试
```cpp
/* black_box.h */
#ifndef BLACK_BOX_H_
#define BLACK_BOX_H_

#include <iostream>

int* init(int *i){
	i = new int;
    *i = 1;
    return i;
 }

#endif
```

```cpp
/* example1.cpp */
#include "black_box.h"

void crash(int *i){
    *i = 1;
}

void f(int *i){
    int *j = i;
    j = init(j);
	crash(j);
}

int main(){
	int i;
	f(&i);
}

```
编译
```shell
g++ -g -std=c++14 example1.cpp -o example1
```

## 调用上下堆栈 `up/down`
在 `crash` 函数处标记一个断点，并且运行
```gdb
(gdb) b crash
Breakpoint 1 at 0x11d7: file example1.cpp, line 3.
(gdb) r
Starting program: /home/xxx/example1

Breakpoint 1, crash (i=0x55555556aeb0) at example1.cpp:3
3       void crash(int *i){
(gdb)
```
此时程序停止在 `crash` 函数定义处，当我们使用 `up` 向上调用堆栈
```gdb
(gdb) up
#1  0x0000555555555224 in f (i=0x7fffffffdb54) at example1.cpp:10
10           crash(j);
```
又回到了 `crash` 函数被调用的地方
而 `crash` 又是在函数 `f` 定义中被调用的，此时如果继续向上，会回到 `f` 的地方
```gdb
(gdb) up
#2  0x000055555555524e in main () at example1.cpp:15
15           f(&i);
```
`down` 的用法也类似
但 `up/down` 对 `next` 的起始位置没有影响，也就是说执行到某个断点的位置不变
```gdb
(gdb) n
4            *i = 1;
```

## 显示/不显示变量所在地址 `display/undisplay`
与 `print` 不同，`display` 可以在每次执行程序（或gdb指令）后一直显示所指定的变量的地址
```gdb
(gdb) display j
1: j = (int *) 0x55555556aeb0
(gdb) display i
2: i = (int *) 0x7fffffffdb54
(gdb) n
5       }
(gdb) n
f (i=0x7fffffffdb54) at example1.cpp:11
11      }
1: j = (int *) 0x55555556aeb0
2: i = (int *) 0x7fffffffdb54
```
反之，如果不再继续显示，则用 `undisplay` 指定即可

## 输出整个调用堆栈 `backtrace/bt`
在之前的程序中，有 `main` 调用 `f`，`f` 调用 `crash`，输出整个堆栈
```gdb
(gdb) backtrace
#0  crash (i=0x55555556aeb0) at example1.cpp:3
#1  0x0000555555555224 in f (i=0x7fffffffdb54) at example1.cpp:10
#2  0x000055555555524e in main () at example1.cpp:15
```
在递归中，函数的堆栈调用较多
例如
```cpp
#include <iostream>

int factorial(const int &n){
	if(n){
		return n * factorial(n - 1);
	}else{ // zero base case
		return 1;
	}
}

int main(){
	int n;

	std::cout << "Please enter a positive integer: ";
	if(std::cin >> n && n >= 0){
		std::cout << n << " != " << factorial(n) << std::endl;
	}else{
		 std::cout << "That's not a positive integer!" << std::endl;
	}
```

```gdb
(gdb) bt
#0  factorial (n=@0x7ffff7c73632: 1307312461) at recursive.cpp:3
#1  0x0000555555555271 in factorial (n=@0x7fffffffda94: 1) at recursive.cpp:5
#2  0x0000555555555271 in factorial (n=@0x7fffffffdad4: 2) at recursive.cpp:5
#3  0x0000555555555271 in factorial (n=@0x7fffffffdb14: 3) at recursive.cpp:5
#4  0x0000555555555271 in factorial (n=@0x7fffffffdb44: 4) at recursive.cpp:5
#5  0x0000555555555337 in main () at recursive.cpp:16
```

## `step/continue/finish`
进入断点之后，需要运行断点以后的内容，而 `run` 是从头运行，而 `step` 是继续运行一步，`continue` 是运行到下一个断点或结束，`finish` 则是运行到某一函数结束
```gdb
(gdb) step
4            if(n){
(gdb)
5                return n * factorial(n - 1);
(gdb) c
Continuing.

Breakpoint 1, factorial (n=@0x555555558040: -134548528) at recursive.cpp:3
3       int factorial(const int &n){
(gdb)
Continuing.

Breakpoint 1, factorial (n=<error reading variable>) at recursive.cpp:3
3       int factorial(const int &n){

...

(gdb)finish
Run till exit from #0  factorial (n=@0x7ffff7c73632: 1307312461) at recursive.cpp:3
factorial (n=@0x7fffffffda94: 1) at recursive.cpp:5
5                    return n * factorial(n - 1);
Value returned is $1 = 1
(gdb)
Run till exit from #0  factorial (n=@0x7fffffffda94: 1) at recursive.cpp:5
factorial (n=@0x7fffffffdad4: 2) at recursive.cpp:5
5                    return n * factorial(n - 1);
Value returned is $2 = 1

...

Run till exit from #0  factorial (n=@0x7fffffffdb44: 4) at recursive.cpp:5
0x0000555555555337 in main () at recursive.cpp:16
16                      std::cout << n << " != " << factorial(n) << std::endl;
Value returned is $5 = 24
```