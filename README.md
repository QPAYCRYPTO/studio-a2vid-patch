# studio-a2vid-patch

`wlsdml1114/Infinitetalk_Runpod_hub` worker'ının `handler.py` dosyasının
**tek değişiklikli** yamalı kopyası.

- Kaynak: https://github.com/wlsdml1114/Infinitetalk_Runpod_hub
- Temel alınan ref: `skyreelsV3_v0.7_blackwell`
- Lisans: Apache License 2.0 (orijinal projeyle aynı, bkz. `LICENSE`)

## Düzeltilen hata

İki aşamalı SkyReels V3 A2V iş akışı ComfyUI geçmişine **birden fazla çıktı
düğümü** bırakıyor:

| Düğüm | Dosya | İçerik |
|---|---|---|
| 328 | `..._00003.mp4` | ara referans videosu — **81 kare, sessiz** |
| 131 | `..._00004-audio.mp4` | nihai video — **tam kare sayısı, sesli** |

Orijinal `handler.py` "ilk boş olmayan düğümü" seçiyordu ve node 328 önce
geldiği için **ara videoyu** döndürüyordu. Ölçülen sonuç: 4,78 saniyelik ses
için 3,24 saniyelik sessiz video; konuşmanın son 1,5 saniyesi görüntüsüz.

Worker günlüğünden kanıt:

```
Sampling 119 frames in 2 windows, at 720x1280 with 4 steps
노드 328에서 1개의 비디오 발견: ..._00003.mp4       (1.149.405 bytes)
노드 131에서 1개의 비디오 발견: ..._00004-audio.mp4 (1.813.204 bytes)
노드 328에서 출력 비디오 발견                        <-- yanlış seçim
```

## Yama

Çıktı seçimi şu sıraya geçti:

1. adı `-audio.mp4` ile biten adaylar — aralarında **en büyük** dosya,
2. hiç yoksa tüm adaylar arasında en büyük dosya.

Başka hiçbir davranış değişmedi. Seçilen aday ve tüm adaylar `[STUDIO]`
önekiyle loglanır.

## Nasıl kullanılıyor

İmaj yeniden derlenmiyor. Worker'ın tam imajı özel bir registry'de
(`registry.runpod.net`, kimlik doğrulaması istiyor) ve yazarın Docker Hub'ında
yalnızca temel imajlar var; sıfırdan derlemek ~40 GB model indirmek demek,
GitHub Actions'ın disk sınırına sığmaz.

Bu yüzden RunPod Serverless endpoint'inin konteyner komutu, worker açılırken
bu dosyayı `/handler.py` üzerine indirecek şekilde ayarlanmıştır. İndirme
başarısız olursa worker **bilinçli olarak** hata verir — sessizce yanlış
(ara) videoyu döndürmesindense görünür şekilde durması tercih edilmiştir.
