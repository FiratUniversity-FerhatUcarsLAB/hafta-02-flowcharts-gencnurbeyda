BEGIN

  WHILE TRUE DO          // Sistem 7/24 çalışır

    IF sistemAktifMi() THEN   // Sistem aktif kontrolü

      // Sensör okuma döngüsü
      hareketAlgilandi = hareketSensorOku()
      kapiPencereAcik = kapiPencereSensorOku()

      IF hareketAlgilandi OR kapiPencereAcik THEN   // Tehdit algılama

        // Kamera kontrolü
        IF kameraAktifMi() THEN
          kameraAktiveEt()
        ENDIF

        // Yanlış alarm kontrolü
        IF evSahibiEvdeMi() THEN
          alarmSeviyesi = 1   // Düşük alarm seviyesi, yanlış alarm olabilir
        ELSE
          // Alarm seviyesi belirleme
          IF hareketAlgilandi AND kapiPencereAcik THEN
            alarmSeviyesi = 3   // Yüksek seviye
          ELSE IF hareketAlgilandi OR kapiPencereAcik THEN
            alarmSeviyesi = 2   // Orta seviye
          ENDIF
        ENDIF

        // Bildirim gönder
        bildirimGonder("Alarm Seviyesi: " + alarmSeviyesi + ". Tehdit algılandı!")

        // Alarm sıfırlama veya devam ettirme kontrolü
        IF alarmSifirlandiMi() THEN
          alarmSeviyesi = 0
          CONTINUE    // Alarm sıfırlandı, döngü devam eder
        ELSE
          // Alarm devam ediyor
          // İstenirse burada alarm sesi veya başka önlemler devreye girer
        ENDIF

      ENDIF

    ENDIF

    BEKLE(5 saniye)    // Sensörleri belirli aralıklarla kontrol et

  ENDWHILE

END
