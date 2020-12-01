# Veri Kaynağı Plugin Geliştirme Rehberi

## 1. Giriş

Grafana Prometheus, MySQL, PostgreSQL, Datadog dahil olmak üzere birçok veri tabanını destekler. Kurduğunuz sistemleri zaten görselleştirme imkanınızın olma ihtimali çok yüksektir. Fakat bazı durumlarda Grafana dashboard'larına kendi yazdığınız bir veri kaynağı çözümünüzü eklemek isteyebilirsiniz. Bu rehber size o veri kaynağını nasıl destekleyeceğinizi gösterir.

Bu rehberde şunları yapacaksınız:

* Bir sinüs dalgasını görselleştirmek için veri kaynağı yaratmak
* Query editor kullanarak sorgularınızı oluşturmak
* Config editor kullanarak veri kaynağınızı düzenlemek

### Gereklilikler

* Grafana 7.0
* NodeJS 12.x
* yarn

## 2. Çalışma ortamınızı ayarlayın
Plugin geliştirmeye başlamadan önce çalışma ortamınızı ayarlamanız gerekir.

Pluginleri keşfetmek için Grafana plugins klasörü arar, bu klasörün konumu işletim sisteminize göre değişiklik gösterir.

1. Çalışma ortamınızda `grafana-plugins` adında bir klasör açın.

2. Grafana konfigürasyon dosyasında `plugins` satırını bulun ve dosya yolunu kendi `grafana-plugins` klasörünüze ayarlayın. Grafana konfigürasyon dosyası alakalı daha fazla bilgi için dökümantasyona ulaşın.

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

Grafana çalıştırıldığında plugin klasöründe `plugin.json` dosyası içeren bütün alt klasörleri tarar. `plugin.json` dosyası plugininiz hakkında bilgi içerir ve Grafana'ya plugininizin yapabildiklerini ve gerekli bağımlılıklarını iletir.

Bazı plugin tipleri farklı konfigürasyon seçeneklerine sahip olsa da zorunlu olanlara bakalım:

- `type` Grafana'ya plugininizin türünü söyler. Grafana üç farklı plugin türünü destekler: `panel`, `datasource` ve `app`.

- `name` kullanıcıların plugin listesinde göreceği isimdir. Eğer bir data source yaratıyorsanız bu genellikle bağlandığı veri tabanın ismini alır. Örnek: Prometheus, PostgreSQL, Stackdriver.

