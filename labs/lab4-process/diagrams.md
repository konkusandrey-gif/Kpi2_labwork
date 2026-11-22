# Activity-діаграма процесу оформлення та оплати замовлення (`UC_Checkout`)

Діаграма описує наскрізну поведінку процесу `UC_Checkout` (`SC-01`–`SC-04`, `FR-01`–`FR-08`, `NFR-02`) у єдиній декларативній нотації **Mermaid `flowchart`** із зонами відповідальності учасників (`subgraph`), стандартизованими сигналами (`send signal` / `accept event`), розгалуженнями (`if/else`) та паралельною гілкою сповіщень (`fork/join`).

```mermaid
flowchart TD
    subgraph Client ["Client (Клієнт)"]
        Start(["Start: Ініціація оформлення"])
        C1["Натиснути 'Перейти до оплати' (clickProceedToPayment)"]
        Acc_Cli[/"Accept: sendPaymentConfirmation(userId, orderId, finalAmount)"/]
        C_Receipt["Отримати чек про оплату (FR-06)"]
    end

    subgraph OrderSystem ["OrderSystem (CheckoutUI, OrderController, OrderRepository, PromoService, NotificationService)"]
        S1["OrderController / OrderRepository: Завантажити чернетку Order (Draft) та позиції OrderItem"]
        D1{"itemsCount > 0?"}
        S_Empty["CheckoutUI: Показати попередження 'Кошик порожній'"]

        D2{"addressId != null? (SC-04)"}
        S_NoAddr["CheckoutUI: highlightAddressField() та блокування оплати (FR-01)"]
        S_Addr["OrderRepository: Зберегти адресу доставки saveOrderAddress(orderId, addressId)"]

        D3{"Введено промокод? (SC-02)"}
        S_Base["OrderController: Розрахувати базову суму замовлення (SC-01)"]
        S_PromoCheck["PromoService / OrderRepository: validateReferralCode() у Promotion та Friendship"]
        D4{"Промокод валідний?"}
        S_Disc["PromoService / OrderRepository: Застосувати знижку 5% (applyOrderDiscount)"]
        S_PromoErr["CheckoutUI: showPromoError() та залишити базову суму"]

        S_Fix["OrderRepository: Зафіксувати підсумкову суму (finalAmount)"]
        Sig_Pay>"Send: chargePayment(orderId, finalAmount)"]

        Acc_Ok[/"Accept: paymentResponse(APPROVED)"/]
        Acc_Decl[/"Accept: paymentResponse(DECLINED)"/]
        Acc_Tout[/"Accept: Timeout > 15s (No Response)"/]

        S_Decl["OrderRepository / CheckoutUI: keepDraftAndPreserveCart(), showPaymentErrorWithRetry() (SC-03)"]
        S_Tout["OrderRepository / CheckoutUI: Зафіксувати таймаут (NFR-02), зберегти Draft, показати 'Повторити спробу'"]

        S_Paid["OrderRepository: Оновити статус замовлення updateOrderStatus(orderId, 'Paid')"]
        Fork[["Fork"]]
        Sig_Mgr>"Send: notifyRestaurantManager(orderId)"]
        Sig_Cli>"Send: sendPaymentConfirmation(userId, orderId, finalAmount)"]
        Join[["Join"]]

        S_Done["CheckoutUI: Відобразити підтвердження замовлення showOrderConfirmation(orderId)"]
        EndFail(["End: Оплату не завершено (Draft збережено)"])
        EndOk(["End: Замовлення успішно оформлено (Paid)"])
    end

    subgraph PaymentGateway ["PaymentGateway (Платіжний шлюз)"]
        PG_Acc[/"Accept: chargePayment(orderId, finalAmount)"/]
        PG_Proc["Обробити платіжну транзакцію"]
        PG_Out{"Результат обробки шлюзом"}
        PG_Ok>"Send: paymentResponse(APPROVED, txId)"]
        PG_Decl>"Send: paymentResponse(DECLINED)"]
        PG_None["Відповідь не надійшла"]
    end

    subgraph RestaurantManager ["RestaurantManager (Менеджер ресторану)"]
        Acc_Mgr[/"Accept: notifyRestaurantManager(orderId)"/]
        RM_Act["Отримати сповіщення про нове оплачене замовлення (FR-05)"]
    end

    %% Потік виконання
    Start --> C1
    C1 --> S1
    S1 --> D1

    D1 -- "Ні" --> S_Empty
    S_Empty --> EndFail
    D1 -- "Так" --> D2

    D2 -- "Ні" --> S_NoAddr
    S_NoAddr --> EndFail
    D2 -- "Так" --> S_Addr

    S_Addr --> D3
    D3 -- "Ні" --> S_Base
    D3 -- "Так" --> S_PromoCheck
    S_PromoCheck --> D4
    D4 -- "Так" --> S_Disc
    D4 -- "Ні" --> S_PromoErr

    S_Base --> S_Fix
    S_Disc --> S_Fix
    S_PromoErr --> S_Fix

    S_Fix --> Sig_Pay
    Sig_Pay --> PG_Acc
    PG_Acc --> PG_Proc
    PG_Proc --> PG_Out

    PG_Out -- "Списання успішне" --> PG_Ok
    PG_Out -- "Відмова в оплаті" --> PG_Decl
    PG_Out -- "Збій / затримка мережі" --> PG_None

    PG_Ok --> Acc_Ok
    PG_Decl --> Acc_Decl
    PG_None -. "таймаут > 15 с" .-> Acc_Tout

    Acc_Decl --> S_Decl
    S_Decl --> EndFail
    Acc_Tout --> S_Tout
    S_Tout --> EndFail

    Acc_Ok --> S_Paid
    S_Paid --> Fork

    Fork --> Sig_Mgr
    Fork --> Sig_Cli

    Sig_Mgr --> Acc_Mgr
    Acc_Mgr --> RM_Act
    Sig_Mgr -- "асинхронно (фоновий ретрай при збої)" --> Join

    Sig_Cli --> Acc_Cli
    Acc_Cli --> C_Receipt
    Sig_Cli -- "асинхронно" --> Join

    Join --> S_Done
    S_Done --> EndOk
