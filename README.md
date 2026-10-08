# Koza İpek Katalog — v6 (Firebase)

v5 ile aynı site, veriler Firebase'de:

- **Firestore**: ürünler, site bilgileri, siparişler
- **Authentication**: yönetici girişi (e-posta + şifre)
- **Storage**: ürün ve banner fotoğrafları (Blaze planı gerekir)
- **Hosting**: sitenin yayınlandığı yer → https://kozaipek-katalog.web.app

Sepet ve müşteri bilgileri yine sadece müşterinin tarayıcısında kalır.

## Değişen dosyalar (v5'e göre)

```
js/depo.js              Firestore okuma/yazma, fotoğraf yükleme, sipariş kaydetme
js/ayarlar.js           FIREBASE_AYARLARI ve canlı site adresi eklendi
js/yonetim.js           Gerçek giriş, çıkış, fotoğraflar Storage'a, "Yedek indir"
js/siparis-formu.js     Sipariş Firestore'a yazılır, link sadece kimliği taşır (#siparis=ID)
js/siparis-sayfasi.js   #siparis=ID linkini Firestore'dan açar (eski #s= linkleri de açılır)
js/yonlendirme.js       #siparis=ID adresi eklendi
js/uygulama.js          Firebase'i başlatır; önbellekle anında açar, sonra Firestore'dan günceller
index.html              Firebase kütüphaneleri, e-posta alanı, Çıkış butonu
firebase.json, .firebaserc, firestore.rules, storage.rules   Firebase ayarları
```

## İlk kurulum (bir kez)

**Firebase konsolunda**
1. **Build → Authentication → Başlayın** → Sign-in method → **E-posta/Şifre**'yi aç.
   Users sekmesi → **Kullanıcı ekle** (yönetici e-postası + şifre). **User UID**'yi kopyala.
2. **Build → Firestore Database → Veritabanı oluştur** → konum `eur3` veya `europe-west3` → Production mode.
3. Firestore'da **Koleksiyon başlat** → koleksiyon kimliği `adminler` → belge kimliği = **kopyaladığın UID** →
   bir alan ekle (ör. `ad` = `Yönetici`) → Kaydet. Bu belge o hesabı yönetici yapar.
4. (Blaze'e geçince) **Build → Storage → Başlayın** → konum `us-central1` → Production mode.

**Bilgisayarında** (Node.js kurulu olmalı)
```
npm install -g firebase-tools
firebase login
cd v6
firebase deploy --only hosting,firestore
```
Storage açıldıktan sonra: `firebase deploy` (hepsini yayınlar).

5. Siteyi aç → en alttaki **Yönetici girişi** → giriş yap.
   Veritabanı boşsa örnek ürünleri yüklemeyi önerir.

## Güncelleme yayınlamak

Dosyaları değiştir, sonra `v6` klasöründe: `firebase deploy --only hosting`

## Blaze planı olmadan (Spark) ne çalışır

- Katalog, sepet, sipariş, sipariş linki, yönetici girişi, ürün/banner/site düzenleme: **çalışır**.
- Fotoğraf yükleme: **çalışmaz**. Yüklemeye çalışınca fotoğrafın internet adresini yapıştırmanız istenir
  (ör. kozaipek.net medya kütüphanesindeki bir fotoğrafın adresi).

## Güvenlik

- `firebase.json` içindeki apiKey gizli değildir; erişimi `firestore.rules` ve `storage.rules` belirler.
- Ürün ve site bilgisini sadece `adminler` listesindeki hesaplar değiştirebilir.
- Siparişler listelenemez; sadece linki (rastgele 20 karakterlik kimlik) bilen açabilir.
- Sipariş oluştururken kurallar alanları, adet (en az 15) ve kalem sayısını kontrol eder.
  **Fiyatlar tarayıcıda hesaplandığı için kurallarla doğrulanamaz.** Siparişi WhatsApp'ta onaylarken
  tutarı kontrol edin. Kesin çözüm, Blaze ile siparişi bir Cloud Function'ın oluşturması.

## Firestore veri yapısı

```
urunler/{id}       { id, ad, kat, fiyat, indirimli, stok, desen, desenler[], yayin, aciklama,
                     varyantlar: [{ ad, hex, img }], sira }
site/genel         { marka, waNo, telefon, ..., banners: [...] }
siparisler/{id}    { no, t, m: [ad, soyad, tel, adres, il, ilce, odeme],
                     i: [{ p, vi, ad, de, rn, hx, q, b, img }], a, f: [{ ad, tutar }], g,
                     adet, durum: 'yeni', olusturma }
adminler/{uid}     { ad }
```

Siparişleri şimdilik Firebase konsolu → Firestore → `siparisler` koleksiyonundan görebilirsiniz.
