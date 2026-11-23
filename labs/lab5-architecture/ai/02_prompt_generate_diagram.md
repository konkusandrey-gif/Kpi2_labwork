# Промпт 2: Генерація UML Component Diagram системи `FoodDelivery`

На основі моєї специфікації архітектури (`labs/lab5-architecture/spec.md`), моделі даних `er_v2.mmd` (9 сутностей), вимог Завдання 2 (`FR-01`–`FR-08`, `NFR-01`–`NFR-05`) та класів Завдань 3–4 згенеруй декларативний код діаграми компонентів системи `FoodDelivery` для файлу `labs/lab5-architecture/diagrams.md`.

### Зафіксовані архітектурні рішення (1А + 2А + 3А):
1. **Нотація та інтерфейси (Рішення 1А):**
   - Використай єдину нотацію **Mermaid `flowchart`** (`flowchart TB`) у блоці ````mermaid`.
   - Відобрази чітку межу системи через зовнішній контейнер `subgraph FoodDelivery ["System Boundary: FoodDelivery"]`.
   - Покажи `provided`- та `required`-інтерфейси у вигляді окремих вузлів-портів `([I...])` із підписами зв'язків `<<provide>>` та `<<require>>` між усіма рівнями й зовнішніми шлюзами.

2. **Внутрішні шари та компоненти в межах `FoodDelivery` (Рішення 2А):**
   - **Presentation Layer (UI):**
     - `WebMobileUI` («Web/Mobile Client UI (incl. CheckoutUI)») — клієнтський інтерфейс (`UC_Login`, `UC_ViewMenu`, `UC_Checkout`, `UC_Track`).
     - `StaffPortalUI` («Staff Portal UI») — інтерфейс менеджера ресторану та кур'єра (`UC_ManageMenu`, `UC_ConfirmOrder`, `UC_TakeDelivery`, `UC_UpdateStatus`).
   - **API / Backend Layer:**
     - `APIGateway` («API Gateway / Controllers: AuthController, OrderController, CatalogController, DeliveryController») — прийом HTTPS-запитів (`NFR-04`), маршрутизація та первинна валідація.
   - **Business Logic Layer (Services):**
     - `AuthService` — внутрішня автентифікація та перевірка ролей `User.role` (Рішення 3А).
     - `OrderService` — оркестрація оформлення та оплати замовлення (`Draft` → `Paid`), перевірка кошика й адреси (`FR-01`, `FR-03`, `FR-04`, `FR-07`).
     - `PromoService` — перевірка та застосування реферальних промокодів 5% (`FR-02`).
     - `NotificationService` — асинхронне формування та ретрай сповіщень клієнту й менеджеру (`FR-05`, `FR-06`).
     - `CatalogService` — управління меню та обраними стравами (`UC_ViewMenu`, `UC_ManageMenu`).
     - `DeliveryService` — призначення кур'єрів та оновлення статусів доставки (`UC_TakeDelivery`, `UC_UpdateStatus`, `UC_Track`).
   - **Data Access Layer (Repositories):**
     - `UserRepository` (таблиця `User`).
     - `OrderRepository` (таблиці `Order`, `OrderItem`, `Address` — із Завдання 3).
     - `PromoRepository` (таблиці `Promotion`, `Friendship`).
     - `CatalogDeliveryRepository` (таблиці `MenuItem`, `Favorite`, `Courier`).
   - **Database Layer:**
     - `PostgreSQL` («FoodDelivery DB (PostgreSQL): User, Address, Courier, Order, OrderItem, MenuItem, Promotion, Friendship, Favorite»).

3. **Зовнішні інтеграції поза межею `FoodDelivery` (Рішення 3А):**
   - `PaymentGateway` — зовнішній платіжний шлюз (інтерфейс `IPaymentGateway`), що забезпечує оплату без зберігання карткових даних і CVV у БД (`NFR-04`) та таймаут 15 с (`NFR-02`).
   - `NotificationProvider` («SMS / Email Provider») — зовнішній шлюз доставки сповіщень (інтерфейс `INotificationGateway`).
   - Зовнішні `OAuth / Identity Provider` та картографічні сервіси свідомо винесено поза обсяг (`out of scope`).

### Критерії якості діаграми:
- Усі залежності спрямовані строго зверху вниз (`UI → API → Services → Data Access → DB`), без прямих звернень UI до БД і без циклічних залежностей.
- Кожен компонент надає щонайменше один `provided`-інтерфейс або споживає відповідний `required`-інтерфейс.
- Синтаксис Mermaid має рендеритися на GitHub без жодної помилки та без накладання стрілок одна на одну.
