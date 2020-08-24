Title: Three.js　シーングラフ
Description: シーングラフとはなにか？
TOC: シーングラフ

This article is part of a series of articles about three.js. The
first article is [three.js fundamentals](threejs-fundamentals.html). If
you haven't read that yet you might want to consider starting there.
この記事はthree.jsについてのシリーズ記事の一つです。
最初の記事は[Three.jsの基礎](threejs-fundamentals.html)です。
まだ読んでない人は、そちらから先に読んでみるといいかもしれません。

Three.js's core is arguably its scene graph. A scene graph in a 3D
engine is a hierarchy of nodes in a graph where each node represents
a local space.
Three.jsの核心は間違いなくシーングラフです。
3Dエンジンのシーングラフは、各ノードがローカルな空間を表現しているグラフのノードの階層です。

<img src="resources/images/scenegraph-generic.svg" align="center">

That's kind of abstract so let's try to give some examples.
抽象的なので、例をいくつか挙げてみましょう。

One example might be solar system, sun, earth, moon.
例の一つは太陽系、太陽・地球・月でしょうか。

<img src="resources/images/scenegraph-solarsystem.svg" align="center">

The Earth orbits the Sun. The Moon orbits the Earth. The Moon
moves in a circle around the Earth. From the Moon's point of
view it's rotating in the "local space" of the Earth. Even though
its motion relative to the Sun is some crazy spirograph like
curve from the Moon's point of view it just has to concern itself with rotating
around the Earth's local space.
地球は太陽を回っています。月は地球を回っています。
月は地球の周りを円を描いて回って移動しています。月の視点からだと、地球の"ローカルな空間"を回っていることになります。
太陽との相対的な動きは、月の視点から見るとクレイジーな螺旋のような曲線ですが、地球のローカルな空間の周りを回転していると捉える必要があります。

{{{diagram url="resources/moon-orbit.html" }}}

To think of it another way, you living on the Earth do not have to think
about the Earth's rotation on its axis nor its rotation around the
Sun. You just walk or drive or swim or run as though the Earth is
not moving or rotating at all. You walk, drive, swim, run, and live
in the Earth's "local space" even though relative to the sun you are
spinning around the earth at around 1000 miles per hour and around
the sun at around 67,000 miles per hour. Your position in the solar
system is similar to that of the moon above but you don't have to concern
yourself. You just worry about your position relative to the earth in its
"local space".
それを別の方法で考えると、地球に住んでいるあなたが、地球が地軸の周りを自転していることも、
太陽の周りを公転していることも、考える必要はありません。
皆さんは全くもって地球が動きも回りもしていないかのように、歩いたりドライブしたり
泳いだり走ったりするだけです。
地球の"ローカルな空間"で歩いたり、ドライブしたり、泳いだり、走ったり、そして生活したりしていても、みなさんは地球の上で、太陽と相対的に1,600km/h、太陽の周りを107,200km/hの速度で回っています。

Let's take it one step at a time. Imagine we want to make
a diagram of the sun, earth, and moon. We'll start with the sun by
just making a sphere and putting it at the origin. Note: We're using
sun, earth, moon as a demonstration of how to use a scene graph. Of course
the real sun, earth, and moon use physics but for our purposes we'll
fake it with a scene graph.

一歩進みましょう。私たちは太陽と地球と月との図を作りたいと想像してください。
まず、太陽から始めましょう。ただ球体を作り原点に置くだけです。
シーングラフを使う方法のデモンストレーションとして、太陽、地球、月を使うことは、気を留めておいてください。
もちろん、現実の太陽、地球、月は物理学を使いますが、目的のため、私たちはシーングラフで代用します。


```js
// an array of objects whose rotation to update
const objects = [];

// use just one sphere for everything
const radius = 1;
const widthSegments = 6;
const heightSegments = 6;
const sphereGeometry = new THREE.SphereBufferGeometry(
    radius, widthSegments, heightSegments);

const sunMaterial = new THREE.MeshPhongMaterial({emissive: 0xFFFF00});
const sunMesh = new THREE.Mesh(sphereGeometry, sunMaterial);
sunMesh.scale.set(5, 5, 5);  // make the sun large
scene.add(sunMesh);
objects.push(sunMesh);
```

We're using a really low-polygon sphere. Only 6 subdivisions around its equator.
This is so it's easy to see the rotation.
本当に少ないポリゴンから成る球体を使います。緯度方向にたった6分割です。
これで、回転が見やすくなります。

