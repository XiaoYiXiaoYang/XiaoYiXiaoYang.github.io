---
title: mem系列函数(memcpy和memmove不同)
date: 2019-12-14 22:54:42
tags: [C++]
categories: [技术分享]
---

 同样是内存拷贝，效率不同，memcpy、memmove

<!--more-->

## 源码


**mencpy**

```
void *memcpy(void *dst, const void *src, int count)
          从内存中读取src字符串指定字符个数copy到dst中，其中dst与src内存不能重叠，切拷贝以后需要将dst末尾
		  
void *memcpy(void *dest, const void *source, size_t count)  
{  
 assert((NULL != dest) && (NULL != source));  
 char *tmp_dest = (char *)dest;  
 char *tmp_source = (char *)source;  
 while(count --)//不对是否存在重叠区域进行判断  
   *tmp_dest ++ = *tmp_source ++;  
 return dest;  
}  
```

**memmove**

```
void *memmove(void *dst, const void *src, int count)
作用与memcpy相似，但是改善了memcpy不能copy内存重叠字符串的情况

void *memmove(void *dest, const void *source, size_t count)  
{  
 assert((NULL != dest) && (NULL != source));  
 char *tmp_source, *tmp_dest;  
 tmp_source = (char *)source;  
 tmp_dest = (char *)dest;  
 if((dest + count<source) || (source + count) <dest))  
 {// 如果没有重叠区域  
   while(count--)  
     *tmp_dest++ = *tmp_source++;  
}  
else  
{ //如果有重叠  
 tmp_source += count - 1;  
 tmp_dest += count - 1;  
 while(count--)  
   *--tmp_dest = *--tmp;  
}  
return dest;  
}  
```

总结：memmove存在对重叠区域的判断，然后复制的时候就倒着复制，保证复制正确性。
而memcpy没有判断，直接复制，如果两者内存有交叉，则复制到最后，拷贝的数据不是原来的数据，拷贝到目标区域的数据也是错的。


## 其他mem系列


**memset**

```
void *memset(void*s ,int ch,size_t n);

函数描述：将内存地址s处的n个字节的每个字节都替换为ch，并返回s。
```

**memcmp**
```
int memcmp(const void *buffer1, const void ,*buffer2, int count)
作用：比较内存区内由用户指定字符个数的大小，即比较buffer1与buffer2前count个字节的大小


```


**memchr**

```
void *memchr(const void *buffer, int ch, int count)
作用：从buffer所指内存区域的前count个字节查找字符ch， 当第一次遇到字符ch时停止查找。

```










2019-10-18 20:01:15 星期五