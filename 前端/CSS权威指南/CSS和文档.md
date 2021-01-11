# 1.元素

## 替换元素

```html
<img src="a.gif">
```

## 非替换元素

## 块级元素

## 行内元素

# 2.引入CSS样式表

## 使用link标签

```HTML
<!--首选样式表,media代表媒体表现形式,如screen-屏幕,projection-幻灯片-->
<link rel="stylesheet" type="text/css" href="xxx.css" media="all">
<!--候选样式表-->
<link rel="alternate stylesheet" type="text/css" href="xxx.css" media="all" title="Big Text">
```

## 使用style元素

```HTML
<style type="text/css" media="all">
    @import url(b.css) screen; /*引入外部css*/
</style>
```

## 内联样式

除了在body外部出现的标记；只能放声明块。