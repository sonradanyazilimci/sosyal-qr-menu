# Sosyal Menü — QR Dijital Menü Sistemi

QR kod ile açılan, video/gradyan karşılama efektli, mobil öncelikli dijital menü. Tamamen statik HTML/CSS/JS dosyalarından oluşur — Node.js, `npm install`, build adımı **gerekmez**. Veri, dosya depolama ve admin girişi [Firebase](https://firebase.google.com) (Firestore + Storage + Authentication) üzerinden yönetilir. Bu sayede proje klasörü sadece birkaç küçük dosyadan oluşur ve doğrudan GitHub'a manuel olarak yüklenebilir.

## Dosya yapısı

```
index.html          → müşteri menüsü
admin.html           → yönetim paneli
css/style.css         → menü tasarımı
css/admin.css         → panel tasarımı
js/menu.js            → menüyü Firestore'dan okur, canlı günceller
js/admin.js           → panel mantığı (giriş, kategori/ürün yönetimi, dosya yükleme)
js/firebase-init.js   → Firebase SDK başlatma (dokunmanıza gerek yok)
js/firebase-config.js → SADECE bu dosyayı kendi Firebase bilgilerinizle doldurun
```

## 1) Firebase projesi kurulumu

1. [console.firebase.google.com](https://console.firebase.google.com) adresinden yeni bir proje oluşturun.
2. Sol menüden **Build > Firestore Database** → *Create database* → herhangi bir bölge seçip **production mode** ile oluşturun.
3. Sol menüden **Build > Storage** → *Get started* → production mode ile oluşturun.
4. Sol menüden **Build > Authentication** → *Get started* → **Sign-in method** sekmesinden **Email/Password** sağlayıcısını etkinleştirin.
5. **Authentication > Users** sekmesinden *Add user* ile kendinize bir yönetici e-posta/şifresi oluşturun (bu bilgilerle admin paneline giriş yapacaksınız).
6. Proje ayarları (dişli ikonu) **> Project settings > General** sekmesinde en altta **"Your apps"** bölümünden *Web* (`</>`) simgesine tıklayıp bir uygulama kaydedin. Size verilen `firebaseConfig` nesnesini kopyalayın.
7. Kopyaladığınız bilgileri `js/firebase-config.js` dosyasındaki `firebaseConfig` içine yapıştırın (tüm `BURAYA_...` yazan yerleri değiştirin).

> `apiKey` gizli bir bilgi değildir, GitHub'a yüklemekten çekinmeyin — güvenlik aşağıdaki kurallarla sağlanır.

## 2) Firestore ve Storage güvenlik kuralları

**Firestore Database > Rules** sekmesine gidip aşağıdakini yapıştırıp yayınlayın:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /config/menu {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

**Storage > Rules** sekmesine gidip aşağıdakini yapıştırıp yayınlayın:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /uploads/{allPaths=**} {
      allow read: if true;
      allow write: if request.auth != null && request.resource.size < 50 * 1024 * 1024;
    }
  }
}
```

Bu kurallar sayesinde menü herkes tarafından görüntülenebilir, ama sadece giriş yapmış (Authentication'a eklediğiniz) yönetici değişiklik yapabilir.

## 3) Yerel test

Tarayıcılar güvenlik nedeniyle ES modüllerini `file://` üzerinden çalıştırmaz, bu yüzden basit bir yerel sunucu ile açın:

```
npx serve .
```

veya Python varsa:

```
python -m http.server 5500
```

Sonra `http://localhost:5500/index.html` ve `http://localhost:5500/admin.html` adreslerini açın.

## 4) GitHub'a yükleme ve yayınlama

1. Bu klasörü (node_modules **yoktur**, tüm dosyalar birkaç KB) olduğu gibi bir GitHub reposuna yükleyin.
2. Repo **Settings > Pages** bölümünden *Branch* olarak `main` (veya kullandığınız dal) ve `/ (root)` klasörünü seçip kaydedin.
3. Birkaç dakika içinde `https://kullaniciadi.github.io/repo-adi/` adresinde menünüz yayında olur.
4. Admin paneline `https://kullaniciadi.github.io/repo-adi/admin.html` üzerinden, Firebase'e eklediğiniz e-posta/şifre ile giriş yapın.

QR kodunuzu bu adrese göre oluşturup masalara yerleştirebilirsiniz.

## Admin panelinden yapılabilecekler

**Ayarlar sekmesi:** işletme adı, handle, alt başlık, para birimi, vurgu rengi, logo, karşılama videosu/fotoğrafı, indirim metni, alerjen bilgisi, şifre değiştirme.

**Menü sekmesi:** kategori → alt kategori → ürün (isim, açıklama, fiyat, kalori, rozet emoji, fotoğraf) ekle/düzenle/sil. Menü boşsa tek tıkla örnek içerik yükleyebilirsiniz.

Yaptığınız her değişiklik Firestore'a anında yazılır ve açık olan müşteri menüsü ekranlarında **otomatik olarak, sayfa yenilenmeden** görünür (canlı güncelleme).

## Notlar

- Karşılama videosu yüklenmezse otomatik olarak hareketli "kor ateşi" gradyan animasyonu gösterilir.
- Yüklenen görsel/videolar Firebase Storage'da `uploads/` altında saklanır; bir dosyayı panelden "kaldır" dediğinizde sadece menüden referansı silinir, dosyanın kendisi Storage'da kalmaya devam eder (isterseniz Firebase Console'dan elle temizleyebilirsiniz).
- Firebase'in ücretsiz (Spark) planı küçük/orta ölçekli bir menü sitesi için fazlasıyla yeterlidir.
