Title: Three.jsのテクスチャ
Description: Three.jsのテクスチャの使い方
TOC: テクスチャ

This article is one in a series of articles about three.js.
The first article was [about three.js fundamentals](threejs-fundamentals.html).
The [previous article](threejs-setup.html) was about setting up for this article.
If you haven't read that yet you might want to start there.
この記事はthree.jsについてのシリーズ記事の一つです。
最初の記事は[Three.jsの基礎知識](threejs-fundamentals.html)です。
まだ読んでない人は、そちらから先に読んでみるといいかもしれません。

Textures are a kind of large topic in Three.js and
I'm not 100% sure at what level to explain them but I will try.
There are many topics and many of them interrelate so it's hard to explain
them all at once. Here's quick table of contents for this article.
テクスチャはThree.jsの大きなトピックの一つです。
どのレベルで説明するといいか100%承知してはいませんが、やってみようと思います。 
Three.jsにはたくさんのトピックがあり、互いに関係しているので、一度に説明するのが難しいのです。
これがこの記事の内容の早見表です。
<ul>
<li><a href="#hello">Hello Texture</a></li>
<li><a href="#six">6 textures, a different one on each face of a cube</a></li>
<li><a href="#loading">Loading textures</a></li>
<ul>
  <li><a href="#easy">The easy way</a></li>
  <li><a href="#wait1">Waiting for a texture to load</a></li>
  <li><a href="#waitmany">Waiting for multiple textures to load</a></li>
  <li><a href="#cors">Loading textures from other origins</a></li>
</ul>
<li><a href="#memory">Memory usage</a></li>
<li><a href="#format">JPG vs PNG</a></li>
<li><a href="#filtering-and-mips">Filtering and mips</a></li>
<li><a href="#uvmanipulation">Repeating, offseting, rotating, wrapping</a></li>
</ul>

<ul>
<li><a href="#hello">ハロー・テクスチャ</a></li>
<li><a href="#six">立方体の各面に異なる6つのテクスチャを貼り付ける</a></li>
<li><a href="#loading">テクスチャの読み込み</a></li>
<ul>
  <li><a href="#easy">簡単な方法</a></li>
  <li><a href="#wait1">テクスチャの読み込みを待つ</a></li>
  <li><a href="#waitmany">複数テクスチャの読み込みを待つ</a></li>
  <li><a href="#cors">異なるオリジンからのテクスチャの読み込み</a></li>
</ul>
<li><a href="#memory">メモリ使用</a></li>
<li><a href="#format">JPG vs PNG</a></li>
<li><a href="#filtering-and-mips">フィルタリングとMIP</a></li>
<li><a href="#uvmanipulation">テクスチャの繰り返し、オフセット、回転、ラッピング</a></li>
</ul>

## <a name="hello"></a> ハロー・テクスチャ

Textures are *generally* images that are most often created
in some 3rd party program like Photoshop or GIMP. For example let's
put this image on cube.
テクスチャは*一般的に*PhotoshopやGIMPのような3rdパーティーのプログラムで最もよく作られる画像です。

<div class="threejs_center">
  <img src="../resources/images/wall.jpg" style="width: 600px;" class="border" >
</div>

We'll modify one of our first samples. All we need to do is create a `TextureLoader`. Call its
[`load`](TextureLoader.load) method with the URL of an
image and and set the material's `map` property to the result instead of setting its `color`.
最初の例を修正してみましょう。`TextureLoader`を作ることで、必要なことはすべてできます。
[`load`](TextureLoader.load)を画像のURLを引数にして呼び、`color`を設定する代わりに、
マテリアルの`map`属性にその結果を渡してください。

```js
+const loader = new THREE.TextureLoader();

const material = new THREE.MeshBasicMaterial({
-  color: 0xFF8844,
+  map: loader.load('resources/images/wall.jpg'),
});
```

Note that we're using `MeshBasicMaterial` so no need for any lights.
`MeshBasicMaterial`を使っているので、光源が必要ないことに注意してください。

{{{example url="../threejs-textured-cube.html" }}}

## <a name="six"></a> 6 Textures, a different one on each face of a cube
## <a name="six"></a> 立方体の各面に異なる6つのテクスチャを貼り付ける

How about 6 textures, one on each face of a cube?
立方体の各面に貼り付ける、6つのテクスチャはどのようなものでしょうか。

