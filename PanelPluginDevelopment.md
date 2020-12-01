# Panel Plugin Geliştirme

### Giriş
Paneller Grafana'nın temel bileşenleridir. Çeşitli verileri farklı yollarla görselleştirmenize izin verirler. Grafana'da hali hazırda bulunan değişik panel tipleri varken başka görselleştirmeler yapmak için kendi panelinizi yaratabilirsiniz.

Panellerle alakalı daha fazla bilgi için Grafana'nın Panels hakkındaki dökümantasyonunu okuyabilirsiniz.

### Gereklilikler
- Grafana 7.0
- NodeJS 12.x
- yarn

Set up your environment
Before you can get started building plugins, you need to set up your environment for plugin development.

To discover plugins, Grafana scans a plugin directory, the location of which depends on your operating system.

Create a directory called grafana-plugins in your preferred workspace.

Find the plugins property in the Grafana configuration file and set the plugins property to the path of your grafana-plugins directory. Refer to the Grafana configuration documentation for more information.

### Çalışma ortamınızı ayarlayın
Plugin geliştirmeye başlamadan önce çalışma ortamınızı ayarlamanız gerekir.

Pluginleri keşfetmek için Grafana plugins klasörü arar, bu klasörün konumu işletim sisteminize göre değişiklik gösterir.

1. Çalışma ortamınızda `grafana-plugins` adında bir klasör açın.

2. Grafana konfigürasyon dosyasında `plugins` satırını bulun ve dosya yolunu kendi `grafana-plugins` klasörünüze ayarlayın. Grafana konfigürasyon dosyası alakalaı daha fazla bilgi için dökümantasyona ulaşın.

```
[paths]
plugins = "/path/to/grafana-plugins"
```
3. Grafana hali hazırda çalışıyorsa yeni konfigürasyonu çalıştırmak için yeniden başlatın.

### Alternatif yol: Docker
Eğer Grafana'yı yerel makinenize kurmak istemiyorsanız Docker kullanabilirsiniz.

Grafana'yı plugin geliştirme için ayarlarken Docker kullanmak istiyorsanız aşağıdaki komutu çalıştırın:

```
docker run -d -p 3000:3000 -v "$(pwd)"/grafana-plugins:/var/lib/grafana/plugins --name=grafana grafana/grafana:7.0.0
```

Grafana pluginleri sadece açılışta yüklediği için herhangi bir plugin eklediğinizde ya da kaldırdığınızda container'ı yeniden başlatmanız gerekmektedir.

```
docker restart grafana
```
