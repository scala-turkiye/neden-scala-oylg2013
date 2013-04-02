## Ölçekleme

### Yatay ölçekleme (Scala out/Scale down)

Add more nodes to a system, such as adding a new computer to a distributed software application.

### Dikey ölçekleme (Scale up)

Add resources to a single node in a system, typically involving the addition of CPUs or memory to a single computer.

#### Parallelism

Paralel donanımlar üzerinde programları daha hızlı koşmak.

#### Concurrency

İş parçacıklarının eş-zamanlı olarak koşmalarını yönetmek.

#### Temel problem
```scala
var x = 0
async { x = x + 1 }
async { x = x * 2 }
// muhtemel sonuçlar 0, 1, 2
```
Eş-zamanlı iş parçacıklarının paylaşımlı değişken durumlara(mutable shared state) erişmesi rasgeleliğin sebebidir.

**non-determinism = paralel işleme + değişken durum**

Rasgele olmayan işleme istiyorsak değişken durumdan(mutable state) sakınmamız gerekli.

#### Senkronizasyon maliyeti

Eğer mutable state varsa eş zamanlı çalışmanın senkronize edilmesi gerekir. Java’da aşağıdaki senkronizasyon araçları mevcuttur.

* Atomik veri yapıları (AtomicInteger, …)
* Synchronized bloklar
* CountDownLatch
* CyclicBarrier
* Semaphore

Eğer uygulamanızda mutable state yoksa senkronizasyon gibi bir sorununuz da yoktur.


## Sınırsız thread oluşturmanın dezavantajları

### Thread Lifecycle overhead

Thread oluşturma ve öldürme maliyetsiz değildir. Çoğu sunucu uygulamasında olduğu gibi istekler sık ve küçük ise her bir istek için yeni bir thread oluşturmak oldukça yüksek kaynak tüketimine sebep olur.

### Resource consumption

Aktif threadler sistem kaynaklarını(özellikle bellek) kullanırlar. Kullanılabilir işlemci sayısından daha fazla thread var ise threadler boş dururlar. Boş duran threadler gereksiz yere bellek kullanırlar. İşlemciler için yarışan çok sayıda thread var ise bu durum ayrıca performans kaybına sebep olabilir.

### Stability

Oluşturulabilecek thread sayısı sınırlıdır. Bu sınır stack size gibi parametrelere ve işletim sisteminin sınırlarına bağlıdır. Bu sınıra ulaştığınızda OOM gibi bir yanıt alırsınız. Bu durumu recover etmeye çalışmak risklidir. Bunun yerine uygulamanızı bu sınıra ulaşmayacak şekilde tasarlamanız daha kolaydır.


## Threaded vs Evented

Threaded: Servlets, Ruby on Rails, Django, PHP
Evented: Play, Node.js, Twisted


**Blocking Scala kodu**

```scala
// Use Apache HttpClient to make a remote call to example.com

val client = new HttpClient()
val method = new GetMethod("http://www.example.com/")

// executeMethod is a blocking, synchronous call
val statusCode = client.executeMethod(method)

// When this line is executed, we are guaranteed that the call to example.com
// has completed
println("Server responsed with %d".format(statusCode))
```

**Non-blocking Javascript kodu**
```js
// Use jQuery to make an Ajax call to /example

var callback = function(data) {
  console.log("Callback fired with: " + data);
};

// $.get is a non-blocking, asynchronous call
$.get('/example', callback);

// When this line is executed, we are not garaunteed that the call to /example
// has completed
console.log('This line may execute before the callback!');
```

## Play Framework ile async programlama

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

