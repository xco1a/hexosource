---
title: sass语法小记
date: 2018-05-18 14:33:28
tags: 
- 学习笔记
categories: 
- 前端
---

&emsp; &emsp; 在接触sass后，回想之前初学时写过的css文件，确实简陋粗糙。如果能用sass重写估计会节省很多时间，以及维护修改时的工作量。sass的关键字以@开头，变量则以$开头，基础语法还是挺简单的，只是目前缺乏实践，尚不能自如运用。

#####记录所学的基础语法~

```sass
@charset "UTF-8";
@import "commen";//  导入
$fontsize : "16px";//变量
ul{
  font-size: $fontsize;
}
//嵌套
.banner{
  width:100%;
  .box{
    height:100px;
    .img{
      float: left;
    }
  }
}
//继承
ul{
  width:100px;
  height:100px;
  color: #ccc;
  font-size: $fontsize;
}
div{
  @extend ul;
}

//混入
@mixin setProp($a,$b,$c){
  width:$a;
  height:$b;
  color: $c;
}
div{
  @include setProp(100px,100px,#ccc);
}
//函数
@function dar($a){
  @return $a*10px;
}
div{
  width:dar(10);
}
//循环
@for $i from 1 through 10{
  div:nth-of-type(#{$i}){
    width: $i;
  }
}
//判断
@if true {
  div{
    width: 10px;
  }
}@else{
  div{
    height:100px;
  }
}
```

