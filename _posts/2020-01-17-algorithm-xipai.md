---
layout: post
title:  洗牌算法
date:   2020-01-17
categories: algorithm
permalink : algorithm/xipai.html
description: 洗牌算法
show : true
---

```php
$array = [1,2,3,4,5];
$n = count($array);
for($i =$n-1, $i>=0; $i--){
	swap($array[$i], $array[mt_rand(0, $i)]);//交换位置
}
```

