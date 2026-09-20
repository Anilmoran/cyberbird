# Flappy Klon — Cyberpunk

Unity ile geliştirilen dikey 2D flappy bird türevi. Tema, arayüz, boru görselleri ve
müzik sahneye yerleştirilmiş varlıklar yerine çalışma anında kod tarafından üretilir.

| Ana menü | Oyun |
|---|---|
| <img src="docs/menu.png" width="320"> | <img src="docs/gameplay.png" width="320"> |

## Teknik özet

| | |
|---|---|
| Motor | Unity 6000.4.5f1, Built-in pipeline |
| Boyut | 2D, dikey (1080×1920 referans), orthographic size 5 |
| Metin | TextMeshPro, dynamic atlas |
| Kayıt | PlayerPrefs |
| Kapsam | 2 sahne, 32 runtime + 5 editor script |

## Klasör yapısı

```
Assets/
├── scripts/            Oyun mantığı (kuş, boru, skor, menü)
│   └── Cyberpunk/      Tema, arayüz, ses, efektler, ilerleme
├── Editor/             Kurulum araçları, özel Inspector'lar
├── CyberBirds/         Kuş ve arka plan varlıkları + üretilen prefab/animasyonlar
├── Resources/          CyberContent.asset (içerik kataloğu)
└── Scenes/             MainMenu, oyun1
```

## Mimari

**Tema sistemi.** `CyberpunkTheme`, `[RuntimeInitializeOnLoadMethod(AfterSceneLoad)]`
ve `SceneManager.sceneLoaded` üzerinden her sahnede kendini kurar; sahneye bileşen
eklemek gerekmez. Davranış opsiyonel `CyberpunkSettings` bileşeniyle geçersiz kılınır.

**İçerik kataloğu.** Kuşlar, arka planlar ve arayüz fontu
`Assets/Resources/CyberContent.asset` içinde tek listede tutulur, runtime'da
`Resources.Load` ile okunur. Aynı listenin iki sahnede iki ayrı Inspector dizisinde
tutulması, sıralar bozulduğunda menüde seçilen kuş ile oyunda çıkan kuşun
farklılaşmasına yol açar; tek katalog bu bağımlılığı kaldırır.

## Oyun sistemleri

**Animasyon.** 6 kuş × 4 animasyon (Flap 8/14fps, Hit 6/15, Death 10/12, Fall 4/12).
Klipler ve AnimatorController'lar editor scripti ile üretilir. Oyun bittiğinde
`Time.timeScale` sıfırlandığı için Animator `UnscaledTime` moduna alınır; aksi halde
ölüm animasyonu ilk karede donar.

**Boyutlandırma.** Sprite karesi 512×512 / 256 PPU → 2×2 birim, kuş karenin ~%55'ini
doldurur. Ölçek kare üzerinden değil görünen yükseklik üzerinden hesaplanır:

```
görünen = kamera yüksekliği × birdScreenHeightPercent / 100
görünen = min(görünen, boru aralığı × maxGapFill)
ölçek   = (görünen / spriteFillRatio) / kare yüksekliği
```

Çarpışma yarıçapı da görünen yüksekliğe oranlıdır, değerler Play modunda anında uygulanır.

**Borular.** `CartoonPipe` prefabın çocuklarını çalışma anında silip boruyu prosedürel
sprite'lardan yeniden kurar. Gövde ve kapak `SpriteDrawMode.Sliced` ile çizilir, böylece
uzatıldığında kontur kalınlığı bozulmaz. Spawn yüksekliği aralık ve ekran payına göre
kameradan hesaplanır.

**Kredi ve ilerleme.** Kredi kaynakları: aralıktaki çipler (+1) ve tur sonunda `skor / 3`.
Kuş fiyatları 0 / 40 / 80 / 140 / 220 / 320, kilitler bitmask olarak saklanır.

## Prosedürel üretim

**Görsel.** Panel, buton, boru parçaları, kredi çipi ve efekt dokuları koddan üretilir.
Paneller yuvarlatılmış dikdörtgen signed distance function ile çizilip 9-slice border ile
oluşturulur, üretilen sprite'lar cache'lenir.

**Ses.** Müzik yazılım sentezleyici ile üretilir: menü 96 BPM, oyun 128 BPM, 32 vuruşluk
loop, 32 kHz mono, Am–F–C–G ilerleyişi. Kick, clap, hat, bas, acid arp ve pad ayrı
sentezlenip toplanır; çıkışa echo, loop uçlarına crossfade, sonrasında tepe normalizasyonu
ve yumuşak kırpma uygulanır. Üretim arka plan thread'inde yapılır — orada yalnızca `Mathf`
kullanılır, `AudioClip` ana thread'de oluşturulur.

## Arayüz ve efektler

| Bileşen | Kapsam |
|---|---|
| `CyberHUD` | Skor, kredi sayacı, hız göstergesi, duraklatma, tur sonu özeti |
| `CyberMenuUI` | Kredi, rekor, kilit durumu, müzik/efekt düğmeleri |
| `CyberUISkin` | Sahnedeki mevcut butonların yeniden boyanması |
| `NeonOverlay` | Tarama çizgileri, ızgara, vignette, aralıklı parazit |
| `NeonTrail` | Kanat çırpışında kuş silueti hayaleti, ince neon iz |
| `CameraJuice` | Skorda zoom vuruşu, ölümde sarsıntı ve flash |

Arayüz `ScaleWithScreenSize` canvas üzerinde kurulur, ölçüler ekran yüzdesi olarak verilir.
Overlay katmanları raycast almaz. `CameraJuice` ve parazit efektleri `Time.timeScale`
sıfırken de çalışması için unscaled zaman kullanır.

Ekranda görünen tüm yazılar `CyberText.cs` içinde tutulur. Kontur ve parlama
`text.fontMaterial` üzerinden verilir — dynamic font asset'lerinde materyalin elle
kopyalanması, atlas büyüdüğünde kopyanın eski dokuya bakmasına yol açar.

## Editor araçları

| Menü | İşlev |
|---|---|
| `Oyuna Bagla (tam kurulum)` | Animasyon ve prefab üretimi, katalog, sahne bağlantıları |
| `Icerik Listesini Yenile` | Yalnızca kataloğu günceller |
| `Olculeri Yazdir` | Sprite alfa sınırı ve boru aralığı ölçümü |

`BackgroundManager` için özel Inspector, arka planı sayı yerine isimle seçtirir.

## Kurulum

Depo Unity Hub üzerinden **6000.4.5f1** ile açılır; scriptler derlendikten sonra animasyon,
prefab ve katalog otomatik üretilir. Üretim tetiklenmezse
`Tools > Cyber Birds > Oyuna Bagla (tam kurulum)` çalıştırılır.

## Varlıklar

Kuş ve arka plan görselleri `Assets/CyberBirds` altındadır, font Luckiest Guy. Üçüncü taraf
varlıkların lisans koşulları depo yayınlanmadan önce kontrol edilmelidir. Panel, buton,
boru, kredi çipi görselleri ile müzik ve efekt sesleri koddan üretilir.
