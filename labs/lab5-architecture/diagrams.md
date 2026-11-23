# Компонентна модель архітектури системи `FoodDelivery` (`diagrams.md`)

Діаграма побудована в єдиній декларативній нотації **Mermaid `flowchart TB`** відповідно до `spec.md` та артефактів Завдань 1–4:
* **Межа системи (`System Boundary: FoodDelivery`)** відокремлює внутрішні модулі від зовнішніх платіжних та комунікаційних шлюзів.
* **Порти інтерфейсів `([I...])`:** зв'язок `---|"provide"|` показує компонент, що надає контракт, а `-->|"require"|` — компонент, який його споживає.

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

            OrderService -->|"require"| I_Promo
            I_Promo ---|"provide"| PromoService

            OrderService -->|"require"| I_Notif
            I_Notif ---|"provide"| NotificationService
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
        WebMobileUI -->|"require"| I_REST
        StaffPortalUI -->|"require"| I_REST
        I_REST ---|"provide"| APIGateway

        %% API -> Services
        APIGateway -->|"require"| I_Auth
        APIGateway -->|"require"| I_Order
        APIGateway -->|"require"| I_Catalog
        APIGateway -->|"require"| I_Delivery

        I_Auth ---|"provide"| AuthService
        I_Order ---|"provide"| OrderService
        I_Catalog ---|"provide"| CatalogService
        I_Delivery ---|"provide"| DeliveryService

        %% Services -> Data Access
        AuthService -->|"require"| I_UserRepo
        I_UserRepo ---|"provide"| UserRepository

        OrderService -->|"require"| I_OrderRepo
        I_OrderRepo ---|"provide"| OrderRepository

        PromoService -->|"require"| I_PromoRepo
        I_PromoRepo ---|"provide"| PromoRepository

        CatalogService -->|"require"| I_CatDelRepo
        DeliveryService -->|"require"| I_CatDelRepo
        I_CatDelRepo ---|"provide"| CatalogDeliveryRepository

        %% Data Access -> Database
        UserRepository -->|"require"| I_SQL
        OrderRepository -->|"require"| I_SQL
        PromoRepository -->|"require"| I_SQL
        CatalogDeliveryRepository -->|"require"| I_SQL
        I_SQL ---|"provide"| PostgreSQL
    end

    %% External Integrations (Outside System Boundary)
    I_Payment(["IPaymentGateway"])
    PaymentGateway["[External System]<br/>PaymentGateway<br/><i>Оплата без зберігання карт (NFR-04, timeout 15s)</i>"]

    I_GatewayNotif(["INotificationGateway"])
    NotificationProvider["[External System]<br/>SMS / Email Provider<br/><i>Доставка SMS/Email/Push сповіщень</i>"]

    OrderService -->|"require"| I_Payment
    I_Payment ---|"provide"| PaymentGateway

    NotificationService -->|"require"| I_GatewayNotif
    I_GatewayNotif ---|"provide"| NotificationProvider

    %% Styling
    classDef component fill:#1f2937,stroke:#60a5fa,stroke-width:1.5px,color:#f3f4f6;
    classDef interface fill:#312e81,stroke:#a5b4fc,stroke-width:1.5px,color:#eef2ff;
    classDef db fill:#064e3b,stroke:#34d399,stroke-width:1.5px,color:#ecfdf5;
    classDef external fill:#3f1d2e,stroke:#f472b6,stroke-width:1.5px,color:#fdf2f8;

    class WebMobileUI,StaffPortalUI,APIGateway,AuthService,OrderService,PromoService,NotificationService,CatalogService,DeliveryService,UserRepository,OrderRepository,PromoRepository,CatalogDeliveryRepository component;
    class I_REST,I_Auth,I_Order,I_Catalog,I_Delivery,I_Promo,I_Notif,I_UserRepo,I_OrderRepo,I_PromoRepo,I_CatDelRepo,I_SQL,I_Payment,I_GatewayNotif interface;
    class PostgreSQL db;
    class PaymentGateway,NotificationProvider external;
