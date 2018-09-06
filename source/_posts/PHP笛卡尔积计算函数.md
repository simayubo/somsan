---
title: PHP笛卡尔积计算函数
date: 2018-08-27 19:08:56
tags: [php]
categories: [php]
---

> 今天写商品规格匹配的时候，用到了多数组交叉匹配，所以就用到了这个东西。

## 笛卡尔乘积
笛卡尔乘积是指在数学中，两个集合X和Y的笛卡尓积，又称直积，表示为X × Y，第一个对象是X的成员而第二个对象是Y的所有可能有序对的其中一个成员。
假设集合A={a, b}，集合B={0, 1, 2}，则两个集合的笛卡尔积为{(a, 0), (a, 1), (a, 2), (b, 0), (b, 1), (b, 2)}。

<!--more-->

## 代码：
```php
function descartes($arr){
    $arr1 = array();
    $result = array_shift($arr);
    while($arr2 = array_shift($arr)){
        $arr1 = $result;
        $result = array();
        foreach($arr1 as $v){
            foreach($arr2 as $v2){
                if(!is_array($v))$v = array($v);
                if(!is_array($v2))$v2 = array($v2);
                $result[] = array_merge_recursive($v,$v2);
            }
        }
    }
    return $result;
}
```

函数可以实现任意个数数组的计算，比如下面：

![QQ截图20170707141512.png][1]


  [1]: https://simayubocc.oss-cn-hangzhou.aliyuncs.com/img/2017/07/316064195.png