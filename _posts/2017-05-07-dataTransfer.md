---
layout: post
date: 2017-05-07
title: JS中的dataTransfer对象
tags: FrontEnd
---
　　在进行拖放操作的时候，dataTransfer对象可以用来保存被拖动的数据。它可以保存一项或多项数据、一种或多数数据类型。通谷一点讲，就是可以通过它来传输被拖动的数据，以便在拖拽结束的时候，对数据进行其他的操作。

　　在项目拖动的开始事件(`ondragstart`)的时候，可以对dataTransfer进行设值操作(`setData`)。

```javascript
document.getElementById('source').ondragstart = function (event) {
    event.dataTransfer.setData('some-key', 'some-value');
};
```

　　在项目拖动结束的时候，可以获取dataTransfer对象里的值。

```javascript
document.getElementById('target').ondrop = function(event){
    event.preventDefault(); // 拖动的默认处理方式是在新窗口打开，所以阻止其执行
    let value = event.dataTransfer.getData('some-key');
    // some-value
}
```

　　在其它的事件(如`ondragover`、`ondragleave`等），是无法获取dataTransfer里面的值了。这是由于W3C要求对dataTransfer里的值进行保护[[参考](https://www.w3.org/TR/html51/editing.html#drag-and-drop)]。因此，如果需要在这些事件里获取数据，只能通过一个全局变量等其它方式来实现了。





