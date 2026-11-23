# Компонентна модель архітектури системи `FoodDelivery` (`diagrams.md`)

Діаграма побудована в єдиній декларативній нотації **Mermaid `flowchart TB`** відповідно до `spec.md` та артефактів Завдань 1–4:
* **Межа системи (`System Boundary: FoodDelivery`)** відокремлює внутрішні модулі від зовнішніх платіжних та комунікаційних шлюзів.
* **Порти інтерфейсів `([I...])`:** зв'язок `---|"<<provide>>"|` показує компонент, що надає контракт, а `-->|"<<require>>"|` — компонент, який його споживає.

```mermaid
flowchart TB
    subgraph FoodDelivery ["System Boundary: FoodDelivery"]
        direction TB

        subgraph UI ["Presentation Layer (UI)"]
            direction LR
            WebMobileUI["Web/Mobile Client UI<br/>(incl. CheckoutUI)<br/><i>UC_Login, UC_ViewMenu, UC_Checkout, UC_Track</i>"]
            StaffPortalUI["Staff Portal UI<br/><i>UC_ManageMenu, UC_ConfirmOrder,<br/>UC_TakeDelivery, UC_UpdateStatus</i>"]
        end

        I_REST(["I_RestAPI (HTTPS / TLS 1.3)"])

        subgraph API ["API / Backend Layer"]
            APIGateway["API Gateway / Controllers<br/><i>AuthController, OrderController,<br/>CatalogController, DeliveryController</i>"]
        end

        I_Auth(["IAuthService"])
        I_Order(["IOrderService"])
        I_Catalog(["ICatalogService"])
        I_Delivery(["IDeliveryService"])

        subgraph Services ["Business Logic Layer (Services)"]
            direction TB
            AuthService["AuthService<br/><i>Автентифікація та ролі User.role</i>"]
            OrderService["OrderService<br/><i>Оркестрація UC_Checkout (Draft → Paid)</i>"]
            PromoService["PromoService<br/><i>Реферальні промокоди 5% (FR-02)</i>"]
            NotificationService["NotificationService<br/><i>Асинхронні сповіщення та ретрай (FR-05, FR-06)</i>"]
            CatalogService["CatalogService<br/><i>Управління меню та обраним</i>"]
            DeliveryService["DeliveryService<br/><i>Призначення кур'єрів та статуси доставки</i>"]

            I_Promo(["IPromoService"])
            I_Notif(["INotificationService"])

            OrderService -->|"&lt;&lt;require&gt;&gt;"| I_Promo
            I_Promo ---|"&lt;&lt;provide&gt;&gt;"| PromoService

            OrderService -->|"&lt;&lt;require&gt;&gt;"| I_Notif
            I_Notif ---|"&lt;&lt;provide&gt;&gt;"| NotificationService
        end

        I_UserRepo(["IUserRepository"])
        I_OrderRepo(["IOrderRepository"])
        I_PromoRepo(["IPromoRepository"])
        I_CatDelRepo(["ICatalogDeliveryRepository"])

        subgraph DataAccess ["Data Access Layer (Repositories)"]
            direction LR
            UserRepository["UserRepository<br/><i>Таблиця: User</i>"]
            OrderRepository["OrderRepository<br/><i>Таблиці: Order, OrderItem, Address</i>"]
            PromoRepository["PromoRepository<br/><i>Таблиці: Promotion, Friendship</i>"]
            CatalogDeliveryRepository["CatalogDeliveryRepository<br/><i>Таблиці: MenuItem, Favorite, Courier</i>"]
        end

        I_SQL(["ISqlDataAccess (PostgreSQL Driver)"])

        subgraph Database ["Database Layer"]
            PostgreSQL[("FoodDelivery DB (PostgreSQL)<br/>User, Address, Courier, Order, OrderItem,<br/>MenuItem, Promotion, Friendship, Favorite")]
        end

        %% UI -> API
        WebMobileUI -->|"&lt;&lt;require&gt;&gt;"| I_REST
        StaffPortalUI -->|"&lt;&lt;require&gt;&gt;"| I_REST
        I_REST ---|"&lt;&lt;provide&gt;&gt;"| APIGateway

        %% API -> Services
        APIGateway -->|"&lt;&lt;require&gt;&gt;"| I_Auth
        APIGateway -->|"&lt;&lt;require&gt;&gt;"| I_Order
        APIGateway -->|"&lt;&lt;require&gt;&gt;"| I_Catalog
        APIGateway -->|"&lt;&lt;require&gt;&gt;"| I_Delivery

        I_Auth ---|"&lt;&lt;provide&gt;&gt;"| AuthService
        I_Order ---|"&lt;&lt;provide&gt;&gt;"| OrderService
        I_Catalog ---|"&lt;&lt;provide&gt;&gt;"| CatalogService
        I_Delivery ---|"&lt;&lt;provide&gt;&gt;"| DeliveryService

        %% Services -> Data Access
        AuthService -->|"&lt;&lt;require&gt;&gt;"| I_UserRepo
        I_UserRepo ---|"&lt;&lt;provide&gt;&gt;"| UserRepository

        OrderService -->|"&lt;&lt;require&gt;&gt;"| I_OrderRepo
        I_OrderRepo ---|"&lt;&lt;provide&gt;&gt;"| OrderRepository

        PromoService -->|"&lt;&lt;require&gt;&gt;"| I_PromoRepo
        I_PromoRepo ---|"&lt;&lt;provide&gt;&gt;"| PromoRepository

        CatalogService -->|"&lt;&lt;require&gt;&gt;"| I_CatDelRepo
        DeliveryService -->|"&lt;&lt;require&gt;&gt;"| I_CatDelRepo
        I_CatDelRepo ---|"&lt;&lt;provide&gt;&gt;"| CatalogDeliveryRepository

        %% Data Access -> Database
        UserRepository -->|"&lt;&lt;require&gt;&gt;"| I_SQL
        OrderRepository -->|"&lt;&lt;require&gt;&gt;"| I_SQL
        PromoRepository -->|"&lt;&lt;require&gt;&gt;"| I_SQL
        CatalogDeliveryRepository -->|"&lt;&lt;require&gt;&gt;"| I_SQL
        I_SQL ---|"&lt;&lt;provide&gt;&gt;"| PostgreSQL
    end

    %% External Integrations (Outside System Boundary)
    I_Payment(["IPaymentGateway"])
    PaymentGateway["«External System»<br/>PaymentGateway<br/><i>Оплата без зберігання карт (NFR-04, timeout 15s)</i>"]

    I_GatewayNotif(["INotificationGateway"])
    NotificationProvider["«External System»<br/>SMS / Email Provider<br/><i>Доставка SMS/Email/Push сповіщень</i>"]

    OrderService -->|"&lt;&lt;require&gt;&gt;"| I_Payment
    I_Payment ---|"&lt;&lt;provide&gt;&gt;"| PaymentGateway

    NotificationService -->|"&lt;&lt;require&gt;&gt;"| I_GatewayNotif
    I_GatewayNotif ---|"&lt;&lt;provide&gt;&gt;"| NotificationProvider

    %% Styling
    classDef component fill:#1f2937,stroke:#60a5fa,stroke-width:1.5px,color:#f3f4f6;
    classDef interface fill:#312e81,stroke:#a5b4fc,stroke-width:1.5px,color:#eef2ff;
    classDef db fill:#064e3b,stroke:#34d399,stroke-width:1.5px,color:#ecfdf5;
    classDef external fill:#3f1d2e,stroke:#f472b6,stroke-width:1.5px,color:#fdf2f8;

    class WebMobileUI,StaffPortalUI,APIGateway,AuthService,OrderService,PromoService,NotificationService,CatalogService,DeliveryService,UserRepository,OrderRepository,PromoRepository,CatalogDeliveryRepository component;
    class I_REST,I_Auth,I_Order,I_Catalog,I_Delivery,I_Promo,I_Notif,I_UserRepo,I_OrderRepo,I_PromoRepo,I_CatDelRepo,I_SQL,I_Payment,I_GatewayNotif interface;
    class PostgreSQL db;
    class PaymentGateway,NotificationProvider external;