We're going to reuse the same sphere for everything so we'll set a scale
for the sun mesh of 5x.
同じ球体を全ての球体に再利用するつもりなので、太陽のメッシュの大きさを5倍にしておきます。

We also set the phong material's `emissive` property to yellow. A phong material's
emissive property is basically the color that will be drawn with no light hitting
the surface. Light is added to that color.
また、phong materialの`emissive`属性を黄色に設定します。
phong materialのemissive属性は、基本的に光が当たっていない表面に描かれる色です。

Let's also put a single point light in the center of the scene. We'll go into more
details about point lights later but for now the simple version is a point light
represents light that emanates from a single point.
次に、シーンの真ん中に1つ点光源を置きましょう。後ほどより詳細に点光源について説明しますが、
とりあえず簡単な説明は、一点から発せられる明かりです。

```js
{
  const color = 0xFFFFFF;
  const intensity = 3;
  const light = new THREE.PointLight(color, intensity);
  scene.add(light);
}
```

To make it easy to see we're going to put the camera directly above the origin
looking down. The easiest way to do that is to use the `lookAt` function. The `lookAt`
function will orient the camera from its position to "look at" the position
we pass to `lookAt`. Before we do that though we need to tell the camera
which way the top of the camera is facing or rather which way is "up" for the
camera. For most situations positive Y being up is good enough but since
we are looking straight down we need to tell the camera that positive Z is up.
見やすくするために、直接原点を見下ろすようにカメラをおきましょう。
最も簡単な方法は `lookAt`関数を使うことです。
`lookAt`関数は、引数に渡した位置を「見る」ようにカメラの向きを向けます。
それをする前に、カメラの上部がどの方向を向いているか、またはカメラが
どの方向を向いているかを、カメラに伝える必要があります。
ほとんどの場合、正のYが上になるのが十分に良いですが、
見下ろすために、正のZが上だとカメラに伝えるのが必要です。

```js
const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
camera.position.set(0, 50, 0);
camera.up.set(0, 0, 1);
camera.lookAt(0, 0, 0);
```

In the render loop, adapted from previous examples, we're rotating all
objects in our `objects` array with this code.
レンダリングループの中で、前の例を参考にして、以下のコードで、`objects`配列の中の全てのオブジェクトを回転させています。

```js
objects.forEach((obj) => {
  obj.rotation.y = time;
});
```

Since we added the `sunMesh` to the `objects` array it will rotate.
`sunMesh`を`objects`配列に追加したので、回転します。

{{{example url="../threejs-scenegraph-sun.html" }}}

Now let's add in the earth.
さて、地球を追加してみましょう。

```js
const earthMaterial = new THREE.MeshPhongMaterial({color: 0x2233FF, emissive: 0x112244});
const earthMesh = new THREE.Mesh(sphereGeometry, earthMaterial);
earthMesh.position.x = 10;
scene.add(earthMesh);
objects.push(earthMesh);
```

We make a material that is blue but we gave it a small amount of *emissive* blue
so that it will show up against our black background.
青いマテリアルを作っていますが、黒背景に対して目立つよう、
*emissive*に少し青色を設定します。

We use the same `sphereGeometry` with our new blue `earthMaterial` to make
an `earthMesh`. We position that 10 units to the left of the sun
and add it to the scene.  Since we added it to our `objects` array it will
rotate too.
同じ`sphereGeometry`を使っていますが、`earthMesh`を作るため、新しい青色の`earthMaterial`を使っています。それを太陽の10ユニット左側に置き、シーンに追加します。
`objects`配列にそれを追加したので、回転します。


{{{example url="../threejs-scenegraph-sun-earth.html" }}}

You can see both the sun and the earth are rotating but the earth is not
going around the sun. Let's make the earth a child of the sun
太陽と地球の両方が回転して見えますが、地球は太陽の周りを公転していません。
地球を太陽の子にしてみましょう。

```js
-scene.add(earthMesh);
+sunMesh.add(earthMesh);
```

and...

{{{example url="../threejs-scenegraph-sun-earth-orbit.html" }}}

What happened? Why is the earth the same size as the sun and why is it so far away?
I actually had to move the camera from 50 units above to 150 units above to see the earth.
なにが起きましたか？なぜ地球が太陽と同じ大きさで、こんなに離れているのでしょうか。
地球を見るためには、実際にカメラを50ユニット上から、150ユニット上に動かす必要がありました。