<div class="threejs_center">
  <div>
    <img src="../resources/images/flower-1.jpg" style="width: 100px;" class="border" >
    <img src="../resources/images/flower-2.jpg" style="width: 100px;" class="border" >
    <img src="../resources/images/flower-3.jpg" style="width: 100px;" class="border" >
  </div>
  <div>
    <img src="../resources/images/flower-4.jpg" style="width: 100px;" class="border" >
    <img src="../resources/images/flower-5.jpg" style="width: 100px;" class="border" >
    <img src="../resources/images/flower-6.jpg" style="width: 100px;" class="border" >
  </div>
</div>

We just make 6 materials and pass them as an array when we create the `Mesh`
`Mesh`を作るときに、単に6つのマテリアルを作り、配列として渡します。

```js
const loader = new THREE.TextureLoader();

-const material = new THREE.MeshBasicMaterial({
-  map: loader.load('resources/images/wall.jpg'),
-});
+const materials = [
+  new THREE.MeshBasicMaterial({map: loader.load('resources/images/flower-1.jpg')}),
+  new THREE.MeshBasicMaterial({map: loader.load('resources/images/flower-2.jpg')}),
+  new THREE.MeshBasicMaterial({map: loader.load('resources/images/flower-3.jpg')}),
+  new THREE.MeshBasicMaterial({map: loader.load('resources/images/flower-4.jpg')}),
+  new THREE.MeshBasicMaterial({map: loader.load('resources/images/flower-5.jpg')}),
+  new THREE.MeshBasicMaterial({map: loader.load('resources/images/flower-6.jpg')}),
+];
-const cube = new THREE.Mesh(geometry, material);
+const cube = new THREE.Mesh(geometry, materials);
```

動きました！

{{{example url="../threejs-textured-cube-6-textures.html" }}}

It should be noted though that not all geometry types supports multiple
materials. `BoxGeometry` and `BoxBufferGeometry` can use 6 materials one for each face.
`ConeGeometry` and `ConeBufferGeometry` can use 2 materials, one for the bottom and one for the cone.
`CylinderGeometry` and `CylinderBufferGeometry` can use 3 materials, bottom, top, and side.
For other cases you will need to build or load custom geometry and/or modify texture coordinates.
ただし、全ての種類のジオメトリが複数のマテリアルに対応しているわけではないことに注意してください。
`BoxGeometry`と`BoxBufferGeometry`は、それぞれの面に6つのマテリアルを使えます。
`ConeGeometry`と`ConeBufferGeometry`は2つのマテリアルを使うことができ、一つは底面、一つは円錐面に適用されます。
`CylinderGeometry`と`CylinderBufferGeometry`は3つのマテリアルを使うことができ、一つは底面、一つは上面、一つは側面に適用されます。
その他のケースでは、カスタムジオメトリのビルドや読み込み、テクスチャの座標の修正が必要になります。


