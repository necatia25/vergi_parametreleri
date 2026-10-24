# vergi_parametreleri

**Vergi Hesaplama** uygulamasının yıllık vergi parametreleri. Uygulama bu dosyayı arka planda çeker; yeni yıl eklemek için Play Store sürümü gerekmez.

`kira_takip` / `TUFE_Rates` ile aynı desen — orada TÜFE oranları, burada vergi parametreleri. Fark şu: TÜFE'yi TCMB EVDS'den bir script otomatik çekiyordu. Vergi parametreleri için böyle bir API **yok**; tebliğ PDF olarak yayımlanıyor ve elle girilmesi gerekiyor. Yılda bir kez, yaklaşık on dakika.

## Depo herkese açık olmalı

Uygulama kimlik doğrulaması olmadan `raw.githubusercontent.com` üzerinden okuyor. Depo private olursa güncelleme sessizce başarısız olur ve uygulama gömülü sürüme düşer — çöker değil, ama yeni yıl görünmez.

## Her Aralık ayında yapılacaklar

Gelir Vergisi Genel Tebliği genelde **Aralık ayının son günlerinde** Resmî Gazete'de yayımlanır.

1. `gelir_vergisi.json` içine yeni bir yıl bloğu ekleyin (mevcut bir yılı kopyalayıp değiştirmek en kolayı)
2. Kök seviyedeki **`surum`** alanını bir artırın — uygulama güncellemeyi bununla algılıyor
3. `guncellenme` tarihini bugüne çekin
4. Yeni yılı **`"durum": "taslak"`** ile ekleyin; doğrulayana kadar uygulama kullanıcıyı uyarır
5. Aşağıdaki listeyi tebliğle karşılaştırın, hepsi tuttuğunda `"dogrulandi"` yapın
6. Commit + push. Kullanıcılar bir sonraki açılışta yeni yılı görür.

## Doğrulama listesi

Her alan resmî bir kaynaktan teyit edilmelidir. Vergi hesabında tahmin edilmiş bir rakam, eksik bir yıldan daha kötüdür.

| Alan | Nereden |
|---|---|
| `tarife.ucretDisi` · `tarife.ucret` | GVK md. 103 · yıllık tebliğ. Ücret ve ücret dışı 3. dilimler **farklıdır**, ikisini de girin |
| `gmsi.konutIstisnasi` | GVK md. 21 · tebliğ |
| `gmsi.istisnaGelirSiniri` | Ücret tarifesinin 3. dilim tutarı |
| `gmsi.stopajliBeyanSiniri` | GVK md. 86/1-c · genelde tarifenin 2. dilimi |
| `gmsi.tevkifatsizBeyanSiniri` | GVK md. 86/1-d · tebliğ |
| `gmsi.goturuGiderOrani` | GVK md. 74. **2017'den önce 0.25, 2017'den itibaren 0.15** (7061 sayılı Kanun) |
| `gmsi.isyeriStopajOrani` | GVK md. 94/5-a. Genelde 0.20, ama **2020-2021 arasında Cumhurbaşkanı Kararı ile geçici olarak düşürülmüştü** — o yıllar için karar tarihlerini kontrol edin |
| `gmsi.konutFaizGideriIndirilebilir` | 2025 öncesi `true`, 2025 ve sonrası `false` (7566 sayılı Kanun md. 2). İşyeri faizi her yıl indirilebilir |
| `indirimTavanlari.sahisSigortaYillikAsgariUcret` | Yıllık brüt asgari ücret. Bilinmiyorsa `null` bırakın — yalnızca oran tavanı uygulanır |

Doğrulama kaynakları: `gib.gov.tr` yıllık tarife sayfaları, Resmî Gazete tebliğ metni, PwC / EY / PKF yıllık vergi rehberleri.

## Geçmiş yıllar

Uygulama şu an yalnızca tebliğle birebir doğrulanmış yılları içeriyor. Geriye dönük yıl eklemek isterseniz iki şeye dikkat:

- **Zamanaşımı.** Vergi Usul Kanunu md. 114'e göre tarh zamanaşımı 5 yıl. Beş yıldan eski bir beyan için düzeltme beyannamesi verilemez, vergi dairesi de tarhiyat yapamaz. Pratikte son 5-6 yıl dışındaki hesaplamalar yalnızca meraka hizmet eder.
- **2005 para birimi reformu.** 1 Ocak 2005'te altı sıfır atıldı. 2004 ve öncesi tutarlar eski Türk Lirası cinsindendir; motorun kuruş tabanlı tam sayı mantığı bu ölçekte taşar. Bu yıllar için ayrı bir dönüşüm katmanı gerekir.

## Dosya biçimi

```jsonc
{
  "surum": 1,              // her degisiklikte artir
  "guncellenme": "2026-08-26",
  "yillar": {
    "2025": {
      "tebligNo": "329",
      "taksitAylari": [3, 7],
      "durum": "dogrulandi",  // veya "taslak"
      "tarife": { "ucretDisi": [...], "ucret": [...] },
      "gmsi": { ... },
      "indirimTavanlari": { ... }
    }
  }
}
```

Tarife dilimlerinde son dilimin `ustSinir` değeri `null` olmalıdır (açık uçlu dilim).

## Değişiklikten sonra

Uygulamanın `packages/vergi_core/lib/src/params/gomulu_parametreler.dart` dosyasındaki gömülü kopyayı da bir sonraki sürümde güncelleyin. Gömülü sürüm, interneti olmayan veya uygulamayı ilk kez açan kullanıcılar için son çare.
