# fprintf()

``` C
int fprintf(FILE *stream, const char *format, ...)
```

* **stream** -- 这是指向 FILE 对象的指针，该 FILE 对象标识了流。
* **format** -- 这是 C 字符串，包含了要被写入到流 stream 中的文本。它可以包含嵌入的 format 标签，format 标签可被随后的附加参数中指定的值替换，并按需求进行格式化。
* **返回值** -- 写入字符总数，失败返回一个负数

``` C
#include <stdio.h>
#include <stdlib.h>

int main()
{
   FILE * fp;

   fp = fopen ("file.txt", "w+");
   fprintf(fp, "%s %s %s %d", "We", "are", "in", 2014);
   
   fclose(fp);
   
   return(0);
}
```

# printf()

``` C
int printf(const char *format, ...)
```

* **format** -- 这是 C 字符串，包含了要被写入到流 stream 中的文本。它可以包含嵌入的 format 标签，format 标签可被随后的附加参数中指定的值替换，并按需求进行格式化。
* **返回值** -- 写入字符总数，失败返回一个负数

``` C
#include <stdio.h>
 
int main ()
{
   int ch;
 
   for( ch = 75 ; ch <= 100; ch++ ) {
      printf("ASCII 值 = %d, 字符 = %c\n", ch , ch );
   }
 
   return(0);
}
```

# sprintf()

```C
int sprintf(char *str, const char *format, ...)
```

- **str** -- 这是指向一个字符数组的指针，该数组存储了 C 字符串。
- **format** -- 这是字符串，包含了要被写入到字符串 str 的文本。它可以包含嵌入的 format 标签，format 标签可被随后的附加参数中指定的值替换，并按需求进行格式化
- **返回值** -- 字符串的总长度，失败返回一个负数

```C
#include <stdio.h>
#include <math.h>

int main()
{
   char str[80];

   sprintf(str, "Pi 的值 = %f", M_PI);
   puts(str);
   
   return(0);
}
```

> ```
> Pi 的值 = 3.141593
> ```

# snprintf()

C 库函数 **int snprintf(char \*str, size_t size, const char \*format, ...)** 设将可变参数(...)按照 format 格式化成字符串，并将字符串复制到 str 中，size 为要写入的字符的最大数目，超过 size 会被截断。

``` C
int snprintf ( char * str, size_t size, const char * format, ... )
```

- **str** -- 目标字符串。
- **size** -- 拷贝字节数(Bytes)。
- **format** -- 格式化成字符串。
- **...** -- 可变参数。
- **返回值** -- 字符串的总长度，不会考虑size带来的影响，效果同sprintf，失败返回一个负数

``` C
#include <stdio.h>
 
int main()
{
    char buffer[50];
    char* s = "runoobcom";
 
    // 读取字符串并存储在 buffer 中
    int j = snprintf(buffer, 6, "%s\n", s);
 
    // 输出 buffer及字符数
    printf("string:\n%s\ncharacter count = %d\n", buffer, j);
 
    return 0;
}
```

> ```
> string:
> runoo
> character count = 10
> ```

# vfprintf()

C 库函数 **int vfprintf(FILE \*stream, const char \*format, va_list arg)** 使用参数列表发送格式化输出到流 stream 中。

``` C
int vfprintf(FILE *stream, const char *format, va_list arg)
```

- **stream** -- 这是指向 FILE 对象的指针，该 FILE 对象标识了流。
- **format** -- 这是 C 字符串，包含了要被写入到流 stream 中的文本。它可以包含嵌入的 format 标签，format 标签可被随后的附加参数中指定的值替换，并按需求进行格式化。

``` C
#include <stdio.h>
#include <stdarg.h>

void WriteFrmtd(FILE *stream, char *format, ...)
{
   va_list args;

   va_start(args, format);
   vfprintf(stream, format, args);
   va_end(args);
}

int main ()
{
   FILE *fp;

   fp = fopen("file.txt","w");

   WriteFrmtd(fp, "This is just one argument %d \n", 10);

   fclose(fp);
   
   return(0);
}
```

> ```
> This is just one argument 10
> ```

# vprintf()

C 库函数 **int vprintf(const char \*format, va_list arg)** 使用参数列表发送格式化输出到标准输出 stdout。

``` C
int vprintf(const char *format, va_list arg)
```

