Title: Three.js Responsive Tasarım
Description: Three.js'inizi farklı boyuttaki ekranlara sığdırmak.
TOC: Responsive Tasarım

Bu, three.js makaleleri serisinin ikinci makalesidir. İlk makale [temel bilgiler](three.js-fundamentals.html) hakkındaydı. Eğer o makaleyi henüz okumadıysanız önce oradan başlamak isteyebilirsiniz.

Bu makale, three.js uygulamanızı her türlü durumda nasıl responsive yapacağınız hakkındadır. Bir web sayfasını responsive yapmak demek genellikle o sayfanın masaüstü bilgisayardan tabletlere ve telefonlara kadar farklı boyuttaki ekranlarda güzel görünmesini sağlamak anlamına gelir.

Hatta three.js için dikkate alınması gereken daha fazla durum vardır. Örneğin; solda, sağda, yukarıda veya aşağıda kontrolleri olan bir 3D editörü bizim ilgilenmemiz gereken bir şeydir. Dökümanın ortasında bulunan bir canlı diyagram (live diagram) bir başka örnektir.

Kullanmış olduğumuz son örnek, CSS'i ve boyutu olmayan düz bir canvastı

```html
<canvas id="c"></canvas>
```

Bu canvas varsayılan olarak 300x150 piksel boyutlarındadır.

Web platformunda bir şeylerin boyutlarını değiştirmekte tercih edilen yol CSS kullanmaktır.

Şimdi CSS ekleyerek canvasın sayfayı doldurmasını sağlayalım

```html
<style>
html, body {
   margin: 0;
   height: 100%;
}
#c {
   width: 100%;
   height: 100%;
   display: block;
}
</style>
```

HTML'de body'nin varsayılan olarak 5 piksellik bir margini vardır ve margin değerini 0 yaparak bu margin kaldırılmış olur. Html ve body'nin height değeri 100% yapılarak tüm pencereyi kaplaması sağlanır. Aksi halde sadece sahip oldukları içerik kadar büyük olabilirler.

Ardından `id=c` özelliğine sahip olan elementin boyutlarını da 100% yaparak kendisini kapsayan container elementinin yani body elementinin boyutlarına ulaştırmış oluyoruz.

Son olarak `display` özelliğini `block` olarak ayarlıyoruz. Bir canvasın varsayılan display özelliği `inline`dır. Inline elementler görüntülenen şeye whitespace ekleyerek sonlanabilir. Canvasın display'ini `block` olarak seçince bu sorun ortadan kalkar.

İşte sonuç

{{{example url="../threejs-responsive-no-resize.html" }}}

Şimdi canvasın sayfayı doldurduğunu görebiliyorsunuz ancak iki problem var. Birincisi küplerimiz esnedi. Küpten çok kutulara benzediler. Fazla uzun veya fazla geniş oldular. Örnek resmi yeni bir pencerede açın ve sayfa boyutu ile oynayın. Küplerin nasıl genişlediğini ve uzadığını göreceksiniz.

<img src="resources/images/resize-incorrect-aspect.png" width="407" class="threejs_center nobg">

İkinci problem ise küplerin düşük çözünürlüklü veya pikselleşmiş ve bulanık görünmesi. Pencereyi iyice genişletin ve sorunu siz de görün.

<img src="resources/images/resize-low-res.png" class="threejs_center nobg">

İlk önce esneme problemini çözelim. Bunu yapmak için kameranın aspect'ini canvasın display boyutlarına ayarlamamız gerekiyor. Bunu canvasın `clientWidth` ve `clientHeight` özelliklerine bakarak yapabiliriz.

Render loopumuzu şu şekilde güncelliyoruz

```js
function render(time) {
  time *= 0.001;

+  const canvas = renderer.domElement;
+  camera.aspect = canvas.clientWidth / canvas.clientHeight;
+  camera.updateProjectionMatrix();

  ...
```

Şimdi küplerin bozulması durmuş olmalı.

{{{example url="../threejs-responsive-update-camera.html" }}}

Örneği yeni pencerede açın ve ekranı yeniden boyutlandırın. Böylece küplerin artık genişleyip uzamadığını göreceksiniz. Pencere boyutundan bağımsız olarak doğru aspect'te kalmaktalar.

<img src="resources/images/resize-correct-aspect.png" width="407" class="threejs_center nobg">

Şimdi pikselleşmeyi düzeltelim.

