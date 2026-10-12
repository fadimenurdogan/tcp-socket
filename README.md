[README.md](https://github.com/user-attachments/files/33324694/README.md)
# UR5e Socket Measurement

Bu repo, UR5e robotu için yazılmış bir URScript programı içerir. Robot, RoboDK ile oluşturulmuş bir dizi noktaya sırayla gider. Her noktada TCP/IP üzerinden bir ölçüm cihazına komut gönderir ve cihazın cevabını okur. Ardından bu cevabı robotun o anki TCP pozuyla birlikte bir PC'ye iletir.

## Akış

```
movej (başlangıç)
  └─ her nokta için:
       movel → sendcom() → respcom()
```

1. **`conn()`**: Ölçüm cihazına iki socket açar: gönderici (`sender`) ve dinleyici (`listener`).
2. **`sendcom()`**: Cihaza `%MR02**` ve `%S76` komutlarını CR (`\r`) ile sonlandırarak gönderir.
3. **`respcom()`**: Cihazın cevabını okur ve `get_actual_tcp_pose()` ile robotun pozunu alır. Sonra `cevap,poz` formatındaki mesajı PC'ye gönderir.

## Ağ Ayarları

| Cihaz          | IP               | Port  | Socket adı  |
|----------------|------------------|-------|-------------|
| Ölçüm cihazı   | 169.254.231.54   | 8604  | `sender`    |
| Ölçüm cihazı   | 169.254.231.54   | 8601  | `listener`  |
| PC             | 169.254.231.190  | 65525 | `pc_sender` |

Kendi kurulumunuza göre IP ve portları `conn()` ile `respcom()` içinde değiştirin.

## PC Tarafı

PC'de 65525 portunu dinleyen basit bir TCP sunucusu çalışmalıdır. Her ölçüm noktası için şu formatta bir satır gelir:

```
<cihaz_cevabı>,p[x, y, z, rx, ry, rz]
```

## Gereksinimler

- Universal Robots UR5e (PolyScope 5.x)
- RoboDK v5.7.0 (programı yeniden oluşturmak için)
- Robot, ölçüm cihazı ve PC'nin aynı ağda bulunması

## Kullanım

1. `.script` dosyasını robota yükleyin. Bunun için USB ya da bir `Script` komutu kullanabilirsiniz.
2. PC tarafındaki TCP sunucusunu başlatın.
3. Ölçüm cihazının açık ve ağa bağlı olduğundan emin olun.
4. Programı çalıştırın.

## Notlar

- Referans frame ve TCP değerleri RoboDK istasyonuna özeldir. Farklı bir kurulumda yeniden kalibre edilmelidir.
- `SprayOn`, `Extruder` ve `MoveAxis` fonksiyonları RoboDK'nin varsayılan şablonundan gelir ve bu programda kullanılmaz.
