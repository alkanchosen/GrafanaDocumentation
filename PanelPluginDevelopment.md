# Grafana Panel Plugin Geliştirme Rehberi

## 1. Giriş
Paneller Grafana'nın temel bileşenleridir. Çeşitli verileri farklı yollarla görselleştirmenize izin verirler. Grafana'da hali hazırda bulunan değişik panel tipleri varken başka görselleştirmeler yapmak için kendi panelinizi yaratabilirsiniz.

Panellerle alakalı daha fazla bilgi için Grafana'nın Panels hakkındaki dökümantasyonunu okuyabilirsiniz.

### Gereklilikler
- Grafana 7.0
- NodeJS 12.x
- yarn


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

## 5. Panel Pluginleri

Grafana 6.x'ten beri paneller ReactJS [bileşen sınıflarıdır](https://reactjs.org/docs/components-and-props.html). (component)

Grafana 6.0'dan önce pluginler [AngularJS](https://angular.io/) ile yazılmıştı. AngularJS ile yazılan pluginleri hala desteklememize rağmen yeni pluginleri ReactJS ile yazmanızı şiddetle öneriyoruz.

### Panel özellikleri

[PanelProps](https://github.com/grafana/grafana/blob/747b546c260f9a448e2cb56319f796d0301f4bb9/packages/grafana-data/src/types/panel.ts#L27-L40) arayüzü (interface) panel boyutları, zaman aralığı gibi panel bilgileri içerir. 

Panel özelliklerine `props` aracılığıyla ulaşabilirsiniz.

**src/SimplePanel.tsx**

```javascript
const { options, data, width, height } = props;
```

### Geliştirme akışı

Şimdi panelinize değişiklik yapmayı, paneli oluşturmayı ve Grafana'yı yeniden başlatmayı içeren bir akış öğreneceksiniz.

Önce panelinizi dashboard'a eklemeniz gerekmektedir:

1. Grafana'yı tarayıcınızda açın.
2. Yeni bir dashboard oluşturun ve yeni bir panel ekleyin.
3. Görselleştirme tiplerinden sizin panelinizi seçin.
4. Dashboard'u kaydedin.

Şimdi panelinizi gördüğünüze göre panel plugininde değişiklik yapmayı deneyin:

1. `SimplePanel.tsx` dosyasında çemberin `fill` rengini değiştirin.
2. Plugini oluşturmak için `yarn dev` komutunu çalıştırın.
3. Yeni değişikleri görmek için tarayıcınızda Grafana'yı tekrardan çalıştırın.

## 6. Panel seçenekleri ekleyin

Bazen plugininizin davranışı düzenlemek için kullanıcılara seçenek vermek isteyebilirsiniz. Plugininizde panel seçenekleri ekleyerek panelinizin kullanıcı komutlarını kabul etmesini sağlayabilirsiniz.

Önceki adımda kodunuzda çemberin `fill` rengini değiştirmiştiniz. Şimdi kodu kullanıcıların panel editörü kullanarak bu `fill` rengini değiştirmelerine izin verecek şekilde düzenleyelim.

### Bir seçenek ekleyin

Panel seçenekleri bir *panel options objesinde* bulunmaktadır. `SimpleOptions` bu objeyi açıklayan bir arayüzdür.

1. `types.ts` dosyasında kullanıcıların aralarından seçebileceği bir `CircleColor` type'ı ekleyin.

```typescript
type CircleColor = 'red' | 'green' | 'blue';
```
2. `SimpleOptions` arayüzünde `color` adında yeni bir seçenek ekleyin.

```typescript
color: CircleColor;
```

Güncellenmiş `types.ts` şu şekildedir:

**src/types.ts**

```typescript
type SeriesSize = 'sm' | 'md' | 'lg';
type CircleColor = 'red' | 'green' | 'blue';

// interface defining panel options type
export interface SimpleOptions {
  text: string;
  showSeriesCount: boolean;
  seriesCountSize: SeriesSize;
  color: CircleColor;
}
```

### Seçenek kontrolü ekleyin

Seçeneği panel editöründen düzenlemek için `color` seçeneğini bir seçenek kontrolüne bağlamanız gerekmektedir.

Grafana çeşitli seçenek kontrollerini destekler: Yazı girişleri, switchler, radio butonlar...

Hadi bir radio kontrolü yaratalım ve `color` seçeneğine bağlayalım.

1. Kontrolü `src/module.ts` dosyasında builder'in sonuna ekleyin.

```typescript
.addRadio({
  path: 'color',
  name: 'Circle color',
  defaultValue: 'red',
  settings: {
    options: [
      {
        value: 'red',
        label: 'Red',
      },
      {
        value: 'green',
        label: 'Green',
      },
      {
        value: 'blue',
        label: 'Blue',
      },
    ],
  }
});
```

`path`, kontrolü bir seçeneğe bağlamak içindir. Kontrolü iç içe bir seçeneğe tam konumunu göstererek bağlayabilirsiniz. Örnek: `colors.background` .

Grafana sizin için bir seçenek editörü yaratır ve bunu **Display** bölümündeki *panel editor* kısmında gösterir.

### Yeni seçeneği kullanın

Neredeyse bitti. Yeni bir seçenek ve bu seçeneğin değerini değiştirmek için gerekli kontrolü eklediniz. Fakat plugin bu seçeneği hala kullanmıyor. Hadi bunu düzeltelim.

1. Seçenek değerini mevcut tema tarafından kullanılan renkler yerine koymak için `SimplePanel.tsx` dosyasına `return` kısmından önce yeni bir `switch` kısmı ekleyin.

**src/SimplePanel.tsx**

```typescript
let color: string;
switch (options.color) {
  case 'red':
    color = theme.palette.redBase;
    break;
  case 'green':
    color = theme.palette.greenBase;
    break;
  case 'blue':
    color = theme.palette.blue95;
    break;
}
```

2. Çemberi yeni rengi kullanması için değiştirin.

```typescript
<g>
  <circle style={{ fill: color }} r={100} />
</g>
```

Artık panel editörde rengi değiştirdiğinizde çemberin rengininde değiştiğini göreceksiniz. 

## 7. Data frame'leri kullanarak dinamik paneller yaratın

Çoğu panel Grafana data source'dan gelen dinamik veriyi görselleştirir. Bu adımda, her bir serideki sayıyı kullanarak için yarıçapı bu sayı olan yeni çemberler yaratacaksınız.

> Panelinizde veri tabanından gelen verileri kullanmak için data source kurmanız gerekmektedir. 
> Eğer mevcut bir tane yoksa, geçici olarak [TestData DB'i](https://grafana.com/docs/grafana/latest/features/datasources/testdata) kullanabilirsiniz.

Data source'dan gelen veriler panel bileşen sınıfının (component) içindeki `data` değişkeninde bulunur.

```typescript
const { data } = props;
```

`data.series` data source'dan gelen serileri içerir. Her bir seri *data frame* denilen bir veri yapısı şeklinde gösterilir. *Data frame* gelen verileri sütunlara koyup yeni bir tablo oluşturur. Her bir sütundaki veri aynı veri yapısındadır: string, int veya time. 

Aşağıda `Time` ve `Value` sütununa sahip örnek bir *data frame* görebilirsiniz.

Time | Value
------------ | -------------
1589189388597 | 32.4
1589189406480 | 27.2
1589189513721 | 15.0

Hadi bu *data frame*'den nasıl veri çekip görselleştireceğinize bakalım:

1. Her sütundan `number` veri yapısındaki son veriyi alın ve `SimplePanel.tsx` dosyasına `return` kısmından önce aşağıdakini kullanarak ekleyin:

```typescript
const radii = data.series
 .map(series => series.fields.find(field => field.type === 'number'))
 .map(field => field?.values.get(field.values.length - 1));
```

`radii` data source'dan gelen serilerden en sondaki verileri içerecek. Bu verileri her çemberin yarıçapını ayarlamak için kullanabilirsiniz.

2. `svg` elementini aşağıdakine göre değiştirin:

```typescript
<svg
  className={styles.svg}
  width={width}
  height={height}
  xmlns="http://www.w3.org/2000/svg"
  xmlnsXlink="http://www.w3.org/1999/xlink"
  viewBox={`0 -${height / 2} ${width} ${height}`}
>
  <g fill={color}>
    {radii.map((radius, index) => {
      const step = width / radii.length;
      return <circle r={radius} transform={`translate(${index * step + step / 2}, 0)`} />;
    })}
  </g>
</svg>
```

Her `radii` değeri için nasıl `<circle>` elementi oluşturduğumuza dikkat edin:

```typescript
{radii.map((radius, index) => {
  const step = width / radii.length;
  return <circle r={radius} transform={`translate(${index * step + step / 2}, 0)`} />;
})}
```

Oluşan çemberi yatay olarak dağıtmak için `transform`'u kullanıyoruz.

3. Plugininizi yeniden build edin ve dashboard'u yenileyin.

Eğer *data frame*'ler hakkında daha çok bilgi sahibi olmak istiyorsanız [Data frames](https://grafana.com/docs/grafana/latest/developers/plugins/data-frames/) sayfasına gidin.

## Tebrikler!

Grafana'da panel plugin oluşturma rehberimizin sonuna geldiniz.
