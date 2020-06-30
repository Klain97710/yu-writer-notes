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

## css当文字过多时以...省略

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
