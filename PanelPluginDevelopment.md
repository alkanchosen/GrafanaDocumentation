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

Create a new plugin
Tooling for modern web development can be tricky to wrap your head around. While you certainly can write your own webpack configuration, for this guide, you’ll be using grafana-toolkit.

grafana-toolkit is a CLI application that simplifies Grafana plugin development, so that you can focus on code. The toolkit takes care of building and testing for you.

In the plugin directory, create a plugin from template using the plugin:create command:

npx @grafana/toolkit plugin:create my-plugin
Change directory.

cd my-plugin
Download necessary dependencies:

yarn install
Build the plugin:

yarn dev
Restart the Grafana server for Grafana to discover your plugin.

Open Grafana and go to Configuration -> Plugins. Make sure that your plugin is there.

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

