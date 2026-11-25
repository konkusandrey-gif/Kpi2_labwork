# Промпт 02: Усунення розбіжностей рев'ю у сценарії SC-02 

**Завдання до AI:**
Онови сценарій `SC-02` у файлі `labs/lab3-sequence/diagrams.md` відповідно до уточнених критеріїв прийняття у `labs/lab3-sequence/spec.md`  та вимог `FR-05`, `FR-06`:

1. Усередині `PromoService.validateReferralCode()` додай виклик до `OrderRepository.findReferralPromo(userId, promoCode)` для реальної перевірки промокоду та реферального зв'язку у БД (`Promotion`, `Friendship`).
2. У другій фазі `SC-02` (`submitOrder`) перед викликом `PaymentGateway.chargePayment()` додай збереження адреси доставки `OrderRepository.saveOrderAddress(orderId, addressId)`.
3. Усередині фрагмента `opt Оплата успішна` додай виклик `NotificationService.sendPaymentConfirmation(userId, orderId, finalAmount)` для сповіщення клієнта (`FR-06`) після сповіщення менеджера (`FR-05`).
4. Збережи безперервність усіх `activation bars` (`CheckoutUI`, `OrderController`, `PromoService`).
