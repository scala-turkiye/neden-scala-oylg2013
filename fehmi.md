## Ölçekleme

### Yatay ölçekleme (Scala out/Scale down)

Dağıtık bir uygulamaya yeni bir düğüm(node) eklemek ya da çıkarmak.

### Dikey ölçekleme (Scale up)

Tek bir düğüme CPU ya da bellek gibi yeni kaynak eklemek.

#### Parallelism

Paralel donanımlar üzerinde programları daha hızlı koşmak.

#### Concurrency (Eş zamanlılık)

İş parçacıklarının eş-zamanlı olarak koşmalarını yönetmek.

#### Concurrency problemi
```scala
var x = 0
async { x = x + 1 }
async { x = x * 2 }
// muhtemel sonuçlar 0, 1, 2
```
Eş-zamanlı iş parçacıklarının paylaşımlı değişken durumlara(mutable shared state) erişmesi rasgeleliğin sebebidir.

**non-determinism = paralel işleme + değişken durum**

** Rasgele olmayan işleme istiyorsak değişken durumdan(mutable state) sakınmamız gerekli.**

#### Senkronizasyon maliyeti

Eğer mutable state varsa eş zamanlı çalışmanın senkronize edilmesi gerekir. Java’da aşağıdaki senkronizasyon araçları mevcuttur.

* Atomik veri yapıları (AtomicInteger, ...)
* Synchronized bloklar
* CountDownLatch
* CyclicBarrier
* Semaphore

**Eğer uygulamanızda mutable state yoksa senkronizasyon gibi bir sorununuz da yoktur.**

## Threaded vs Evented

* Threaded: Servlets, Ruby on Rails, Django, PHP

* Evented: Play, Node.js, Twisted

### Threaded sunucular

Threaded sunucular her bir isteğe bir thread atarlar ve o thread cevap gönderilene kadar tüm işleri gerçekleştirirler. Uzak bir servis çağrısı gibi herhangi bir I/O senkrondur. Bu sebeple thread bloklanır ve I/O işlemi tamamlanana kadar boş oturur.

**Blocking Scala kodu**

```scala
// Apache HttpClient ile example.com'dan bir istek yapılıyor.
val client = new HttpClient()
val method = new GetMethod("http://www.example.com/")

// executeMethod blocking, senkron bir çağrıdır
val statusCode = client.executeMethod(method)

// Bu satır çalıştığında example.com'dan cevap alındığından eminiz.
println("Server responsed with %d".format(statusCode))
```

### Sınırsız thread oluşturmanın dezavantajları

#### Thread yaşam döngüsü maliyeti

Thread oluşturma ve öldürme maliyetsiz değildir. 
Çoğu sunucu uygulamasında olduğu gibi istekler sık ve küçük ise her bir istek için yeni bir thread oluşturmak 
oldukça yüksek kaynak tüketimine sebep olur.

#### Kaynak tüketimi

Aktif threadler sistem kaynaklarını(özellikle bellek) kullanırlar. 
Kullanılabilir işlemci sayısından daha fazla thread var ise threadler boş dururlar. 
Boş duran threadler gereksiz yere bellek kullanırlar. 
İşlemciler için yarışan çok sayıda thread var ise bu durum ayrıca performans kaybına sebep olabilir.

#### Kararlılık

Oluşturulabilecek thread sayısı sınırlıdır. 
Bu sınır stack size gibi parametrelere ve işletim sisteminin sınırlarına bağlıdır. 
Bu sınıra ulaştığınızda OOM gibi bir yanıt alırsınız. Bu durumu düzeltmeye çalışmak risklidir. 
Bunun yerine uygulamanızı bu sınıra ulaşmayacak şekilde tasarlamanız daha kolaydır.

### Evented sunucular

**Non-blocking Javascript kodu**
```js
var callback = function(data) {
  console.log("Callback fired with: " + data);
};

$.get('/example', callback);

// Bu satır çalıştığında uzak servis çağrısıdan büyük ihtimalle henüz cevap dönmemiş olacak. 
console.log(''Bu satır callbackten önce çalışabilir!');
```

#### Play Framework ile async örneği

```scala
object ProxyController extends Controller {

  def proxy = Action {
    val responseFuture: Future[Response] = WS.url("http://example.com").get()

    val resultFuture: Future[Result] = responseFuture.map { resp =>
      // Create a Result that uses the http status, body, and content-type
      // from the example.com Response
      Status(resp.status)(resp.body).as(resp.ahcResponse.getContentType)
    }

    Async(resultFuture)
  }

}
```

```scala
// Make 3 parallel async calls
val fooFuture = WS.url("http://foo.com").get()
val barFuture = WS.url("http://bar.com").get()
val bazFuture = WS.url("http://baz.com").get()

for {
  foo <- fooFuture
  bar <- barFuture
  baz <- bazFuture
} yield {
  // Build a Result using foo, bar, and baz

  Ok(...)
}
```

