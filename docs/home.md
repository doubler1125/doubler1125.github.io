# PHP

## 介绍与安装
*PHP*，即“PHP: Hypertext Preprocessor”，是一种被广泛应用的开源通用脚本语言，尤其适用于 Web 开发并可嵌入 HTML 中去。它的语法利用了 C、Java 和 Perl，易于学习。该语言的主要目标是允许 web 开发人员快速编写动态生成的 web 页面，但 PHP 的用途远不只于此。

官方文档：https://www.php.net/manual/zh/index.php

安装：https://www.php.net/manual/zh/install.php

## 底层原理

**变量在内存中的存储结构**

PHP变量是通过zval结构体来存储的，文件Zend/zend.h
```c
struct _zval_struct {
	zvalue_value value;
    zend_uint refcount_gc;
    zend_uchar type;
    zend_uchar is_ref_gc;
}
```
```php
$a = 3; 

// 一个结构体产生了
{
	union_zvalue {long 3}
    type IS_LONG
    refcount_gc: 1
    is_ref_gc: 0
}
```

PHP变量的值是放在zval结构体中的value段中的，文件Zend/zend.h
```c
typedef union _zvalue_value {
	long lval;
    double dval;
    struct {
    	char *val;
        int len;
    } str;
    HashTable *ht;
    zend_object_value obj;
} zvalue_value;
```
```php
$b = 'hello';

{
    {
    	char: 'hello'
        len: 5
    }
    type: IS_STRING
    refcount_gc: 1
    is_ref_gc: 0
}

// 可以看出PHP中字符串的长度直接体现在结构体中，strlen（）速度很快，时间复杂度为O(1)
```
符号表symbol_table，是一张哈希表，里面存储了变量名->变量zval结构体地址的映射

**写时复制**
```php
$a = 3;

{
	zvalue: 3
    type: IS_LONG
    refcount_gc: 1
    is_ref_gc: 0
}

$b = $a;

{
	zvalue: 3
    type: IS_LONG
    refcount_gc: 2
    is_ref_gc: 0
}

$b = 5; // 写时复制

$a
{
	zvalue: 3
    type: IS_LONG
    refcount_gc: 1
    is_ref_gc: 0
}
$b
{
	zvalue: 5
    type: IS_LONG
    refcount_gc: 1
    is_ref_gc: 0
}
```
# Golang

## 介绍与安装


# 数据库

## 非关系型


# 组件和工具