# Panel Plugin Geliştirme

## 1. Giriş
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

## 2. Çalışma ortamınızı ayarlayın
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

## 3. Yeni bir plugin oluşturun

Modern web geliştirme için araç geliştirmek zor olabilir. Kendi webpack konfigürasyonunuzu yazabilme imkanınız varken, bu rehber için, grafana-toolkit'i kullanacağız.

[grafana-toolkit](https://github.com/grafana/grafana/tree/master/packages/grafana-toolkit) Grafana için plugin geliştirmeyi kolaylaştıran bir CLI (Komut Satırı Arayüzü) uygulamasıdır. Böylece kod yazmaya daha fazla odaklanabilirsiniz. Geri kalan yapı ve test işlerini grafana-toolkit sizin için halleder.

1. Plugin klasöründe `plugin:create` komunutu çalıştırarak yeni bir şablon oluşturun.

```
npx @grafana/toolkit plugin:create my-plugin
```

2. Klasör değiştirin.
```
cd my-plugin
```

3. Gerekli bağımlılıkları yükleyin.
```
yarn install
```

4. Plugini oluşturun.
```
yarn dev
```

5. Grafana'nın yeni plugininizi keşfetmesi için Grafana sunucusunu yeniden başlatın.

6. Grafana'yı açın ve **Configuration -> Plugins** kısmına gidin. Plugininizin orada olduğundan emin olun.

Grafana varsayılan olarak yeni bir plugin keşfettiğinde log dosyasına kayıt alır.
```
INFO[01-01|12:00:00] Registering plugin       logger=plugins name=my-plugin
```

## 4. Pluginin yapısı

Pluginler farklı şekil ve büyüklüklerde gelir. Daha detaylı incelemeden önce hepsi tarafından kullanılan bazı dosyalara bakalım.

Oluşturduğunuz her plugin en az iki dosyaya ihtiyaç duyacak: `plugin.json` and `module.ts`.

### plugin.json

Grafana çalıştırıldığında plugin klasöründe `plugin.json` dosyası içeren bütün alt klasörleri tarar. `plugin.json` dosyası plugininizin hakkında bilgi içerir ve Grafana'ya plugininizin yapabildiklerini ve gerekli bağımlılıklarını iletir.

Bazı plugin tipleri farklı konfigürasyon seçeneklerine sahip olsa da zorunlu olanlara bakalım:

- `type` Grafana'ya plugininizin türünü söyler. Grafana üç farklı plugin türünü destekler: `panel`, `datasource` ve `app`.

- `name` kullanıcıların plugin listesinde göreceği isimdir. Eğer bir data source yaratıyorsanız bu genellikle bağlandığı veri tabanın ismini alır. Örnek: Prometheus, PostgreSQL, Stackdriver.

- `id` plugininizi özgün şekilde ifade eder ve diğer pluginlerle çakışmayı önlemek için Grafana kullanıcı adınızla başlamalıdır. Grafana kullanıcı adı almak için [Grafana'ya kaydolun](https://grafana.com/signup).

`plugin.json` dosyasındaki bütün konfigürasyon ayarlarına bakmak için [plugin.json şemasına](https://grafana.com/docs/grafana/latest/plugins/developing/plugin.json) bakın.

### module.ts

Grafana plugininizi keşfettikten sonra `module.ts` dosyasını yükler, bu dosya plugininizin giriş noktasıdır. `module.ts` plugininizin yazılımını açığa çıkarır, bu yazılım oluşturduğunuz plugin tipine göre değişir.

Spesifik olarak `module.ts` [GrafanaPlugin'i](https://github.com/grafana/grafana/blob/08bf2a54523526a7f59f7c6a8dafaace79ab87db/packages/grafana-data/src/types/plugin.ts#L124) extend eden bir objeyi açığa çıkarmalıdır. Bu aşağıdakilerden herhangi birisi olabilir:

* [PanelPlugin](https://github.com/grafana/grafana/blob/08bf2a54523526a7f59f7c6a8dafaace79ab87db/packages/grafana-data/src/types/panel.ts#L73)
* [DataSourcePlugin](https://github.com/grafana/grafana/blob/08bf2a54523526a7f59f7c6a8dafaace79ab87db/packages/grafana-data/src/types/datasource.ts#L33)
* [AppPlugin](https://github.com/grafana/grafana/blob/45b7de1910819ad0faa7a8aeac2481e675870ad9/packages/grafana-data/src/types/app.ts#L27)
