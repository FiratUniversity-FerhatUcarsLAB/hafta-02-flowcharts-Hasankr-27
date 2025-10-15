FONKSIYON HastaneSistemiAnaBaslangic()
    // 1. Kimlik Doğrulama
    KULLANICI_KIMLIK_DOGRULAMA()
    
    EĞER (KimlikDogrulamaBasarili ISE)
        BASLAT AnaMenu(KullaniciID)
    DEĞİLSE
        CIKTI("Hatalı kimlik bilgileri. Lütfen tekrar deneyin.")
    SON_EĞER
SON_FONKSIYON

// ===========================================
// ANA MENÜ VE YÖNLENDİRME
// ===========================================
FONKSIYON AnaMenu(KullaniciID)
    DONGU_BASLAT
        CIKTI("--- HASTANE İŞLEMLERİ ANA MENÜSÜ ---")
        CIKTI("1. Randevu İşlemleri (Alma, İptal, Görüntüleme)")
        CIKTI("2. Tahlil Sonuçları Görüntüleme")
        CIKTI("3. Çıkış")
        
        GIRDI(Secim)
        
        EĞER (Secim == "1")
            BASLAT RandevuAlimSureci(KullaniciID) // Randevu alma ve yönetimi
        DEĞİLSE EĞER (Secim == "2")
            BASLAT TahlilSorgulamaSureci(KullaniciID) // Tahlil sonuçlarını görüntüleme
        DEĞİLSE EĞER (Secim == "3")
            DONGU_BITIR
        DEĞİLSE
            CIKTI("Geçersiz seçim. Lütfen tekrar deneyin.")
        SON_EĞER
    DONGU_BITIR
    CIKTI("Oturum kapatıldı.")
SON_FONKSIYON

// ===========================================
// RANDEVU İŞLEMLERİ MODÜLÜ (Önceki Algoritma 2'nin detaylı adımı)
// ===========================================
FONKSIYON RandevuAlimSureci(HastaID)
    CIKTI("--- RANDEVU ALMA İŞLEMLERİ ---")
    // (Burada Randevu iptal/görüntüleme gibi alt seçenekler de sunulabilir)
    
    // 1. Hastane Seçimi
    HastaneID = HASTANE_SECIMI() 
    EĞER (HastaneID == HATA) DÖN 
    
    // 2. Poliklinik Seçimi
    PoliklinikID = POLIKLINIK_SECIMI(HastaneID) 
    EĞER (PoliklinikID == HATA) DÖN 
    
    // 3. Doktor Seçimi
    DoktorListesi = DOKTOR_LISTESI_GETIR(HastaneID, PoliklinikID)
    EĞER (UZUNLUK(DoktorListesi) > 0)
        DOKTORLARI_LISTELE(DoktorListesi)
        GIRDI(SecilenDoktorID)
    DEĞİLSE CIKTI("Uygun doktor yok."); DÖN
    
    // 4. Uygun Saatleri Gösterme ve Seçim
    GIRDI(IstenilenTarih) 
    UygunSaatler = UYGUN_SAATLERI_BUL(SecilenDoktorID, IstenilenTarih)
    
    EĞER (UZUNLUK(UygunSaatler) > 0)
        SAATLERI_LISTELE(UygunSaatler)
        GIRDI(SecilenSaat)
        
        // 5. Randevu Onaylama
        ONAY_DURUMU = RANDEVU_ONAYLA(HastaID, SecilenDoktorID, IstenilenTarih, SecilenSaat)
        
        EĞER (ONAY_DURUMU == BASARILI)
            CIKTI("Randevunuz başarıyla oluşturuldu.")
            // 6. SMS Gönderme
            SMS_GONDER(HastaID, "Randevu onaylandı: " + IstenilenTarih + " " + SecilenSaat)
        DEĞİLSE
            CIKTI("Randevu oluşturulurken hata oluştu.")
        SON_EĞER
    DEĞİLSE
        CIKTI("Seçilen tarih için uygun saat bulunamadı.")
    SON_EĞER
SON_FONKSIYON

// ===========================================
// TAHLİL SONUÇLARI MODÜLÜ (Önceki Algoritma 3'ün detaylı adımı)
// ===========================================
FONKSIYON TahlilSorgulamaSureci(HastaID)
    CIKTI("--- TAHLİL SONUÇLARI İŞLEMLERİ ---")

    // 1. Tahlil Varlığı Kontrolü ve Listeleme
    TahlilListesi = TAHLIL_LISTESI_GETIR(HastaID)
    
    EĞER (UZUNLUK(TahlilListesi) > 0)
        TAHLILLERI_LISTELE(TahlilListesi)
        GIRDI(SecilenTahlilID)
        
        TahlilKaydi = LISTEDEN_TAHLIL_BUL(TahlilListesi, SecilenTahlilID)
        
        EĞER (TahlilKaydi BULUNDU ISE)
            
            // 2. Sonuç Hazır mı Kontrolü (Onay Durumu)
            SONUC_DURUMU = TAHLIL_SONUC_DURUMU_KONTROL(TahlilKaydi.TahlilID)
            
            EĞER (SONUC_DURUMU == "HAZIR_VE_ONAYLI")
                
                // 3. Sonucu Görüntüleme ve PDF İndirme Seçeneği
                SONUC_VERILERI = TAHLIL_SONUCU_GETIR(TahlilKaydi.TahlilID)
                CIKTI("--- TAHLİL SONUCU GÖSTERİLİYOR ---")
                
                CIKTI("PDF olarak indirmek ister misiniz? (E/H)")
                GIRDI(IndirmeSecimi)
                
                EĞER (IndirmeSecimi == "E")
                    PDF_OLUSTUR_VE_INDIR(TahlilKaydi.TahlilID, SONUC_VERILERI)
                SON_EĞER
                
            DEĞİLSE EĞER (SONUC_DURUMU == "HAZIR_BEKLEMEDE")
                CIKTI("Sonuçlar laboratuvarda hazır, doktor onayı beklenmektedir.")
            
            DEĞİLSE EĞER (SONUC_DURUMU == "ISLEMDE")
                CIKTI("Tahlil sonuçları henüz hazırlanıyor (İşlemde).")
            
            DEĞİLSE
                CIKTI("Tahlil sonucu durumu belirsiz.")
            SON_EĞER
            
        DEĞİLSE
            CIKTI("Geçersiz Tahlil ID'si seçimi.")
        SON_EĞER
        
    DEĞİLSE
        CIKTI("Size ait kayıtlı herhangi bir tahlil isteği veya sonucu bulunmamaktadır.")
    SON_EĞER
SON_FONKSIYON


// ===========================================
// YARDIMCI FONKSİYONLAR (Önceki algoritmalardan)
// ===========================================
// ... (HASTANE_SECIMI, POLIKLINIK_SECIMI, DOKTOR_LISTESI_GETIR, UYGUN_SAATLERI_BUL, RANDEVU_ONAYLA, SMS_GONDER, TAHLIL_LISTESI_GETIR, TAHLIL_SONUC_DURUMU_KONTROL, TAHLIL_SONUCU_GETIR, PDF_OLUSTUR_VE_INDIR, KULLANICI_KIMLIK_DOGRULAMA vb. fonksiyonlar burada yer alacaktır.)