We made the `earthMesh` a child of the `sunMesh`. The `sunMesh` has
its scale set to 5x with `sunMesh.scale.set(5, 5, 5)`. That means the
`sunMesh`s local space is 5 times as big. Anything put in that space
 will be multiplied by 5. That means the earth is now 5x larger and
 it's distance from the sun (`earthMesh.position.x = 10`) is also
 5x as well.
`earthMesh`を`sunMesh`の子としていました。
`sunMesh`は`sunMesh.scale.set(5, 5, 5)`によって5倍に大きさを設定しています。
よって、`sunMesh`のローカルな空間は5倍大きくなりました。
その空間におかれるあらゆるものは5倍されるのです。
つまり、地球が5倍大きくなり、太陽からの距離も同じく5倍（`earthMesh.position.x = 10`）になりました。

 Our scene graph currently looks like this
 シーングラフは、このように見えます。

<img src="resources/images/scenegraph-sun-earth.svg" align="center">

To fix it let's add an empty scene graph node. We'll parent both the sun and the earth
to that node.
修正するため。シーングラフに空のノードを追加しましょう。
そして、太陽と地球の両方をそのノードの親にしましょう。

```js
+const solarSystem = new THREE.Object3D();
+scene.add(solarSystem);
+objects.push(solarSystem);

const sunMaterial = new THREE.MeshPhongMaterial({emissive: 0xFFFF00});
const sunMesh = new THREE.Mesh(sphereGeometry, sunMaterial);
sunMesh.scale.set(5, 5, 5);
-scene.add(sunMesh);
+solarSystem.add(sunMesh);
objects.push(sunMesh);

const earthMaterial = new THREE.MeshPhongMaterial({color: 0x2233FF, emissive: 0x112244});
const earthMesh = new THREE.Mesh(sphereGeometry, earthMaterial);
earthMesh.position.x = 10;
-sunMesh.add(earthMesh);
+solarSystem.add(earthMesh);
objects.push(earthMesh);
```

Here we made an `Object3D`. Like a `Mesh` it is also a node in the scene graph
but unlike a `Mesh` it has no material or geometry. It just represents a local space.
ここで`Object3D`を作りました。`Mesh`のように、シーングラフのノードですが、`Mesh`とは異なり、マテリアルかジオメトリを持ちません。
ただローカルな空間を表現するだけです。

Our new scene graph looks like this
新しいシーングラフはこのように見えます。

<img src="resources/images/scenegraph-sun-earth-fixed.svg" align="center">

Both the `sunMesh` and the `earthMesh` are children of the `solarSystem`. All 3
are being rotated and now because the `earthMesh` is not a child of the `sunMesh`
it is no longer scaled by 5x.
`sunMesh`と`earthMesh`は共に`solarSystem`の子要素です。3つ全部が回転していますが、
いま`earthMesh`は`sunMesh`の子要素ではないので、5倍にスケールされません。

{{{example url="../threejs-scenegraph-sun-earth-orbit-fixed.html" }}}

Much better. The earth is smaller than the sun and it's rotating around the sun
and rotating itself.
とてもよくなりました。地球は太陽よりも小さく、太陽の周りを公転しつつ、自転しています。

Continuing that same pattern let's add a moon.
続けて、同様の方法で月を追加してみましょう。

```js
+const earthOrbit = new THREE.Object3D();
+earthOrbit.position.x = 10;
+solarSystem.add(earthOrbit);
+objects.push(earthOrbit);

const earthMaterial = new THREE.MeshPhongMaterial({color: 0x2233FF, emissive: 0x112244});
const earthMesh = new THREE.Mesh(sphereGeometry, earthMaterial);
-solarSystem.add(earthMesh);
+earthOrbit.add(earthMesh);
objects.push(earthMesh);

+const moonOrbit = new THREE.Object3D();
+moonOrbit.position.x = 2;
+earthOrbit.add(moonOrbit);

+const moonMaterial = new THREE.MeshPhongMaterial({color: 0x888888, emissive: 0x222222});
+const moonMesh = new THREE.Mesh(sphereGeometry, moonMaterial);
+moonMesh.scale.set(.5, .5, .5);
+moonOrbit.add(moonMesh);
+objects.push(moonMesh);
```

