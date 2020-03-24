# 打印

- **Tips-33:** 打印ASCII和宽字符字符串

**示例代码：**

```c
#include <stdio.h>
#include <wchar.h>

int main(void)
{
        char str1[] = "abcd";
        wchar_t str2[] = L"abcd";
        
        return 0;
}
```

**技巧：**
用gdb调试程序时，可以使用`x/s`命令打印`ASCII`字符串。以上面程序为例：
```shell
Temporary breakpoint 1, main () at a.c:6
6               char str1[] = "abcd";
(gdb) n
7               wchar_t str2[] = L"abcd";
(gdb) 
9               return 0;
(gdb) x/s str1
0x804779f:      "abcd"
```
可以看到打印出了`str1`字符串的值。

打印宽字符字符串时，要根据宽字符的长度决定如何打印。仍以上面程序为例：
```shell
Temporary breakpoint 1, main () at a.c:6
6               char str1[] = "abcd";
(gdb) n
7               wchar_t str2[] = L"abcd";
(gdb) 
9               return 0;
(gdb) p sizeof(wchar_t)
$1 = 4
(gdb) x/ws str2
0x8047788:      U"abcd"
```
由于当前平台宽字符的长度为`4`个字节，则用`x/ws`命令。如果是`2`个字节，则用`x/hs`命令。


- **Tips-34:** 打印STL容器中的内容

**示例代码：**

```c
#include <iostream>
#include <vector>

using namespace std;

int main ()
{
  vector<int> vec(10); // 10 zero-initialized elements

  for (int i = 0; i < vec.size(); i++)
    vec[i] = i;

  cout << "vec contains:";
  for (int i = 0; i < vec.size(); i++)
    cout << ' ' << vec[i];
  cout << '\n';

  return 0;
}
```

**技巧一：**
在gdb中，如果要打印`C++ STL`容器的内容，缺省的显示结果可读性很差：
```shell
(gdb) p vec
$1 = {<std::_Vector_base<int, std::allocator<int> >> = {
    _M_impl = {<std::allocator<int>> = {<__gnu_cxx::new_allocator<int>> = {<No data fields>}, <No data fields>}, _M_start = 0x404010, _M_finish = 0x404038, 
          _M_end_of_storage = 0x404038}}, <No data fields>}
```

`gdb 7.0`之后，可以使用`gcc`提供的`python`脚本，来改善显示结果：

```shell
(gdb) p vec
$1 = std::vector of length 10, capacity 10 = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
```
某些发行版(Fedora 11+)，不需要额外的设置工作。可在gdb命令行下验证(若没有显示，可按下文的方法进行设置)。
```shell
(gdb) info pretty-printer
```
方法如下:

1、获得python脚本，建议使用gcc默认安装的
```shell
 sudo find / -name "*libstdcxx*"
```
2、若本机查找不到python脚本，建议下载gcc对应版本源码包，相对目录如下
```shell
gcc-4.8.1/libstdc++-v3/python
```
3、也可直接下载最新版本
```shell
svn co svn://gcc.gnu.org/svn/gcc/trunk/libstdc++-v3/python
```
4、将如下代码添加到.gdbinit文件中
```shell
python
 import sys
 sys.path.insert(0, '/home/maude/gdb_printers/python')
 from libstdcxx.v6.printers import register_libstdcxx_printers
 register_libstdcxx_printers (None)
 end
```
**技巧二：**
`p vec`的输出无法阅读，但能给我们提示，从而得到无需脚本支持的技巧：

```shell
(gdb) p *(vec._M_impl._M_start)@vec.size()
$2 = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
```