- `id` plugininizi özgün şekilde ifade eder ve diğer pluginlerle çakışmayı önlemek için Grafana kullanıcı adınızla başlamalıdır. Grafana kullanıcı adı almak için [Grafana'ya kaydolun](https://grafana.com/signup).

`plugin.json` dosyasındaki bütün konfigürasyon ayarlarına bakmak için [plugin.json şemasına](https://grafana.com/docs/grafana/latest/plugins/developing/plugin.json) bakın.

### module.ts

Grafana plugininizi keşfettikten sonra `module.ts` dosyasını yükler, bu dosya plugininizin giriş noktasıdır. `module.ts` plugininizin yazılımını içerir, bu yazılım oluşturduğunuz plugin tipine göre değişir.

Spesifik olarak `module.ts` [GrafanaPlugin'i](https://github.com/grafana/grafana/blob/08bf2a54523526a7f59f7c6a8dafaace79ab87db/packages/grafana-data/src/types/plugin.ts#L124) extend eden bir objeyi içermelidir. Bu aşağıdakilerden herhangi birisi olabilir:

* [PanelPlugin](https://github.com/grafana/grafana/blob/08bf2a54523526a7f59f7c6a8dafaace79ab87db/packages/grafana-data/src/types/panel.ts#L73)
* [DataSourcePlugin](https://github.com/grafana/grafana/blob/08bf2a54523526a7f59f7c6a8dafaace79ab87db/packages/grafana-data/src/types/datasource.ts#L33)
* [AppPlugin](https://github.com/grafana/grafana/blob/45b7de1910819ad0faa7a8aeac2481e675870ad9/packages/grafana-data/src/types/app.ts#L27)

## 5. Veri kaynağı pluginleri

Grafana'da bir veri kaynağı `DataSourceApi` arayüzünü extend etmelidir. Bu durumda `query` ve `testDatasource` adlı iki tane metot yaratmanız gerekir.

### `query` metodu

`query` metodu bir veri kaynağının kalbidir. Kullanıcıdan bir sorguyu alır, veriyi dış bir veri tabanından çeker ve veriyi Grafana'nın tanıyabileceği bir formatta geri döndürür.

```
async query(options: DataQueryRequest<MyQuery>): Promise<DataQueryResponse>
```

`options` objesi kullanıcığın yaptığı sorguları ve mevcut zaman aralığı gibi çeşitli bilgileri içerir. Dış bir veri tabanından veri çekmek için bu bilgiyi kullanın.

### Veri kaynağınızı test edin

`testDatasource` metodu veri kaynağınızda bir sağlık taraması yürütür. Örnek olarak, kullanıcı her bağlantı ayarlarını değiştirip **Kaydet & Çık** butonuna bastığında Grafana bu metodu çalıştırır. 

```
async testDatasource()
```

## 6. Data frame'leri

Bu günlerde sayısız veri tabanı bulunmaktadır ve her birinin kendi sorgu metodları vardır. Bütün bu farklı veri yapılarını desteklemek için Grafana alınan verileri *data frame* adındaki bir veri yapısına dönüştürür.

Hadi `query` metodu nasıl bir data frame oluşturup döndüreceğinizi görelim. Bu adımda starter plugin'de bulunan kodu değiştirip bir [sinüs dalgası](https://en.wikipedia.org/wiki/Sine_wave) göndermesini sağlayacaksınız.

1. Mevcut `query` metodunda bulunan `map` fonksiyonun içerisindeki kodu kaldırın.

`query` metodu şu şekilde gözükmeli:

```typescript
async query(options: DataQueryRequest<MyQuery>): Promise<DataQueryResponse> {
  const { range } = options;
  const from = range!.from.valueOf();
  const to = range!.to.valueOf();

  const data = options.targets.map(target => {
    // Your code goes here.
  });

  return { data };
}
```

2. `map` fonksiyonunda, ayarlanmamış sorgu özelliklerine varsayılan değerler koymak için `lodash/defaults` paketini kullanın.

```typescript
const query = defaults(target, defaultQuery);
```

3. Zaman ve sayı sütunları bulunan bir data frame yaratın.

```typescript
const frame = new MutableDataFrame({
  refId: query.refId,
  fields: [
    { name: 'time', type: FieldType.time },
    { name: 'value', type: FieldType.number },
  ],
});
```

`refId` ayarlanmalıdır çünkü Grafana hangi sorgunun bu data frame'i yarattığını buna bakarak anlar.

Şimdi, data frame'a gerçek veriler ekleyeceğiz. Sayıların hesaplanmasında kullanılan matematik konusunda endişelenmeyin.

1. Birkaç yardımcı değişken yaratın.

```typescript
// duration of the time range, in milliseconds.
const duration = to - from;

// step determines how close in time (ms) the points will be to each other.
const step = duration / 1000;
```

2. Data frame'e yeni değerleri ekleyin.

```typescript
for (let t = 0; t < duration; t += step) {
  frame.add({ time: from + t, value: Math.sin((2 * Math.PI * t) / duration) });
}
```

`frame.add()` anahtar isimleri data frame'de bulunan sütun isimleriyle aynı olan objeleri kabul eder.

3. Data frame'i döndürün.

```typescript
return frame;
```

4. Plugin'i tekrardan build edin ve deneyin.

Veri kaynağınız artık Grafana'nın görselleştirebileceği data frame'leri gönderiyor. Gelecek kısımda sorgu yaratarak nasıl sinüs dalgasının frekansını ayarlayabileceğinize bakacağız.

## 7. Sorgu tanımlamak

