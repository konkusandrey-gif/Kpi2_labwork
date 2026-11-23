# Специфікація архітектури системи `FoodDelivery` (`spec.md`)

## 1. Мета, нотація та межа системи (Рішення 1А)
Побудувати компонентну модель архітектури `FoodDelivery` на основі Завдань 1–4 (`er_v2.mmd`, `uc.mmd`, `requirements.md`, `scenarios.md`, діаграм послідовності та процесу `UC_Checkout`).
* **Нотація:** єдина декларативна нотація **Mermaid `flowchart` (`flowchart TB`)** із нативним рендером на GitHub.
* **Межа системи (`System Boundary`):** контейнер `subgraph FoodDelivery`, що відокремлює внутрішні модулі від зовнішніх шлюзів.
* **Інтерфейси:** позначаються окремими вузлами-портами `([I...])` зі зв'язками `<<provide>>` (надає контракт) та `<<require>>` (споживає контракт).

## 2. Внутрішні шари та компоненти в межах `FoodDelivery` (Рішення 2А)
1. **Presentation Layer (UI):**
   * `WebMobileUI` (включно з `CheckoutUI`) — інтерфейс клієнта (`UC_Login`, `UC_ViewMenu`, `UC_Checkout`, `UC_Track`).
   * `StaffPortalUI` — інтерфейс менеджера ресторану та кур'єра (`UC_ManageMenu`, `UC_ConfirmOrder`, `UC_TakeDelivery`, `UC_UpdateStatus`).
2. **API / Backend Layer:**
   * `APIGateway` (`AuthController`, `OrderController`, `CatalogController`, `DeliveryController`) — точка входу HTTPS (`NFR-04`), маршрутизація та первинна валідація.
3. **Business Logic Layer (Services):**
   * `AuthService` — внутрішня автентифікація та перевірка ролей `User.role` (Рішення 3А).
   * `OrderService` — оркестрація замовлення (`Draft` → `Paid`), перевірка кошика й адреси (`FR-01`, `FR-03`, `FR-04`, `FR-07`).
   * `PromoService` — валідація та застосування реферальних промокодів 5% (`FR-02`).
   * `NotificationService` — асинхронна відправка й ретрай сповіщень клієнту та менеджеру (`FR-05`, `FR-06`).
   * `CatalogService` — управління меню та обраними стравами (`UC_ViewMenu`, `UC_ManageMenu`).
   * `DeliveryService` — призначення кур'єрів і оновлення статусів доставки (`UC_TakeDelivery`, `UC_UpdateStatus`, `UC_Track`).
4. **Data Access Layer (Repositories):**
   * `UserRepository` (`User`), `OrderRepository` (`Order`, `OrderItem`, `Address`), `PromoRepository` (`Promotion`, `Friendship`), `CatalogDeliveryRepository` (`MenuItem`, `Favorite`, `Courier`).
5. **Database Layer:**
   * `PostgreSQL` — реляційна БД із 9 таблицями з `er_v2.mmd`.

## 3. Зовнішні інтеграції та поза обсягом (Рішення 3А)
* **Зовнішні сервіси (поза `System Boundary`):**
  * `PaymentGateway` (`IPaymentGateway`) — проведення оплати без зберігання карткових даних у БД (`NFR-04`, таймаут 15 с за `NFR-02`).
  * `NotificationProvider` (`INotificationGateway`) — зовнішній SMS/Email-шлюз доставки сповіщень.
* **Поза обсягом (`Out of Scope`):** зовнішній `OAuth / Identity Provider` (вхід реалізовано через внутрішній `AuthService`) та картографічні сервіси.

## 4. Критерії прийняття
* Залежності спрямовані строго зверху вниз (`UI → API → Services → Data Access → DB`), без прямих звернень UI до БД і без циклічних залежностей.
* Кожен компонент пов'язаний через явні `provided`/`required`-інтерфейси та трасується до моделей Завдань 1–4.
