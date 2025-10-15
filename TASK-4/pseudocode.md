// --- SİSTEMDE TANIMLI OLDUĞU VARSAYILAN VERİ YAPILARI ---
// Ogrenci: no, sifre, gpa, tamamlananDersler[]
// Ders: kod, ad, kredi, kontenjan, kayitliSayisi, zamanTablosu[], onKosullar[]

// --- İŞLEM İÇİN GEREKLİ DEĞİŞKENLER ---
DEĞİŞKEN secilenDersler : Dizi // Öğrencinin seçtiği dersleri tutar
DEĞİŞKEN toplamKredi : Tamsayı = 0
DEĞİŞKEN aktifOgrenci : Ogrenci // Giriş yapan öğrencinin bilgileri
DEĞİŞKEN kayitDevamEdiyor : Mantıksal = TRUE
PROSEDÜR UniversiteDersKayitSistemi()

  // --- ADIM 1: ÖĞRENCİ GİRİŞİ ---
  YAZ "Öğrenci No Giriniz:"
  OKU ogrenciNo
  YAZ "Şifre Giriniz:"
  OKU sifre

  EĞER KullaniciDogrula(ogrenciNo, sifre) == DOĞRU İSE
    aktifOgrenci = OgrenciBilgileriniGetir(ogrenciNo)
  DEĞİLSE
    YAZ "HATA: Öğrenci Numarası veya Şifre Yanlış."
    ÇIKIŞ // Prosedürü sonlandır
  SON EĞER

  // --- ADIM 2: ANA İŞLEM DÖNGÜSÜ ---
  DÖNGÜ (kayitDevamEdiyor == DOĞRU OLDUĞU SÜRECE)
    
    YAZ "--- ANA MENÜ ---"
    YAZ "1. Ders Ekle"
    YAZ "2. Seçilen Dersleri Göster ve Ders Çıkar"
    YAZ "3. Kaydı Tamamla ve Çıkış"
    OKU menuSecim

    EĞER menuSecim == 1 İSE
      // --- DERS EKLEME İŞLEMİ VE KONTROLLER ---
      TumDersleriGoster()
      YAZ "Eklemek istediğiniz dersin kodunu girin:"
      OKU eklenecekDersKodu
      
      eklenecekDers = DersBilgisiniGetir(eklenecekDersKodu)
      
      // --- KONTROL 1: KREDİ LİMİTİ KONTROLÜ ---
      EĞER (toplamKredi + eklenecekDers.kredi) <= 35 İSE
      
        // --- KONTROL 2: KONTENJAN KONTROLÜ ---
        EĞER eklenecekDers.kayitliSayisi < eklenecekDers.kontenjan İSE
          
          // --- KONTROL 3: ÖN KOŞUL KONTROLÜ ---
          EĞER OnKosulSaglandiMi(aktifOgrenci.tamamlananDersler, eklenecekDers.onKosullar) == DOĞRU İSE
            
            // --- KONTROL 4: ZAMAN ÇAKIŞMASI KONTROLÜ ---
            EĞER ZamanCakismasiVarMi(eklenecekDers, secilenDersler) == YANLIŞ İSE
              
              // --- TÜM KONTROLLER BAŞARILI: DERSİ EKLE ---
              secilenDersler.EKLE(eklenecekDers)
              toplamKredi = toplamKredi + eklenecekDers.kredi
              YAZ "BİLGİ: '" + eklenecekDers.ad + "' dersi başarıyla eklendi."
              
            DEĞİLSE // Zaman Çakışması Kontrolü Başarısız
              YAZ "HATA: Ders saati, seçtiğiniz başka bir ders ile çakışıyor."
            SON EĞER
            
          DEĞİLSE // Ön Koşul Kontrolü Başarısız
            YAZ "HATA: Bu ders için gerekli ön koşul derslerini tamamlamadınız."
          SON EĞER
          
        DEĞİLSE // Kontenjan Kontrolü Başarısız
          YAZ "HATA: Ders kontenjanı dolu."
        SON EĞER
        
      DEĞİLSE // Kredi Limiti Kontrolü Başarısız
        YAZ "HATA: Bu dersi eklerseniz maksimum kredi limitini (35) aşıyorsunuz."
      SON EĞER
    
    //---------------------------------------------------------

    ELSE IF menuSecim == 2 İSE
      // --- DERS ÇIKARMA İŞLEMİ ---
      SecilenDersleriGoster(secilenDersler)
      YAZ "Çıkarmak istediğiniz dersin kodunu girin (iptal için '0' girin):"
      OKU cikarilacakDersKodu
      
      EĞER cikarilacakDersKodu != "0" İSE
        cikarilanDers = secilenDersler.BUL(cikarilacakDersKodu)
        secilenDersler.SİL(cikarilanDers)
        toplamKredi = toplamKredi - cikarilanDers.kredi
        YAZ "BİLGİ: '" + cikarilanDers.ad + "' dersi çıkarıldı."
      SON EĞER
      
    //---------------------------------------------------------
    
    ELSE IF menuSecim == 3 İSE
      // --- KAYDI TAMAMLAMA İŞLEMİ ---
      kayitDevamEdiyor = YANLIŞ // Döngüyü sonlandır
      
    DEĞİLSE
      YAZ "UYARI: Geçersiz bir seçim yaptınız."
    SON EĞER
    
  SON DÖNGÜ

  // --- ADIM 3: KAYIT ÖZETİ VE SON ONAY ---
  YAZ "\n--- KAYIT ÖZETİ ---"
  SecilenDersleriGoster(secilenDersler)
  YAZ "Toplam Kredi: " + toplamKredi
  YAZ "--------------------"

  // --- KONTROL 5: DANIŞMAN ONAYI KONTROLÜ ---
  EĞER aktifOgrenci.gpa < 2.50 İSE
    YAZ "UYARI: Not ortalamanız 2.50'nin altında olduğu için kaydınız danışman onayına gönderilecektir."
    kayitDurumu = "Onay Bekliyor"
  DEĞİLSE
    YAZ "Kaydınız doğrudan onaylanacaktır."
    kayitDurumu = "Onaylandı"
  SON EĞER

  YAZ "Bu şekilde kaydı tamamlamak istiyor musunuz? (EVET/HAYIR)"
  OKU sonOnay
  
  EĞER sonOnay == "EVET" İSE
    KaydiVeritabaninaIsle(aktifOgrenci.no, secilenDersler, kayitDurumu)
    YAZ "Ders kaydınız başarıyla tamamlandı. Sistemden çıkış yapılıyor."
  DEĞİLSE
    YAZ "Kayıt işlemi iptal edildi. Sistemden çıkış yapılıyor."
  SON EĞER
  
SON PROSEDÜR


