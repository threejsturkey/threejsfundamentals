Title: Three.js Temelleri
Description: Temeller ile birlikte ilk Three.js dersiniz
TOC: Temel Bilgiler

Bu three.js ile alakalı makale serisindeki ilk makaledir. [Three.js](https://threejs.org) bir web sayfasına 3D içerik sağlamayı mümkün olduğunca kolaylaştırmaya çalışan bir 3D kütüphanedir.

Three.js her zaman olmasa da öncesine göre daha sıklıkla WebGl ile karıştırılır, three.js 3D çizim için WebGL kullanır.
[WebGL yalnızca noktalar, çizgiler ve üçgenler çizen çok düşük seviyeli bir sistemdir](https://webglfundamentals.org).
WebGL ile herhangi yararlı bir şey yapmak birazcık kod gerektirir ve three.js burada devreye girer.
Sahneler, ışıklar, gölgeler, malzemeler, dokular, 3d matematik gibi şeyleri halleder, eğer direkt olarak WebGL kullansaydınız bunların hepsini kendiniz yazmak zorunda kalırdınız.

Bu dersler halihazırda Javascript bildiğinizi varsayar ve çoğu bölümünde ES6 stilini kullanacaklar. [Halihazırda bilmenizin beklendiği şeylerin kısa listesine buradan bakabilirsiniz](threejs-prerequisites.html).
Three.js'i destekleyen çoğu tarayıcı otomatik olarak desteklendiğinden çoğu kullanıcı bu kodu çalıştırabilir. Eğer bu kodu gerçekten eski tarayıcılarda çalıştırmak istiyorsanız [Babel](https://babeljs.io) gibi bir aktarıcıya bakın.
Elbette gerçekten eski tarayıcılarda çalışan kullanıcıların makineleri muhtemelen three.js'i çalıştırmayacaktır.

Çoğu programlama dilini öğrenirken insanların ilk yaptığı şey
bilgisayara `"Hello World!"` yazısını yazdırmaktır.
3D için ise en yaygın olarak yapılan ilk şeylerden biri 3D bir küp yapmaktır.
Öyleyse gelin "Hello Cube!" ile başlayalım!

Başlamadan önce three.js uygulamasının mimarisi hakkında fikir vermeye çalışalım. Bir three.js uygulaması birçok objelerin oluşturulmasına ve onların birbirine bağlanmasına ihtiyaç duyar. Burada küçük bir three.js uygulamasını temsil eden bir diyagram var.

<div class="threejs_center"><img src="resources/images/threejs-structure.svg" style="width: 768px;"></div>

Yukarıdaki şema hakkında dikkat edilmesi gerekenler.

- Burada bir `Renderer` var. Bu tartışmalı bir şekilde three.js'nin ana nesnesidir. `Renderer`'a bir `Scene` ve bir `Camera` iletilir ve 3 boyutlu sahnenin kameranın kesik kısmı içindeki kısmını bir 2 boyutlu görüntü olarak tuvale işler (çizer).

- Bir `Scene` nesnesi, `Mesh` nesneleri, `Light` nesneleri, `Group`, `Object3D` ve `Camera` objeleri gibi çeşitli nesnelerden oluşan, ağaç benzeri bir yapı gibi bir [sahne grafiği](threejs-scenegraph.html) vardır. Bir `Scene` objesi arkaplan rengi ve sis gibi özellikler içerir ve sahne grafiğinin kökünü tanımlar. Bu nesneler hiyerarşik bir üst/alt ağaç benzeri yapı tanımlar ve nesnelerin nerede göründüğünü ve nasıl yönlendirildiklerini temsil eder. Altlar (çocuklar) üstlerine (ebeveynlerine) göre konumlandırılır ve yönlendirilir. Örneğin; arabanın tekerleri arabanın çocuğu olabilir, böylece araba nesnesini hareket ettirmek ve yönlendirmek, tekerlekleri otomatik olarak hareket ettirir. Daha fazlası için sahne grafiği ile alakalı [buradaki](threejs-scenegraph.html) makaleyi okuyabilirsiniz.

  Şemadaki notta `Camera` sahne grafiğinin yarı yarıya dışarısındadır. Bu three.js'te, diğer nesnelerin aksine, bir `Camera`'nın çalışması için sahne grafiğinde olması gerekmez. Aynı diğer objeler gibi, bir `Camera`, başka bir nesnenin çocuğu olarak, ebeveyn nesnesine hareket edecek ve yönlendirilecektir. [Burada](threejs-scenegraph.html) sahne grafikleri ile ilgili makalenin sonunda bir sahne grafiğine birden fazla `Camera` nesnesi yerleştirme örneği vardır.

- `Mesh` nesneleri spesifik bir `Geometry`'i spesifik bir `Material` ile çizmeyi temsil eder. Hem `Material` hem de `Geometry` objeleri birden çok `Mesh` nesneleri tarafından kullanılabilir. Örneğin; farklı konumlardaki iki mavi kübü çizmek için her iki kübün konum ve yönünü temsil eden iki `Mesh` nesnesine ihtiyacımız vardır. Küpün tepe noktası verilerini tutmak için yalnızca bir `Geometry`'e ve mavi rengi belirmek için sadece bir `Material`'a ihtiyacımız olacaktı. Her iki `Mesh` nesnesi aynı `Geometry` nesnesi ve aynı `Material` nesnesini referans eder.

- `Geometry` nesneleri küre, küp, uçak, köpek, kedi, insan, ağaç, bina gibi bazı geometri parçalarının tepe noktası verilerini temsil eder. Three.js birçok türde yerleşik [geometri ilkelleri](threejs-primitives.html) sağlar. Ayrıca [dosyalardan geometri yükleyebileceğiniz](threejs-load-obj.html) gibi [özel geometri](threejs-custom-buffergeometry.html) de oluşturabilirsiniz.

- `Material` nesneleri ne kadar parlak olacağını belirleyen renk gibi şeyleri içeren [geometri çizerken kullanılan yüzey özellikleri](threejs-materials.html)ni temsil eder. Bir `Material` ayrıca bir ya da birden fazla kullanılan `Texture` nesnesini referans edebilir, örneğin; bir görüntüyü bir geometrinin yüzeyine sarmak için.

- `Texture` nesneleri genellikle ya [resim dosyalarından yüklenen resimleri](threejs-textures.html), [tuvalden oluşturulan](threejs-canvas-textures.html) ya da [başka bir sahneden işlenen](threejs-rendertargets.html) resimleri temsil eder.

- `Light` nesneleri [farklı ışık türleri](threejs-lights.html)ni temsil eder.

Tüm bunları göz önüne alarak en küçük _"Hello Cube"_ kurulumunu yapacağız ki tam olarak şuna benziyor:

<div class="threejs_center"><img src="resources/images/threejs-1cube-no-light-scene.svg" style="width: 500px;"></div>

İlk olarak three.js'i yükleyelim!

```html
<script type="module">
  import * as THREE from "./resources/threejs/r125/build/three.module.js";
</script>
```

Script etiketine `type="module"`'ü koymanız önemlidir. Bu bize three.js'i yüklerken `import`'u kullanabilmemizi sağlar. Three.js'i yüklemek için başka yollar da vardır ama r106'dan itibaren modülleri kullanmak önerilen yoldur. Modüller ihtiyacı olan diğer modülleri kolayca içeri aktarmayı sağlayan bir avantaja sahiptir. Bu bizi fazladan manuel olarak bağımlı olan ekstra script dosyalarını yüklemekten kurtarıyor.

Sırada bir `<canvas>` etiketine ihtiyacımız var, yani...

```html
<body>
  <canvas id="c"></canvas>
</body>
```

Three.js'den bu tuvali çizmesini isteyeceğiz, bu yüzden ona bir göz atmamız gerekiyor.

```html
<script type="module">
  import * as THREE from './resources/threejs/r125/build/three.module.js';

  +function main() {
  +  const canvas = document.querySelector('#c');
  +  const renderer = new THREE.WebGLRenderer({canvas});
  +  ...
</script>
```

Tuvale göz attıktan sonra bir `WebGLRenderer` oluşturuyoruz. Oluşturucu tuvalin oluşturulmasında ve sağladığınız tüm verilerin alınmasında asıl sorumlu olan şeydir. Geçmişte `CSSRenderer`, `CanvasRenderer` gibi diğer oluşturucular vardı ve gelecekte belki de `WebGL2Renderer` ya da `WebGPURenderer` gibileri olabilir. Şimdilik tuvalde 3 boyutlu oluşturmak için WebGL'i kullanan `WebGLRenderer` var.

Burada bazı ezoterik (belirli bir insan topluluğunun dışında kimseye bildirilmeyen, yalnızca sınırlı, dar bir çevreye aktarılan her türlü bilgi) ayrıntılar olduğuna dikkat edin. Eğer bir tuvali three.js'e geçirmezseniz sizin için bir tane oluşturur fakat sonrasında belgenize eklemeniz gerekir. Nereye ekleyeceğiniz kullanım durumunuza bağlı olarak değişebilir ve kodunuzu değiştirmeniz gerekecektir, böylece bir tuval geçirirken three.js daha esnek hissettiriyor.

Next up we need a camera. We'll create a `PerspectiveCamera`.

Sırada bir kameraya ihtiyacımız var. Bir `PerspectiveCamera` oluşturacağız.

```js
const fov = 75;
const aspect = 2; // the canvas default
const near = 0.1;
const far = 5;
const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
```

`fov` `field of view` (görüş alanı)'nın kısaltmasıdır. Bu örnekte dikeyde 75 derece boyutta. Three.js'deki açıların çoğunun radyan cinsinden olduğuna dikkat edin, fakat bazı sebeplerden ötürü perspektif kamera derece alır.

`aspect` tuvalin görüntü yönüdür. Bu konunun detaylarını [başka bir makalede](threejs-responsive.html) irdeleyeceğiz ama varsayılan olarak bir tuval 300x150 pikseldir bu da görünümü 300/150 ya da 2 yapar.

`near` ve `far` oluşturulacak görüntüde kameranın önündeki boşluğu temsil eder. Bu aralıktan önce veya sonra herhangi bir şey kırpılır (çizilmez).

Buradaki dört ayar bir _"frustum"_ tanımlar. Bir _frustum_ ucu kesilmiş bir piramit gibi bir 3 boyutlu şeklin adıdır. Diğer bir deyişle "frustum" kelimesi küre, küp, prizma, frustum gibi başka bir 3 boyutlu şeklin yerine kullanılabilir.

<img src="resources/frustum-3d.svg" width="500" class="threejs_center"/>

Yakın ve uzak düzlemlerin yüksekliği görüş alanı tarafından belirlenir.
Her iki yüzeylerin genişliği görüş alanı ve yönüne göre belirlenir.

Tanımlanan frustum içindeki herhangi bir şey çizilecektir. Dışarıdaki herhangi bir şey ise çizilmeyecektir.

Kamera varsayılan olarak +Y yukarı ile -Z aşağı eksenine bakar. Kübümüzü orjine koyacağız, bu yüzden bir şey görebilmek için kamerayı başlangıç noktasından biraz geriye çekmemiz gerekiyor.

```js
camera.position.z = 2;
```

İşte hedeflediğimiz şey.


<img src="resources/scene-down.svg" width="500" class="threejs_center"/>

Yukarıdaki şemada kameramızın `z = 2`'de olduğunu görebiliriz. -Z eksenine bakıyoruz. Frustumumuz kameranın önünde 0.1 birim başlıyor ve kamera önünde 5 birim gidiyor. Çünkü bu şemada aşağı bakıyoruz, görüş alanı açıdan etkilenir. Tuvalimiz boyundan iki kat daha geniştir, bu nedenle tuval boyunca görüş alanı, dikey görüş alanı olarak söylediğimiz 75 dereceden çok daha geniş olacaktır.

Next we make a `Scene`. A `Scene` in three.js is the root of a form of scene graph.
Anything you want three.js to draw needs to be added to the scene. We'll
cover more details of [how scenes work in a future article](threejs-scenegraph.html).

Sırada bir `Scene` yapalım. Bir `Scene` Three.js'de sahne grafiği formundaki köklerden biridir. Three.js'in çizmesini istediğiniz herhangi bir şey sahneye eklenmelidir. Bu konunun detaylarını [sahnelerin nasıl çalıştığı ile alakalı ileriki makale](threejs-scenegraph.html)lerden birinde göreceğiz.

```js
const scene = new THREE.Scene();
```

Next up we create a `BoxGeometry` which contains the data for a box.
Almost anything we want to display in Three.js needs geometry which defines
the vertices that make up our 3D object.

```js
const boxWidth = 1;
const boxHeight = 1;
const boxDepth = 1;
const geometry = new THREE.BoxGeometry(boxWidth, boxHeight, boxDepth);
```

We then create a basic material and set its color. Colors can
be specified using standard CSS style 6 digit hex color values.

```js
const material = new THREE.MeshBasicMaterial({ color: 0x44aa88 });
```

We then create a `Mesh`. A `Mesh` in three represents the combination
of a three things

1. A `Geometry` (the shape of the object)
2. A `Material` (how to draw the object, shiny or flat, what color, what texture(s) to apply. Etc.)
3. The position, orientation, and scale of that object in the scene relative to its parent. In the code below that parent is the scene.

```js
const cube = new THREE.Mesh(geometry, material);
```

And finally we add that mesh to the scene

```js
scene.add(cube);
```

We can then render the scene by calling the renderer's render function
and passing it the scene and the camera

```js
renderer.render(scene, camera);
```

Here's a working example

{{{example url="../threejs-fundamentals.html" }}}

It's kind of hard to tell that is a 3D cube since we're viewing
it directly down the -Z axis and the cube itself is axis aligned
so we're only seeing a single face.

Let's animate it spinning and hopefully that will make
it clear it's being drawn in 3D. To animate it we'll render inside a render loop using
[`requestAnimationFrame`](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame).

Here's our loop

```js
function render(time) {
  time *= 0.001; // convert time to seconds

  cube.rotation.x = time;
  cube.rotation.y = time;

  renderer.render(scene, camera);

  requestAnimationFrame(render);
}
requestAnimationFrame(render);
```

`requestAnimationFrame` is a request to the browser that you want to animate something.
You pass it a function to be called. In our case that function is `render`. The browser
will call your function and if you update anything related to the display of the
page the browser will re-render the page. In our case we are calling three's
`renderer.render` function which will draw our scene.

`requestAnimationFrame` passes the time since the page loaded to
our function. That time is passed in milliseconds. I find it's much
easier to work with seconds so here we're converting that to seconds.

We then set the cube's X and Y rotation to the current time. These rotations
are in [radians](https://en.wikipedia.org/wiki/Radian). There are 2 pi radians
in a circle so our cube should turn around once on each axis in about 6.28
seconds.

We then render the scene and request another animation frame to continue
our loop.

Outside the loop we call `requestAnimationFrame` one time to start the loop.

{{{example url="../threejs-fundamentals-with-animation.html" }}}

It's a little better but it's still hard to see the 3d. What would help is to
add some lighting so let's add a light. There are many kinds of lights in
three.js which we'll go over in [a future article](threejs-lights.html). For now let's create a directional light.

```js
{
  const color = 0xffffff;
  const intensity = 1;
  const light = new THREE.DirectionalLight(color, intensity);
  light.position.set(-1, 2, 4);
  scene.add(light);
}
```

Directional lights have a position and a target. Both default to 0, 0, 0. In our
case we're setting the light's position to -1, 2, 4 so it's slightly on the left,
above, and behind our camera. The target is still 0, 0, 0 so it will shine
toward the origin.

We also need to change the material. The `MeshBasicMaterial` is not affected by
lights. Let's change it to a `MeshPhongMaterial` which is affected by lights.

```js
-const material = new THREE.MeshBasicMaterial({color: 0x44aa88});  // greenish blue
+const material = new THREE.MeshPhongMaterial({color: 0x44aa88});  // greenish blue
```

Here is our new program structure

<div class="threejs_center"><img src="resources/images/threejs-1cube-with-directionallight.svg" style="width: 500px;"></div>

And here it is working.

{{{example url="../threejs-fundamentals-with-light.html" }}}

It should now be pretty clearly 3D.

Just for the fun of it let's add 2 more cubes.

We'll use the same geometry for each cube but make a different
material so each cube can be a different color.

First we'll make a function that creates a new material
with the specified color. Then it creates a mesh using
the specified geometry and adds it to the scene and
sets its X position.

```js
function makeInstance(geometry, color, x) {
  const material = new THREE.MeshPhongMaterial({ color });

  const cube = new THREE.Mesh(geometry, material);
  scene.add(cube);

  cube.position.x = x;

  return cube;
}
```

Then we'll call it 3 times with 3 different colors and X positions
saving the `Mesh` instances in an array.

```js
const cubes = [
  makeInstance(geometry, 0x44aa88, 0),
  makeInstance(geometry, 0x8844aa, -2),
  makeInstance(geometry, 0xaa8844, 2),
];
```

Finally we'll spin all 3 cubes in our render function. We
compute a slightly different rotation for each one.

```js
function render(time) {
  time *= 0.001;  // convert time to seconds

  cubes.forEach((cube, ndx) => {
    const speed = 1 + ndx * .1;
    const rot = time * speed;
    cube.rotation.x = rot;
    cube.rotation.y = rot;
  });

  ...
```

and here's that.

{{{example url="../threejs-fundamentals-3-cubes.html" }}}

If you compare it to the top down diagram above you can see
it matches our expectations. With cubes at X = -2 and X = +2
they are partially outside our frustum. They are also
somewhat exaggeratedly warped since the field of view
across the canvas is so extreme.

Our program now has this structure

<div class="threejs_center"><img src="resources/images/threejs-3cubes-scene.svg" style="width: 610px;"></div>

As you can see we have 3 `Mesh` objects each referencing the same `BoxGeometry`.
Each `Mesh` references a unique `MeshPhongMaterial` so that each cube can have
a different color.

I hope this short intro helps to get things started. [Next up we'll cover
making our code responsive so it is adaptable to multiple situations](threejs-responsive.html).

<div id="es6" class="threejs_bottombar">
<h3>es6 modules, three.js, and folder structure</h3>
<p>As of version r106 the preferred way to use three.js is via <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import">es6 modules</a>.</p>
<p>
es6 modules can be loaded via the <code>import</code> keyword in a script
or inline via a <code>&lt;script type="module"&gt;</code> tag. Here's an example of
both
</p>
<pre class=prettyprint>
&lt;script type="module"&gt;
import * as THREE from './resources/threejs/r125/build/three.module.js';

...

&lt;/script&gt;

</pre>
<p>
Paths must be absolute or relative. Relative paths always start with <code>./</code> or <code>../</code>
which is different than other tags like <code>&lt;img&gt;</code> and <code>&lt;a&gt;</code>.
</p>
<p>
References to the same script will only be loaded once as long as their absolute paths
are exactly the same. For three.js this means it's required that you put all the examples
libraries in the correct folder structure
</p>
<pre class="dos">
someFolder
 |
 ├-build
 | |
 | +-three.module.js
 |
 +-examples
   |
   +-jsm
     |
     +-controls
     | |
     | +-OrbitControls.js
     | +-TrackballControls.js
     | +-...
     |
     +-loaders
     | |
     | +-GLTFLoader.js
     | +-...
     |
     ...
</pre>
<p>
The reason this folder structure is required is because the scripts in the
examples like <a href="https://github.com/mrdoob/three.js/blob/master/examples/jsm/controls/OrbitControls.js"><code>OrbitControls.js</code></a>
have hard coded relative paths like
</p>
<pre class="prettyprint">
import * as THREE from '../../../build/three.module.js';
</pre>
<p>
Using the same structure assures then when you import both three and one of the example
libraries they'll both reference the same <code>three.module.js</code> file.
</p>
<pre class="prettyprint">
import * as THREE from './someFolder/build/three.module.js';
import {OrbitControls} from './someFolder/examples/jsm/controls/OrbitControls.js';
</pre>
<p>This includes when using a CDN. Be sure your path to <code>three.module.js</code> ends with
<code>/build/three.modules.js</code>. For example</p>
<pre class="prettyprint">
import * as THREE from 'https://unpkg.com/three@0.108.0<b>/build/three.module.js</b>';
import {OrbitControls} from 'https://unpkg.com/three@0.108.0/examples/jsm/controls/OrbitControls.js';
</pre>
<p>If you'd prefer the old <code>&lt;script src="path/to/three.js"&gt;&lt;/script&gt;</code> style
you can check out <a href="https://r105.threejsfundamentals.org">an older version of this site</a>.
Three.js has a policy of not worrying about backward compatibility. They expect you to use a specific
version, as in you're expected to download the code and put it in your project. When upgrading to a newer version
you can read the <a href="https://github.com/mrdoob/three.js/wiki/Migration-Guide">migration guide</a> to
see what you need to change. It would be too much work to maintain both an es6 module and a class script
version of this site so going forward this site will only show es6 module style. As stated elsewhere,
to support legacy browsers look into a <a href="https://babeljs.io">transpiler</a>.</p>
</div>

<!-- needed in English only to prevent warning from outdated translations -->

<a href="threejs-geometry.html"></a>
<a href="Geometry"></a>
