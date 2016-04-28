---
layout: post
title: "蕙泉斋"
description: ""
category: 
tags: []
---
{% include JB/setup %}

## javascript中的连续赋值


此前一直不太了解javascript的连续赋值，直到今天下午，看到一位同事大哥在群里发了这样一道题目：

var foo = {n: 1};
var bar = foo;
foo.x = foo = {n: 2};

这个foo.x的值是多少？

初看这道题目，逐一分析代码：
第一行：foo指向对象{n: 1}的引用；
第二行：foo将自身存储的引用地址（指向{n: 1}）赋值给bar；
第三行：foo存储的{n: 1}的引用地址被覆盖，现在foo中存储的是{n: 2}的引用地址；而foo.x读取的是{n: 2}中的x属性，由于x属性不存在，javascript引擎将会在{n: 2}中自动新增一个x属性，并将{n: 2}赋值给x属性。

按照这样的分析，最终foo.x = {n: 2};

然而运行这段代码后却发现foo.x = undefined;

这很有趣，然而问题的前两步分析非常简单，坑一定是埋在第三步的。

同事给出的解释是：“第三行代码执行时，先从左往右取好引用地址，再从右往左依次赋值”。结合题目分析就是，在第三行代码执行的过程中，foo.x 和 foo 分别首先指向了 {n: 1}的x属性 和{n: 1}的引用，然后再赋值；赋值是从右向左的，也就是foo中存储的{n: 1}的引用被{n: 2}的引用覆盖了，随后{n: 1}的x属性中存储了{n: 2}的引用。在第三行代码执行完毕后，foo中存储的就是指向{n: 2}的引用了，而此时两个对象的状态分别是{n: 1, x:{n: 2}}, {n: 2}，但由于不再存在指向{n: 1, x:{n: 2}}的引用了，所以我们无法访问到x属性，而此时foo指向的{n: 2}中是不存在x属性的，因此访问foo.x时，会返回undefined。

这很有趣，然而为什么取引用地址是先于赋值运算的呢？查了资料才发现，原来是这个不起眼的对象成员访问运算符.在作怪。
参考资料：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence
在javascript中，等号的优先级是很低的，然而通常我们只关注算数运算符的优先级，却忽略了对象成员访问运算符的优先级。在这道题目中，由于第三行代码中的成员访问运算符的优先级很高，因此javascript引擎在执行这行代码时，会先执行foo.x，此时foo仍然是指向{n: 1}的，因此也就出现了取得引用地址会先于赋值运算执行的现象。

（ps：据我所知，貌似只有赋值运算是从右向左执行的，因此同事的得出结论是：取引用地址是从左向右的，而赋值是从右向左的）。
