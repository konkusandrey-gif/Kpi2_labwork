# Специфікація архітектури системи `FoodDelivery` (`spec.md`)

## 1. Мета та обсяг
Побудувати статичну компонентну модель архітектури системи `FoodDelivery` (UML Component Diagram) із чіткою межею системи (`System Boundary`), шаруватою структурою, `provided`/`required`-інтерфейсами та зовнішніми інтеграціями на основі артефактів Завдань 1–4 (`er_v2.mmd`, `uc.mmd`, `requirements.md`, `scenarios.md`, діаграм послідовності та процесу `UC_Checkout`).

## 2. Внутрішні шари та компоненти (у межах `FoodDelivery`)
1. **Presentation Layer (UI):**
   * `Web/Mobile Client UI` (включно з `CheckoutUI` із Завдання 3) — інтерфейс клієнта (`UC_Login`, `UC_ViewMenu`, `UC_Checkout`, `UC_Track`).
   * `Staff Portal UI` — інтерфейс менеджера ресторану та кур'єра (`UC_ManageMenu`, `UC_ConfirmOrder`, `UC_TakeDelivery`, `UC_UpdateStatus`).
2. **API / Backend Layer:**
   * `API Gateway / Controllers` (включно з `OrderController`, а також контролерами автентифікації, меню та доставки) — прийом HTTPS-запитів (`NFR-04`), маршрутизація та первинна валідація.
3. **Business Logic Layer (Services):**
   * `OrderService` — управління життєвим циклом замовлення (`Draft` → `Paid` → доставка), перевірка кошика й адреси (`FR-01`, `FR-03`, `FR-04`, `FR-07`).
   * `PromoService` — перевірка та застосування реферальних промокодів 5% (`FR-02`).
   * `NotificationService` — формування та асинхронна відправка сповіщень клієнту та менеджеру (`FR-05`, `FR-06`).
   * `Catalog & Delivery Service` — управління меню, обраним (`Favorite`) та призначенням кур'єрів (`UC_ViewMenu`, `UC_TakeDelivery`).
4. **Data Access Layer:**
   * `OrderRepository` (із Завдання 3) — доступ до замовлень, позицій та адрес.
   * `User & Promo Repository` — доступ до користувачів, ролей, промокодів і реферальних зв'язків.
   * `Menu & Delivery Repository` — доступ до страв, обраного та кур'єрів.
5. **Database Layer:**
   * `FoodDelivery DB (PostgreSQL)` — реляційна БД із 9 таблицями (`User`, `Address`, `Courier`, `Order`, `OrderItem`, `MenuItem`, `Promotion`, `Friendship`, `Favorite`).

## 3. Зовнішні інтеграції (поза межею системи)
* `PaymentGateway` — зовнішній платіжний шлюз для списання коштів без зберігання карткових даних у системі (`NFR-04`, таймаут 15 с за `NFR-02`).
* `SMS / Email Provider` — зовнішній шлюз доставки push/SMS/email-сповіщень клієнтам і менеджерам.
* `OAuth / Identity Provider` — зовнішній сервіс автентифікації користувачів (`UC_Login`).

## 4. Інтерфейси (`provided` / `required`) та критерії прийняття
* Кожен компонент надає чіткий контракт (`provided`, напр. `IOrderRepository`, `IPromoService`, `IPaymentGateway`, `INotificationProvider`) і залежить лише від інтерфейсів нижчого рівня або зовнішніх шлюзів (`required`).
* Залежності спрямовані зверху вниз (`UI → API → Services → Data Access → DB`), без прямих звернень UI до БД і без циклічних залежностей.
* Використано єдину декларативну нотацію (PlantUML Component Diagram або Mermaid) із графічним рендером.
