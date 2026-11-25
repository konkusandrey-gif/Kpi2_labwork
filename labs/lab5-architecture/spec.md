# Специфікація архітектури системи FoodDelivery

1. Мета, обрана нотація та межа системи
Мета цього завдання — описати компонентну архітектуру системи FoodDelivery на основі моделей Завдань 1–4 (er_v2.mmd, uc.mmd, requirements.md, scenarios.md, діаграм послідовності та процесу UC_Checkout).
Для моделювання обрано єдину нотацію Mermaid flowchart TB, оскільки вона зберігається у вигляді звичайного тексту в репозиторії й одразу рендериться на GitHub без додаткових зображень (детальніше обґрунтовано в adr/01-layered-component-architecture.md). Межа системи (System Boundary: FoodDelivery) відділяє внутрішні програмні модулі від зовнішніх сервісів. Контракти взаємодії між компонентами позначено круглими вузлами інтерфейсів: зв'язок provide показує, який компонент реалізує інтерфейс, а require — який компонент його використовує.

2. Внутрішні шари та компоненти в межах FoodDelivery
Архітектуру побудовано за п'ятишаровою моделлю із залежностями зверху вниз:
- Шар інтерфейсу користувача (Presentation Layer): WebMobileUI (включає CheckoutUI, обслуговує сценарії клієнта UC_Login, UC_ViewMenu, UC_Checkout, UC_Track) та StaffPortalUI (робочий кабінет менеджера ресторану й кур'єра для UC_ManageMenu, UC_ConfirmOrder, UC_TakeDelivery, UC_UpdateStatus).
- Шар API та маршрутизації (API / Backend Layer): APIGateway із контролерами AuthController, OrderController, CatalogController та DeliveryController. Забезпечує точку входу по HTTPS (NFR-04), первинну валідацію запитів і передачу викликів у сервісний шар.
- Шар бізнес-логіки (Business Logic Layer): шість спеціалізованих сервісів — AuthService (вхід у систему та перевірка ролей User.role), OrderService (оформлення замовлення, перевірка кошика й адреси, перехід статусів Draft → Paid), PromoService (перевірка реферальних промокодів зі знижкою 5% за FR-02), NotificationService (асинхронна відправка та повторні спроби сповіщень за FR-05 і FR-06), CatalogService (робота з меню та обраними стравами) і DeliveryService (призначення кур'єрів та оновлення статусів доставки).
- Шар доступу до даних (Data Access Layer): чотири репозиторії, що інкапсулюють роботу з таблицями БД — UserRepository (таблиця User), OrderRepository (Order, OrderItem, Address), PromoRepository (Promotion, Friendship) та CatalogDeliveryRepository (MenuItem, Favorite, Courier).
- Шар збереження даних (Database Layer): реляційна база даних PostgreSQL, що містить усі 9 сутностей з er_v2.mmd.

3. Зовнішні інтеграції та межі обсягу (Out of Scope)
За межу FoodDelivery винесено лише два зовнішні сервіси, необхідні за вимогами: PaymentGateway (інтерфейс IPaymentGateway для проведення оплати без збереження реквізитів банківських карток за NFR-04 та з таймаутом 15 с за NFR-02) і NotificationProvider (інтерфейс INotificationGateway для доставки SMS та Email сповіщень за FR-05, FR-06).
Поза обсягом цієї архітектурної моделі свідомо залишено зовнішню OAuth-автентифікацію (вхід реалізовано через внутрішній AuthService) та сторонні картографічні сервіси навігації.

4. Правила залежностей
Усі залежності спрямовані суворо зверху вниз (UI → API → Services → Data Access → DB) через визначені інтерфейси, без прямих звернень інтерфейсу користувача до бази даних і без циклічних зв'язків між модулями.
