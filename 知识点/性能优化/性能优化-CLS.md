## 什么是CLS以及如何优化它

累计布局偏移: Cumulative Layout Shift, 简称CLS，是Google提出的一项非常重要的网页性能指标，以用户为中心的内容视觉稳定性指标，因为它有助于量化用户体验到意外布局移位的频率，较低的CLS有助于确保页面用户视觉和交互体验。

当用户浏览一个页面的时候，若是想要点击一个按钮或者其他交互时，页面的布局突然出然抖动，可以会造成用户的交互行为造成期望之外的结果。

![CLS1](/Users/nordon/Desktop/me/markdowns/%E7%9F%A5%E8%AF%86%E7%82%B9/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/CLS1.gif)

页面内容的意外移动通常发生在资源异步加载或DOM元素被动态添加到现有内容之上的页面上。罪魁祸首可能是尺寸未知的图像或视频、显示大于或小于回退的字体、或动态调整自身大小的第三方广告或小部件。

使这个问题更加严重的是，一个项目在开发过程中的功能通常与用户的体验有很大的不同。个性化或第三方内容在开发时的表现通常与在生产时不一样，测试图像通常已经存在于开发人员的浏览器缓存中，本地运行的API调用通常非常快，因此延迟并不明显。

比如常见的页面banner图，当图片资源加载缓慢的时候，并不影响浏览器解析DOM、CSSOM合成Layout Tree等操作，当页面整体渲染完成之后图片突然加载完成并渲染到页面上，便会造成严重的CLS。

Google一直是将用户体验作为首要任务，但是在现实中往往事与愿违，很多流行且具有影响的网站并没有将用户体验作为首要任务，少数前端开发者也并没有过多关注用户体验，而CLS便是影响用户体验的重要因素之一。

### CLS分数

CLS得分越低,代表页面的布局越摁钉。官方CLS分数使用谷歌的性能工具如下

![image-20201210131931006](/Users/nordon/Desktop/me/markdowns/%E7%9F%A5%E8%AF%86%E7%82%B9/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/image-20201210131931006.png)

- Good

  CLS评分低于0.1分，代表页面很稳定

- Needs

  CLS评分在0.1-0.25之间，代表页面稳定性欠缺，需要对页面进行优化

- Poor

  CLS评分超过0.25，代表页面的稳定性已经比较渣渣了，页面的优化已经迫在眉睫

### 评分工具

知道CLS分数，接下来是官方给出的一些CLS评分工具

#### Field tools 

- [Chrome User Experience Report](https://developers.google.com/web/tools/chrome-user-experience-report)
- [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)
- [Search Console (Core Web Vitals report)](https://support.google.com/webmasters/answer/9205520)
- [`web-vitals` JavaScript library](https://github.com/GoogleChrome/web-vitals)

#### Lab tools

- [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/)

![image-20201210143742204](/Users/nordon/Desktop/me/markdowns/%E7%9F%A5%E8%AF%86%E7%82%B9/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/image-20201210143742204.png)

- [Lighthouse](https://developers.google.com/web/tools/lighthouse/)

  ![image-20201210143648918](/Users/nordon/Desktop/me/markdowns/%E7%9F%A5%E8%AF%86%E7%82%B9/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/image-20201210143648918.png)

- [WebPageTest](https://webpagetest.org/)

### 影响CLS因素

- 没有为图片设置尺寸
- 没有为广告位、iframes等设置尺寸
- 动态注入渲染内容
- 网页字体导致FOIT/TOUT
- 在一些异步操作(newwork)之前对DOM进行操作

### 如何优化CLS

#### 字体优化

字体在未下载完成时,浏览器隐藏或者自动降级,导致字体闪烁 

- FOIT: Flash Of Invisible Text
- FOUT: Flash Of Unstyled text

使用 font-display属性解决闪烁的问题

```css
@font-face {
    font-dispaly: block | swap | fallback | optional | auto;
}
```

 使用css font loading API

```css
@font-face {
     unicode-range: ....; // 若是一次性将所有的字体都引入,会导致加载的内容非常多,例如汉字那么多,不可能一次性全部加载,因此可以将汉字使用频率进行拆分,然后按照优先级进行加载,且只有当其中的字体被使用的时候才会进行加载
}
```

使用**preload**改变资源加载优先级进行预加载

```html
<!-- 必须使用corssorigin="anonymous" 解决跨域问题 -->
<link rel="preload" href="./xxx.woff2" as="font" type="font/woff2" crossorigin="anonymous" >
```

### 图片、视频、iframes、广告位等优化

为**image**、**video**等元素设置尺寸属性，可以保证在其资源未加载完成之前有一个很好的占位效果，不会造成CLS

```html
<img src="xx.png" width="200" height="200" alt="png" />
```