It's far more common in other 3D engines and far more performant to use a
[Texture Atlas](https://en.wikipedia.org/wiki/Texture_atlas)
if you want to allow multiple images on a single geometry. A Texture atlas
is where you put multiple images in a single texture and then use texture coordinates
on the vertices of your geometry to select which parts of a texture are used on
each triangle in your geometry.
1つのジオメトリに複数の画像を適用したいなら、
[テクスチャアトラス](https://en.wikipedia.org/wiki/Texture_atlas)を使うのが、ほかの3Dエンジンでははるかに一般的で、はるかに高性能です。
テクスチャアトラスは、一つのテクスチャに複数の画像を配置し、ジオメトリの頂点の座標を使って
テクスチャのどの部分がジオメトリのおのおのの三角形に使われるか、選択するものです。

What are texture coordinates? They are data added to each vertex of a piece of geometry
that specify what part of the texture corresponds to that specific vertex.
We'll go over them when we start [building custom geometry](threejs-custom-geometry.html).
テクスチャの座標とはなんでしょうか？ジオメトリ頂点に与えられたデータのことで、
テクスチャのどの部分がその頂点に対応するか指定するものです。
[カスタムジオメトリの構築](threejs-custom-geometry.html)を始めるときに説明します。

## <a name="loading"></a> Loading Textures
## <a name="loading"></a> テクスチャの読み込み

### <a name="easy"></a> The Easy Way
### <a name="easy"></a> 簡単な方法

Most of the code on this site uses the easiest method of loading textures.
We create a `TextureLoader` and then call its [`load`](TextureLoader.load) method.
This returns a `Texture` object.
このサイトのコードのほとんどは、もっとも簡単なテクスチャの読み込み方を使っています。
`TextureLoader`を作り、その[`load`](TextureLoader.load)メソッドを呼びます。
これは`Texture`オブジェクトを返します。


```js
const texture = loader.load('resources/images/flower-1.jpg');
```

It's important to note that using this method our texture will be transparent until
the image is loaded asynchronously by three.js at which point it will update the texture
with the downloaded image.
このメソッドを使うと、画像がthree.jsによって非同期的に読み込まれるまで、テクスチャ透明になってしましまいます。読み込まれた時点で、テクスチャをダウンロードした画像に更新します。


This has the big advantage that we don't have to wait for the texture to load and our
page will start rendering immediately. That's probably okay for a great many use cases
but if we want we can ask three.js to tell us when the texture has finished downloading.
この方法では、テクスチャの読み込みを待つ必要がなく、ページをすぐにレンダリングし始めることができるという、大きな利点があります。
多くのケースでこの方法で問題ありませんが、テクスチャをダウンロードし終えたときにthree.jsに通知してもらうこともできます。

### <a name="wait1"></a> Waiting for a texture to load
### <a name="wait1"></a> テクスチャの読み込みを待つ

To wait for a texture to load the `load` method of the texture loader takes a callback
that will be called when the texture has finished loading. Going back to our top example
we can wait for the texture to load before creating our `Mesh` and adding it to scene
like this
テクスチャの読み込みを待つために、テクスチャローダーの`load`メソッドは、テクスチャの読み込みが終了したときに呼ばれるコールバックを取ります。
冒頭の例に戻り、このように、`Mesh`を作りシーンに追加する前に、テクスチャの読み込みを待つことができます。


```js
const loader = new THREE.TextureLoader();
loader.load('resources/images/wall.jpg', (texture) => {
  const material = new THREE.MeshBasicMaterial({
    map: texture,
  });
  const cube = new THREE.Mesh(geometry, material);
  scene.add(cube);
  cubes.push(cube);  // add to our list of cubes to rotate
});
```

Unless you clear your browser's cache and have a slow connection you're unlikely
to see the any difference but rest assured it is waiting for the texture to load.
ブラウザのキャッシュをクリアし、接続が遅くならない限り、違いが分かることはないと思いますが、
テクスチャが読み込まれるのを待っているので、安心してください。

{{{example url="../threejs-textured-cube-wait-for-texture.html" }}}

### <a name="waitmany"></a> Waiting for multiple textures to load
### <a name="waitmany"></a> 複数テクスチャの読み込みを待つ


To wait until all textures have loaded you can use a `LoadingManager`. Create one
and pass it to the `TextureLoader` then set its  [`onLoad`](LoadingManager.onLoad)
property to a callback.
全てのテクスチャが読み込まれたことを待つために、`LoadingManager`を使うことができます。
`TextureLoader`を渡すと、[`onLoad`](LoadingManager.onLoad)属性がコールバックに設定されます。

```js
+const loadManager = new THREE.LoadingManager();
*const loader = new THREE.TextureLoader(loadManager);

const materials = [
  new THREE.MeshBasicMaterial({map: loader.load('resources/images/flower-1.jpg')}),
  new THREE.MeshBasicMaterial({map: loader.load('resources/images/flower-2.jpg')}),
  new THREE.MeshBasicMaterial({map: loader.load('resources/images/flower-3.jpg')}),
  new THREE.MeshBasicMaterial({map: loader.load('resources/images/flower-4.jpg')}),
  new THREE.MeshBasicMaterial({map: loader.load('resources/images/flower-5.jpg')}),
  new THREE.MeshBasicMaterial({map: loader.load('resources/images/flower-6.jpg')}),
];

+loadManager.onLoad = () => {
+  const cube = new THREE.Mesh(geometry, materials);
+  scene.add(cube);
+  cubes.push(cube);  // add to our list of cubes to rotate
+};
```

The `LoadingManager` also has an [`onProgress`](LoadingManager.onProgress) property
we can set to another callback to show a progress indicator.
`LoadingManager`は[`onProgress`](LoadingManager.onProgress)属性もあり、
プログレスインジケーターを表示するためのコールバックを設定できます。

First we'll add a progress bar in HTML
まず、HTMLにプログレスバーを追加しましょう。

```html
<body>
  <canvas id="c"></canvas>
+  <div id="loading">
+    <div class="progress"><div class="progressbar"></div></div>
+  </div>
</body>
```

and the CSS for it
そしてCSSにも追加します。

```css
#loading {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
}
#loading .progress {
    margin: 1.5em;
    border: 1px solid white;
    width: 50vw;
}
#loading .progressbar {
    margin: 2px;
    background: white;
    height: 1em;
    transform-origin: top left;
    transform: scaleX(0);
}
```

Then in the code we'll update the scale of the `progressbar` in our `onProgress` callback. It gets
called with the URL of the last item loaded, the number of items loaded so far, and the total
number of items loaded.
そうすると、コード内で`onProgress`コールバックの`progressbar`のスケールが更新できます。
これは、最後のアイテムが読み込まれるURL、いま読み込まれているアイテムの数、アイテムの合計数と一緒に呼ばれます。

```js
+const loadingElem = document.querySelector('#loading');
+const progressBarElem = loadingElem.querySelector('.progressbar');

loadManager.onLoad = () => {
+  loadingElem.style.display = 'none';
  const cube = new THREE.Mesh(geometry, materials);
  scene.add(cube);
  cubes.push(cube);  // add to our list of cubes to rotate
};

+loadManager.onProgress = (urlOfLastItemLoaded, itemsLoaded, itemsTotal) => {
+  const progress = itemsLoaded / itemsTotal;
+  progressBarElem.style.transform = `scaleX(${progress})`;
+};
```

Unless you clear your cache and have a slow connection you might not see
the loading bar.
キャッシュを削除して低速なコネクションを作らない限りは、プログレスバーを見ることできないかもしれません。

{{{example url="../threejs-textured-cube-wait-for-all-textures.html" }}}

## <a name="cors"></a> Loading textures from other origins
## <a name="cors"></a> 異なるオリジンからのテクスチャの読み込み

To use images from other servers those servers need to send the correct headers.
If they don't you cannot use the images in three.js and will get an error.
If you run the server providing the images make sure it
[sends the correct headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).
If you don't control the server hosting the images and it does not send the
permission headers then you can't use the images from that server.
異なるサーバーの画像を使うため、そのサーバーは正しいヘッダーを送る必要があります。
そうしないと、three.jsでその画像を使うことができず、エラーを受け取るでしょう。
もし皆さんが画像を提供するサーバーを運用しているなら、
[正しいヘッダーを送る](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)を確認してください。
画像をホスティングしているサーバーに手を入れられず、権限用のヘッダーを送ることができないなら、
そのサーバーからの画像を使うことはできません。


For example [imgur](https://imgur.com), [flickr](https://flickr.com), and
[github](https://github.com) all send headers allowing you to use images
hosted on their servers in three.js. Most other websites do not.
例えば、[imgur](https://imgur.com)、[flickr](https://flickr.com)、そして
[github](https://github.com)は全て、ホストしている画像を
three.jsで使うことができるようなヘッダーを送っています。

## <a name="memory"></a> Memory Usage
## <a name="memory"></a>メモリ使用

Textures are often the part of a three.js app that use the most memory. It's important to understand
that *in general*, textures take `width * height * 4 * 1.33` bytes of memory.
多くの場合、テクスチャはthree.jsアプリの中で最もメモリを使っています。
*一般的に*テクスチャは`幅 * 高さ * 4 * 1.33`バイトのメモリを消費していることを理解するのは重要です。

Notice that says nothing about compression. I can make a .jpg image and set its compression super
high. For example let's say I was making a scene of a house. Inside the house there is a table
and I decide to put this wood texture on the top surface of the table
圧縮については言及していないことに注意してください。.jpgイメージを作り、超高圧縮することもできます。
例えば、家のシーンを作っているとしましょう。家の中には、テーブルがあり、上面に木目のテクスチャを置くことに決めました。

<div class="threejs_center"><img class="border" src="resources/images/compressed-but-large-wood-texture.jpg" align="center" style="width: 300px"></div>

That image is only 157k so it will download relatively quickly but [it is actually
3024 x 3761 pixels in size](resources/images/compressed-but-large-wood-texture.jpg).
Following the equation above that's
このイメージはたった157kなので、比較的速くダウンロードすることができます。しかし、
[実際には3024 x 3761ピクセルの大きさ](resources/images/compressed-but-large-wood-texture.jpg)です。
前述した式によると、

    3024 * 3761 * 4 * 1.33 = 60505764.5

That image will take **60 MEG OF MEMORY!** in three.js.
A few textures like that and you'll be out of memory.
となり、three.jsの**60メガのメモリ！**を消費するでしょう。
このようなテクスチャが少しあれば、メモリリークしてしまうでしょう。

I bring this up because it's important to know that using textures has a hidden cost.
In order for three.js to use the texture it has to hand it off to the GPU and the
GPU *in general* requires the texture data to be uncompressed.
この例を持ち出したのは、テクスチャ使用の隠れたコストを知っていることが重要だからです。
three.jsでテクスチャを使うためには、テクスチャのデータをGPUに渡し、*一般的に*非圧縮にしておく必要があります。

The moral of the story is make your textures small in dimensions not just small
in file size. Small in file size = fast to download. Small in dimensions = takes
less memory. How small should you make them?
As small as you can and still look as good as you need them to look.
この話の教訓は、テクスチャをファイルサイズだけでなく、次元も小さくすることです。
ファイルサイズの小ささ = 高速なダウンロードです。次元の小ささ = 省メモリです。
では、どのように小さくできるのでしょうか？

## <a name="format"></a> JPG vs PNG
## <a name="format"></a> JPG vs PNG

This is pretty much the same as regular HTML in that JPGs have lossy compression,
PNGs have lossless compression so PNGs are generally slower to download.
But, PNGs support transparency. PNGs are also probably the appropriate format
for non-image data like normal maps, and other kinds of non-image maps which we'll go over later.
これは通常のHTMLとほぼ同じで、PNGはロスレス圧縮なので、lossy圧縮のJPGよりも
一般的にダウンロードが遅くなります。

It's important to remember that a JPG doesn't use
less memory than a PNG in WebGL. See above.
WebGLにおいて、上で見たように、JPGがPNGよりも省メモリではないことを覚えておいてください。

## <a name="filtering-and-mips"></a> Filtering and Mips
## <a name="filtering-and-mips"></a> フィルタリングとMIP

Let's apply this 16x16 texture
この16x16のテクスチャを

<div class="threejs_center"><img src="resources/images/mip-low-res-enlarged.png" class="nobg" align="center"></div>

To a cube
立方体に適用してみます。

<div class="spread"><div data-diagram="filterCube"></div></div>

Let's draw that cube really small
この立方体をとても小さく描画してみましょう。

<div class="spread"><div data-diagram="filterCubeSmall"></div></div>

Hmmm, I guess that's hard to see. Let's magnify that tiny cube
ふーむ、見えにくいです。小さな立方体を拡大してみましょう。

<div class="spread"><div data-diagram="filterCubeSmallLowRes"></div></div>

How does the GPU know which colors to make each pixel it's drawing for the tiny cube?
What if the cube was so small that it's just 1 or 2 pixels?
GPUは小さな立方体のどのピクセルにどの色を使うか、どうやって知るのでしょうか？
立方体が小さすぎて1、2ピクセルしかないとしたらどうでしょうか？

This is what filtering is about.
フィルタリングとはこういうものです。

If it was Photoshop, Photoshop would average nearly all the pixels together to figure out what color
to make those 1 or 2 pixels. That would be a very slow operation. GPUs solve this issue
using mipmaps.
もしフォトショップなら近くの全てのピクセルを平均して、1、2ピクセルの色を見つけます。
これはとても遅い操作です。GPUはミップマップを使ってこの問題を解決します。

Mips are copies of the texture, each one half as wide and half as tall as the previous
mip where the pixels have been blended to make the next smaller mip. Mips are created
until we get all the way to a 1x1 pixel mip. For the image above all of the mips would
end up being something like this
MIPはテクスチャのコピーで、ピクセルがブレンドされて次の小さいMIPを作られるために、前のMIPの半分の幅と半分の高さになっています。
MIPは1x1ピクセルのMIPが得られるまで作られます。
全てのMIP上の画像はこのようになります。

<div class="threejs_center"><img src="resources/images/mipmap-low-res-enlarged.png" class="nobg" align="center"></div>

Now, when the cube is drawn so small that it's only 1 or 2 pixels large the GPU can choose
to use just the smallest or next to smallest mip level to decide what color to make the
tiny cube.
さて、立方体が1、2ピクセルの小ささに描かれたとき、どんな色にするか決めるため、GPUは最も小さなMIPか次に小さいMIPか選ぶことができます。


In three you can choose what happens both when the texture is drawn
larger than its original size and what happens when it's drawn smaller than its
original size.
three.jsでは、テクスチャが元の大きさより大きく描かれたときと、小さく描かれたときの両方で、起こることを選ぶことができます。


For setting the filter when the texture is drawn larger than its original size
you set [`texture.magFilter`](Texture.magFilter) property to either `THREE.NearestFilter` or
 `THREE.LinearFilter`.  `NearestFilter` means
just pick the closet single pixel from the original texture. With a low
resolution texture this gives you a very pixelated look like Minecraft.
テクスチャが元の大きさより大きく描かれたときのフィルタ設定として、[`texture.magFilter`](Texture.magFilter)属性に`THREE.NearestFilter`か`THREE.LinearFilter`を設定することができます。
`NearestFilter`は元のテクスチャから最も近い1ピクセルを使用するということです。
低解像度のテクスチャでは、マインクラフトのようにピクセル化された見た目になります。

`LinearFilter` means choose the 4 pixels from the texture that are closest
to the where we should be choosing a color from and blend them in the
appropriate proportions relative to how far away the actual point is from
each of the 4 pixels.
`LinearFilter`はテクスチャから、色を決めたいピクセルに最も近い4ピクセルを選び、
実際の点が4つのピクセルからどれだけ離れているかに応じて適切な比率で混ぜ合わせます。

<div class="spread">
  <div>
    <div data-diagram="filterCubeMagNearest" style="height: 250px;"></div>
    <div class="code">Nearest</div>
  </div>
  <div>
    <div data-diagram="filterCubeMagLinear" style="height: 250px;"></div>
    <div class="code">Linear</div>
  </div>
</div>

For setting the filter when the texture is drawn smaller than its original size
you set the [`texture.minFilter`](Texture.minFilter) property to one of 6 values.
元の大きさよりもテクスチャが小さく描画された時のフィルタ設定では、
[`texture.minFilter`](Texture.minFilter)属性を6つの値から一つ設定できます。

* `THREE.NearestFilter`

   same as above, choose the closest pixel in the texture
   上と同様に、テクスチャの最も近いピクセルを選ぶ。

* `THREE.LinearFilter`

   same as above, choose 4 pixels from the texture and blend them
   上と同様に、テクスチャから4ピクセルを選んで混ぜ合わせる。

* `THREE.NearestMipmapNearestFilter`

   choose the appropriate mip then choose one pixel
   適切なMIPを選んでピクセルを一つ選ぶ。

* `THREE.NearestMipmapLinearFilter`

   choose 2 mips, choose one pixel from each, blend the 2 pixels
   2つMIPを選び、それぞれからピクセルを選んで、その2つを混ぜる。

* `THREE.LinearMipmapNearestFilter`

   chose the appropriate mip then choose 4 pixels and blend them
   適切なMIPを選び、4ピクセルを選んで混ぜ合わせる。

*  `THREE.LinearMipmapLinearFilter`

   choose 2 mips, choose 4 pixels from each and blend all 8 into 1 pixel
   2つMIPを選び、それぞれから4ピクセルを選んで、8つ全部を混ぜ合わせて1ピクセルにする。

Here's an example showing all 6 settings
ここで6つ全ての設定の例を見せましょう。

<div class="spread">
  <div data-diagram="filterModes" style="
    height: 450px;
    position: relative;
  ">
    <div style="
      width: 100%;
      height: 100%;
      display: flex;
      align-items: center;
      justify-content: flex-start;
    ">
      <div style="
        background: rgba(255,0,0,.8);
        color: white;
        padding: .5em;
        margin: 1em;
        font-size: small;
        border-radius: .5em;
        line-height: 1.2;
        user-select: none;"
      >click to<br/>change<br/>texture</div>
    </div>
    <div class="filter-caption" style="left: 0.5em; top: 0.5em;">nearest</div>
    <div class="filter-caption" style="width: 100%; text-align: center; top: 0.5em;">linear</div>
    <div class="filter-caption" style="right: 0.5em; text-align: right; top: 0.5em;">nearest<br/>mipmap<br/>nearest</div>
    <div class="filter-caption" style="left: 0.5em; text-align: left; bottom: 0.5em;">nearest<br/>mipmap<br/>linear</div>
    <div class="filter-caption" style="width: 100%; text-align: center; bottom: 0.5em;">linear<br/>mipmap<br/>nearest</div>
    <div class="filter-caption" style="right: 0.5em; text-align: right; bottom: 0.5em;">linear<br/>mipmap<br/>linear</div>
  </div>
</div>

One thing to notice is the top left and top middle using `NearestFilter` and `LinearFilter`
don't use the mips. Because of that they flicker in the distance because the GPU is
picking pixels from the original texture. On the left just one pixel is chosen and
in the middle 4 are chosen and blended but it's not enough come up with a good
representative color. The other 4 strips do better with the bottom right,
`LinearMipmapLinearFilter` being best.
注意することは、左上と中央上は`NearestFilter`を使っていて、`LinearFilter`はMIPを使っていないことです。GPUが元のテクスチャからピクセルを選ぶので、遠くはちらついて見えます。
左側はたった一つのピクセルが選ばれ、中央は4つのピクセルが選ばれて混ぜ合わされます。しかし、
良い色の表現には至っていません。
ほかの4つの中では、右下の`LinearMipmapLinearFilter`が一番良いです。

If you click the picture above it will toggle between the texture we've been using above
and a texture where every mip level is a different color.
上の画像をクリックすると、上で使用しているテクスチャと、MIPごとに色が異なるテクスチャが切り替わります。

<div class="threejs_center">
  <div data-texture-diagram="differentColoredMips"></div>
</div>

This makes it more clear what is happening.
You can see in the top left and top middle the first mip is used all the way
into the distance. The top right and bottom middle you can clearly see where a different mip
is used.
これで、起きていることが分かりやすいでしょう。
左上と中央上は、最初のMIPがずっと遠くまで使われているのが分かります。
右上と中央下は、別のMIPが使われているのがよく分かります。

Switching back to the original texture you can see the bottom right is the smoothest,
highest quality. You might ask why not always use that mode. The most obvious reason
is sometimes you want things to be pixelated for a retro look or some other reason.
The next most common reason is that reading 8 pixels and blending them is slower
than reading 1 pixel and blending. While it's unlikely that a single texture is going
to be the difference between fast and slow as we progress further into these articles
we'll eventually have materials that use 4 or 5 textures all at once. 4 textures * 8
pixels per texture is looking up 32 pixels for ever pixel rendered.
This can be especially important to consider on mobile devices.
元のテクスチャに切り替えると、右下が滑らか、つまり高品質であることが分かります。
なぜ常にこのモードにしないのか聞きたいかもしれません。
最も明らかな理由は、レトロ感を出すために、ピクセル化してほしいとかです。
次の理由は、8ピクセルを読んで混ぜ合わせることは、1ピクセルを読んで混ぜ合わせるよりも遅いことです。
1つのテクスチャの速度では違いが出るように思えないかもしれませんが、
記事が進むにつれて、最終的に4、5のテクスチャを一度に持つマテリアルが出てくるでしょう。

## <a name="uvmanipulation"></a> Repeating, offseting, rotating, wrapping a texture
## <a name="uvmanipulation"></a> テクスチャの繰り返し、オフセット、回転、ラッピング

Textures have settings for repeating, offseting, and rotating a texture.
テクスチャは、繰り返し、オフセット、回転の設定があります。

By default textures in three.js do not repeat. To set whether or not a
texture repeats there are 2 properties, [`wrapS`](Texture.wrapS) for horizontal wrapping
and [`wrapT`](Texture.wrapT) for vertical wrapping.
three.jsのデフォルトのテクスチャは繰り返されません。
テクスチャが繰り返されるかどうかの設定には、2つの属性があります。
水平方向のラッピングに[`wrapS`](Texture.wrapS)と、垂直方向のラッピングに[`wrapT`](Texture.wrapT)です。

They can be set to one of:
以下のどれかが設定されます：

* `THREE.ClampToEdgeWrapping`

   the last pixel on each edge is repeated forever
   それぞれ角の最後のピクセルが永遠に繰り返されます。

* `THREE.RepeatWrapping`

   the texture is repeated
   テクスチャが繰り返されます。

* `THREE.MirroredRepeatWrapping`

   the texture is mirrored and repeated
   テクスチャの鏡像が取られ、繰り返されます。

For example to turn on wrapping in both directions:
例えば、両方向にラッピングすると、

```js
someTexture.wrapS = THREE.RepeatWrapping;
someTexture.wrapT = THREE.RepeatWrapping;
```

Repeating is set with the [repeat] repeat property.
繰り返しは`repeat`属性で設定されます。

```js
const timesToRepeatHorizontally = 4;
const timesToRepeatVertically = 2;
someTexture.repeat.set(timesToRepeatHorizontally, timesToRepeatVertically);
```

Offseting the texture can be done by setting the `offset` property. Textures
are offset with units where 1 unit = 1 texture size. On other words 0 = no offset
and 1 = offset one full texture amount.
テクスチャのオフセットは`offset`属性でできます。
テクスチャは1単位 = 1テクスチャの大きさにオフセットされます。
言い換えると、0 = オフセットなし、1 = テクスチャ全体の大きさということです。

```js
const xOffset = .5;   // offset by half the texture
const yOffset = .25;  // offset by 1/4 the texture
someTexture.offset.set(xOffset, yOffset);
```

Rotating the texture can be set by setting the `rotation` property in radians
as well as the `center` property for choosing the center of rotation.
It defaults to 0,0 which rotates from the bottom left corner. Like offset
these units are in texture size so setting them to `.5, .5` would rotate
around the center of the texture.
テクスチャの回転は、`rotation`属性で、ラジアンで指定します。
同様に `center`属性で回転の中心を指定します。
デフォルトは0,0で、左下の角で回転します。
オフセットと同じように、単位はテクスチャの大きさなので、`.5, .5`に設定すると、
テクスチャの中心での回転になります。

```js
someTexture.center.set(.5, .5);
someTexture.rotation = THREE.MathUtils.degToRad(45);
```

Let's modify the top sample above to play with these values
最初に取り上げたサンプルでこれらの値を試してみましょう。

First we'll keep a reference to the texture so we can manipulate it
最初に、テクスチャを操作できるように参照を保持しておきます。

```js
+const texture = loader.load('resources/images/wall.jpg');
const material = new THREE.MeshBasicMaterial({
-  map: loader.load('resources/images/wall.jpg');
+  map: texture,
});
```

Then we'll use [dat.GUI](https://github.com/dataarts/dat.gui) again to provide a simple interface.
ここでも、簡単なインターフェースを提供するために[dat.GUI](https://github.com/dataarts/dat.gui)を使います。

```js
import {GUI} from '../3rdparty/dat.gui.module.js';
```

As we did in previous dat.GUI examples we'll use a simple class to
give dat.GUI an object that it can manipulate in degrees
but that will set a property in radians.
以前のdat.GUIの例でしたように、dat.GUIに度数で操作できるオブジェクトを与え、
ラジアン単位でプロパティを設定する簡単なクラスを使います。

```js
class DegRadHelper {
  constructor(obj, prop) {
    this.obj = obj;
    this.prop = prop;
  }
  get value() {
    return THREE.MathUtils.radToDeg(this.obj[this.prop]);
  }
  set value(v) {
    this.obj[this.prop] = THREE.MathUtils.degToRad(v);
  }
}
```

We also need a class that will convert from a string like `"123"` into
a number like `123` since three.js requires numbers for enum settings
like `wrapS` and `wrapT` but dat.GUI only uses strings for enums.
`"123"`といった文字列から`123`といった数値に変換するクラスも必要です。
これは、three.jsは`wrapS`や`wrapT`のようなenumの設定として数値が必要ですが、
dat.GUIはenumに文字列のみを使うためです。

```js
class StringToNumberHelper {
  constructor(obj, prop) {
    this.obj = obj;
    this.prop = prop;
  }
  get value() {
    return this.obj[this.prop];
  }
  set value(v) {
    this.obj[this.prop] = parseFloat(v);
  }
}
```

Using those classes we can setup a simple GUI for the settings above
このクラスを使って、上記設定のための簡単なGUIをセットアップできます。

```js
const wrapModes = {
  'ClampToEdgeWrapping': THREE.ClampToEdgeWrapping,
  'RepeatWrapping': THREE.RepeatWrapping,
  'MirroredRepeatWrapping': THREE.MirroredRepeatWrapping,
};

function updateTexture() {
  texture.needsUpdate = true;
}

const gui = new GUI();
gui.add(new StringToNumberHelper(texture, 'wrapS'), 'value', wrapModes)
  .name('texture.wrapS')
  .onChange(updateTexture);
gui.add(new StringToNumberHelper(texture, 'wrapT'), 'value', wrapModes)
  .name('texture.wrapT')
  .onChange(updateTexture);
gui.add(texture.repeat, 'x', 0, 5, .01).name('texture.repeat.x');
gui.add(texture.repeat, 'y', 0, 5, .01).name('texture.repeat.y');
gui.add(texture.offset, 'x', -2, 2, .01).name('texture.offset.x');
gui.add(texture.offset, 'y', -2, 2, .01).name('texture.offset.y');
gui.add(texture.center, 'x', -.5, 1.5, .01).name('texture.center.x');
gui.add(texture.center, 'y', -.5, 1.5, .01).name('texture.center.y');
gui.add(new DegRadHelper(texture, 'rotation'), 'value', -360, 360)
  .name('texture.rotation');
```

The last thing to note about the example is that if you change `wrapS` or
`wrapT` on the texture you must also set [`texture.needsUpdate`](Texture.needsUpdate)
so three.js knows to apply those settings. The other settings are automatically applied.
最後に特記することは、もしテクスチャの`wrapS`や`wrapT`を変えるなら、
three.jsが設定の適用を知るために、[`texture.needsUpdate`](Texture.needsUpdate)も設定しなければならないことです。




{{{example url="../threejs-textured-cube-adjust.html" }}}

This is only one step into the topic of textures. At some point we'll go over
texture coordinates as well as 9 other types of textures that can be applied
to materials.
これはテクスチャのトピックへの第一歩にすぎません。
ある時点で、テクスチャの座標や、マテリアルが適用できる別の9種のテクスチャについても説明します。

For now let's move on to [lights](threejs-lights.html).
今のところは、[光源](threejs-lights.html)に進みましょう。

<!--
alpha
ao
env
light
specular
bumpmap ?
normalmap ?
metalness
roughness
-->

<link rel="stylesheet" href="resources/threejs-textures.css">
<script type="module" src="resources/threejs-textures.js"></script>
