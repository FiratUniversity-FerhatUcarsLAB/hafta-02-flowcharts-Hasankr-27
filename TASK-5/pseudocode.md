PROSEDÜR Akıllı_Guvenlik_Sistemi()
// Başlangıç Değerleri
SISTEM_AKTIF = YANLIS
ALARM_DURUMU = SIFIRLANMIS

// Sonsuz Ana Döngü (Sürekli Çalışır)
DÖNGÜ SÜRECE (DOGRU)
BEKLE(100 milisaniye) // CPU yükünü azaltmak için kısa bekleme

// 1. Sistem Aktivasyon Kontrolü
EĞER SISTEM_AKTIF == YANLIS İSE
  DEVAM_ET // Döngünün başına dön ve sistemi tekrar kontrol et
SON_EĞER

// 2. Sensör Okuma ve Tehdit Algılama
HAREKET_ALGILANDI = SENSOR_OKU("Hareket")
KAPI_PENCERE_ACIK = SENSOR_OKU("KapiPencere")

TEHDIT_ALGILANDI = HAREKET_ALGILANDI VEYA KAPI_PENCERE_ACIK

EĞER TEHDIT_ALGILANDI İSE
  // 3. Tehdit Seviyesi Belirleme
  ALARM_SEVIYESI = 1 // Varsayılan Düşük Seviye
  
  EĞER HAREKET_ALGILANDI VE KAPI_PENCERE_ACIK İSE
    ALARM_SEVIYESI = 3 // Yüksek
  YALNIZCA_EĞER HAREKET_ALGILANDI VEYA KAPI_PENCERE_ACIK İSE
    ALARM_SEVIYESI = 2 // Orta
  SON_EĞER
  
  // 4. Yanlış Alarm Kontrolü (Ev Sahibi Evde Mi?)
  EV_SAHIBI_EVDE = KONUM_KONTROL_ET() // (Geofencing VEYA Dahili Sensor)
  
  EĞER EV_SAHIBI_EVDE == DOGRU VE ALARM_SEVIYESI < 3 İSE
    // Potansiyel Yanlış Alarm: Uyarı ver, hemen alarm çalma.
    BILDIRIM_GÖNDER("App", "Evde hareket algılandı. Onaylıyor musunuz?")
    
    SÜRE_SAYAC = 0
    DÖNGÜ SÜRECE (SÜRE_SAYAC < 60 VE ALARM_DURUMU == SIFIRLANMIS) // 60 saniye bekle
      EĞER KOMUT_AL("Sıfırla") İSE
        ALARM_DURUMU = SIFIRLANMIS
        ÇIKIŞ_DÖNGÜSÜ
      YALNIZCA_EĞER KOMUT_AL("Devam") İSE
        ALARM_DURUMU = ALARM_AKTIF
        ÇIKIŞ_DÖNGÜSÜ
      SON_EĞER
      SÜRE_SAYAC = SÜRE_SAYAC + 1
      BEKLE(1000)
    SON_DÖNGÜ
  
    EĞER ALARM_DURUMU == SIFIRLANMIS İSE
      DEVAM_ET // Ana döngünün başına dön
    SON_EĞER
  
  // 5. Tehdit Onaylandı (Veya Yüksek Seviyede Başladı)
  KAMERA_AKTIVE_ET()
  
  EĞER ALARM_SEVIYESI == 3 İSE
    SESLI_ALARM_CAL()
    BILDIRIM_GÖNDER("SMS", "YÜKSEK SEVİYE TEHDİT. Güvenlik birimi aranıyor.")
    BILDIRIM_GÖNDER("Email", "YÜKSEK SEVİYE TEHDİT BİLDİRİMİ.")
    BILDIRIM_GÖNDER("App", "YÜKSEK SEVİYE TEHDİT. Siren çalıyor.")
  YALNIZCA_EĞER ALARM_SEVIYESI == 2 İSE
    BILDIRIM_GÖNDER("App", "Orta Seviye Tehdit. (Kapı Açıldı)")
    BILDIRIM_GÖNDER("Email", "Orta Seviye Tehdit Bildirimi.")
  YALNIZCA_EĞER ALARM_SEVIYESI == 1 İSE
    BILDIRIM_GÖNDER("App", "Düşük Seviye Tehdit. (Hafif hareket algılandı)")
  SON_EĞER
  
  ALARM_DURUMU = ALARM_AKTIF
  
  // 6. Alarm Sıfırlama veya Devam Ettirme Döngüsü
  DÖNGÜ SÜRECE (ALARM_DURUMU == ALARM_AKTIF)
    BEKLE(5000) // 5 saniye bekle ve tekrar kontrol et
    
    EĞER KOMUT_AL("Sıfırla") İSE
      ALARM_DURUMU = SIFIRLANMIS
      ALARM_SEVIYESI = 0
    YALNIZCA_EĞER SENSOR_OKU("Tehdit_Bitti") İSE // Tehdit geçtiyse
      ALARM_DURUMU = SIFIRLANMIS
    YALNIZCA_EĞER SISTEM_AKTIF == YANLIS İSE // Sistem devre dışı bırakıldıysa
      ALARM_DURUMU = SIFIRLANMIS
    YALNIZCA_EĞER ALARM_SEVIYESI == 3 İSE // Yüksek seviye alarm devam ediyorsa
      BILDIRIM_GÖNDER("Email", "Alarm durumu devam ediyor. Acil durum yetkilisi aranıyor.")
    SON_EĞER
  SON_DÖNGÜ
  
SON_EĞER // Tehdit algılanmadıysa

SON_DÖNGÜ // Ana döngü sonu
SON_PROSEDÜR
