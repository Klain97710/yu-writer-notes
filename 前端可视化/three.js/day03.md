# day03

## 点

在三维空间中的某一个点可以用一个坐标点来表示。一个坐标点由x,y,z三个分量构成。在three.js中，点可以在右手坐标系中表示：

空间几何中，点可以用一个向量来表示，在Three.js中也是用一个向量来表示的，代码如下所示：

```javascript
THREE.Vector3 = function ( x, y, z ) {
  this.x = x || 0;
  this.y = y || 0;
  this.z = z || 0;
};
```

定义一个点：

```javascript
// 构造函数定义
var point1 =  new  THREE.Vecotr3(4,8,9);

// 内置方法定义
var point1 = new THREE.Vector3();
point1.set(4,8,9);
```

