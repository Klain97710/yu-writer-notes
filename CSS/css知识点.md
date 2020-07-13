# css知识点

## background简写

```css
.example {
  background: color 
              url(img.png) 
              no-repeat 
              scroll 
              center center / 50% 
              content-box content-box;
}
```

## clip(裁剪)属性

元素设置绝对定位后，元素矩形截取`clip:rect(A, B, C, D);`
![clip-rect]($resource/clip-rect.png)

## CSS3 filter(滤镜)属性

```css
/* 修改所有图片的颜色为黑白 (100% 灰度): */
img {
  -webkit-filter: grayscale(100%); /* Chrome, Safari, Opera */
  filter: grayscale(100%);
}
```

## 当文字过多时以...省略

```css
/**单行文本**/
overflow: hidden;
text-overflow: ellipsis;
white-space: nowrap;

/**多行**/
overflow: hidden;
text-overflow: ellipsis;
display: -webkit-box;
-webkit-line-clamp: 2; /**几行**/
-webkit-box-orient: vertical;
```

## 强制文本换行

`word-break: break-all`

## 详解linear-gradient

```css
.notching{
  width: 200px;
  height: 120px;
  border: 1px dashed red;
  background:
  linear-gradient(45deg,red 20px, yellow 30px 50px, red 60px 80px, yellow 90px) center center;
  background-size: 50% 50%;
  background-repeat: no-repeat;
}
```

![linear-gradient]($resource/linear-gradient.png)

分析：
`linear-gradient(45deg,red 20px, yellow 30px 50px, red 60px 80px, yellow 90px) center center;`

* `45deg`：渐变的方向；
* `red (默认0开始) 20px`：第一组色标不用设置起始点，最后一组不用设置结束点；
* 起始、结束点的效果：在起始、结束点之间，背景为设置的色标的颜色，起始、结束点之外的地方，模糊渐变。


