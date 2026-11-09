# Akor Cebi — proje notları

Bu dosya, claude.ai'de başlayan sohbetin devamı için yazıldı. Claude Code bu klasörde açıldığında bunu otomatik okur. Kullanıcıyla **Türkçe** konuş.

## Ne bu?
Gitar çalanlar için tek dosyalık mobil web uygulaması: `index.html` (HTML + CSS + JS, harici kütüphane yok, internet gerekmez). Tasarım hedefi: **iPhone'un kendi uygulamaları gibi sade ve kolay** (sistem yazı tipi, gruplu listeler, büyük başlıklar, mavi vurgu, bulanık alt çubuk). Kullanıcı için en önemli şey kullanım kolaylığı.

Çalıştırmak için: `index.html` dosyasını tarayıcıda aç. Telefonda denemek ya da mikrofon kullanmak için yerel sunucu: `python3 -m http.server 8000` → `http://localhost:8000` (aynı ağdaki telefondan bilgisayarın IP'si ile; mikrofon için HTTPS ya da localhost gerekir).

## Ekranlar (alt çubukta 4 sekme)
1. **Akort** — Çalgı seçici (yazısız, küçük kafa ikonları kaydırılarak): 3+3 burgu (Gibson tarzı), tek sıra 6 burgu (Fender tarzı), 12 telli (6+6), 7 telli, Bas 4, Bas 5, Ukulele. Altında akort düzeni şeridi (Standart, Drop D, DADGAD…). Ekranda kafa + burgular SVG olarak çizilir; burguya dokununca o telin referans sesi çalar (sinüs + harmonikler). A4 referansı 430–450 Hz. **Mikrofon yok** (claude.ai artifact ortamı izin vermiyordu).
2. **Akorlar** — Büyük kök nota şeridi (kaydır), altında akor türü şeridi (Majör, Minör, 7, maj7…), notalar, akor şeması (tellerin üstünden parmakla sürükleyince tel tel çalar), şemanın altında şekil şeridi (Açık, 3. perde…). Üstte "Son" akorlar.
3. **Ton** — Kök + gam türü şeritleri (Penta minör/majör, Blues, Majör, Minör, Dorian, Mixolydian, Harmonik minör). **Arka plan müziği** kartı: ilerleme + tarz (Pop, Rock, Balad, Blues shuffle) seçip çal; davul/bas/gitar Web Audio ile sentezlenir, 1 ölçü sayım, anlık akor mavi yanar, klavyede o akorun notaları öne çıkar. Küçük klavye (12 perde) + pozisyonlar + Nota/Derece; **Tam ekran** yatay gerçekçi klavye (15 perde, gerçek perde aralıkları; dikey tutulunca CSS ile 90° döner). Tonun diatonik akorları (+7'li) ve popüler ilerlemeler ("Arkada çal" düğmesi).
4. **Şarkılar** — Şarkı defteri (localStorage). Söz içinde `[Am]Akşam iner` biçimi; sadece akordan oluşan satırlar düğme olur. Ton değiştirme, kapo (öneriyle), şarkıya özel tempo, otomatik kaydırma. Akora dokununca alt sayfada şema açılır.

Her ekranda: alt çubuğun üstünde **mini metronom** şeridi (dokununca tam metronom alt sayfası: BPM, 2/4 3/4 4/4 6/8, tempo vur, **ritim kalıpları** ↓↑ — istenirse seçili akorla çalar). Arka plan müziği ile metronom aynı anda çalmaz. Sağ üstte **dişli → Ayarlar**: Do Re Mi yazımı, bemol, solak modu (şema/klavye aynalanır), ekran açık kalsın (Wake Lock). Her ekranda bir kez gösterilen kısa ipuçları ("Tamam" ile kapanır).

## Kod yapısı (`index.html`)
- `<style>`: renkler `:root` belirteçleri; koyu tema `prefers-color-scheme` + `[data-theme]` blokları. Yeni renkleri hep belirteç olarak ekle.
- 1. `<script>` **müzik mantığı** (DOM'a dokunmaz, Node'da test edilebilir): `NAMES`, `state`, `QT` (akor türleri), `SH` (taşınabilir şekiller), `OPENV` (açık akorlar), `voicings()`, `parseTok()` (akor adı ayrıştırıcı, Do/Re/Mi destekli), `SCALES`, `positions()`, `diatonic()`, `PROGS`, `INST` (çalgılar + akortlar), `SIGS`, `PATS` (ritim kalıpları), `STYLES` (arka plan tarzları).
- 2. `<script>` **arayüz**: `save()/load()` (localStorage, anahtar `akorcebi`, hepsi try/catch içinde), kaydırmalı seçici bileşeni (`carousel()`, `swipeHead()`, `layoutCar()`, `settle()`, `CARS` tablosu, `stepRoot()`, pointer olayları), `diagram()`, `board()`, `headSVG()`, `instIcon()`, `render*()` fonksiyonları, ses motoru (`ensureAudio`, iOS sessiz mod kilidi `unlock`, `pluckAt`), akort tonu, metronom (`metroTick` 25 ms zamanlayıcı + ileriye planlama), arka plan (`btTick`, `kick/snare/hat/bassNote`, `showBar`), şarkılar, alt sayfa (`openSheet/renderSheet`), tek bir `click` dinleyicisi (`data-act` ile `switch`).
- Yeni bir kaydırmalı seçici eklemek: `carousel('anahtar', p=>etiket, 'mini'|'sm'|'icons')` + `CARS.anahtar = {get,set,n,wrap,render}`.

## Kullanıcının tercihleri (sohbetten)
- Sade, iOS stok uygulama hissi; kalabalık yazı istemiyor (ör. çalgı adlarını ikonla değiştirdik).
- Seçimler **kaydırmalı** olsun; komşu öğeler silik görünsün; geçişler yumuşak (parmağı takip, bırakınca oturma).
- Önemli kontroller yukarıda ve görünür olsun, aşağıda kaybolmasın.
- Akor şemasında "Çal" düğmesi yerine tellere parmakla sürterek çalma.

## Sıradaki işler (kullanıcıyla konuşulan, henüz yapılmadı)
- **Mikrofonlu otomatik akort** (kullanıcı istedi): "Otomatik" (çalınan teli kendisi bulur) ve "Manuel" (teli kendin seç) iki mod. Artık yerelde çalıştığı için `getUserMedia` + perde algılama (ör. autocorrelation / YIN) ile yapılabilir; ibre/sent göstergesi, yeşil = tam. Mikrofon yoksa mevcut referans ses modu kalmalı.
- **Sırayla akort modu**: teller 6→1 sırayla birkaç saniye çalıp geçsin (eller serbest).
- **Sahne modu** (şarkıda büyük yazı, dokununca kaydırma), **akor arama** ("F#m7" yaz), **favori akorlar**, **tek elle kullanım** (ana kontroller altta).
- Ters akor bulucu, akor geçiş alıştırması (1 dakika sayaç + ilerleme), hızlanan metronom, klavye nota oyunu, kulak eğitimi.

## Test yaklaşımı
- Mantık: 1. script'i Node'da çalıştırıp kontrol et (ör. tüm akor şekillerinin sadece akor notalarını içermesi, diatonik akorların gamın içinde kalması, ritim kalıbı uzunlukları).
- Görünüm: Playwright ile 390×844 (telefon) ve 844×390 (yatay) ekran görüntüsü; yatay taşma (`scrollWidth`) ve konsol hatası kontrolü. Sentetik sesler insan kulağıyla denenmeli.
