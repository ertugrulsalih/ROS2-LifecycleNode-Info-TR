# ROS2 Lifecycle Node

## Lifecycle Node Nedir?
Standart bir Node'un yanı sıra, **Lifecycle Node** kullanarak bir Node’un aşamalarını gözlemleyebilir ve kontrol edebiliriz. Bu özellik, geliştiricilerin Node state geçişlerini yöneterek daha sağlam, güvenilir ve bakımı kolay yazılımlar oluşturmasını sağlar. ROS 2'deki lifecycle modeli, düğümlerin belirli bir sırayla oluşturulması, başlatılması, çalıştırılması, durdurulması, sıfırlanması ve yok edilmesini kontrol eden bir arayüz sunar. 

Bir **Lifecycle Node**, ilk başlatıldığında `Unconfigured` durumundadır. Bu aşamada, Node başlatılmıştır ancak henüz aktif bir görev yapmaz. Aşağıdaki şemada her bir state ve geçiş görevi açıklanmaktadır (Şekil 1.1). Bu senaryo, belirlenen lifecycle aşamaları dışına çıkmadan Node yönetimi sağlar.

![Lifecycle Node Şeması](link-to-image)

## Örnek Senaryo: Kamera Konfigürasyonu
Bir robotun nesne algılama ve engellerden kaçınma işlevlerini yerine getirmesi için bir kameraya bağlı olduğunu düşünelim. Güvenilir verilerin sağlanabilmesi için kamera yapılandırması son derece önemlidir. Bu noktada, standart bir Node yerine bir **Lifecycle Node** kullanarak, kameranın çözünürlük, kare hızı ve pozlama gibi parametrelerini yapılandırabiliriz. Ayrıca, kamera verilerini, algılama ve engellerden kaçınma bileşenlerine güvenilir bir şekilde yayınlayabiliriz.

Bu yaklaşım, kamera Node'unun doğru şekilde kurulmasını ve verilerinin güvenilir olmasını sağlayarak, yaşam döngüsünün daha etkili bir şekilde yönetilmesine olanak tanır.

## Örnek Senaryo: ROS Navigation Stack
**ROS Navigation Stack**, map server, cost map generator, local planner ve global planner gibi çeşitli bileşenleri içeren bir yazılım paketidir. Bu bileşenlerin belirli bir sırayla başlatılması ve durdurulması gerekir. Örneğin, map server ve sensör topic’lerinin, cost map ve planner düğümlerinden önce yüklenmesi gerekmektedir.

ROS 2 Lifecycle Node API'si ile, her bileşen için başlatma, kapatma ve durumlar arası geçişler için gerekli adımları içeren bir lifecycle state machine tanımlayabilirsiniz.

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

## Lifecycle Node Yapısı
ROS 2'de bir Lifecycle Node, `rclcpp_lifecycle::LifecycleNode` sınıfı kullanılarak tanımlanır. Bu sınıf, durum geçişlerini ve ilgili geri çağrıları (callbacks) yönetmemize olanak tanır. Örneğin:

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