Again we added another invisible scene graph node, an `Object3D` called `earthOrbit`
and added both the `earthMesh` and the `moonMesh` to it. The new scene graph looks like
this.
再び、描画されないシーングラフのノードを追加しました。これは、`earthOrbit`と呼ばれる`Object3D`です。
そして、このノードに`earthMesh`と`moonMesh`の両方を追加しました。
新しいシーングラフはこのように見えます。

<img src="resources/images/scenegraph-sun-earth-moon.svg" align="center">

and here's that
そして、このように描画されます。

{{{example url="../threejs-scenegraph-sun-earth-moon.html" }}}

You can see the moon follows the spirograph pattern shown at the top
of this article but we didn't have to manually compute it. We just
setup our scene graph to do it for us.
記事の上部でお見せした螺旋のパターンに沿った月が見えます。しかし、手動で操作する必要はありませんでした。
ただシーングラフをそれが可能なようにします。

It is often useful to draw something to visualize the nodes in the scene graph.
Three.js has some helpful ummmm, helpers to ummm, ... help with this.
シーンフラフでノードが見えるように何かを書くことは便利です。
Three.jsはいくつかの便利なうーむを持っていて、うーむを補助し、これを助けます。

One is called an `AxesHelper`. It draws 3 lines representing the local
<span style="color:red">X</span>,
<span style="color:green">Y</span>, and
<span style="color:blue">Z</span> axes. Let's add one to every node we
created.

```js
// add an AxesHelper to each node
objects.forEach((node) => {
  const axes = new THREE.AxesHelper();
  axes.material.depthTest = false;
  axes.renderOrder = 1;
  node.add(axes);
});
```

On our case we want the axes to appear even though they are inside the spheres.
To do this we set their material's `depthTest` to false which means they will
not check to see if they are drawing behind something else. We also
set their `renderOrder` to 1 (the default is 0) so that they get drawn after
all the spheres. Otherwise a sphere might draw over them and cover them up.
私たちの場合、たとえ球体の内側であったとしても、軸を表示させたいです。
これをするために、マテリアルの`depthTest`をfalseにします。
これによって、軸がなにかの内側に描画されているかどうかチェックしなくなります。
全ての球体の後に描画されるように、`renderOrder`も1に設定します（デフォルト値は0です）。
そうしないと、球体が軸の上に描画され、軸を覆ってしまう可能性があります。

{{{example url="../threejs-scenegraph-sun-earth-moon-axes.html" }}}

We can see the
<span style="color:red">x (red)</span> and
<span style="color:blue">z (blue)</span> axes. Since we are looking
straight down and each of our objects is only rotating around its
y axis we don't see much of the <span style="color:green">y (green)</span> axes.
<span style="color:red">x (赤)</span> と<span style="color:blue">z (青)</span>の
軸が見えます。私たちはオブジェクトをまっすぐ見下ろしていて、オブジェクトはy軸を中心に
回転しているので、 <span style="color:green">y (緑)</span>軸があまり見えません。

It might be hard to see some of them as there are 2 pairs of overlapping axes. Both the `sunMesh`
and the `solarSystem` are at the same position. Similarly the `earthMesh` and
`earthOrbit` are at the same position. Let's add some simple controls to allow us
to turn them on/off for each node.
While we're at it let's also add another helper called the `GridHelper`. It
makes a 2D grid on the X,Z plane. By default the grid is 10x10 units.
位置が重なった軸が2組あるので、見づらいかもしれません。
`sunMesh`と`solarSystem`は同じ場所にあります。
同様に、`earthMesh`と`earthOrbit`は同じ場所にあります。
各ノードに対してオン/オフできるように、簡単な操作を加えてみましょう。
そのついでに、`GridHelper` というヘルパー関数も追加しておきましょう。
これはX,Z平面に2次元グリッドを作ります。デフォルトでは、グリッドは10x10ユニットです。

