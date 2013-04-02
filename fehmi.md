## Sınırsız thread oluşturmanın dezavantajları

* Thread Lifecycle overhead

Thread oluşturma ve öldürme maliyetsiz değildir. Çoğu sunucu uygulamasında olduğu gibi istekler sık ve küçük ise her bir istek için yeni bir thread oluşturmak oldukça yüksek kaynak tüketimine sebep olur.

* Resource consumption

Aktif threadler sistem kaynaklarını(özellikle bellek) kullanırlar. Kullanılabilir işlemci sayısından daha fazla thread var ise threadler boş dururlar. Boş duran threadler gereksiz yere bellek kullanırlar. İşlemciler için yarışan çok sayıda thread var ise bu durum ayrıca performans kaybına sebep olabilir.

* Stability

Oluşturulabilecek thread sayısı sınırlıdır. Bu sınır stack size gibi parametrelere ve işletim sisteminin sınırlarına bağlıdır. Bu sınıra ulaştığınızda OOM gibi bir yanıt alırsınız. Bu durumu recover etmeye çalışmak risklidir. Bunun yerine uygulamanızı bu sınıra ulaşmayacak şekilde tasarlamanız daha kolaydır.