Canvas elementlerinin iki farklı boyutu vardır. Birisi canvasın sayfada görünen boyutudur. CSS ile ayarlamış olduğumuz budur. Diğer ebat ise canvasın kendisinde bulunan piksel sayısıdır. Bu herhangi bir resim dosyasından farklı değildir. Örneğin 128x64 piksellik bir resim dosyamız olabilir ve CSS kullanarak onu 400x200 piksel olarak gösterebiliriz.

```html
<img src="some128x64image.jpg" style="width:400px; height:200px">
```

Bir canvasın iç boyutları, onun çözünürlüğü sıklıkla onun drawingbuffer boyutu olarak adlandırılır. Three.js'de canvasın drawingbuffer ölçülerini `renderer.setSize` ile ayarlayabiliriz. Hangi boyutları seçmeliyiz? En bariz cevap "canvasın görüntülendiği ölçülerle aynı ölçüler"dir.
Tekrar söylemek gerekirse, bunu yapmak için canvas'ın `clientWidth` ve  `clientHeight` özelliklerine bakabiliriz.

Renderer'ın canvasının halihazırda görüntülen boyutlarda olup olmadığını kontrol eden ve eğer o boyutlarda değilse olacak şekilde yeniden boyutlandıran bir fonksiyon yazalım.

```js
function resizeRendererToDisplaySize(renderer) {
  const canvas = renderer.domElement;
  const width = canvas.clientWidth;
  const height = canvas.clientHeight;
  const needResize = canvas.width !== width || canvas.height !== height;
  if (needResize) {
    renderer.setSize(width, height, false);
  }
  return needResize;
}
```

Canvasın gerçekten yeniden boyutlandırmaya ihtiyacı olup olmadığını kontrol ettiğimize dikkat edin. Canvası yeninden boyutlandırmak, canvas spesifikasyonunun ilginç bir parçasıdır ve eğer zaten istediğimiz ölçülerdeyse, aynı ölçülere ayarlamamak en iyisidir.

Yeniden boyutlandırmaya ihtiyacımız olup olmadığını öğrendikten sonra artık `renderer.setSize`ı çağırabilir ve yeni width and height değerlerini verebiliriz. Sonda `false` argümanını kullanmak önemlidir. `renderer.setSize` varsayılan olarak canvasın CSS ölçülerini set eder ki bu bizim istediğimiz şey değil. Biz tarayıcının, görüntülenme boyutlarını belirleyen CSS kullanan tüm diğer elementler için nasıl çalışıyorsa öyle çalışmaya devam etmesini istiyoruz. Biz three tarafından kullanılan canvasların diğer elementlerden farklı olmasını istemiyoruz.

Eğer canvas yeniden boyutlandırılırsa fonksiyonumuzun true değeri döndürdüğüne dikkat edin. Biz bunu güncellememiz gerekn başka şeylerin olup olmadığını kontrol etmek amacıyla kullanabiliriz. Render loop'umuzu yeni fonksiyonu kullanacak şekilde modifiye edelim

```js
function render(time) {
  time *= 0.001;

+  if (resizeRendererToDisplaySize(renderer)) {
+    const canvas = renderer.domElement;
+    camera.aspect = canvas.clientWidth / canvas.clientHeight;
+    camera.updateProjectionMatrix();
+  }

  ...
```

Aspect sadece canvasın display ölçüleri değiştinde değişeceği için, camera'nın aspect'ini sadece eğer `resizeRendererToDisplaySize` `true` değeri döndürürse ayarlarız.

{{{example url="../threejs-responsive.html" }}}

Artık canvasın görüntülenme ebatlarıyla eşleşen bir çözünürlükte renderlanıyor olmalı.

Yeniden boyutlandırmayı CSS'in halletmesine izin vermek için kodlarımızı alalım ve [ayrı bir .js dosyası](../threejs-responsive.js)na koyalım.
İşte şimdi, ebatları CSS'in seçtiği ve çalışması için bizim hiç kod değiştirmek zorunda olmadığımız birkaç örnek daha.

Küplerimizi bir metin paragrafının ortasına koyalım.

{{{example url="../threejs-responsive-paragraph.html" startPane="html" }}}

ve burada kodumuzun aynısı, sağdaki kontrol alanının yeniden boyutlandırılabilir olduğu editör stili düzeninde kullanılıyor.

{{{example url="../threejs-responsive-editor.html" startPane="html" }}}

Dikkat edilmesi gereken nokta kod değişmedi. Sadece HTML ve CSS değişti.

## HD-DPI ekranları kullanma

HD-DPI, high-density dot per inch displays'in kısaltmasıdır. Günümüzde neredeyse tüm akıllı telefonlar, çoğu Mac ve birçok Windows cihaz böyledir.

