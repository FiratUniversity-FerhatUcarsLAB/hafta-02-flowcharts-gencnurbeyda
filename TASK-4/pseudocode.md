BEGIN

  // Giriş
  DISPLAY "Öğrenci numaranızı giriniz:"
  INPUT ogrenciNo

  DISPLAY "Şifrenizi giriniz:"
  INPUT sifre

  IF NOT ogrenciGirisKontrol(ogrenciNo, sifre) THEN
      DISPLAY "Giriş başarısız. Sistemden çıkılıyor."
      EXIT
  ENDIF

  toplamKredi = 0
  secilenDersler = []

  dersListesi = getDersListesi()
  DISPLAY "Ders Listesi:"

  FOR ders IN dersListesi DO
      DISPLAY ders
  ENDFOR

  WHILE TRUE DO
      DISPLAY "Ders eklemek için ders kodunu girin (Çıkmak için -1):"
      INPUT dersKodu

      IF dersKodu == -1 THEN
          BREAK
      ENDIF

      ders = getDersByKodu(dersKodu)

      // Kontenjan kontrolü
      IF ders.kontenjan <= 0 THEN
          DISPLAY "Bu dersin kontenjanı dolmuştur."
          CONTINUE
      ENDIF

      // Ön koşul kontrolü
      IF ders.onKosul != NULL AND NOT ogrenciGecmisDersiAlmisMi(ogrenciNo, ders.onKosul) THEN
          DISPLAY "Bu dersi alabilmek için ön koşul olan " + ders.onKosul + " dersini geçmiş olmanız gerekir."
          CONTINUE
      ENDIF

      // Zaman çakışması kontrolü
      IF zamanCakismaKontrol(ders, secilenDersler) THEN
          DISPLAY "Bu ders seçtiğiniz diğer bir dersle zaman çakışması içeriyor."
          CONTINUE
      ENDIF

      // Kredi limiti kontrolü
      IF toplamKredi + ders.kredi > 35 THEN
          DISPLAY "Bu dersi seçerseniz kredi limiti aşılır. Kredi limiti: 35"
          CONTINUE
      ENDIF

      // Ders ekleme
      secilenDersler.ADD(ders)
      toplamKredi = toplamKredi + ders.kredi
      DISPLAY ders.dersAdi + " dersi eklendi. Toplam kredi: " + toplamKredi

  ENDWHILE

  // Ders çıkarma döngüsü (isteğe bağlı)
  WHILE TRUE DO
      DISPLAY "Ders çıkarmak istiyor musunuz? (evet/hayır)"
      INPUT cevap

      IF cevap == "hayır" THEN
          BREAK
      ENDIF

      DISPLAY "Çıkarmak istediğiniz dersin kodunu giriniz:"
      INPUT dersKodu

      ders = getDersByKodu(dersKodu)
      IF ders IN secilenDersler THEN
          secilenDersler.REMOVE(ders)
          toplamKredi = toplamKredi - ders.kredi
          DISPLAY ders.dersAdi + " dersi çıkarıldı. Yeni toplam kredi: " + toplamKredi
      ELSE
          DISPLAY "Bu dersi seçmemişsiniz."
      ENDIF
  ENDWHILE

  // GPA kontrolü
  ogrenciGPA = getGPA(ogrenciNo)
  IF ogrenciGPA < 2.5 THEN
      DISPLAY "GPA'niz 2.5'in altında. Danışman onayı gereklidir."
      IF NOT danismanOnayiAlindiMi(ogrenciNo) THEN
          DISPLAY "Danışman onayı alınmamış. Kayıt tamamlanamaz."
          EXIT
      ENDIF
  ENDIF

  // Kayıt özeti
  DISPLAY "Kayıt Özeti:"
  FOR ders IN secilenDersler DO
      DISPLAY ders.dersKodu + " - " + ders.dersAdi + " (" + ders.kredi + " kredi)"
  ENDFOR
  DISPLAY "Toplam Kredi: " + toplamKredi

  DISPLAY "Ders kaydını onaylıyor musunuz? (evet/hayır)"
  INPUT onay

  IF onay == "evet" THEN
      kaydiTamamla(ogrenciNo, secilenDersler)
      DISPLAY "Ders kaydınız başarıyla tamamlandı."
  ELSE
      DISPLAY "Ders kaydı iptal edildi."
  ENDIF

END
