# 信息显示
- **Tips-1:** 显示gdb版本信息
使用gdb时，如果想查看gdb版本信息，可以使用`show version`命令:
```shell
(gdb) show version
GNU gdb (Ubuntu 8.1-0ubuntu3.2) 8.1.0.20180409-git
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word".
```

- **Tips-2:** 显示gdb版权相关信息
使用gdb时，如果想查看gdb版权相关信息，可以使用`show copying`命令:
```shell
(gdb) show copying 
                    GNU GENERAL PUBLIC LICENSE
                       Version 3, 29 June 2007

 Copyright (C) 2007 Free Software Foundation, Inc. <http://fsf.org/>
 Everyone is permitted to copy and distribute verbatim copies
 of this license document, but changing it is not allowed.

                            Preamble

  The GNU General Public License is a free, copyleft license for
software and other kinds of works.
  ............
```

- **Tips-3:** 启动时不显示提示信息
gdb在启动时会显示提示信息，如果不想显示这个信息，则可以使用-q选项把提示信息关掉，如下所示：
```shell
$ gdb -q
(gdb)
```
当然，可以在~/.bashrc中，为gdb设置一个别名：
```shell
alias gdb="gdb -q"
```

- **Tips-4:** 退出时不显示提示信息
gdb在退出时会提示：
```shell
A debugging session is active.

    Inferior 1 [process 29686    ] will be killed.

Quit anyway? (y or n) n
```
如果不想显示这个信息，则可以在gdb中使用如下命令把提示信息关掉：
```shell
(gdb) set confirm off
```
当然，也可以把这个命令加到.gdbinit文件里。

- **Tips-5:** 输出信息多时不暂停输出
有时当gdb输出信息较多时，gdb会暂停输出，并会打印`---Type <return> to continue, or q <return> to quit---`这样的提示信息，如下面所示：
```shell
 81 process 2639102      0xff04af84 in __lwp_park () from /usr/lib/libc.so.1
 80 process 2573566      0xff04af84 in __lwp_park () from /usr/lib/libc.so.1
---Type <return> to continue, or q <return> to quit---Quit
```
解决办法是使用`set pagination off`或者`set height 0`命令。这样gdb就会全部输出，中间不会暂停。