Bunun tarayıcıda çalışma şekli, ekranın ne kadar yüksek çözünürlüklü olduğuna bakılmaksızın aynı olması gereken boyutları ayarlamak için CSS piksellerini kullanmalarıdır.Tarayıcı, metni daha ayrıntılı ancak aynı fiziksel boyutta oluşturacaktır. 

Three.js ile HD-DPI işlemenin çeşitli yolları vardır. 

Birincisi, özel bir şey yapmamaktır. Bu tartışmasız en yaygın olanıdır. 3D grafikler oluşturmak çok fazla GPU işlem gücü gerektirir. Mobil GPU'lar, en azından 2018 itibariyle masaüstü bilgisayarlardan daha az güce sahiptir ve yine de cep telefonları genellikle çok yüksek çözünürlüklü ekranlara sahiptir. Şu anki en üst seviye telefonlar, HD-DPI olmayan bir ekrana göre her bir piksel için 3x'lik bir HD-DPI oranına sahiptir, yani bir piksele karşılık bu telefonların 9 pikseli vardır. Bu da, 9 kat fazla render işlemi yapmaları gerektiği anlamına gelir.

9 kat fazla pikseli hesaplamak çok iş demektir, yani eğer biz kodu olduğu gibi bırakırsak pikselleri 1x olarak hesaplayacağız ve tarayıcı boyutları 3x olarak çizecektir. (3x kere 3x = 9x piksel)

Herhangi bir ağır three.js uygulaması için istediğiniz şey muhtemelen budur. Aksi takdirde yavaş bir framerate elde etmeniz olasıdır.

Bununla birlikte, gerçekten cihazın çözünürlüğünde görüntü oluşturmak istiyorsanız, bunu three.js'de yapmanın birkaç yolu vardır. 

Bir yöntem, `renderer.setPixelRatio` kullanarak çözünürlük multiplier'ı kullanmaktır. Tarayıcıya CSS piksellerinden cihaz piksellerine olan multiplier'ın kaç olduğunu sorarsınız ve bu değeri three.js'e geçersiniz

     renderer.setPixelRatio(window.devicePixelRatio);

Sonrasında, `renderer.setSize`a yapılan her çağrı, sizin pass etmiş olduğunuz multiplier oranı ile hesaplanmış olarak sihirli bir şekilde talep ettiğiniz ölçüleri kullanır. **Bu yöntem tavsiye EDİLMEZ** Aşağıyı okuyun

Diğer yol ise canvası yeniden boyutlandırdığınızda bunu kendinizin yapmasıdır.

```js
    function resizeRendererToDisplaySize(renderer) {
      const canvas = renderer.domElement;
      const pixelRatio = window.devicePixelRatio;
      const width  = canvas.clientWidth  * pixelRatio | 0;
      const height = canvas.clientHeight * pixelRatio | 0;
      const needResize = canvas.width !== width || canvas.height !== height;
      if (needResize) {
        renderer.setSize(width, height, false);
      }
      return needResize;
    }
```

Bu ikinci yol tarafsız olarak daha iyidir. Neden? Çünkü bu, ne istediysem onu elde ettim anlamına gelir. Three.js kullanırken tuvalin DrawingBuffer'ının gerçek boyutunu bilmemiz gereken birçok durum vardır. Örneğin, bir işlem sonrası filtre yaparken veya `gl_FragCoord`a erişen bir gölgelendirici yapıyorsak, bir ekran görüntüsü alıyorsak veya GPU seçme için pikselleri okuyorsak, 2B tuvale çizim yapmak için vb... `setPixelRatio` kullanırsak, gerçek boyutumuzun istediğimiz boyuttan farklı olacağı ve istediğimiz boyutu ne zaman kullanacağımızı ve three.js boyutunu ne zaman kullanacağımızı tahmin etmemiz gereken birçok durum vardır. Kendimiz yaparak, kullanılan boyutun her zaman istediğimiz boyut olduğunu biliyoruz. Perde arkasında sihirbazlık yapılan özel bir durum yoktur. 

Yukarıdaki kodun kullanıldığı bir örnek

{{{example url="../threejs-responsive-hd-dpi.html" }}}

Farkı görmek zor olabilir, ancak bir HD-DPI ekranınız varsa ve bu örneği yukarıdakilerle karşılaştırırsanız, kenarların daha net olduğunu fark edeceksiniz. 

Bu makale çok temel bir konuyu kapsamaktadır. Şimdi [three.js'in sağladığı temel ilkeleri](threejs-primitives.html) hızlıca gözden geçirelim.

