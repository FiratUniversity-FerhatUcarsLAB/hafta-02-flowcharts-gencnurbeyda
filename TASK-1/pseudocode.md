// Başlangıç değişkenleri (gerçek sistemde banka/veritabanından gelir)
cardInserted <- FALSE
cardBlocked <- FALSE
correctPIN <- "1234"                // örnek; gerçek sistemde güvenli saklanır
accountBalance <- 5000.00           // TL
dailyWithdrawn <- 0.00              // bugün çekilen toplam
dailyLimit <- 2000.00               // günlük çekim limiti (örnek)
MAX_PIN_ATTEMPTS <- 3

// ATM ana döngüsü: kart takılana kadar bekle
WHILE TRUE:
    WAIT_FOR_CARD_INSERTION()
    cardInserted <- TRUE

    IF cardInserted == TRUE:
        IF cardBlocked == TRUE:
            DISPLAY("Kartınız bloke edilmiştir. Lütfen banka ile iletişime geçin.")
            EJECT_CARD()
            cardInserted <- FALSE
            CONTINUE   // ana döngüye dön
        ENDIF

        // PIN doğrulama döngüsü
        pinAttempts <- 0
        authenticated <- FALSE

        WHILE pinAttempts < MAX_PIN_ATTEMPTS AND authenticated == FALSE:
            enteredPIN <- PROMPT_PIN("Lütfen PIN giriniz:")
            IF enteredPIN == correctPIN:
                authenticated <- TRUE
            ELSE
                pinAttempts <- pinAttempts + 1
                remaining <- MAX_PIN_ATTEMPTS - pinAttempts
                IF remaining > 0:
                    DISPLAY("Hatalı PIN. Kalan deneme: " + remaining)
                ENDIF
            ENDIF
        ENDWHILE

        // Eğer 3 hatalı deneme olduysa kart bloke et ve iade et
        IF authenticated == FALSE:
            cardBlocked <- TRUE
            DISPLAY("3 hatalı PIN girişinden dolayı kartınız bloke edildi.")
            RETAIN_CARD()   // ATM kartı tutar veya güvenlik prosedürü uygulayıp bildirir
            // veya EJECT_CARD() ve bloke bilgisi göster
            cardInserted <- FALSE
            CONTINUE
        ENDIF

        // Başarılı giriş sonrası işlem döngüsü (kullanıcı birden fazla işlem yapabilir)
        continueTransactions <- TRUE
        WHILE continueTransactions == TRUE:
            // İşlem menüsü (sadece çekme işlemi uygulanacak şekilde düzenlenmiş)
            DISPLAY("1) Bakiye Sorgula")
            DISPLAY("2) Para Çek")
            DISPLAY("3) Kart İade ve Çıkış")
            choice <- PROMPT("Seçiminiz (1/2/3):")

            IF choice == "1":
                DISPLAY("Mevcut bakiye: " + FORMAT_CURRENCY(accountBalance))
                DISPLAY("Bugünkü toplam çekim: " + FORMAT_CURRENCY(dailyWithdrawn))
                // ardından başka işlem isteyip istemediğini sor
                ans <- PROMPT_YN("Başka işlem yapmak ister misiniz? (E/H):")
                IF ans == "E": continueTransactions <- TRUE ELSE
                    DISPLAY("Lütfen kartınızı alın.")
                    EJECT_CARD()
                    cardInserted <- FALSE
                    continueTransactions <- FALSE
                ENDIF

            ELSE IF choice == "2":
                // Para çekme adımları
                DISPLAY("Para çekme ekranı. Lütfen tutarı giriniz.")
                amount <- PROMPT_AMOUNT("Çekmek istediğiniz tutarı giriniz (TL):")

                // İptal koşulu
                IF amount == 0 OR USER_CANCELLED(): 
                    DISPLAY("İşlem iptal edildi.")
                    ans <- PROMPT_YN("Başka işlem yapmak ister misiniz? (E/H):")
                    IF ans == "E": continueTransactions <- TRUE ELSE
                        DISPLAY("Lütfen kartınızı alın.")
                        EJECT_CARD()
                        cardInserted <- FALSE
                        continueTransactions <- FALSE
                    ENDIF
                    CONTINUE
                ENDIF

                // Koşul 1: 20 TL'nin katı olmalı
                IF (amount MOD 20) != 0:
                    DISPLAY("Hata: Çekilecek tutar 20 TL'nin katı olmalıdır.")
                    CONTINUE   // tekrar menüye dön
                ENDIF

                // Koşul 2: Yetersiz bakiye kontrolü
                IF amount > accountBalance:
                    DISPLAY("Hata: Yetersiz bakiye.")
                    CONTINUE
                ENDIF

                // Koşul 3: Günlük limit kontrolü
                IF (dailyWithdrawn + amount) > dailyLimit:
                    remainingLimit <- dailyLimit - dailyWithdrawn
                    DISPLAY("Hata: Günlük limit aşılıyor. Kalan limit: " + FORMAT_CURRENCY(remainingLimit))
                    CONTINUE
                ENDIF

                // Tüm kontroller geçti -> para verilir ve bakiye güncellenir
                DISPENSE_CASH(amount)      // ATM fiziksel haznesinden para ver
                PRINT_RECEIPT(amount, accountBalance - amount, dailyWithdrawn + amount)
                accountBalance <- accountBalance - amount
                dailyWithdrawn <- dailyWithdrawn + amount

                DISPLAY("Lütfen paranızı ve fişinizi alınız.")

                // Başka işlem isteyip istemediğini sor
                ans <- PROMPT_YN("Başka işlem yapmak ister misiniz? (E/H):")
                IF ans == "E": continueTransactions <- TRUE ELSE
                    DISPLAY("Lütfen kartınızı alın.")
                    EJECT_CARD()
                    cardInserted <- FALSE
                    continueTransactions <- FALSE
                ENDIF

            ELSE IF choice == "3":
                DISPLAY("İşlemler sonlandırılıyor. Lütfen kartınızı alın.")
                EJECT_CARD()
                cardInserted <- FALSE
                continueTransactions <- FALSE

            ELSE
                DISPLAY("Geçersiz seçim. Lütfen tekrar deneyin.")
            ENDIF

        ENDWHILE // işlem döngüsü sonu

    ENDIF // cardInserted check sonu

ENDWHILE // ana döngü sonu
