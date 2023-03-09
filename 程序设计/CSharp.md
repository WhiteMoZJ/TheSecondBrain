# 入门

**关于C#中与C/C++相同的地方（程序结构，数据类型及赋值等）不作赘述**
## 第一个C#程序
```cs
using System;

namespace HelloWorld
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World");
        }
    }
}
```

***程序的基本结构***
- 命名空间（可不包含）
- 类
- 类的方法（主方法Main每个程序只能有一个）
- 方法主体

## 输出/输入
`Console.WriteLine()` 是一个输出方法，将括号内的内容输出到一行，可以理解为输出文本之和自动换行；而 `Console.Write()` 则是不换行
```cs
Console.Write("Hello ");
Console.WriteLine("Hello World");
Console.Write("Hello");
```

```
Hello Hello World
Hello
```

C#中的输出内容和Python类似可以支持拼接
**要注意运算结果与字符拼接时要先计算出结果**
```cs
int a = 3, b = 4;
Console.WriteLine(a + b);
Console.WriteLine(a + "+" + b);
Console.WriteLine("a + b" + a + b);
Console.WriteLine("a + b" + (a + b));
```

```
7
3+4
a + b34
a + b7
```

在字符串前加上“`@`”可以使转义字符当作普通字符输出
```cs
string str1 = @"a\tb\tc\td";
string str2 = "a\tb\tc\td";
Console.WriteLine(str1);
Console.WriteLine(str2);
```

```
a\tb\tc\td
a       b       c       d
```
**格式化输出**：使用{}+序号可以使用字符串的格式化输出
```cs
Console.WriteLine("{0}+{1}={2}", "w", 3, 5.4);
```

通过使用 `Console.ReadLine()` 来获取用户输入信息，其返回类型为 `string` 
```cs
string str = Console.ReadLine();
Console.WriteLine(str);
```

```
> helloworld
helloworld
```
`int` 类型需要用 `Convert.ToInt32()` 来进行类型转换（具体类型有不同的转换方法）
```cs
string str = Console.ReadLine();
int a = Convert.ToInt32(str);
Console.WriteLine(a);
```
或
```cs
int a = Convert.ToInt32((string)Console.ReadLine());
Console.WriteLine(a);
```

```
> 12
12
```
需要注意的是，此方法在输入使如果输入内容非整数，则会报错
```
未经处理的异常:  System.FormatException: 输入字符串的格式不正确。
```

## 数组
与C/C++不同，C#的数组结构为
```cs
int[] arr = { 2, 3, 4, 5, 6 };
int[] arr = new int[5] { 2, 3, 4, 5, 6 };
int[] arr = new int[] { 2, 3, 4, 5, 6 };

int[5] arr; //error
```
其构建规则与C/C++有所不同
- 定义各个元素，可以不用声明大小
- 若需声明大小，需使用 `new int[+num]` 来声明
除字符数组外，其他数组需要使用遍历输出，只用单一 `Console.WriteLine()` / `Console.Write()` 会输出其类型
```cs
for (int i = 0; i < arr.Length; i++)
{
    Console.Write(arr[i] + " ");
}
Console.WriteLine(arr);
```

```
2 3 4 5 6 System.Int32[]
```
C#数组自带类变量 `Length` 用来获取数组长度

数组遍历也可以使用
```cs
foreach (int num in arr)
```
对于以数组为参数的函数，定义时需要声明为**静态 `static`**

### 字符数组 char[]
字符数组可以通过使用 `Console.WriteLine()` / `Console.Write()` 直接输出字符数组内容
```cs
char[] str = { 'a', 'b', 'c' };
Console.WriteLine(str);
```

```
abc
```

## 字符串 string
在C#中 `string` 类型提供了几种方法
- `ToLower()` 全部小写
- `ToUpper()` 全部大写
- `Trim()` 省略全部空白(空格)
- `TrimStart()` 省略字符串开头的空格
- `TrimEnd()` 省略字符串结尾的空格

```cs
string str = " Hello World ";
Console.WriteLine(str);
Console.WriteLine(str.ToLower());
Console.WriteLine(str.ToUpper());
Console.WriteLine(str.Trim());
Console.WriteLine(str.TrimStart());
Console.WriteLine(str.TrimEnd());
```

```
 Hello World 
 hello world 
 HELLO WORLD 
Hello World
Hello World 
 Hello World
```

`Split()` 可以根据指定的括号中的内容指定从什么地方将字符串分割(若空白则默认为 `' '`)，返回类型为 `string[]`，被分割的部分不会输出，但会占一个空位/空行
```cs
string[] strp = str.Split(' ');
foreach (string s in strp)
{
    Console.WriteLine(s);
}
```

```

Hello
World

```
**字符数组可以字符串转换**
```cs
char[] charstr = str.ToCharArray();
```
但是字符串到字符数组直接使用 `ToString()` 只能将字符数组类型 `System.Char[]` 本身转换为字符串
```cs
str = charstr.ToString();
Console.WriteLine(str);
```

```
System.Char[]
```
**应当遍历字符数组中的每一个字符，转化为字符串，并作累加**
```cs
string strc = "";
foreach (char c in charstr) 
{ 
	 strc += c.ToString();
}
Console.WriteLine(strc);
```

```
 Hello World 
```

## 委托 delegate
委托是一种存储函数引用的类型，定义制定了一个返回类型和一个参数列表
定义之后，可以声明委托类型变量，之和就可以把一个与委托类型、参数列表相同的函数赋值给委托
```cs
delegate double MyDelegate(double a, double b);
static double Multiply(double a, double b)
{
    return a * b;
}
static double Divide(double a, double b)
{
    return a / b;
}

double a = 34, b = 21;
MyDelegate de;
de = Multiply;         
Console.WriteLine(de(a, b));
de = Divide;
Console.WriteLine(de(a, b));
```

```
714
1.61904761904762
```

