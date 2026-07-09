# TS EDGE PRO — Kurulum Rehberi

Kurulum bir kez yapılır, yaklaşık 10 dakika sürer. Sonrasında her şey otomatiktir ve telefonundan tek linkle takip edersin.

## 1. GitHub hesabı aç
- Bilgisayarında github.com adresine git, "Sign up" ile ücretsiz hesap aç.

## 2. Yeni depo (repository) oluştur
- Sağ üstteki **+** işaretine bas → **New repository**
- Repository name: `ts-edge` yaz
- **Public** seçili kalsın (Pages'in ücretsiz çalışması için gerekli)
- **Create repository** butonuna bas

## 3. Dosyaları yükle
- Açılan sayfada **uploading an existing file** linkine tıkla
- Bu klasördeki TÜM dosyaları (tarama.js, KURULUM.md, docs klasörü,
  .github klasörü) pencereye sürükle-bırak
- ÖNEMLİ: `.github` gizli klasör olduğu için sürüklemede gelmeyebilir.
  Gelmezse: depoda **Add file → Create new file** de, dosya adı olarak
  `.github/workflows/tarama.yml` yaz (eğik çizgiler klasör oluşturur),
  içine bu paketteki tarama.yml içeriğini yapıştır.
- Alt taraftaki **Commit changes** butonuna bas

## 4. GitHub Pages'i aç
- Depoda **Settings → Pages** bölümüne gir
- "Source": **Deploy from a branch**
- "Branch": **main** seç, klasör olarak **/docs** seç → **Save**
- 1-2 dakika sonra sayfanın üstünde adresin görünür:
  `https://KULLANICIADIN.github.io/ts-edge/`

## 5. İlk taramayı elle başlat
- Depoda **Actions** sekmesine gir
- İlk girişte "enable workflows" sorarsa onayla
- Soldan **TS EDGE Tarama** seç → sağda **Run workflow** → yeşil butona bas
- 2-3 dakikada biter; bitince panel linkinde sonuçlar görünür

## 6. Telefonuna ekle
- Telefonunda Chrome ile panel linkini aç
- Menü (⋮) → **Ana ekrana ekle**
- Artık uygulama simgesi gibi tek dokunuşla açılır

## Nasıl çalışır?
- Hafta içi 09:00-21:00 arası her 30 dakikada bir otomatik tarama yapılır
- Panel her açılışta en son taramanın sonucunu gösterir
- Veriler Yahoo Finance kaynaklıdır ve 15 dk civarı gecikmelidir

## Sık sorulanlar
**Ücret çıkar mı?** Hayır. GitHub Actions ve Pages, herkese açık depolarda
ücretsizdir; bu tarama aylık ücretsiz limitin çok altında kalır.

**Tarama sıklığını değiştirmek?** `.github/workflows/tarama.yml` içindeki
cron satırını düzenle. Örn. saatte bir için: `0 6-18 * * 1-5`

**Sembol eklemek/çıkarmak?** `tarama.js` dosyasının başındaki BIST30
listesini düzenle, Commit'le. Sonraki taramada geçerli olur.

**Uyarı:** Bu sistem sinyal üretir; yatırım tavsiyesi değildir.
