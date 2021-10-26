---
title: 父元素设置了 min-height ，子元素 height 100% 不生效
date: 2021-10-26 22:57:43
tags: [前端]
---
## 错误分析
因为百分比是根据父元素的 `height` 来计算的，而不是 `min-height`。所以当父元素的 `height` 未被设置时，子元素是根据默认值 `auto` 来计算的，所以表现为根据内容自动设置高度。
## 解决方法
在父元素使用 `flex` 或 `grid` 布局，此时子元素的 `align-self` 是 `stretch`，即 100%。  
````css
.container{
    display: flex;
    min-height: 100vh;
}

.item{
    
}
````

````html
<div class="container">
    <div class="item">
        Hello, World!
    </div>
</div>
````
## 参考
[align-self - CSS（层叠样式表）](https://developer.mozilla.org/zh-CN/docs/Web/CSS/align-self)