We're also going to use [dat.GUI](https://github.com/dataarts/dat.gui) which is
a UI library that is very popular with three.js projects. dat.GUI takes an
object and a property name on that object and based on the type of the property
automatically makes a UI to manipulate that property.
[dat.GUI](https://github.com/dataarts/dat.gui)も使います。
これはthree.jsプロジェクトでとても一般的なUIライブラリです。
dat.GUIはオブジェクトとそのオブジェクトの属性名を受け取り、
プロパティの型に基づいて自動的にその属性を操作するUIを作成します。

We want to make both a `GridHelper` and an `AxesHelper` for each node. We need
a label for each node so we'll get rid of the old loop and switch to calling
some function to add the helpers for each node
それぞれのノードに対して、`GridHelper`と`AxesHelper`の両方を作りたいです。
それぞれのノートにラベルが必要なので、古いループを削除し、
各ノードのhelperを加える関数を呼ぶ形に切り替えていきます。

```js
-// add an AxesHelper to each node
-objects.forEach((node) => {
-  const axes = new THREE.AxesHelper();
-  axes.material.depthTest = false;
-  axes.renderOrder = 1;
-  node.add(axes);
-});

+function makeAxisGrid(node, label, units) {
+  const helper = new AxisGridHelper(node, units);
+  gui.add(helper, 'visible').name(label);
+}
+
+makeAxisGrid(solarSystem, 'solarSystem', 25);
+makeAxisGrid(sunMesh, 'sunMesh');
+makeAxisGrid(earthOrbit, 'earthOrbit');
+makeAxisGrid(earthMesh, 'earthMesh');
+makeAxisGrid(moonMesh, 'moonMesh');
```

`makeAxisGrid` makes an `AxisGridHelper` which is a class we'll create
to make dat.GUI happy. Like it says above dat.GUI
will automagically make a UI that manipulates the named property
of some object. It will create a different UI depending on the type
of property. We want it to create a checkbox so we need to specify
a `bool` property. But, we want both the axes and the grid
to appear/disappear based on a single property so we'll make a class
that has a getter and setter for a property. That way we can let dat.GUI
think it's manipulating a single property but internally we can set
the visible property of both the `AxesHelper` and `GridHelper` for a node.
`makeAxisGrid`は、dat.GUIをハッピーにする`AxisGridHelper`クラスを作ります。
前述したように、dat.GUIは、オブジェクトの名前が付いた属性を操作するUIを自動的に生成します。
属性の型に応じて異なるUIが作成されます。
チェックボックスを作って欲しいので、`bool`属性を指定する必要があります。
しかし、軸とグリッドの両方を一つの属性で表示/非表示にしたいので、属性のgetterとsetterを
持ったクラスを作成します。
この方法で、dat.GUIに一つの属性を操作するように思わせることができますが、
内部的には各ノードに`AxesHelper`と`GridHelper`の両方のvisibleプロパティを設定することができます。


```js
// Turns both axes and grid visible on/off
// dat.GUI requires a property that returns a bool
// to decide to make a checkbox so we make a setter
// and getter for `visible` which we can tell dat.GUI
// to look at.
class AxisGridHelper {
  constructor(node, units = 10) {
    const axes = new THREE.AxesHelper();
    axes.material.depthTest = false;
    axes.renderOrder = 2;  // after the grid
    node.add(axes);

    const grid = new THREE.GridHelper(units, units);
    grid.material.depthTest = false;
    grid.renderOrder = 1;
    node.add(grid);

    this.grid = grid;
    this.axes = axes;
    this.visible = false;
  }
  get visible() {
    return this._visible;
  }
  set visible(v) {
    this._visible = v;
    this.grid.visible = v;
    this.axes.visible = v;
  }
}
```

One thing to notice is we set the `renderOrder` of the `AxesHelper`
to 2 and for the `GridHelper` to 1 so that the axes get drawn after the grid.
Otherwise the grid might overwrite the axes.
注意することは`AxesHelper`の`renderOrder`を2に設定し、`GridHelper`には1を設定することです。
こうすることで、軸はグリッドの後に描画されます。
そうしないと、グリッドが軸を上書きしてしまうかもしれません。

{{{example url="../threejs-scenegraph-sun-earth-moon-axes-grids.html" }}}

Turn on the `solarSystem` and you'll see how the earth is exactly 10
units out from the center just like we set above. You can see how the
earth is in the *local space* of the `solarSystem`. Similarly if you
turn on the `earthOrbit` you'll see how the moon is exactly 2 units
from the center of the *local space* of the `earthOrbit`.
`solarSystem`のチェックをオンにすると、まさに先ほど設定したように、
どのように地球が中心からちょうど10ユニットにあるか分かるでしょう。
地球が`solarSystem`の*ローカルな空間*にどのように存在するか見ることができます。
同様に、もし`earthOrbit`のチェックをオンにすると、
どのように月が`earthOrbit`の*ローカルな空間*の中心から、ちょうど2ユニットあるか分かるでしょう。

A few more examples of scene graphs. An automobile in a simple game world might have a scene graph like this
もう少しシーングラフの例を紹介します。
簡単なゲームの世界の自動車はこのようなシーングラフを持っているかもしれません。

<img src="resources/images/scenegraph-car.svg" align="center">

If you move the car's body all the wheels will move with it. If you wanted the body
to bounce separate from the wheels you might parent the body and the wheels to a "frame" node
that represents the car's frame.
もし車のbody全体を動かすと、それに伴ってwheelsが動くでしょう。
もしbodyにwheelsとは別にバウンドして欲しいとすると、
bodyとwheelsを、車のフレームを表す"frame"ノードの親とすることができます。

Another example is a human in a game world.
別の例はゲームの世界の人間です。

<img src="resources/images/scenegraph-human.svg" align="center">

You can see the scene graph gets pretty complex for a human. In fact
that scene graph above is simplified. For example you might extend it
to cover every finger (at least another 28 nodes) and every toe
(yet another 28 nodes) plus ones for the face and jaw, the eyes and maybe more.
とても複雑になった人間のシーングラフを見てください。
実際は、上記のシーングラフは単純化されています。例えば、全ての手の指(少なくとも28ノード)、
全ての足の指(さらに28ノード)、加えて顔と顎、目とたぶんもっとほかの部位を
カバーするように、拡張できるかもしれません。


Let's make one semi-complex scene graph. We'll make a tank. The tank will have
6 wheels and a turret. The tank will follow a path. There will be a sphere that
moves around and the tank will target the sphere.
もう少し複雑なシーングラフを作りましょう。戦車を作ります。
戦車は6つの車輪と砲塔があります。戦車はある道筋に沿って走ります。
そこら中を移動する球体があり、戦車はその球体を狙うとしましょう。

Here's the scene graph. The meshes are colored in green, the `Object3D`s in blue,
the lights in gold, and the cameras in purple. One camera has not been added
to the scene graph.
これがシーングラフです。メッシュは緑色、`Object3D`は青色、明かりは金色、カメラは紫色です。
シーングラフに追加されていないカメラが一つあります。

<div class="threejs_center"><img src="resources/images/scenegraph-tank.svg" style="width: 800px;"></div>

Look in the code to see the setup of all of these nodes.
コードを見て、これらのノードの設定を確認してください。

For the target, the thing the tank is aiming at, there is a `targetOrbit`
(`Object3D`) which just rotates similar to the `earthOrbit` above. A
`targetElevation` (`Object3D`) which is a child of the `targetOrbit` provides an
offset from the `targetOrbit` and a base elevation. Childed to that is another
`Object3D` called `targetBob` which just bobs up and down relative to the
`targetElevation`. Finally there's the `targetMesh` which is just a cube we
rotate and change it's colors
ターゲット、つまり戦車が狙っているもののために、`targetOrbit`(`Object3D`) があります。
これはちょうど前述の`earthOrbit`と同じように回転します。
`targetOrbit`の子要素である`targetElevation` (`Object3D`)は、`targetOrbit`と基準となる高さからの
オフセットを提供します。
この子要素には、`targetElevation`に対して相対的に浮き沈みする、`targetBob`と呼ばれる別の`Object3D`があります。
最後に、回転させて色を変えることができるだけの立方体である`targetMesh`があります。


```js
// move target
targetOrbit.rotation.y = time * .27;
targetBob.position.y = Math.sin(time * 2) * 4;
targetMesh.rotation.x = time * 7;
targetMesh.rotation.y = time * 13;
targetMaterial.emissive.setHSL(time * 10 % 1, 1, .25);
targetMaterial.color.setHSL(time * 10 % 1, 1, .25);
```

For the tank there's an `Object3D` called `tank` which is used to move everything
below it around. The code uses a `SplineCurve` which it can ask for positions
along that curve. 0.0 is the start of the curve. 1.0 is the end of the curve. It
asks for the current position where it puts the tank. It then asks for a
position slightly further down the curve and uses that to point the tank in that
direction using `Object3D.lookAt`.
戦車には、`tank`と呼ばれる`Object3D`があります。
これを使って戦車の下のものをすべて移動させることができます。
コードでは`SplineCurve`を使っています。これは曲線に沿った位置を求めることができます。
0.0は曲線の始点です。1.0は曲線の終点です。これにより、戦車がある現在地を求めます。
次に、カーブの少し下の方の位置を求めて、`Object3D.lookAt`を使い、戦車をその方向に向けます。


```js
const tankPosition = new THREE.Vector2();
const tankTarget = new THREE.Vector2();

...

// move tank
const tankTime = time * .05;
curve.getPointAt(tankTime % 1, tankPosition);
curve.getPointAt((tankTime + 0.01) % 1, tankTarget);
tank.position.set(tankPosition.x, 0, tankPosition.y);
tank.lookAt(tankTarget.x, 0, tankTarget.y);
```

The turret on top of the tank is moved automatically by being a child
of the tank. To point it at the target we just ask for the target's world position
and then again use `Object3D.lookAt`
戦車のてっぺんに付いている砲塔は、戦車の子要素なので自動的に動かされます。
ターゲットの方を向かせるのに、ターゲットの世界の位置を求め、次に再度`Object3D.lookAt`を使うだけです。

```js
const targetPosition = new THREE.Vector3();

...

// face turret at target
targetMesh.getWorldPosition(targetPosition);
turretPivot.lookAt(targetPosition);
```

There's a `turretCamera` which is a child of the `turretMesh` so
it will move up and down and rotate with the turret. We make that
aim at the target.
`turretMesh`の子要素である`turretCamera`があるので、砲塔と一緒に上下に動き、回転します。


```js
// make the turretCamera look at target
turretCamera.lookAt(targetPosition);
```

There is also a `targetCameraPivot` which is a child of `targetBob` so it floats
around with the target. We aim that back at the tank. It's purpose is to allow the
`targetCamera` to be offset from the target itself. If we instead made the camera
a child of `targetBob` and just aimed the camera itself it would be inside the
target.
`targetBob`の子要素である`targetCameraPivot`もあります。これはターゲットと浮遊します。
戦車に狙いを定めましょう。`targetCamera`にターゲット自身に高さを合わせるためです。
もし代わりにカメラを`targetBob`の子要素にして、カメラ自身に狙いを定めさせると、
ターゲットの内側になってしまうでしょう。

```js
// make the targetCameraPivot look at the tank
tank.getWorldPosition(targetPosition);
targetCameraPivot.lookAt(targetPosition);
```

Finally we rotate all the wheels
最後に、全ての車輪を回転させます。

```js
wheelMeshes.forEach((obj) => {
  obj.rotation.x = time * 3;
});
```

For the cameras we setup an array of all 4 cameras at init time with descriptions.
初期化時に、4つ全てのカメラの配列を設定します。

```js
const cameras = [
  { cam: camera, desc: 'detached camera', },
  { cam: turretCamera, desc: 'on turret looking at target', },
  { cam: targetCamera, desc: 'near target looking at tank', },
  { cam: tankCamera, desc: 'above back of tank', },
];

const infoElem = document.querySelector('#info');
```

and cycle through our cameras at render time.
描画時にカメラを周回させます。

```js
const camera = cameras[time * .25 % cameras.length | 0];
infoElem.textContent = camera.desc;
```

{{{example url="../threejs-scenegraph-tank.html"}}}

I hope this gives some idea of how scene graphs work and how you might use them.
Making `Object3D` nodes and parenting things to them is an important step to using
a 3D engine like three.js well. Often it might seem like some complex math is necessary
to make something move and rotate the way you want. For example without a scene graph
computing the motion of the moon or where to put the wheels of the car relative to its
body would be very complicated but using a scene graph it becomes much easier.
どのようにシーングラフが動作し、皆さんがそれを使えるかのアイデアをこの例から得られることを願っています。
`Object3D`ノードを作り、物体をその親にすることは、three.jsのような3Dエンジンを上手く使うために
重要なステップです。
思い通りになにかを動かしたり回転させたりすることは、しばしば複雑な数学が必要に見えるかもしれません。
例えばシーングラフなしで、月の動きを操作したり、車の車体に対して壮太知的に車輪を置いたりすることは、
とても難しいかもしれません。しかし、シーングラフを使うことでとても簡単になるのです。

[Next up we'll go over materials](threejs-materials.html).
[次はマテリアルを説明します](threejs-materials.html)。