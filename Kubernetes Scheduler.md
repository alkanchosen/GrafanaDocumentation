### Kubernetes Scheduler
Kubernetes'te scheduling, Kubelet'in Pod'ların çalıştırabilmesi için Node'ların Pod'lar ile eşleştiğinden emin olmayı ifade eder.

##### Scheduling'e genel bakış
Bir scheduler, atanmış Node'u olmayan yeni Pod'ları izler. Scheduler keşfettiği her Pod için o Pod'un üzerinde çalışacağı en iyi Node'u bulmaktan sorumlu olur. Scheduler, aşağıda açıklanan programlama prensiplerini dikkate alarak bu yerleştirme kararına ulaşır.

Pod'ların neden belirli bir Node'a yerleştirildiğini anlamak istiyorsanız veya kendiniz bir özel scheduler uygulamayı planlıyorsanız, bu sayfa scheduler hakkında bilgi edinmenize yardımcı olacaktır.

##### kube-scheduler

[kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/), Kubernetes için varsayılan scheduler'dir ve kontrol düzleminin bir parçası olarak çalışır. kube-scheduler, isterseniz ve gerekirse kendi programlama bileşeninizi yazıp onun yerine kullanabilmeniz için tasarlanmıştır.

kube-scheduler, her yeni oluşturulan Pod veya diğer planlanmamış Pod'lar için üzerinde çalışacakları en uygun Node'u seçer. Ancak, Pod'lardaki her konteynerin kaynaklar için farklı gereksinimleri vardır ve her Node'un da farklı gereksinimleri vardır. Bu nedenle, mevcut Node'ları belirli programlama gereksinimlerine göre filtrelenmesi gerekir.

Bir kümede, bir Pod için scheduling gereksinimlerini karşılayan Node'lar, uygun node'lar olarak adlandırılır. Node'ların hiçbiri uygun değilse, Pod, scheduler tarafından yerleştirilene kadar planlanmamış olarak kalır.

Scheduler, bir Pod için uygun Node'lar bulur ve ardından uygun Node'ları puanlamak için bir dizi işlevi çalıştırır. Bunun ardından Pod'u çalıştırmak için uygun olanlar arasından en yüksek puanı alan Node'u seçer. Scheduler daha sonra, *binding* adı verilen bir işlemle bu karar hakkında API sunucusunu bilgilendirir.

Scheduling kararları için dikkate alınması gereken faktörler arasında bireysel ve toplu kaynak gereksinimleri, donanım / yazılım / politika kısıtlamaları, yakınlık ve afinite karşıtı özellikler, veri konumu, iş yükü arası girişim vb. yer alır.

##### kube-scheduler'da Node seçimi

kube-scheduler, 2 adımlı bir işlemde Pod için bir Node seçer:

1. Filtreleme

2. Puanlama

Filtreleme adımı, Pod'un planlanmasının uygun olduğu Node kümesini bulur. Örneğin, *PodFitsResources* filtresi, bir aday Node'un bir Pod'un belirli kaynak isteklerini karşılamak için yeterli kullanılabilir kaynağa sahip olup olmadığını kontrol eder. Bu adımdan sonra, Node listesi herhangi bir uygun Pod'u içerir; genellikle birden fazla olacaktır. Liste boşsa, bu Pod (henüz) planlanabilir değildir.

Puanlama adımında, scheduler kalan Node'ları en uygun Pod yerleşimini seçmek için sıralar. Programlayıcı, filtrelemeden sonra kalan her Node'a, aktif puanlama kurallarına dayandırarak bir puan atar.

Son olarak, kube-scheduler Pod'u en yüksek sıralamaya sahip Node'a atar. Eşit puanlara sahip birden fazla Node varsa, kube-scheduler bunlardan birini rastgele seçer.

Planlayıcının filtreleme ve puanlama davranışını yapılandırmanın desteklenen iki yolu vardır:

1. [Scheduling Policies](https://kubernetes.io/docs/reference/scheduling/policies), filtreleme için tahminleri ve puanlama için öncelikleri yapılandırmanıza olanak tanır.
2. [Scheduling Profiles](https://kubernetes.io/docs/reference/scheduling/config/#profiles), `QueueSort, Filter, Score, Bind, Reserve, Permit` ve diğerleri dahil olmak üzere farklı scheduling aşamalarını uygulayan eklentileri yapılandırmanıza olanak tanır. Ayrıca kube-scheduler'ı farklı profilleri çalıştıracak şekilde yapılandırabilirsiniz.

