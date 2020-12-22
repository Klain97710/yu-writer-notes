# day01

## 一、三大组件(三要素)

### 1. 场景(scene)

场景是所有物体的**容器**，如果要显示一个苹果，就需要将苹果对象加入场景中。
`var scene = new THREE.Scene()`

### 2. 相机(camera)

相机决定了场景中哪个角度的景色会显示出来。相机就像人的眼睛一样，人站在不同位置，抬头或者低头都能够看到不同的景色。

场景只有一种，但是相机却又很多种。和现实中一样，不同的相机确定了呈相的各个方面。比如有的相机适合人像，有的相机适合风景，专业的摄影师根据实际用途不一样，选择不同的相机。对程序员来说，只要设置不同的**相机参数**，就能够让相机产生不一样的效果。

```javascript
// 这是一个透视相机
var camera =  new  THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
```

### 3. 渲染器(renderer)

渲染器决定了渲染的结果应该画在页面的什么元素上面，并且以怎样的**方式**来绘制。

```javascript
var renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);
```

注意，渲染器`renderer`的`domElement`元素，表示渲染器中的画布，所有的渲染都是画在`domElement`上的，所以这里的`appendChild`表示将这个`domElement`挂接在`body`下面，这样渲染的结果就能够在页面中显示了。
