# Buildinfo Motoru — Canlı Test Rehberi

Bu klasör, sistemi sıfırdan GitHub üzerinde denemen için hazırlandı. Aşağıdaki adımları sırasıyla uygula.

## 1) GitHub'da Yeni Repo Oluştur

- GitHub'da yeni, **boş** bir repo oluştur (README eklemeden, .gitignore eklemeden — tamamen boş).
- Repoyu henüz push etme, sadece oluştur.

## 2) Bu Klasördeki Dosyaları Reponun İçine Koy

Bu paket şu dosyaları içeriyor:

```
.buildinfo.yaml
.github/workflows/buildinfo.yml
src/test_servis/hello.go
```

Bunları klonladığın (veya yeni oluşturduğun) reponun **kök dizinine** kopyala. Klasör yapısı birebir aynı kalmalı (özellikle `.github/workflows/` — isim esnekliği yok).

## 3) Kendi `.exe` Dosyanı Ekle

Elindeki `buildinfo-gen.exe` dosyasını da **aynı kök dizine** (diğer dosyalarla yan yana) koy. Son yapı şöyle görünmeli:

```
reponun-adi/
├── .buildinfo.yaml
├── .github/
│   └── workflows/
│       └── buildinfo.yml
├── buildinfo-gen.exe   ← senin eklediğin
└── src/
    └── test_servis/
        └── hello.go
```

## 4) İlk Commit ve Push

Terminalde repo klasörünün içindeyken:

```bash
git add .
git commit -m "chore: buildinfo test kurulumu"
git push -u origin main
```

> Eğer reponun ana branch'i `main` değil `master` ise komutu ona göre değiştir (workflow zaten hem `main` hem `master`'ı dinliyor, bu yüzden sorun olmaz).

## 5) GitHub Actions'ı İzle

- Reponun GitHub sayfasında **Actions** sekmesine git.
- "Buildinfo Otonom Raporlama" adında bir workflow'un otomatik başladığını göreceksin (sarı/turuncu = çalışıyor, yeşil = başarılı, kırmızı = hata).
- Üzerine tıklayıp adım adım logları izleyebilirsin — hangi adımda ne olduğunu (`checkout`, `Buildinfo Aracını Ateşle`, `Raporu Repoya Kaydet`) canlı görürsün.

## 6) Sonucu Kontrol Et

Workflow yeşil (başarılı) bittiyse:

- Reponun ana sayfasına dön, dosya listesine bak.
- `build-manifest.json` adında **yeni bir dosyanın otomatik olarak eklendiğini** görmelisin (bot tarafından atılan yeni bir commit ile).
- `.last-build-tag` adında başka bir dosya daha oluşmuş olmalı.
- `build-manifest.json` içeriğini aç: `hello.go` dosyasının `src/test_servis` altında olduğu için `test_servis` binary'si altında, `risk: "low"` ile raporlandığını görmelisin (`.buildinfo.yaml`'daki kuralla eşleşiyor).

## 7) İkinci Bir Değişiklikle Farkı Test Et

Sistemin gerçekten "sadece farkı" işlediğini görmek için:

```bash
echo 'fmt.Println("ikinci degisiklik")' >> src/test_servis/hello.go
git add src/test_servis/hello.go
git commit -m "test: ikinci degisiklik"
git push
```

Actions tekrar tetiklenecek, ama bu sefer `build-manifest.json` içindeki `previous_commit` alanı bir önceki (ilk) commit'i, `commit` alanı ise bu yeni commit'i gösterecek — yani sistem `.last-build-tag` sayesinde "kaldığı yerden" devam ediyor demektir.

---

## Bir Şeyler Ters Giderse

| Belirti | Muhtemel Sebep |
|---|---|
| Actions sekmesinde hiç workflow görünmüyor | `.github/workflows/buildinfo.yml` yanlış klasörde veya isim hatalı |
| Workflow kırmızı (hata) veriyor, "command not found" gibi bir hata | `buildinfo-gen.exe` kök dizine eklenmemiş veya adı farklı yazılmış |
| Workflow başarılı ama repoya yeni commit gelmiyor | `permissions: contents: write` eksik olabilir, ya da repo ayarlarında Actions'ın yazma izni kapalı olabilir (Settings → Actions → General → Workflow permissions → "Read and write permissions" seçili mi kontrol et) |
| `build-manifest.json` üretildi ama `test_servis` boş/eksik görünüyor | `.buildinfo.yaml`'daki `path_prefix` ile dosya yolu eşleşmiyor olabilir, yolları tekrar kontrol et |