* **format** -- 这是 C 字符串，包含了要被写入到流 stream 中的文本。它可以包含嵌入的 format 标签，format 标签可被随后的附加参数中指定的值替换，并按需求进行格式化。

```c
#include <stdio.h>
#include <stdarg.h>

void WriteFrmtd(char *format, ...)
{
   va_list args;
   
   va_start(args, format);
   vprintf(format, args);
   va_end(args);
}

int main ()
{
   WriteFrmtd("%d variable argument\n", 1);
   WriteFrmtd("%d variable %s\n", 2, "arguments");
   
   return(0);
}
```

> ```
> 1 variable argument
> 2 variable arguments
> ```

# vsprintf()

C 库函数 **int vsprintf(char \*str, const char \*format, va_list arg)** 使用参数列表发送格式化输出到字符串。

``` C
int vsprintf(char *str, const char *format, va_list arg)
```

* **str** -- 这是指向一个字符数组的指针，该数组存储了 C 字符串。
* **format** -- 这是字符串，包含了要被写入到字符串 str 的文本。它可以包含嵌入的 format 标签，format 标签可被随后的附加参数中指定的值替换，并按需求进行格式化

``` C
#include <stdio.h>
#include <stdarg.h>

char buffer[80];
int vspfunc(char *format, ...)
{
   va_list aptr;
   int ret;

   va_start(aptr, format);
   ret = vsprintf(buffer, format, aptr);
   va_end(aptr);

   return(ret);
}

int main()
{
   int i = 5;
   float f = 27.0;
   char str[50] = "runoob.com";

   vspfunc("%d %f %s", i, f, str);
   printf("%s\n", buffer);
   
   return(0);
}
```

> ```
> 5 27.000000 runoob.com
> ```

# fscanf()

``` C
int fscanf(FILE *stream, const char *format, ...)
```

* **stream** -- 这是指向 FILE 对象的指针，该 FILE 对象标识了流。
* **format** -- 这是 C 字符串，包含了以下各项中的一个或多个：*空格字符、非空格字符* 和 *format 说明符*。
* 返回值 --  如果成功，该函数返回成功匹配和赋值的个数。如果到达文件末尾或发生读错误，则返回 EOF。

``` C
#include <stdio.h>
#include <stdlib.h>


int main()
{
   char str1[10], str2[10], str3[10];
   int year;
   FILE * fp;

   fp = fopen ("file.txt", "w+");
   fputs("We are in 2014", fp);
   
   rewind(fp);
   fscanf(fp, "%s %s %s %d", str1, str2, str3, &year);
   
   printf("Read String1 |%s|\n", str1 );
   printf("Read String2 |%s|\n", str2 );
   printf("Read String3 |%s|\n", str3 );
   printf("Read Integer |%d|\n", year );

   fclose(fp);
   
   return(0);
}
```

> ```
> Read String1 |We|
> Read String2 |are|
> Read String3 |in|
> Read Integer |2014|
> ```

# scanf()

``` C
int scanf(const char *format, ...)
```

* **format** -- 这是 C 字符串，包含了以下各项中的一个或多个：*空格字符、非空格字符* 和 *format 说明符*。
* 返回值 --  如果成功，该函数返回成功匹配和赋值的个数。如果到达文件末尾或发生读错误，则返回 EOF。

``` C
#include<stdio.h>
int main(void) 
{ 
    int a,b,c; 
 
    printf("请输入三个数字：");
    scanf("%d%d%d",&a,&b,&c); 
    printf("%d,%d,%d\n",a,b,c);
    return 0; 
}
```

# sscanf()

```C
int sscanf(const char *str, const char *format, ...)
```

- **str** -- 这是 C 字符串，是函数检索数据的源。
- **format** -- 这是 C 字符串，包含了以下各项中的一个或多个：*空格字符、非空格字符* 和 *format 说明符*。
- 返回值 --  如果成功，该函数返回成功匹配和赋值的个数。如果到达文件末尾或发生读错误，则返回 EOF。

``` C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main()
{
   int day, year;
   char weekday[20], month[20], dtm[100];

   strcpy( dtm, "Saturday March 25 1989" );
   sscanf( dtm, "%s %s %d  %d", weekday, month, &day, &year );

   printf("%s %d, %d = %s\n", month, day, year, weekday );
    
   return(0);
}
```

> ```
> March 25, 1989 = Saturday
> ```

# 参考资料

> 菜鸟教程https://www.runoob.com/cprogramming/c-standard-library-stdio-h.html

