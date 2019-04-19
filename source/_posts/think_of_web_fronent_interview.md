---
title: 一道前端面试题引发的思考
date: 2019-04-18 23:02:18
categories: 思考
tags: [vue, interview] 
---

> 最近在看前端面试题的过程中,发现在`stackoverflow`上有这么一道题,题目大概是这样的:

```js
// 请在问号处填写你的答案,使下方等式成立
let a = ?;
if(a == 1 && a == 2 && a == 3) {
    console.log("Hi, I'm Echi");
}
```

一看到这道题,相信大多数人心目中就已经最少有两种实现方案了。我们知道当一个对象在转换为基本类型时，首先它会先调用 `valueOf` 然后调用 `toString`, 所以我们尝试着改写这两种方法。

(一) 使用`valueOf`实现

```js
let a = {
    i: 1,
    valueOf() {
        return a.i++;
    }
};
if(a == 1 && a == 2 && a == 3) {
    console.log("Hi, I'm Echi");
}
```

(二) 使用`toString`实现

```js
let a = {
    i: 1,
    toString() {
        return a.i++;
    }
};
if(a == 1 && a == 2 && a == 3) {
    console.log("Hi, I'm Echi");
}
```

接下来我们来看对象的`Symbol.toPrimitive`属性,该属性会指向一个方法。当对象被转化为原始类型的值时，会调用这个办法，返回该对象对应的原始类型值,且该方法在转基本类型时调用优先级最高。所以我们就有了第三种方案:

(三) 使用`Symbol.toPrimitive`实现

```js
let a = {
    i: 1,
    [Symbol.toPrimitive]() {
        return a.i++;
    }
};
if(a == 1 && a == 2 && a == 3) {
    console.log("Hi, I'm Echi");
}
```

好了,以上就是我对这道题的解题方案,主要还是利用了`==`对变量类型的隐式转换,当然大家可能还有其他解题方案。比如说可以使用`es5`的`Object.defineProperty`,或者`es6`的`Proxy`,这里我就不写出来了;

最后问大家一句,如果将题目中的`==`换成`===`,效果还是一样的吗,为什么?欢迎大家留言讨论
> 更多详情,请自行查看 [stackoverflow](https://stackoverflow.com/questions/48270127/can-a-1-a-2-a-3-ever-evaluate-to-true);
