---
title: 使用 flex 优雅地实现 Web App 布局
layout: post
date: '2018-09-03 17:01:06 +0800'
categories: 程序员日常
---

### 背景：
一般 Web App 是这样的布局
- 固定在顶部的标题栏
- 固定在底部的导航栏
- 中间是内容区，内容区撑满可用空间，即从顶部标题栏结束撑到底部导航栏开始。
内容区还有两个特性：即使是内容很少，用留白也要撑满；如果内容很多，可以显示滚动条，但这个滚动条也只是从标题栏开始到底部导航栏结束。也就是说，超长的内容区既不能被底部遮挡也不能将底部导航栏挤出屏幕外。

### 分析：
要实现这样的布局，一开始很自然让人想到使用 `fixed` 来将顶部与底部固定，但是通过实践，你会发现这样做的话，当内容区内容过多，底下一部分内容会被底部导航栏遮挡。

### 优雅的解决方案：
受[这个答案](http://stackoverflow.com/questions/90178/make-a-div-fill-the-height-of-the-remaining-screen-space)的启发，我们可以使用 `flex` 来布局。这个答案给出的解决方案实现了：
- 固定在顶部的标题栏（使用 `.row.header` 类）
- 固定在底部的导航栏（使用 `.row.content` 类）
- 中间的内容区撑满剩下的可用空间（使用 `.row.footer` 类）
但是，如果中间的内容区内容过长，会将底部导航栏挤出屏幕外，这就遗憾了。
我在此基础上，通过给 `.row.content` 类，增加 `display: flex` 属性，并且将该类的第一个子层最大高度限制了 100%，又给予 `overflow: auto` 的样式，便完美实现了可以灵活扩展的 Web App。

### 完美的方案：
- 固定在顶部的标题栏（使用 `.row.header` 类）
- 固定在底部的导航栏（使用 `.row.footer` 类）
- 中间的内容区撑满剩下的可用空间并且当超长时显示滚动条（不要将内容区直接放在 `.row.content` 层里，而是放在 `.row.content > div` 层里！）

#### 布局的 html 结构：
```html
<body>
  <div class="flex-box">
    <div class="row header">这里是顶部标题栏</div>
    <div class="row content">
      <div>
        这里是内容区，注意内容区出现在 `row content` 的子层里！
      </div>
    </div>
    <div class="row footer">这里是底部导航栏</div>
  </div>
</body>
``` 

#### 布局的 css 代码：
```css
html, body, body > div {
    height: 100%;
    margin: 0
}

.flex-box {
    display: flex;
    flex-flow: column;
    height: 100%;
}

.flex-box .row {
}

.flex-box .row.header {
    flex: 0 1 auto;
    /* The above is shorthand for:
    flex-grow: 0,
    flex-shrink: 1,
    flex-basis: auto
    */
}

.flex-box .row.content {
    flex: 1 1 auto;
    display: flex; /* 加上这个配合子层的高度限制可以防止过长的内容区挤出底部导航栏！ */
}

/* 一定要将内容区放在 `row content` 的子层里，以便当内容超长时，让滚动条显示出来 */
.flex-box .row.content > div {
    max-height: 100%;
    overflow: auto;
}

.flex-box .row.footer {
    flex: 0 1 auto;
}
```

### 使用这种布局的实际的例子：
- https://pareto-fire.pa-pa.me/
可能需要登录之后，才能看到这个布局
- http://past.buzzbuzzenglish.com/s/player?date=2017-08-24&cat=culture&level=B&lesson_id=b1fe0ea0-7e75-11e7-98fa-ed557983c686&video_path=/resources/buzz/smil/2017-07-24-B.json&new_words_path=/resources/buzz/newWords/2017-07-24-B.json&lesson_id=b1fe0ea0-7e75-11e7-98fa-ed557983c686
使用了这种布局的一个变体，没有头部，但是有一个固定高度的视频区，以及一个撑满剩下空间的字幕区，字幕少时也会占满额外空间，字幕多时会显示滚动条。