---
title: LeeTCode两数相加
date: 2018-12-18 19:01:12
tags: 
- 学习笔记
categories: 
- 前端
---

### 题目描述：


![image.png | left | 827x199](https://cdn.nlark.com/lark/0/2018/png/146541/1545125946082-1bfce059-4c3b-4786-bf8c-d4ef7223f3e1.png "")

#### 链表和参数格式：
### ![image.png | left | 333x192](https://cdn.nlark.com/lark/0/2018/png/146541/1545125984985-39df3358-d8c7-4866-b2ff-f43a5896bf41.png "")

思路是对的，但是硬是折腾了将近一天才提交编译通过。。。
### 贴个代码
```javascript
var addTwoNumbers = function(l1, l2) {
  var result, res;
  var p = l1;
  var q = l2;
  var pow = 1;
  temp = {
    value:0
  }
  while(p != null || q != null){
    
    res = new ListNode(calculate(p, q, temp) )
    
    p == null ? null : p = p.next;
    q == null ? null : q = q.next;

    result == undefined ?
      result = res :
      result.addNode(res);
  }
  if(temp.value == 1){
    result.addNode(new ListNode(1));
  }
  return result
};

ListNode.prototype.addNode = function(val) {
  var p = this;
  var q = this;
  while(p != null){
    p.next != null ? q = p.next : null;
    p = p.next;
  }
  q.next = val;
}

function calculate (v1, v2, temp) {
  var result = 0;
  v1 == null ? v1 = {val:0} : null;
  v2 == null ? v2 = {val:0} : null;
  v1.val + v2.val + temp.value < 10 ?
    (result = temp.value + v1.val + v2.val, temp.value = 0) :
    (result = temp.value + v1.val + v2.val - 10,temp.value = 1);
  return result;
}
```

用链表存储数字的一个显而易见的作用在于，可以突破语言的极限数据大小。在求斐波那契数列第100项之类的问题时，语言本身的变量已经存不下那么大的数了，这个时候就得用链表、数组来模拟数字，逐位数学运算。