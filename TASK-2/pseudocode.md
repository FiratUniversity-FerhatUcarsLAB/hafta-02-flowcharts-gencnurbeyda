BEGIN

  // Kullanıcı Girişi
  DISPLAY "Lütfen giriş yapınız."
  INPUT username, password

  IF loginSuccess(username, password) == FALSE THEN
      DISPLAY "Hatalı giriş. Lütfen tekrar deneyiniz."
      EXIT
  ENDIF

  DISPLAY "Giriş başarılı. Hoş geldiniz!"

  // Ürün Kategorileri Arasında Gezinme
  WHILE TRUE
      DISPLAY "Kategori seçiniz: 1-Elektronik 2-Kıyafet 3-Kitap 4-Çıkış"
      INPUT category

      IF category == 4 THEN
          BREAK
      ENDIF

      productList = getProductsByCategory(category)
      DISPLAY productList

      DISPLAY "Sepete eklemek istediğiniz ürün ID'sini giriniz, çıkmak için -1 yazınız:"
      INPUT productId

      IF productId == -1 THEN
          CONTINUE
      ENDIF

      selectedProduct = getProductById(productId)

      // Stok Kontrolü
      IF selectedProduct.stock == 0 THEN
          DISPLAY "Bu ürün stokta yok."
          CONTINUE
      ENDIF

      DISPLAY "Kaç adet eklemek istersiniz?"
      INPUT quantity

      IF quantity > selectedProduct.stock THEN
          DISPLAY "Yeterli stok yok."
          CONTINUE
      ENDIF

      addToCart(selectedProduct, quantity)
      DISPLAY "Ürün sepete eklendi."

  ENDWHILE

  // Sepeti Görüntüleme ve Düzenleme
  WHILE TRUE
      DISPLAY "Sepetiniz:"
      DISPLAY cartItems

      DISPLAY "1-Ürün sil  2-Adet güncelle  3-Devam et"
      INPUT action

      IF action == 3 THEN
          BREAK
      ENDIF

      INPUT productId

      IF action == 1 THEN
          removeFromCart(productId)
      ELSE IF action == 2 THEN
          DISPLAY "Yeni adet:"
          INPUT newQty
          updateCart(productId, newQty)
      ENDIF

  ENDWHILE

  // İndirim Kodu Uygulama
  DISPLAY "İndirim kodunuz var mı? (evet/hayır)"
  INPUT hasCoupon

  IF hasCoupon == "evet" THEN
      DISPLAY "İndirim kodunu giriniz:"
      INPUT couponCode
      IF isValidCoupon(couponCode) THEN
          applyDiscount(couponCode)
          DISPLAY "İndirim uygulandı."
      ELSE
          DISPLAY "Geçersiz indirim kodu."
      ENDIF
  ENDIF

  // Minimum 50 TL kontrolü
  IF getCartTotal() < 50 THEN
      DISPLAY "Sipariş tutarı minimum 50 TL olmalıdır."
      EXIT
  ENDIF

  // Kargo Ücreti Hesaplama
  IF getCartTotal() >= 200 THEN
      shippingCost = 0
  ELSE
      shippingCost = 20
  ENDIF

  DISPLAY "Kargo ücreti: " + shippingCost + " TL"

  // Ödeme Yöntemi Seçimi
  DISPLAY "Ödeme yöntemi seçiniz: 1-Kredi Kartı  2-Havale/EFT  3-Kapıda Ödeme"
  INPUT paymentMethod

  IF paymentMethod NOT IN [1, 2, 3] THEN
      DISPLAY "Geçersiz ödeme yöntemi."
      EXIT
  ENDIF

  // Sipariş Onayı
  DISPLAY "Sipariş özeti:"
  DISPLAY cartItems
  DISPLAY "Toplam: " + getCartTotal() + " TL"
  DISPLAY "Kargo: " + shippingCost + " TL"
  DISPLAY "Ödeme yöntemi: " + getPaymentMethodName(paymentMethod)

  DISPLAY "Siparişi onaylıyor musunuz? (evet/hayır)"
  INPUT confirm

  IF confirm == "evet" THEN
      placeOrder()
      DISPLAY "Siparişiniz başarıyla alındı. Teşekkür ederiz!"
  ELSE
      DISPLAY "Sipariş iptal edildi."
  ENDIF

END
