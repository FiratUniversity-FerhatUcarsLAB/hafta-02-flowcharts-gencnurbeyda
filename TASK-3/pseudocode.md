BEGIN

  // Kimlik Doğrulama
  DISPLAY "Lütfen TC Kimlik Numaranızı giriniz:"
  INPUT tcNo

  IF isValidTC(tcNo) == FALSE THEN
      DISPLAY "Geçersiz TC Kimlik Numarası. Çıkılıyor..."
      EXIT
  ENDIF

  WHILE TRUE
      DISPLAY "İşlem Seçiniz: 1-Randevu Al, 2-Tahlil Sonuçları Gör, 3-Çıkış"
      INPUT islem

      IF islem == 1 THEN
          // Randevu Modülü
          DISPLAY "Poliklinik Seçiniz:"
          DISPLAY listPoliklinikler()
          INPUT poliklinik

          doktorlar = getDoktorlarByPoliklinik(poliklinik)
          DISPLAY "Doktorlar:"
          DISPLAY doktorlar
          INPUT doktor

          WHILE TRUE
              saatler = getUygunSaatler(doktor)
              DISPLAY "Uygun Saatler:"
              DISPLAY saatler
              
              DISPLAY "Randevu saati seçiniz (iptal için -1):"
              INPUT saat

              IF saat == -1 THEN
                  BREAK
              ENDIF

              IF isSaatUygun(doktor, saat) THEN
                  confirm = getUserConfirmation("Randevuyu onaylıyor musunuz?")
                  IF confirm == TRUE THEN
                      saveRandevu(tcNo, doktor, saat)
                      sendSMS(tcNo, doktor, saat)
                      DISPLAY "Randevunuz başarıyla oluşturuldu. SMS gönderildi."
                      BREAK
                  ELSE
                      DISPLAY "Randevu iptal edildi."
                      BREAK
                  ENDIF
              ELSE
                  DISPLAY "Seçilen saat uygun değil. Lütfen tekrar deneyiniz."
              ENDIF
          ENDWHILE

      ELSE IF islem == 2 THEN
          // Tahlil Modülüne Geç
          CALL TahlilModulu(tcNo)

      ELSE IF islem == 3 THEN
          DISPLAY "Çıkılıyor..."
          BREAK
      ELSE
          DISPLAY "Geçersiz işlem seçimi."
      ENDIF

  ENDWHILE

ENDFUNCTION TahlilModulu(tcNo)

  IF hasTahlil(tcNo) == FALSE THEN
      DISPLAY "Kayıtlı bir tahlil bulunamadı."
      RETURN
  ENDIF

  IF isTahlilReady(tcNo) == FALSE THEN
      DISPLAY "Tahlil sonucunuz henüz hazır değil. Lütfen daha sonra tekrar deneyiniz."
      RETURN
  ENDIF

  DISPLAY "Tahlil Sonuçlarınız:"
  DISPLAY getTahlilSonuc(tcNo)

  DISPLAY "Sonucu PDF olarak indirmek ister misiniz? (evet/hayır)"
  INPUT pdfCevap

  IF pdfCevap == "evet" THEN
      downloadPDF(tcNo)
      DISPLAY "PDF indirildi."
  ENDIF

END FUNCTION
