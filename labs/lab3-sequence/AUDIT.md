# Аудит узгодженості Sequence-діаграм (Lab 3)

## 1. Звірка учасників та даних із ER-моделлю (`er_v2.mmd`)
*   **`Client` та `NotificationService`:** відповідають акторам і ролям `User.role` (`Client`, `Manager`).
*   **`OrderRepository` та `PromoService`:** оперують реальними сутностями та атрибутами з `model/er_v2.mmd` (`Order.status`: `"Draft"`/`"Paid"`, `Order.addressId`, `Order.promoCode`, `Order.discountAmount`, `Order.totalAmount`, `OrderItem.priceAtPurchase`, `OrderItem.quantity`, `Promotion.discountPercentage`, `Friendship`).
*   **`CheckoutUI`, `OrderController`, `PaymentGateway`:** реалізують шари взаємодії (Boundary / Control / External) без підміни сутностей БД.

## 2. Звірка кроків із Gherkin (`scenarios.md`) та вимогами (`FR-01`–`FR-08`)
*   **`SC-01`:** покриває `FR-03`–`FR-06` (цикл `loop` підрахунку позицій кошика, фіксація суми й адреси, оплата, перехід у `"Paid"`, сповіщення менеджера й клієнта).
*   **`SC-02`:** покриває `FR-02`–`FR-06` (перевірка реферального промокоду через БД, розрахунок знижки 5% у блоці `alt`, збереження `addressId`, оплата та обидва сповіщення у блоці `opt`).
*   **`SC-03`:** покриває `FR-03`, `FR-07`, `FR-08` (при статусі `"DECLINED"` від шлюзу замовлення залишається у `"Draft"`, кошик зберігається, менеджер не сповіщується, відображається кнопка «Повторити спробу»).
*   **`SC-04`:** покриває `FR-01` (при `addressId == null` блокується запит до `PaymentGateway`, повертається підказка «Будь ласка, вкажіть адресу доставки»).

## 3. Перевірка синтаксису та `activation bars`
*   У кожній із 4 діаграм використано комбіновані фрагменти (`loop`, `alt`, `opt`).
*   Смуги активності (`activate`/`deactivate`) для `CheckoutUI`, `OrderController` та `PromoService` починаються до умовних фрагментів і завершуються після них, не обриваючись усередині гілок.

## 4. Топ-3 розбіжності (Spec / Вимоги ↔ Вихід AI v1 у коміті `ebaa199`)

| # | Виявлена розбіжність | Слід (вхід / промпт) | Виправлення (коміт) |
| :--- | :--- | :--- | :--- |
| **1** | **Відсутність звернення до БД у `PromoService` (`SC-02`):** у першій версії (`ebaa199`) `validateReferralCode()` повертав `isValid=true` без запиту до таблиць `Promotion` та `Friendship` із `er_v2.mmd`. | Уточнення п. 2 і п. 4.4 у `spec.md` (`7f1c69c`), `ai/02_prompt_sequence_fix.md` (`022a390`) | `85804c9` — додано виклик `PromoService` → `OrderRepository.findReferralPromo()` |
| **2** | **Пропущене збереження `addressId` у `SC-02` (`FR-03`):** після застосування знижки `submitOrder()` одразу викликав `chargePayment()`, не зберігши адресу доставки в сутність `Order`. | Уточнення п. 4.4 у `spec.md` (`7f1c69c`), `ai/02_prompt_sequence_fix.md` (`022a390`) | `85804c9` — додано виклик `OrderRepository.saveOrderAddress()` перед оплатою |
| **3** | **Пропущене сповіщення клієнта у `SC-02` (`FR-06`):** за матрицею трасування `FR-06` охоплює `SC-01` і `SC-02`, але у `ebaa199` всередині `opt Оплата успішна` сповіщувався лише менеджер (`FR-05`). | Уточнення п. 2 і п. 4.4 у `spec.md` (`7f1c69c`), `ai/02_prompt_sequence_fix.md` (`022a390`) | `85804c9` — додано виклик `NotificationService.sendPaymentConfirmation()` |
