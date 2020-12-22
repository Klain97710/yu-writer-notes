# day02

## 添加物体到场景中

```javascript
// geometry: 几何体; CubeGeometry: 立方体
var geometry = new THREE.CubeGeometry(1, 1, 1);
// material: 材质
var material = new THREE.MeshBasicMateria({color: 0x00ff00});
// mesh: 使...结合，纹理
var cube = new THREE.Mesh(geometry, material);

scene.add(cube);
```

`CubeGeometry`是一个正方体或者长方体，究竟是什么，由它的3个参数所决定，`CubeGeometry`的原型如下代码所示：

```javascript
CubeGeometry(width, height, depth, segmentsWidth, segmentsHeight, segmentsDepth, materials, sides)
```

`width`：立方体x轴的长度；
`height`：立方体y轴的长度；
`depth`：立方体z轴的深度，也就是长度。

## 渲染

渲染应该使用渲染器，结合相机和场景来得到结果画面。实现这个功能的函数是
`renderer.render(scene, camera)`
渲染函数的原型如下：
`render( scene, camera, renderTarget, forceClear )`

各个参数的意义是：
`scene`：前面定义的场景；
`camera`：前面定义的相机；
`renderTarget`：渲染的目标，默认是渲染到前面定义的render变量中；
`forceClear`：每次绘制之前都将画布的内容给清除，即使自动清除标志autoClear为false，也会清除。

## 渲染循环

渲染有两种方式：实时渲染和离线渲染 。
举个栗子来理解两者，玩王者荣耀的时候，教学视频的渲染是离线渲染，对决中的渲染是实时渲染。

实时渲染需要不停的对画面进行渲染，即使画面中什么也没有改变，也需要重新渲染。
那怎么进行这种操作呢？**循环**。

```javascript
function render() {
  cube.rotation.x += 0.1;
  cube.rotation.y += 0.1;
  renderer.render(scene, camera);
  requestAnimationFrame(render);
}
```

## 重构

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Three框架</title>
    <script src="js/Three.js" data-ke-src="js/Three.js"></script>
    <style type="text/css">
      div#canvas-frame {
        border: none;
        cursor: pointer;
        width: 100%;
        height: 600px;
        background-color: #EEEEEE;
      }
    </style>
    <script>
      var renderer;

      function initThree() {
        width = document.getElementById('canvas-frame').clientWidth;
        height = document.getElementById('canvas-frame').clientHeight;
        renderer = new THREE.WebGLRenderer({
          antialias: true
        });
        renderer.setSize(width, height);
        document.getElementById('canvas-frame').appendChild(renderer.domElement);
        renderer.setClearColor(0xFFFFFF, 1.0);
      }

      var camera;

      function initCamera() {
        camera = new THREE.PerspectiveCamera(45, width / height, 1, 10000);
        camera.position.x = 0;
        camera.position.y = 1000;
        camera.position.z = 0;
        camera.up.x = 0;
        camera.up.y = 0;
        camera.up.z = 1;
        camera.lookAt({
          x: 0,
          y: 0,
          z: 0
        });
      }

      var scene;

      function initScene() {
        scene = new THREE.Scene();
      }

      var light;

      function initLight() {
        light = new THREE.DirectionalLight(0xFF0000, 1.0, 0);
        light.position.set(100, 100, 200);
        scene.add(light);
      }

      var cube;

      function initObject() {

        var geometry = new THREE.Geometry();
        var material = new THREE.LineBasicMaterial({
          vertexColors: THREE.VertexColors
        });
        var color1 = new THREE.Color(0x444444),
          color2 = new THREE.Color(0xFF0000);

        // 线的材质可以由2点的颜色决定
        var p1 = new THREE.Vector3(-100, 0, 100);
        var p2 = new THREE.Vector3(100, 0, -100);
        geometry.vertices.push(p1);
        geometry.vertices.push(p2);
        geometry.colors.push(color1, color2);

        var line = new THREE.Line(geometry, material, THREE.LinePieces);
        scene.add(line);
      }

      function render() {
        renderer.clear();
        renderer.render(scene, camera);
        requestAnimationFrame(render);
      }

      function threeStart() {
        initThree();
        initCamera();
        initScene();
        initLight();
        initObject();
        render();
      }
    </script>
  </head>

  <body onload="threeStart();">
    <div id="canvas-frame"></div>
  </body>
</html>
```
