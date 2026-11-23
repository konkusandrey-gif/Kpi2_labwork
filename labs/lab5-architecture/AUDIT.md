# Аудит та верифікація компонентної архітектури `FoodDelivery` (`AUDIT.md`)

## 1. Трасування компонентів і залежностей до моделей Завдань 1–4
Усі компоненти фінальної діаграми (`labs/lab5-architecture/diagrams.md`, коміт `d03f926`) перевірено на повну відповідність попереднім артефактам:

| Компонент / Шар | Дані (Завдання 1–2, `er_v2.mmd`) | Вимоги та UC (Завдання 2) | Взаємодія та процес (Завдання 3–4) |
| :--- | :--- | :--- | :--- |
| `WebMobileUI` (UI) | Відображення сутностей `MenuItem`, `Order`, `Address`, `Favorite` | `UC_Login`, `UC_ViewMenu`, `UC_Checkout`, `UC_Track`, `FR-01`, `FR-02`, `FR-08`, `NFR-01`, `NFR-03`, `NFR-05` | Відповідає `CheckoutUI` (Лаба 3) та зоні `Client` (Лаба 4) |
| `StaffPortalUI` (UI) | Робота із сутностями `MenuItem`, `Order`, `Courier` | `UC_ManageMenu`, `UC_ConfirmOrder`, `UC_TakeDelivery`, `UC_UpdateStatus`, `FR-05` | Відповідає ролі `RestaurantManager` (Лаби 3–4) та актору Кур'єр |
| `APIGateway` (API) | Маршрутизація запитів за `userId`, `orderId`, `promoCode` | Точка входу HTTPS TLS 1.3 (`NFR-04`), валідація (`FR-01`–`FR-04`) | Узгоджено з `OrderController` (Лаба 3) та зоною `OrderSystem` (Лаба 4) |
| `AuthService` + `UserRepository` | Таблиця `User` (`id`, `email`, `phone`, `role`) | `UC_Login`, розмежування прав клієнта, менеджера та кур'єра | Передумова авторизованого стану (`Given` у `SC-01`–`SC-04`) |
| `OrderService` + `OrderRepository` | Таблиці `Order`, `OrderItem`, `Address` | `UC_Checkout`, `UC_Pay`, `FR-01`, `FR-03`, `FR-04`, `FR-07`, `NFR-02` | Відповідає `OrderRepository` та життєвому циклу `Draft → Paid` (Лаби 3–4) |
| `PromoService` + `PromoRepository` | Таблиці `Promotion`, `Friendship` | `UC_ApplyPromo`, `FR-02` (реферальна знижка 5%) | Пряме трасування до `PromoService.validateReferralCode()` (Лаба 3) |
| `NotificationService` | Використовує контакти `User` (`email`, `phone`) та `Order.id` | `FR-05` (сповіщення ресторану), `FR-06` (підтвердження клієнту) | Відповідає `NotificationService` та паралельному блоку `Fork`/`Join` (Лаба 4) |
| `CatalogService`, `DeliveryService` + `CatalogDeliveryRepository` | Таблиці `MenuItem`, `Favorite`, `Courier` | `UC_ViewMenu`, `UC_ManageMenu`, `UC_TakeDelivery`, `UC_UpdateStatus`, `UC_Track` | Розширює ядро замовлення на повний цикл доставки `FoodDelivery` |
| `PostgreSQL` (DB) | Усі 9 таблиць з `er_v2.mmd` із первинними та зовнішніми ключами | Без збереження реквізитів банківських карток і CVV (`NFR-04`) | Єдине сховище станів замовлень і довідників |

## 2. Перевірка межі системи та контрактів (`provided` / `required`)
* **Чіткість межі системи (`System Boundary: FoodDelivery`):** внутрішні 5 шарів ізольовані контейнером `FoodDelivery`; назовні винесені лише `PaymentGateway` (`IPaymentGateway`, вимоги `NFR-04` і таймаут 15 с за `NFR-02`) та `SMS / Email Provider` (`INotificationGateway`, вимоги `FR-05`, `FR-06`).
* **Коректність інтерфейсів і напрямку залежностей:** кожен модуль надає явний контракт (`provide`) та споживає інтерфейс нижчого рівня (`require`); залежності спрямовані строго зверху вниз (`UI → API → Services → Data Access → DB`), прямі звернення UI до БД та цикли відсутні.

## 3. Виявлені розбіжності та внесені виправлення (у `spec.md` та діаграмі `diagrams.md`)

| № | Що було не так (у чернетці `spec.md` або першому рендері `diagrams.md`) | Мій слід (промпт / рев'ю / оновлення `spec.md`) | Коміти виправлення та змінені артефакти |
| :--- | :--- | :--- | :--- |
| **1** | **Нечітка декомпозиція сервісів та репозиторіїв (`spec.md` `6542f23`):** у першій чернетці каталог і доставка були злиті в один модуль `Catalog & Delivery Service`, не було виділено `OrderService`, а репозиторії не мали 1:1 прив'язки до 9 таблиць `er_v2.mmd`. | `ai/01_prompt_gap_analysis.md` (`e71b2cd`), відповідь (`f60930d`), рішення **2А** в оновленому `spec.md` (`602ce25`) | `602ce25` (`spec.md`), `d283eb6` (`diagrams.md` — виділено 5 окремих сервісів і 4 доменні репозиторії) |
| **2** | **Суперечність межі системи щодо автентифікації (`spec.md` `6542f23`):** чернетка містила зовнішній `OAuth / Identity Provider`, що суперечило власній таблиці `User` (`email`, `phone`, `role`) в `er_v2.mmd` та вимогам Завдання 2. | `ai/01_prompt_gap_analysis_answer.md` (`f60930d`), рішення **3А** в `spec.md` (`602ce25`) | `602ce25` (`spec.md`), `d283eb6` (`diagrams.md` — `AuthService` перенесено всередину `FoodDelivery`, назовні залишено лише `PaymentGateway` та `SMS / Email Provider`) |
| **3** | **Невизначена нотація компонентів та інтерфейсів (`spec.md` `6542f23`):** у початковій специфікації не було зафіксовано єдину нотацію та формат відображення `provided`/`required`-контрактів. | `ai/01_prompt_gap_analysis.md` (`e71b2cd`), рішення **1А** в `spec.md` (`602ce25`), `ai/02_prompt_generate_diagram.md` (`1c6878e`) | `602ce25` (`spec.md` — зафіксовано Mermaid `flowchart TB` та порти `([I...])`), `d283eb6` (`diagrams.md`) |
| **4** | **Баг візуального рендера підписів на діаграмі (`diagrams.md` `d283eb6`):** після генерації діаграми парсер Mermaid на GitHub сприйняв екрановані `&lt;&lt;require&gt;&gt;` та `&lt;&lt;provide&gt;&gt;` як порожні HTML-теги `<>`, а `«External System»` відобразив як `-External System-`. | Візуальне рев'ю згенерованої діаграми на GitHub та виправлення синтаксису підписів стрілок | `d03f926` (`diagrams.md` — замінено екрановані кутові дужки на чисті підписи `"require"`, `"provide"` та `[External System]`) |
