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

non-determinism = paralel işleme + değişken durum
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
