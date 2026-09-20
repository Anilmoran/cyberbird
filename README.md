# Flappy Klon — Cyberpunk

Unity ile geliştirilen dikey 2D flappy bird türevi. Tema, arayüz, boru görselleri ve
müzik çalışma anında kod tarafından üretilir.

| Ana menü | Oyun |
|---|---|
| <img src="docs/menu.png" width="300"> | <img src="docs/gameplay.png" width="300"> |

## Teknik özet

| | |
|---|---|
| Motor | Unity 6000.4.5f1, Built-in pipeline |
| Boyut | 2D dikey, 1080×1920 referans, orthographic size 5 |
| Metin | TextMeshPro, dynamic atlas |
| Kayıt | PlayerPrefs |
| Kapsam | 2 sahne, 32 runtime + 5 editor script |

## Mimari

- **Tema sistemi.** `CyberpunkTheme`, `[RuntimeInitializeOnLoadMethod]` ve
  `SceneManager.sceneLoaded` üzerinden her sahnede kendini kurar; sahneye bileşen
  eklemek gerekmez.
- **İçerik kataloğu.** Kuşlar, arka planlar ve font tek bir ScriptableObject'te
  (`Resources/CyberContent.asset`) tutulur. Aynı listenin iki sahnede iki ayrı Inspector
  dizisinde durması, sıralar bozulduğunda seçilen kuş ile oyundaki kuşu ayırıyordu.

## Oyun sistemleri

- **Animasyon.** 6 kuş × 4 animasyon (Flap / Hit / Death / Fall). Klipler ve
  AnimatorController'lar editor scripti ile üretilir. Oyun bitince `Time.timeScale`
  sıfırlandığı için Animator `UnscaledTime` moduna alınır.
- **Boyutlandırma.** Kuş 512×512 karenin ~%55'ini doldurduğundan ölçek kare üzerinden
  değil görünen yükseklik üzerinden hesaplanır; boru aralığına göre üst sınırlanır.
  Çarpışma yarıçapı da aynı orana bağlıdır.
- **Borular.** `CartoonPipe` prefabın çocuklarını çalışma anında silip boruyu prosedürel
  sprite'lardan kurar. `SpriteDrawMode.Sliced` ile uzatıldığında kontur bozulmaz.
- **İlerleme.** Kredi: aralıktaki çipler ve tur sonunda `skor / 3`. Kuş fiyatları
  0 / 40 / 80 / 140 / 220 / 320, kilitler bitmask olarak saklanır.

## Prosedürel üretim

- **Görsel.** Panel, buton, boru ve efekt dokuları koddan üretilir. Paneller
  yuvarlatılmış dikdörtgen signed distance function ile çizilip 9-slice border ile
  oluşturulur.
- **Ses.** Yazılım sentezleyici: menü 96 BPM, oyun 128 BPM, 32 vuruşluk loop, 32 kHz mono.
  Kick, clap, hat, bas, acid arp ve pad ayrı sentezlenip toplanır; echo, crossfade,
  normalizasyon uygulanır. Üretim arka plan thread'inde, `AudioClip` ana thread'de.

## Bileşenler

| | |
|---|---|
| `CyberHUD` / `CyberMenuUI` | Skor, kredi, duraklatma, tur sonu / kilit ve ses düğmeleri |
| `CyberUISkin` | Sahnedeki mevcut butonların yeniden boyanması |
| `NeonOverlay` | Tarama çizgileri, ızgara, vignette, parazit |
| `NeonTrail` | Kanat çırpışında kuş silueti hayaleti, neon iz |
| `CameraJuice` | Skorda zoom vuruşu, ölümde sarsıntı ve flash |
| `CyberText` | Ekranda görünen tüm yazılar |

## Kurulum

Unity Hub üzerinden **6000.4.5f1** ile açılır; scriptler derlendikten sonra animasyon,
prefab ve katalog otomatik üretilir. Tetiklenmezse `Tools > Cyber Birds > Oyuna Bagla`.