**技巧三：**
将[dbinit_stl_views](http://www.yolinux.com/TUTORIALS/src/dbinit_stl_views-1.03.txt) 下载下来,，执行命令

```shell
cat dbinit_stl_views-1.03.txt >> ~/.gdbinit
```
即可 一些常用的容器及其对应的命令关系
```shell
std::vector<T>  pvector stl_variable 
std::list<T>  plist stl_variable T 
std::map<T,T>  pmap stl_variable 
std::multimap<T,T>  pmap stl_variable 
std::set<T>  pset stl_variable T 
std::multiset<T>  pset stl_variable 
std::deque<T>  pdequeue stl_variable 
std::stack<T>  pstack stl_variable 
std::queue<T>  pqueue stl_variable 
std::priority_queue<T>  ppqueue stl_variable 
std::bitset<n><td>  pbitset stl_variable 
std::string  pstring stl_variable 
std::widestring  pwstring stl_variable  
```
更多详情，参考配置中的帮助

- **Tips-35:** 打印大数组中的内容

**示例代码：**

```c
int main()
{
  int array[201];
  int i;

  for (i = 0; i < 201; i++)
    array[i] = i;

  return 0;
}

```

**技巧：**
在gdb中，如果要打印大数组的内容，缺省最多会显示200个元素：

```shell
(gdb) p array
$1 = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 
  48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 
  95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119, 120, 121, 122, 123, 124, 125, 126, 127, 128, 129, 130, 131, 132, 
  133, 134, 135, 136, 137, 138, 139, 140, 141, 142, 143, 144, 145, 146, 147, 148, 149, 150, 151, 152, 153, 154, 155, 156, 157, 158, 159, 160, 161, 162, 163, 164, 165, 166, 167, 168, 169, 
  170, 171, 172, 173, 174, 175, 176, 177, 178, 179, 180, 181, 182, 183, 184, 185, 186, 187, 188, 189, 190, 191, 192, 193, 194, 195, 196, 197, 198, 199...}
```
可以使用如下命令，设置这个最大限制数：
```shell
(gdb) set print elements number-of-elements
```
也可以使用如下命令，设置为没有限制：
```shell
(gdb) set print elements 0
```
或
```shell
(gdb) set print elements unlimited
(gdb) p array
$2 = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 
  48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 
  95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119, 120, 121, 122, 123, 124, 125, 126, 127, 128, 129, 130, 131, 132, 
  133, 134, 135, 136, 137, 138, 139, 140, 141, 142, 143, 144, 145, 146, 147, 148, 149, 150, 151, 152, 153, 154, 155, 156, 157, 158, 159, 160, 161, 162, 163, 164, 165, 166, 167, 168, 169, 
  170, 171, 172, 173, 174, 175, 176, 177, 178, 179, 180, 181, 182, 183, 184, 185, 186, 187, 188, 189, 190, 191, 192, 193, 194, 195, 196, 197, 198, 199, 200}
```
- **Tips-36:** 打印数组中任意连续元素值

**示例代码：**

```c
int main(void)
{
  int array[201];
  int i;

  for (i = 0; i < 201; i++)
    array[i] = i;

  return 0;
}
```

**技巧：**

在gdb中，如果要打印数组中任意连续元素的值，可以使用`p array[index]@num`命令（`p`是`print`命令的缩写），其中`index`是数组索引（从`0`开始计数），`num`是连续多少个元素。以上面代码为例：

```shell
(gdb) p array
$8 = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31,
  32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62,
  63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93,
  94, 95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119,
  120, 121, 122, 123, 124, 125, 126, 127, 128, 129, 130, 131, 132, 133, 134, 135, 136, 137, 138, 139, 140, 141, 142, 143, 144,
  145, 146, 147, 148, 149, 150, 151, 152, 153, 154, 155, 156, 157, 158, 159, 160, 161, 162, 163, 164, 165, 166, 167, 168, 169,
  170, 171, 172, 173, 174, 175, 176, 177, 178, 179, 180, 181, 182, 183, 184, 185, 186, 187, 188, 189, 190, 191, 192, 193, 194,
  195, 196, 197, 198, 199...}
(gdb) p array[60]@10
$9 = {60, 61, 62, 63, 64, 65, 66, 67, 68, 69}
```

可以看到打印了`array`数组第60~69个元素的值。
如果要打印从数组开头连续元素的值，也可使用这个命令：`p *array@num`：

```shell
(gdb) p *array@10
$2 = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
```

- **Tips-37:** 打印数组的索引下标

**示例代码：**

```c
#include <stdio.h>

int num[10] = { 
  1 << 0,
  1 << 1,
  1 << 2,
  1 << 3,
  1 << 4,
  1 << 5,
  1 << 6,
  1 << 7,
  1 << 8,
  1 << 9
};

int main (void)
{
  int i;

  for (i = 0; i < 10; i++)
    printf ("num[%d] = %d\n", i, num[i]);

  return 0;
}
```

**技巧：**
在gdb中，当打印一个数组时，缺省是不打印索引下标的：
```shell
(gdb) p num
$1 = {1, 2, 4, 8, 16, 32, 64, 128, 256, 512}
```
如果要打印索引下标，则可以通过如下命令进行设置：
```shell
(gdb) set print array-indexes on

(gdb) p num
$2 = {[0] = 1, [1] = 2, [2] = 4, [3] = 8, [4] = 16, [5] = 32, [6] = 64, [7] = 128, [8] = 256, [9] = 512}
```

- **Tips-38:** 格式化打印数组

**示例代码：**
利用call函数控制数组的输出格式：
```c
#include <stdio.h>

int matrix[10][10];

/* 格式化输出数组 */
void print(int matrix[][10], int m, int n) {
    int i, j;
    for (i = 0; i < m; ++i) {
        for (j = 0; j < n; ++j)
            printf("%d ", matrix[i][j]);
        printf("\n");
    }
}

int main(int argc, char const* argv[]) {
    int i, j;
    for (i = 0; i < 10; ++i)
        for (j = 0; j < 10; ++j)
            matrix[i][j] = i*10 + j;
    return 0;
}
```

**技巧：**

```shell
(gdb) b 20
Breakpoint 1 at 0x40065e: file test.c, line 20.
(gdb) r
Starting program: /home/zhaoyu/codelab/algorithm/a.out 

Breakpoint 1, main (argc=1, argv=0x7fffffffdc88) at test.c:20
20          return 0;
(gdb) call print(matrix, 10, 10) // 通过函数调用格式化输出数组
0 1 2 3 4 5 6 7 8 9 
10 11 12 13 14 15 16 17 18 19 
20 21 22 23 24 25 26 27 28 29 
30 31 32 33 34 35 36 37 38 39 
40 41 42 43 44 45 46 47 48 49 
50 51 52 53 54 55 56 57 58 59 
60 61 62 63 64 65 66 67 68 69 
70 71 72 73 74 75 76 77 78 79 
80 81 82 83 84 85 86 87 88 89 
90 91 92 93 94 95 96 97 98 99 
```

- **Tips-39:** 打印函数局部变量的值

**示例代码：**

```c
#include <stdio.h>

void fun_a(void)
{
	int a = 0;
	printf("%d\n", a);
}

void fun_b(void)
{
	int b = 1;
	fun_a();
	printf("%d\n", b);
}

void fun_c(void)
{
	int c = 2;
	fun_b();
	printf("%d\n", c);
}

void fun_d(void)
{
	int d = 3;
	fun_c();
	printf("%d\n", d);
}

int main(void)
{
	int var = -1;
	fun_d();
	return 0;
}
```

**技巧一：**
如果要打印函数局部变量的值，可以使用`bt full`命令（`bt`是`backtrace`的缩写）。首先我们在函数`fun_a`里打上断点，当程序断住时，显示调用栈信息：

```shell
(gdb) bt
#0  fun_a () at a.c:6
#1  0x000109b0 in fun_b () at a.c:12
#2  0x000109e4 in fun_c () at a.c:19
#3  0x00010a18 in fun_d () at a.c:26
#4  0x00010a4c in main () at a.c:33
```
接下来，用`bt full`命令显示各个函数的局部变量值：
```shell
(gdb) bt full
#0  fun_a () at a.c:6
        a = 0
#1  0x000109b0 in fun_b () at a.c:12
        b = 1
#2  0x000109e4 in fun_c () at a.c:19
        c = 2
#3  0x00010a18 in fun_d () at a.c:26
        d = 3
#4  0x00010a4c in main () at a.c:33
        var = -1
```
也可以使用如下`bt full n`，意思是从内向外显示`n`个栈桢，及其局部变量，例如：
```shell
(gdb) bt full 2
#0  fun_a () at a.c:6
        a = 0
#1  0x000109b0 in fun_b () at a.c:12
        b = 1
(More stack frames follow...)
```
而`bt full -n`，意思是从外向内显示`n`个栈桢，及其局部变量，例如：
```shell
(gdb) bt full -2
#3  0x00010a18 in fun_d () at a.c:26
        d = 3
#4  0x00010a4c in main () at a.c:33
        var = -1
```

**技巧一：**
如果只是想打印当前函数局部变量的值，可以使用如下命令：
```shell
(gdb) info locals
a = 0
```

- **Tips-40:** 打印进程内存信息

**技巧：**
用gdb调试程序时，如果想查看进程的内存映射信息，可以使用`i proc mappings`命令（`i`是`info`命令缩写），例如:
```shell
(gdb) i proc mappings
process 27676 flags:
PR_STOPPED Process (LWP) is stopped
PR_ISTOP Stopped on an event of interest
PR_RLC Run-on-last-close is in effect
PR_MSACCT Microstate accounting enabled
PR_PCOMPAT Micro-state accounting inherited on fork
PR_FAULTED : Incurred a traced hardware fault FLTBPT: Breakpoint trap

Mapped address spaces:

    Start Addr   End Addr       Size     Offset   Flags
     0x8046000  0x8047fff     0x2000 0xfffff000 -s--rwx
     0x8050000  0x8050fff     0x1000          0 ----r-x
     0x8060000  0x8060fff     0x1000          0 ----rwx
    0xfee40000 0xfef4efff   0x10f000          0 ----r-x
    0xfef50000 0xfef55fff     0x6000          0 ----rwx
    0xfef5f000 0xfef66fff     0x8000   0x10f000 ----rwx
    0xfef67000 0xfef68fff     0x2000          0 ----rwx
    0xfef70000 0xfef70fff     0x1000          0 ----rwx
    0xfef80000 0xfef80fff     0x1000          0 ---sr--
    0xfef90000 0xfef90fff     0x1000          0 ----rw-
    0xfefa0000 0xfefa0fff     0x1000          0 ----rw-
    0xfefb0000 0xfefb0fff     0x1000          0 ----rwx
    0xfefc0000 0xfefeafff    0x2b000          0 ----r-x
    0xfeff0000 0xfeff0fff     0x1000          0 ----rwx
    0xfeffb000 0xfeffcfff     0x2000    0x2b000 ----rwx
    0xfeffd000 0xfeffdfff     0x1000          0 ----rwx
```
首先输出了进程的`flags`，接着是进程的内存映射信息。

此外，也可以用`i files`（还有一个同样作用的命令：`i target`）命令，它可以更详细地输出进程的内存信息，包括引用的动态链接库等等，例如：
```shell
(gdb) i files
Symbols from "/data1/nan/a".
Unix /proc child process:
    Using the running image of child Thread 1 (LWP 1) via /proc.
    While running this, GDB does not access memory from...
Local exec file:
    `/data1/nan/a', file type elf32-i386-sol2.
    Entry point: 0x8050950
    0x080500f4 - 0x08050105 is .interp
    0x08050108 - 0x08050114 is .eh_frame_hdr
    0x08050114 - 0x08050218 is .hash
    0x08050218 - 0x08050418 is .dynsym
    0x08050418 - 0x080507e6 is .dynstr
    0x080507e8 - 0x08050818 is .SUNW_version
    0x08050818 - 0x08050858 is .SUNW_versym
    0x08050858 - 0x08050890 is .SUNW_reloc
    0x08050890 - 0x080508c8 is .rel.plt
    0x080508c8 - 0x08050948 is .plt
    ......
	0xfef5fb58 - 0xfef5fc48 is .dynamic in /usr/lib/libc.so.1
    0xfef5fc80 - 0xfef650e2 is .data in /usr/lib/libc.so.1
    0xfef650e2 - 0xfef650e2 is .bssf in /usr/lib/libc.so.1
    0xfef650e8 - 0xfef65be0 is .picdata in /usr/lib/libc.so.1
    0xfef65be0 - 0xfef666a7 is .data1 in /usr/lib/libc.so.1
    0xfef666a8 - 0xfef680dc is .bss in /usr/lib/libc.so.1
```

- **Tips-41:** 打印静态变量的值

**示例代码：**

```c
/* main.c */
extern void print_var_1(void);
extern void print_var_2(void);

int main(void)
{
  print_var_1();
  print_var_2();
  return 0;
}

/* static-1.c */
#include <stdio.h>

static int var = 1;

void print_var_1(void)
{ 
  printf("var = %d\n", var);
} 

/* static-2.c */
#include <stdio.h>

static int var = 2;

void print_var_2(void)
{ 
  printf("var = %d\n", var);
} 
```

**技巧：**
在gdb中，如果直接打印静态变量，则结果并不一定是你想要的：
```shell
$ gcc -g main.c static-1.c static-2.c
$ gdb -q ./a.out
(gdb) start
(gdb) p var
$1 = 2

$ gcc -g main.c static-2.c static-1.c
$ gdb -q ./a.out
(gdb) start
(gdb) p var
$1 = 1
```
你可以显式地指定文件名（上下文）：
```shell
(gdb) p 'static-1.c'::var
$1 = 1
(gdb) p 'static-2.c'::var
$2 = 2
```

- **Tips-42:** 打印变量的类型和所在文件

**示例代码：**

```c
#include <stdio.h>

struct child {
  char name[10];
  enum { boy, girl } gender;
};

struct child he = { "Tom", boy };

int main (void)
{
  static struct child she = { "Jerry", girl };
  printf ("Hello %s %s.\n", he.gender == boy ? "boy" : "girl", he.name);
  printf ("Hello %s %s.\n", she.gender == boy ? "boy" : "girl", she.name);
  return 0;
}
```

**技巧：**
在gdb中，可以使用如下命令查看变量的类型：
```shell
(gdb) whatis he
type = struct child
```

如果想查看详细的类型信息：
```shell
(gdb) ptype he
type = struct child {
    char name[10];
    enum {boy, girl} gender;
}
```

如果想查看定义该变量的文件：
```shell
(gdb) i variables he
All variables matching regular expression "he":

File variable.c:
struct child he;

Non-debugging symbols:
0x0000000000402030  she
0x00007ffff7dd3380  __check_rhosts_file
```

哦，gdb会显示所有包含（匹配）该表达式的变量。如果只想查看完全匹配给定名字的变量：
```shell
(gdb) i variables ^he$
All variables matching regular expression "^he$":

File variable.c:
struct child he;
```
注：`info variables`不会显示局部变量，即使是`static`的也没有太多的信息。

- **Tips-43:** 打印内存的值

**示例代码：**

```c
#include <stdio.h>

int main(void)
{
        int i = 0;
        char a[100];

        for (i = 0; i < sizeof(a); i++)
        {
                a[i] = i;
        }

        return 0;
}
```

**技巧：**
gdb中使用`x`命令来打印内存的值，格式为`x/nfu addr`。含义为以f格式打印从`addr`开始的`n`个长度单元为`u`的内存值。参数具体含义如下：

- `n`：输出单元的个数。

- `f`：是输出格式。比如`x`是以`16`进制形式输出，`o`是以`8`进制形式输出,等等。

- `u`：标明一个单元的长度。`b`是一个`byte`，`h`是两个`byte`（`halfword`），`w`是四个`byte`(`word`)，`g`是八个`byte`(giant word)。

以上面程序为例：

- 以16进制格式打印数组前a16个byte的值：

```shell
(gdb) x/16xb a
0x7fffffffe4a0: 0x00    0x01    0x02    0x03    0x04    0x05    0x06    0x07
0x7fffffffe4a8: 0x08    0x09    0x0a    0x0b    0x0c    0x0d    0x0e    0x0f
```

- 以无符号10进制格式打印数组a前16个byte的值：

```shell
(gdb) x/16ub a
0x7fffffffe4a0: 0       1       2       3       4       5       6       7
0x7fffffffe4a8: 8       9       10      11      12      13      14      15
```
- 以2进制格式打印数组前16个abyte的值：

```shell
(gdb) x/16tb a
0x7fffffffe4a0: 00000000        00000001        00000010        00000011        00000100        00000101        00000110        00000111
0x7fffffffe4a8: 00001000        00001001        00001010        00001011        00001100        00001101        00001110        00001111
```

- 以16进制格式打印数组a前16个word（4个byte）的值：

```shell
(gdb) x/16xw a
0x7fffffffe4a0: 0x03020100      0x07060504      0x0b0a0908      0x0f0e0d0c
0x7fffffffe4b0: 0x13121110      0x17161514      0x1b1a1918      0x1f1e1d1c
0x7fffffffe4c0: 0x23222120      0x27262524      0x2b2a2928      0x2f2e2d2c
0x7fffffffe4d0: 0x33323130      0x37363534      0x3b3a3938      0x3f3e3d3c
```

- **Tips-44:** 打印源代码行

**示例代码：**

```assembly
$ gdb -q `which gdb`
(gdb) l
15	
16	   You should have received a copy of the GNU General Public License
17	   along with this program.  If not, see <http://www.gnu.org/licenses/>.  */
18	
19	#include "defs.h"
20	#include "main.h"
21	#include <string.h>
22	#include "interps.h"
23	
24	int
```

**技巧：**
如上所示，在gdb中可以使用`list`(简写为`l`)命令来显示源代码以及行号。`list`命令可以指定行号，函数：

```shell
(gdb) l 24
(gdb) l main
```
还可以指定向前或向后打印：

```shell
(gdb) l -
(gdb) l +
```
还可以指定范围：

```shell
(gdb) l 1,10
```

- **Tips-45:** 每行打印一个结构体成员

**示例代码：**

```c
#include <stdio.h>
#include <pthread.h>

typedef struct
{
        int a;
        int b;
        int c;
        int d;
        pthread_mutex_t mutex;
}ex_st;

int main(void) {
        ex_st st = {1, 2, 3, 4, PTHREAD_MUTEX_INITIALIZER};
        printf("%d,%d,%d,%d\n", st.a, st.b, st.c, st.d);
        return 0;
}

```

**技巧：**
默认情况下，gdb以一种*紧凑*的方式打印结构体。以上面代码为例：

```shell
(gdb) n
15              printf("%d,%d,%d,%d\n", st.a, st.b, st.c, st.d);
(gdb) p st
$1 = {a = 1, b = 2, c = 3, d = 4, mutex = {__data = {__lock = 0, __count = 0, __owner = 0, __nusers = 0, __kind = 0,
      __spins = 0, __list = {__prev = 0x0, __next = 0x0}}, __size = '\000' <repeats 39 times>, __align = 0}}
```

可以看到结构体的显示很混乱，尤其是结构体里还嵌套着其它结构体时。

可以执行`set print pretty on`命令，这样每行只会显示结构体的一名成员，而且还会根据成员的定义层次进行缩进：
```shell
(gdb) set print pretty on
(gdb) p st
$2 = {
  a = 1,
  b = 2,
  c = 3,
  d = 4,
  mutex = {
    __data = {
      __lock = 0,
      __count = 0,
      __owner = 0,
      __nusers = 0,
      __kind = 0,
      __spins = 0,
      __list = {
        __prev = 0x0,
        __next = 0x0
      }
    },
    __size = '\000' <repeats 39 times>,
    __align = 0
  }
}
```


- **Tips-46:** 按照派生类型打印对象

**示例代码：**

```c
#include <iostream>
using namespace std;

class Shape {
 public:
  virtual void draw () {}
};

class Circle : public Shape {
 int radius;
 public:
  Circle () { radius = 1; }
  void draw () { cout << "drawing a circle...\n"; }
};

class Square : public Shape {
 int height;
 public:
  Square () { height = 2; }
  void draw () { cout << "drawing a square...\n"; }
};

void drawShape (class Shape &p)
{
  p.draw ();
}

int main (void)
{
  Circle a;
  Square b;
  drawShape (a);
  drawShape (b);
  return 0;
}
```

**技巧：**
在gdb中，当打印一个对象时，缺省是按照声明的类型进行打印：

```shell
(gdb) frame
#0  drawShape (p=...) at object.cxx:25
25	  p.draw ();
(gdb) p p
$1 = (Shape &) @0x7fffffffde90: {_vptr.Shape = 0x400a80 <vtable for Circle+16>}
```

在这个例子中，p虽然声明为`class Shape`，但它实际的派生类型可能为`class Circle`和`Square`。如果要缺省按照派生类型进行打印，则可以通过如下命令进行设置：
```shell
(gdb) set print object on

(gdb) p p
$2 = (Circle &) @0x7fffffffde90: {<Shape> = {_vptr.Shape = 0x400a80 <vtable for Circle+16>}, radius = 1}
```

当打印对象类型信息时，该设置也会起作用：
```shell
(gdb) whatis p
type = Shape &
(gdb) ptype p
type = class Shape {
  public:
    virtual void draw(void);
} &

(gdb) set print object on
(gdb) whatis p
type = /* real type = Circle & */
Shape &
(gdb) ptype p
type = /* real type = Circle & */
class Shape {
  public:
    virtual void draw(void);
} &
```

- **Tips-47:** 指定程序的输入输出设备

**示例代码：**

```c
#include <stdio.h>

int main(void)
{
  int i;

  for (i = 0; i < 100; i++)
    {
      printf("i = %d\n", i);
    }

  return 0;
}
```

**技巧：**
在gdb中，缺省情况下程序的输入输出是和gdb使用同一个终端。你也可以为程序指定一个单独的输入输出终端。

首先，打开一个新终端，使用如下命令获得设备文件名：

```shell
$ tty
/dev/pts/2
```

然后，通过命令行选项指定程序的输入输出设备：

```shell
$ gdb -tty /dev/pts/2 ./a.out
(gdb) r
```

或者，在gdb中，使用命令进行设置：

```shell
(gdb) tty /dev/pts/2
```


- **Tips-48:** 使用“$\_”和“$\__”变量

**示例代码：**

```c
#include <stdio.h>

int main(void)
{
        int i = 0;
        char a[100];

        for (i = 0; i < sizeof(a); i++)
        {
                a[i] = i;
        }

        return 0;
}
```

**技巧：**
`x`命令会把最后检查的内存地址值存在`$_`这个`convenience variable`中，并且会把这个地址中的内容放在`$__`这个`convenience variable`，以上面程序为例:

```shell
(gdb) b a.c:13
Breakpoint 1 at 0x4004a0: file a.c, line 13.
(gdb) r
Starting program: /data2/home/nanxiao/a

Breakpoint 1, main () at a.c:13
13              return 0;
(gdb) x/16xb a
0x7fffffffe4a0: 0x00    0x01    0x02    0x03    0x04    0x05    0x06    0x07
0x7fffffffe4a8: 0x08    0x09    0x0a    0x0b    0x0c    0x0d    0x0e    0x0f
(gdb) p $_
$1 = (int8_t *) 0x7fffffffe4af
(gdb) p $__
$2 = 15
```

可以看到`$_`值为`0x7fffffffe4af`，正好是`x`命令检查的最后的内存地址。而`$__`值为`15`。
另外要注意有些命令（像`info line`和`info breakpoint`）会提供一个默认的地址给`x`命令检查，而这些命令也会把`$_`的值变为那个默认地址值：

```shell
(gdb) p $_
$5 = (int8_t *) 0x7fffffffe4af
(gdb) info breakpoint
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x00000000004004a0 in main at a.c:13
        breakpoint already hit 1 time
(gdb) p $_
$6 = (void *) 0x4004a0 <main+44>
```
可以看到使用`info breakpoint`命令后，`$_`值变为`0x4004a0`。


- **Tips-49:** 打印程序动态分配内存的信息

**示例代码：**

```c
#include <stdio.h>
#include <malloc.h>

int main(void)
{
        char *p[10];
        int i = 0;

        for (i = 0; i < sizeof(p)/sizeof(p[0]); i++)
        {
                p[i] = malloc(100000);
        }
        return 0;
}
```

**技巧：**
用gdb调试程序时，可以用下面的自定义命令，打印程序动态分配内存的信息：

```shell
define mallocinfo
  set $__f = fopen("/dev/tty", "w")
  call malloc_info(0, $__f)
  call fclose($__f)
end
```

以上面程序为例：
```shell
Temporary breakpoint 5, main () at a.c:7
7               int i = 0;
(gdb) mallocinfo 
<malloc version="1">
<heap nr="0">
<sizes>
</sizes>
<total type="fast" count="0" size="0"/>
<total type="rest" count="0" size="0"/>
<system type="current" size="135168"/>
<system type="max" size="135168"/>
<aspace type="total" size="135168"/>
<aspace type="mprotect" size="135168"/>
</heap>
<total type="fast" count="0" size="0"/>
<total type="rest" count="0" size="0"/>
<system type="current" size="135168"/>
<system type="max" size="135168"/>
<aspace type="total" size="135168"/>
<aspace type="mprotect" size="135168"/>
</malloc>
$20 = 0
$21 = 0
(gdb) n
9               for (i = 0; i < sizeof(p)/sizeof(p[0]); i++)
(gdb) 
11                      p[i] = malloc(100000);
(gdb) 
9               for (i = 0; i < sizeof(p)/sizeof(p[0]); i++)
(gdb) 
11                      p[i] = malloc(100000);
(gdb) 
9               for (i = 0; i < sizeof(p)/sizeof(p[0]); i++)
(gdb) 
11                      p[i] = malloc(100000);
(gdb) 
9               for (i = 0; i < sizeof(p)/sizeof(p[0]); i++)
(gdb) 
11                      p[i] = malloc(100000);
(gdb) 
9               for (i = 0; i < sizeof(p)/sizeof(p[0]); i++)
(gdb) 
11                      p[i] = malloc(100000);
(gdb) mallocinfo 
<malloc version="1">
<heap nr="0">
<sizes>
</sizes>
<total type="fast" count="0" size="0"/>
<total type="rest" count="0" size="0"/>
<system type="current" size="532480"/>
<system type="max" size="532480"/>
<aspace type="total" size="532480"/>
<aspace type="mprotect" size="532480"/>
</heap>
<total type="fast" count="0" size="0"/>
<total type="rest" count="0" size="0"/>
<system type="current" size="532480"/>
<system type="max" size="532480"/>
<aspace type="total" size="532480"/>
<aspace type="mprotect" size="532480"/>
</malloc>
$22 = 0
$23 = 0
(gdb) n
9               for (i = 0; i < sizeof(p)/sizeof(p[0]); i++)
(gdb) 
11                      p[i] = malloc(100000);
(gdb) 
9               for (i = 0; i < sizeof(p)/sizeof(p[0]); i++)
(gdb) 
11                      p[i] = malloc(100000);
(gdb) 
9               for (i = 0; i < sizeof(p)/sizeof(p[0]); i++)
(gdb) 
11                      p[i] = malloc(100000);
(gdb) 
9               for (i = 0; i < sizeof(p)/sizeof(p[0]); i++)
(gdb) 
11                      p[i] = malloc(100000);
(gdb) 
9               for (i = 0; i < sizeof(p)/sizeof(p[0]); i++)
(gdb) 
11                      p[i] = malloc(100000);
(gdb) 
9               for (i = 0; i < sizeof(p)/sizeof(p[0]); i++)
(gdb) mallocinfo 
<malloc version="1">
<heap nr="0">
<sizes>
</sizes>
<total type="fast" count="0" size="0"/>
<total type="rest" count="0" size="0"/>
<system type="current" size="1134592"/>
<system type="max" size="1134592"/>
<aspace type="total" size="1134592"/>
<aspace type="mprotect" size="1134592"/>
</heap>
<total type="fast" count="0" size="0"/>
<total type="rest" count="0" size="0"/>
<system type="current" size="1134592"/>
<system type="max" size="1134592"/>
<aspace type="total" size="1134592"/>
<aspace type="mprotect" size="1134592"/>
</malloc>
$24 = 0
$25 = 0
```
可以看到gdb输出了动态分配内存的变化信息。

- **Tips-50:** 打印调用栈帧中变量的值

**示例代码：**

```c
#include <stdio.h>

int func1(int a)
{
  int b = 1;
  return b * a;
}

int func2(int a)
{
  int b = 2;
  return b * func1(a);
}

int func3(int a)
{
  int b = 3;
  return b * func2(a);
}

int main(void)
{
  printf("%d\n", func3(10));
  return 0;
}
```

**技巧：**
在gdb中，如果想查看调用栈帧中的变量，可以先切换到该栈帧中，然后打印：

```shell
(gdb) b func1
(gdb) r
(gdb) bt
#0  func1 (a=10) at frame.c:5
#1  0x0000000000400560 in func2 (a=10) at frame.c:12
#2  0x0000000000400582 in func3 (a=10) at frame.c:18
#3  0x0000000000400596 in main () at frame.c:23
(gdb) f 1
(gdb) p b
(gdb) f 2
(gdb) p b
```

也可以不进行切换，直接打印：

```shell
(gdb) p func2::b
$1 = 2
(gdb) p func3::b
$2 = 3
```
同样，对于C++的函数名，需要使用单引号括起来，比如：
```shell
(gdb) p '(anonymous namespace)::SSAA::handleStore'::n->pi->inst->dump()
```


