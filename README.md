# ROS2 Lifecycle Node

## Lifecycle Node Nedir?
Standart bir Node'un yanı sıra, **Lifecycle Node** kullanarak bir Node’un aşamalarını gözlemleyebilir ve kontrol edebiliriz. Bu özellik, geliştiricilerin Node state geçişlerini yöneterek daha sağlam, güvenilir ve bakımı kolay yazılımlar oluşturmasını sağlar. ROS 2'deki lifecycle modeli, düğümlerin belirli bir sırayla oluşturulması, başlatılması, çalıştırılması, durdurulması, sıfırlanması ve yok edilmesini kontrol eden bir arayüz sunar. 

Bir **Lifecycle Node**, ilk başlatıldığında `Unconfigured` durumundadır. Bu aşamada, Node başlatılmıştır ancak henüz aktif bir görev yapmaz. Aşağıdaki şemada her bir state ve geçiş görevi açıklanmaktadır (Şekil 1.1). Bu senaryo, belirlenen lifecycle aşamaları dışına çıkmadan Node yönetimi sağlar.

![Lifecycle Node Şeması](https://github.com/ertugrulsalih/ROS2-LifecycleNode-Info-TR/blob/main/img/life_cycle_sm.png?raw=true)

## Örnek Senaryo: Kamera Konfigürasyonu
Bir robotun nesne algılama ve engellerden kaçınma işlevlerini yerine getirmesi için bir kameraya bağlı olduğunu düşünelim. Güvenilir verilerin sağlanabilmesi için kamera yapılandırması son derece önemlidir. Bu noktada, standart bir Node yerine bir **Lifecycle Node** kullanarak, kameranın çözünürlük, kare hızı ve pozlama gibi parametrelerini yapılandırabiliriz. Ayrıca, kamera verilerini, algılama ve engellerden kaçınma bileşenlerine güvenilir bir şekilde yayınlayabiliriz.

Bu yaklaşım, kamera Node'unun doğru şekilde kurulmasını ve verilerinin güvenilir olmasını sağlayarak, yaşam döngüsünün daha etkili bir şekilde yönetilmesine olanak tanır.

## Örnek Senaryo: ROS Navigation Stack

![Lifecycle Node Şeması](https://github.com/ertugrulsalih/ROS2-LifecycleNode-Info-TR/blob/main/img/nav_stack.png?raw=true)

**ROS Navigation Stack**, map server, cost map generator, local planner ve global planner gibi çeşitli bileşenleri içeren, robotik navigasyon için yaygın olarak kullanılan bir yazılım paketidir. 
Bu bileşenlerin belirli bir sırayla başlatılması ve durdurulması gerekir ve genel navigasyon sisteminin düzgün çalışması için yaşam döngülerinin koordine edilmesi kritik öneme sahiptir. Örneğin, map server ve sensör topic’lerinin, cost map ve planner düğümlerinden önce yüklenmesi gerekmektedir. ROS 2 Lifecycle Node API'si ile, her bileşen için başlatma, kapatma ve durumlar arası geçişler için gerekli adımları içeren bir lifecycle state machine tanımlayabilirsiniz. 
Bu sayede, Navigasyon Yığını bileşenlerinin doğru sırada başlatılmasını ve kapatılmasını sağlayarak, sistemin stabil ve güvenilir bir şekilde çalışmasını temin edebilirsiniz.

ROS 2 Lifecycle Node, bileşenlerin başlatılması ve kapatılmasının yanı sıra, navigasyon yığınının yeniden yapılandırılması veya çevresel değişiklikler gibi runtime olaylarını da yönetmek için kullanılabilir. Örneğin, robotun gezinmesi sırasında yeni bir engelle karşılaşılması durumunda, cost map generator’ın bu yeni engeli hesaba katmak için yeniden yapılandırılması gerekebilir. ROS 2 Lifecycle Node API'sini kullanarak, diğer bileşenlerin çalışmaya devam etmesini sağlarken, bu yeniden yapılandırmayı gerçekleştirecek bir durum geçişi tanımlayabilirsiniz.


## Lifecycle Node State'leri
Lifecycle Node’ların dört tür Primary State durumu vardır:

- **Unconfigured**
- **Inactive**
- **Active**
- **Finalized**

Bir Primary State’den çıkmak için, Active durumda meydana gelen bir hata (error) dışında, harici bir denetleyici sürecin eylemi gereklidir. Ayrıca altı adet Geçiş Durumu bulunur:

- Configuring
- CleaningUp
- ShuttingDown
- Activating
- Deactivating
- ErrorProcessing

Geçiş durumlarında, geçişin başarılı olup olmadığını belirlemek için bir mantıksal işlem yürütülür. Geçişin başarı (success) ya da başarısızlık (failure) durumu, Lifecycle Management Interface aracılığıyla lifecycle management yazılımına iletilir.

## Lifecycle Node Durumları (State)

**Unconfigured State:** Bu, Node'un başlatılmasının hemen ardından içinde bulunduğu durumdur. Aynı zamanda, bir hata oluştuğunda Node’un yeniden ayarlanabileceği durumdur. Bu aşamada, Node’un herhangi bir kalıcı durumu *(stored state)* bulunmaz.

**Inactive State:** Inactive durumu, o anda herhangi bir işlem gerçekleştirmeyen bir Node’u temsil eder. Bu durumun temel amacı, Node’un davranışını etkilemeden yeniden yapılandırılmasına (örneğin, yapılandırma parametrelerinin değiştirilmesi, konu yayınlarının veya aboneliklerinin eklenmesi/kaldırılması vb.) izin vermektir. Bu durumda iken, Node herhangi bir execution time kullanmaz; yani, konuları okumaz, verileri işlemez veya işlevsel hizmet taleplerine yanıt vermez. Ayrıca, *Inactive* durumda olan bir Node, yönetilen konulardan gelen verileri okumaz veya işlemez ve yönetilen hizmet taleplerine yanıt vermez (bu talepler arayan taraf için anında başarısız olarak işaretlenir).

**Active State:** Active durumu, Node’un yaşam döngüsünün ana aşamasıdır. Bu durumda, Node herhangi bir işlem gerçekleştirir, hizmet taleplerine yanıt verir, verileri okur ve işler, çıktı üretir vb.
Bu durumda iken, Node veya sistem tarafından işlenemeyen bir hata oluşursa, Node *ErrorProcessing* durumuna geçer.

**Finalized State:** Finalized durumu, Node’un yok edilmeden önce sona erdiği aşamadır. Bu durumdan tek geçiş, yok edilme sürecidir.
Bu durum, hata ayıklama ve iç gözlem (introspection) desteklemek amacıyla mevcuttur. Başarısız olan bir Node, sistem içi gözlem için görünür kalır ve doğrudan yok edilmek yerine hata ayıklama araçları tarafından iç gözlemlenebilir olabilir. Eğer bir Node, respawn döngüsünde başlatılıyorsa veya bu döngü için bilinen bir nedeni varsa, denetleyici sürecin, Node’u otomatik olarak yok etmek ve yeniden oluşturmak için bir politikaya sahip olması beklenir.

## Durum Geçişleri (Transition)

Lifecycle Node'da, her durum geçişi belirli geri çağrılar (callbacks) ile yönetilir. Örneğin:
- **on_configure:** Node, Unconfigured durumundan Inactive durumuna geçerken çağrılır. Bu aşama, motorların başlatılması veya sensörlerin kalibrasyonu gibi işlemler için idealdir.
- **on_activate:** Node, Inactive durumundan Active durumuna geçerken çağrılır. Bu aşamada, motor kontrol sinyallerinin etkinleştirilmesi gibi işlemler gerçekleştirilir.
- **on_deactivate:** Node, Active durumundan Inactive durumuna geçerken çağrılır. Motorların güvenli bir şekilde durdurulması bu aşamada yapılabilir.
- **on_cleanup:** Node, Inactive durumundan Unconfigured durumuna geçerken çağrılır. Bu aşama, kullanılmayan kaynakların serbest bırakılması için idealdir.
- **on_shutdown:** Node kapanırken çağrılan callback'tir. Sistem kapatıldığında kalan işlemler bu aşamada temizlenir.
Bu geri çağrılar, bir robotun farklı çalışma modlarına sorunsuz bir şekilde geçiş yapmasını sağlar.


## Lifecycle Node Durumları (State)
ROS 2'de bir Lifecycle Node, **rclcpp_lifecycle::LifecycleNode** sınıfı kullanılarak tanımlanır. Bu sınıf, durum geçişlerini ve ilgili geri çağrıları (callbacks) yönetmemize olanak tanır. Örneğin, bir robotun hız kontrol Node'unu oluştururken, *on_configure* geri çağrısında hız kontrol parametrelerini yapılandırabiliriz. Ardından, *on_activate* geri çağrısında bu parametreleri kullanarak motorlara hız komutlarını gönderebiliriz.

```cpp
class MotorController : public rclcpp_lifecycle::LifecycleNode {
public:
  MotorController() : LifecycleNode("motor_controller") {}

protected:
  rclcpp_lifecycle::node_interfaces::LifecycleNodeInterface::CallbackReturn on_configure(const rclcpp_lifecycle::State &) {
    RCLCPP_INFO(this->get_logger(), "Motor Controller Configured.");
    // Motorları başlat
    return rclcpp_lifecycle::node_interfaces::LifecycleNodeInterface::CallbackReturn::SUCCESS;
  }

  rclcpp_lifecycle::node_interfaces::LifecycleNodeInterface::CallbackReturn on_activate(const rclcpp_lifecycle::State &) {
    RCLCPP_INFO(this->get_logger(), "Motor Controller Activated.");
    // Hız kontrol sinyallerini gönder
    return rclcpp_lifecycle::node_interfaces::LifecycleNodeInterface::CallbackReturn::SUCCESS;
  }

  // Diğer callbackler burada yer alır...
};
```
 
## Lifecycle Node Yönetimi

Yönetilen bir Node, yönetimi gerçekleştiren araçlar tarafından görüldüğü şekliyle Lifecycle arayüzü aracılığıyla ROS ekosistemine dahil edilir. Bu arayüz, yönetim araçları tarafından görüldüğünde, lifecycle durumlarının iletişim üzerindeki kısıtlamalarına tabi olmamalıdır.
En yaygın desenler, bir kütüphaneden “managed node implementation” yükleyen ve bir eklenti mimarisi (plugin architecture) aracılığıyla gerekli olan yönetim arayüzünü (managed interface) yöntemler aracılığıyla otomatik olarak ortaya çıkaran bir container sınıfıdır.
Container'lar, kendileri lifecycle yönetimine tabi değildir. Ancak, bu arayüzü sağlayan ve lifecycle ilkelerine uyan herhangi bir uygulama geçerli bir managed node olarak kabul edilir. Aksi durumda, bu hizmetleri sağlayan ancak lifecycle state machine yapısına uygun davranmayan nesneler hatalı biçimlendirilmiş kabul edilir.
Yönetim hizmetleri, hem yerel yönetim (Attributes ve Methods) hem de uzak yönetim (ROS mesajları ve Topic'ler) üzerinden sağlanabilir. ROS arayüzü sağlanırken, belirli Topic'ler kullanılmalı ve uygun bir namespace belirlenmelidir.
Yönetilen bir Node, yönetilmeyen bir sistemde çalıştırıldığında otomatik olarak “configure” ve “activate” için argümanları da isteyebilir.
Yönetilen bir Node’un durumlar arasında geçiş yapabileceği birkaç farklı yol vardır. Çoğu durum geçişinin, Node’un yapılandırılması ve başlatılmasını sağlayan harici bir yönetici (örneğin, bu bir lifecycle_manager_node olabilir) tarafından koordine edilmesi beklenir. Harici yönetim aracının, aynı zamanda lifecycle Node’unu izlemesi ve hata durumunda kurtarma işlemlerini gerçekleştirmesi beklenir. Yerel bir yönetim aracı kullanmak da mümkündür ve bu araç, yöntem seviyesindeki arayüzleri kullanabilir. Bir Node’un kendi kendini yönetmesi de mümkündür, *ancak bu durum, Node’u arayüz üzerinden yönetmeye çalışan dış mantıkla çakışabileceği için önerilmez.*
Bu lifecycle araç zincirinin tüm aşamalarında desteklenmesi gerekmektedir; *bu nedenle, bu tasarımın ek durumlarla genişletilmesi amaçlanmamıştır.* Daha karmaşık, uygulamaya özgü durum makinelerinin herhangi bir lifecycle durumu içinde veya makro düzeyde var olması beklenmektedir. Bu lifecycle durumlarının, bir denetleyici sistemin bir parçası olarak yararlı temel bileşenler olması beklenir.

## Konuya Dair Faydalı Linkler
- [Managed Nodes](https://design.ros2.org/articles/node_lifecycle.html)
- [ROS2 from the Ground Up: Part 8- Simplify Robotic Software Components Management with ROS2 Lifecycle Nodes](https://medium.com/@nullbyte.in/ros2-from-the-ground-up-part-8-simplify-robotic-software-components-management-with-ros2-5fafa2738700)
- [How to Use ROS 2 Lifecycle Nodes](https://docs.ros.org/en/ros2_packages/rolling/api/lifecycle_msgs/index.html)
- [ROS2 - Lifecycle Node Tutorial](https://www.youtube.com/watch?v=_GXHBP5sA70)
- [Managed (Lifecycle) Nodes in ROS 2 (Concept + Cpp Code Demo)](https://www.youtube.com/watch?v=axraRVgFRec)
- [ROS Package: lifecycle](https://index.ros.org/p/lifecycle/)
- [ROS2 -Rolling Demo - Lifecucle](https://github.com/ros2/demos/tree/rolling/lifecycle)
- [Nav2 - Repo - Lifecycle](https://github.com/ros-navigation/navigation2/blob/main/nav2_lifecycle_manager/README.md)
- [rclcpp_lifecycle::LifecycleNode Class Reference](https://docs.ros2.org/latest/api/rclcpp_lifecycle/classrclcpp__lifecycle_1_1LifecycleNode.html)
