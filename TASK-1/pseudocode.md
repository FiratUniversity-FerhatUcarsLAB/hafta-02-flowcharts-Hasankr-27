// ATM Para Çekme Sistemi Pseudocode

// --- Sabitler ve Başlangıç Değerleri ---
SABIT GÜNLÜK_LIMIT = 5000 // Günlük para çekme limiti (örnek değer)
SABIT GİRİŞ_HAKKI_MAKS = 3
SABIT MIN_ÇEKİM_TUTARI = 20
SABIT ÇEKİM_KAT_DEĞERİ = 20

BAŞLA

    // --- Değişken Tanımlamaları ---
    DEĞİŞKEN doğru_PIN = "1234"     // Örnek doğru PIN
    DEĞİŞKEN mevcut_bakiye = 1500  // Örnek hesap bakiyesi
    DEĞİŞKEN günlük_çekilen_tutar = 0  // Gün içinde daha önce çekilen tutar
    DEĞİŞKEN işlem_tekrar = DOĞRU
    DEĞİŞKEN pin_giriş_deneme = 0
    DEĞİŞKEN pin_doğrulandı = YANLIŞ

    // --- Ana İşlem Döngüsü (Tekrar Seçeneği) ---
    DONGÜ (işlem_tekrar EŞİTTİR DOĞRU)

        // --- PIN Doğrulama Bölümü ---
        YAZ "Lütfen kartınızı takın."

        // PIN Giriş Döngüsü (3 Hak)
        pin_giriş_deneme = 0
        pin_doğrulandı = YANLIŞ
        DONGÜ (pin_giriş_deneme KÜÇÜKTÜR GİRİŞ_HAKKI_MAKS)
            YAZ "Lütfen 4 haneli PIN kodunuzu girin:"
            OKU girilen_PIN

            EĞER (girilen_PIN EŞİTTİR doğru_PIN) İSE
                pin_doğrulandı = DOĞRU
                KES DONGÜ
            DEĞİLSE
                pin_giriş_deneme = pin_giriş_deneme + 1
                kalan_hak = GİRİŞ_HAKKI_MAKS - pin_giriş_deneme
                EĞER (kalan_hak BÜYÜKTÜR 0) İSE
                    YAZ "Hatalı PIN. Kalan deneme hakkınız: " + kalan_hak
                DEĞİLSE
                    YAZ "Hatalı PIN. Kartınız bloke edilmiştir. Lütfen bankanızla iletişime geçin."
                SON EĞER
            SON EĞER
        SON DONGÜ // PIN Giriş Döngüsü Sonu

        // --- İşlem Kontrolü ---
        EĞER (pin_doğrulandı EŞİTTİR DOĞRU) İSE

            // --- Para Çekme İşlemi Başlangıcı ---
            YAZ "PIN doğru. Lütfen çekmek istediğiniz tutarı girin (TL):"
            OKU çekim_tutarı

            // 1. Tutar Kontrolü (20 TL Katları)
            EĞER (çekim_tutarı MOD ÇEKİM_KAT_DEĞERİ EŞİTTİR 0) VE (çekim_tutarı BÜYÜK_EŞİT MIN_ÇEKİM_TUTARI) İSE

                // 2. Günlük Limit Kontrolü
                toplam_çekim = günlük_çekilen_tutar + çekim_tutarı
                EĞER (toplam_çekim KÜÇÜK_EŞİT GÜNLÜK_LIMIT) İSE

                    // 3. Bakiye Kontrolü
                    EĞER (çekim_tutarı KÜÇÜK_EŞİT mevcut_bakiye) İSE
                        
                        // --- İşlemi Gerçekleştir ---
                        mevcut_bakiye = mevcut_bakiye - çekim_tutarı
                        günlük_çekilen_tutar = toplam_çekim
                        
                        YAZ "İşleminiz gerçekleştiriliyor..."
                        YAZ çekim_tutarı + " TL hesabınızdan çekilmiştir."
                        YAZ "Güncel bakiyeniz: " + mevcut_bakiye + " TL"
                        YAZ "Kalan günlük çekim limitiniz: " + (GÜNLÜK_LIMIT - günlük_çekilen_tutar) + " TL"
                        YAZ "Lütfen paranızı ve kartınızı alın."

                    DEĞİLSE
                        YAZ "Yetersiz bakiye. Hesabınızda " + mevcut_bakiye + " TL bulunmaktadır. Lütfen daha düşük bir tutar girin."
                    SON EĞER // Bakiye Kontrolü Sonu

                DEĞİLSE
                    kalan_limit = GÜNLÜK_LIMIT - günlük_çekilen_tutar
                    YAZ "Günlük limit aşıldı. Bugün " + günlük_çekilen_tutar + " TL çektiniz. Kalan limitiniz: " + kalan_limit + " TL"
                    YAZ "Lütfen " + kalan_limit + " TL'den düşük bir tutar girin veya yarın tekrar deneyin."
                SON EĞER // Günlük Limit Kontrolü Sonu

            DEĞİLSE
                YAZ "Geçersiz tutar. Lütfen " + ÇEKİM_KAT_DEĞERİ + " TL'nin katları şeklinde (örneğin " + MIN_ÇEKİM_TUTARI + ", 40, 60...) bir tutar girin."
            SON EĞER // Tutar Kontrolü Sonu

        DEĞİLSE
            YAZ "PIN doğrulaması başarısız olduğu için işlem sonlandırılıyor."
        SON EĞER // PIN Doğrulama Kontrolü Sonu

        // --- İşlem Tekrarı Seçeneği ---
        YAZ "Başka bir işlem yapmak ister misiniz? (E/H)"
        OKU tekrar_cevap
        
        EĞER (tekrar_cevap EŞİTTİR "H") YADA (tekrar_cevap EŞİTTİR "h") İSE
            işlem_tekrar = YANLIŞ
        DEĞİLSE
            YAZ "Kartınız iade ediliyor."
        SON EĞER

    SON DONGÜ // Ana İşlem Döngüsü Sonu

    YAZ "İyi günler dileriz. İşlem sonlandı."

SON
