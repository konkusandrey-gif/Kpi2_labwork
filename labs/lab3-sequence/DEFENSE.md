# DEFENSE
Завдання: Завдання 3 — Взаємодія (Sequence Diagram) (FoodDelivery)

**1. Що я просив у AI**
На основі `spec.md` (`fb13cf0`), BDD-сценаріїв `scenarios.md` та `er_v2.mmd` згенерувати 4 окремі Mermaid Sequence-діаграми для сценаріїв `SC-01`–`SC-04` із фрагментами `alt`/`opt`/`loop` та безперервними `activation bars` (`ai/01_prompt_sequence.md`, коміт `4c67e0c`), а після рев'ю — усунути виявлені пропуски у сценарії `SC-02` (`ai/02_prompt_sequence_fix.md`, коміт `022a390`).

**2. Топ-3 розбіжності, які я знайшов і виправив (spec ↔ вихід AI `ebaa199`)**

| # | Що було не так (проти якого пункту spec / вимог) | Мій слід (промпт / дод. spec) | Виправлення (коміт) |
| :--- | :--- | :--- | :--- |
| **1** | **Перевірка промокоду без звернення до БД (`SC-02`):** у першій генерації (`ebaa199`) `PromoService.validateReferralCode()` повертав `isValid=true` без запиту до таблиць `Promotion` і `Friendship` (`er_v2.mmd`). | Оновлення `spec.md` (`7f1c69c`), `ai/02_prompt_sequence_fix.md` (`022a390`) | `85804c9` |
| **2** | **Не зберігався `addressId` перед оплатою (`SC-02` vs `FR-03`):** після перерахунку знижки `submitOrder()` одразу викликав `chargePayment()`, пропустивши запис адреси доставки в `Order`. | Оновлення `spec.md` (`7f1c69c`), `ai/02_prompt_sequence_fix.md` (`022a390`) | `85804c9` |
| **3** | **Забуте сповіщення клієнта після оплати зі знижкою (`SC-02` vs `FR-06`):** у блоці `opt Оплата успішна` викликався лише `notifyRestaurantManager()` (`FR-05`), без `sendPaymentConfirmation()` для клієнта. | Оновлення `spec.md` (`7f1c69c`), `ai/02_prompt_sequence_fix.md` (`022a390`) | `85804c9` |

**3. Ключове рішення:**
Обрано централізовану оркестрацію процесу оформлення через `OrderController` із делегуванням бізнес-логіки та інтеграцій виділеним сервісам (`PromoService`, `NotificationService`, `OrderRepository`) замість прямих викликів із `CheckoutUI` (`adr/01-checkout-orchestration.md`, коміт `b4d601a`). Це виключає обхід перевірок адреси чи знижки на стороні клієнта й ізолює правила акцій і сповіщень.
*Межі сценаріїв та спірні місця:* деталізовано лише внутрішню взаємодію для 4 сценаріїв `UC_Checkout` (`SC-01`–`SC-04`); внутрішня логіка `PaymentGateway` (CVV, токенізація), автентифікація (`UC_Login`) та доставка кур'єром (`UC_TakeDelivery`) свідомо винесені поза обсяг (`spec.md`).

**4. Перевірка:**
*   Усі 4 діаграми у `labs/lab3-sequence/diagrams.md` (`85804c9`) візуально відрендерено на GitHub і звірено з `er_v2.mmd`, `scenarios.md` та вимогами `FR-01`–`FR-08` (повний чеклист у `labs/lab3-sequence/AUDIT.md`).
