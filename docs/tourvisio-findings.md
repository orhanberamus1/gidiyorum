# TourVisio dokümantasyon bulguları

> English version: [tourvisio-findings.en.md](tourvisio-findings.en.md)

Planın dayandığı bilgiler. README'de iddialar **[DOC]** (dokümanda yazıyor), **[SAMPLE]** (yalnızca örnek JSON'da görülüyor, sözleşme sayılmıyor) veya **[Q-n]** (dokümanda yok, bkz. [açık sorular](../README.md#tourvisioya-sorular)) olarak işaretlidir.

## Dokümandaki bilgiler

| # | Bulgu | Kaynak |
|---|---|---|
| D1 | **Yalnızca backend → TourVisio** login: `POST /api/authenticationservice/login`. Girdi `Agency`, `User`, `Password`. Çıktı `token`, `expiresOn`, `tokenId`, `userInfo`. Next.js → backend auth bizim tasarımımız | [Authentication](https://docs.santsg.com/tourvisio/authentication/) |
| D2 | Token `Authorization: Bearer` ile gönderilir. Token ömrü belirtilmemiş. **TourVisio'da refresh token yok**, oturum ancak yeniden `login` çağrılarak uzatılabilir | [Authentication](https://docs.santsg.com/tourvisio/authentication/) |
| D3 | `POST /api/productservice/getdepartureautocomplete` ve `getarrivalautocomplete`. Varış çağrısı `DepartureLocations` alır | [Flight Product](https://docs.santsg.com/tourvisio/flightproduct/) |
| D4 | Autocomplete `type` ile arama `type` farklı enum'lar. Havalimanı 3 → 5, şehir 1 → 2 eşlenir | [Enumerations](https://docs.santsg.com/tourvisio/enumarations/) |
| D5 | `POST /api/productservice/pricesearch`. Alternatif `flightpricesearch` "Optional - To improve performance" olarak işaretli | [Flight Product](https://docs.santsg.com/tourvisio/flightproduct/) |
| D6 | `ServiceTypes`: `"1"` tek yön, `"2"` gidiş-dönüş. Gidiş-dönüş `CheckIn` + `Night` kullanır. Önerilen örnek `supportedFlightReponseListTypes: [3]` gönderir (yazım dokümandaki gibi) | [Flight Product](https://docs.santsg.com/tourvisio/flightproduct/) |
| D7 | Yanıt `body.searchId`, `body.expiresOn` ve `body.flights[]` içerir. **`offers[]` her `flight` nesnesinin içindedir** ve kendi `expiresOn` alanı vardır | [Flight Product](https://docs.santsg.com/tourvisio/flightproduct/) |
| D8 | Gidiş-dönüşte gidiş ve dönüş teklifleri `groupKeys` kesişimiyle eşleştirilir, doğru `offerId` `offerIds[]` içinden seçilir | [Flight Product](https://docs.santsg.com/tourvisio/flightproduct/) |
| D9 | Toplam fiyat kuralı tanımlı: paket tekliflerde yüksek olan fiyat, paket olmayanlarda toplam, ardından `fees` kuralı | [Flight Product](https://docs.santsg.com/tourvisio/flightproduct/) |
| D10 | `POST /api/productservice/getoffers` opsiyonel, `searchId` + `offerIds` alır | [Flight Product](https://docs.santsg.com/tourvisio/flightproduct/) |
| D11 | `POST /api/flightservice/getfarerules` `transactionId` veya `reservationNumber` ile `serviceId` + `segment.id` ister. "Third party flight (SanFlight)" servisleri için tanımlı | [Flight Service](https://docs.santsg.com/tourvisio/flightservice/) |
| D12 | Yanıt yapısı: `header { requestId, success, messages[] }` + `body`. Mesaj tipleri: 1 Error, 2 Success, 3 Information, 4 Warning | [General](https://docs.santsg.com/tourvisio/general/) |

## Eksikler ve çelişkiler

- **Çelişki:** Authentication sayfası token'ın her istekte gönderilmesi gerektiğini söylüyor, ama `pricesearch` tablosunda "Token Required: NO" yazıyor. Plan her çağrıda token gönderir (Q-2).
- **Belgelenmemiş:** rate limit, kota, timeout, retry, hata kodu listesi ve sunucu tarafı sayfalama. `enablePaging: false` yanıtlarda açıklamasız görünüyor.
- **[SAMPLE] Ömür:** iki arama örneğinde `expiresOn`, `responseTime`'dan yaklaşık 210 dakika sonra. Bu değere güvenmiyoruz, her yanıttan okuyoruz.
- **[SAMPLE] Teklif ömrü:** iki tam arama örneğinde de her teklifin `expiresOn`'u `body.expiresOn` ile aynı. Farklı olup olamayacakları belirtilmemiş, plan iki durumu da ele alır (Q-4).
- **[SAMPLE] Fiyat:** örnek fiyatlar, yolcu sayıları ve para birimleri birbiriyle uyuşmuyor. Örnekleri, değerleri temsilî olan alan örnekleri olarak kabul ediyoruz (Q-11).
- **[SAMPLE] Tarihler:** bazı örnek tarihlerde saat dilimi yok, bazıları `Z` ile bitiyor. Saat filtresi saat dilimi dönüşümü yapmaz.
- **[SAMPLE] Aktarma:** bir örnekte tek segmentli, 75 dakikalık bir uçuş `stopCount: 1` gösteriyor. Aktarma sayısının anlamı teyit edilmeli (Q-12).
- **[SAMPLE] Aktarmasız bayrağı:** arama örnekleri `showOnlyNonStopFlight: false` gönderiyor, ama doküman alanı veya varsayılan değerini açıklamıyor (Q-16).
