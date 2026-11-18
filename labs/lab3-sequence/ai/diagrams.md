# Діаграми послідовності (Sequence Diagrams — FoodDelivery)

## SC-01: Успішне оформлення та оплата замовлення (Happy Path)

```mermaid
sequenceDiagram
    actor Client
    participant UI as CheckoutUI
    participant Ctrl as OrderController
    participant Repo as OrderRepository
    participant PG as PaymentGateway
    participant Notif as NotificationService

    Client->>UI: clickCheckout(orderId, addressId)
    activate UI
    UI->>Ctrl: submitOrder(userId, orderId, addressId)
    activate Ctrl

    Ctrl->>Repo: findOrderWithItems(orderId)
    activate Repo
    Repo-->>Ctrl: order(status="Draft"), items[]
    deactivate Repo

    loop Для кожної позиції в OrderItem
        Ctrl->>Ctrl: calcItemSubtotal(item.priceAtPurchase, item.quantity)
    end

    Ctrl->>Repo: saveOrderTotal(orderId, addressId, totalAmount)
    activate Repo
    Repo-->>Ctrl: updatedOrder(totalAmount)
    deactivate Repo

    Ctrl->>PG: chargePayment(orderId, totalAmount)
    activate PG
    PG-->>Ctrl: paymentResponse(status="APPROVED", txId)
    deactivate PG

    alt Платіж підтверджено (APPROVED)
        Ctrl->>Repo: updateOrderStatus(orderId, "Paid")
        activate Repo
        Repo-->>Ctrl: order(status="Paid")
        deactivate Repo

        Ctrl->>Notif: notifyRestaurantManager(orderId)
        activate Notif
        Notif-->>Ctrl: managerNotified(true)
        deactivate Notif

        Ctrl->>Notif: sendPaymentConfirmation(userId, orderId, totalAmount)
        activate Notif
        Notif-->>Ctrl: clientNotified(true)
        deactivate Notif
    else Помилка підтвердження
        Ctrl-->>UI: renderPaymentError("Не вдалося підтвердити оплату")
    end

    Ctrl-->>UI: orderCompleted(orderId, status="Paid")
    deactivate Ctrl
    UI-->>Client: showOrderConfirmation(orderId)
    deactivate UI
```

---

## SC-02: Застосування реферального промокоду перед оплатою (Альтернативний шлях)

```mermaid
sequenceDiagram
    actor Client
    participant UI as CheckoutUI
    participant Ctrl as OrderController
    participant Promo as PromoService
    participant Repo as OrderRepository
    participant PG as PaymentGateway
    participant Notif as NotificationService

    Client->>UI: enterPromoCode(orderId, promoCode)
    activate UI
    UI->>Ctrl: applyReferralPromo(userId, orderId, promoCode)
    activate Ctrl

    Ctrl->>Promo: validateReferralCode(userId, promoCode)
    activate Promo
    Promo-->>Ctrl: promoDetails(isValid=true, discountPercentage=5)
    deactivate Promo

    alt Промокод дійсний та активний
        Ctrl->>Repo: getOrderSubtotal(orderId)
        activate Repo
        Repo-->>Ctrl: subtotalAmount
        deactivate Repo

        Ctrl->>Promo: calculateDiscount(subtotalAmount, 5)
        activate Promo
        Promo-->>Ctrl: discountAmount, finalAmount
        deactivate Promo

        Ctrl->>Repo: applyOrderDiscount(orderId, promoCode, discountAmount, finalAmount)
        activate Repo
        Repo-->>Ctrl: updatedOrder(finalAmount)
        deactivate Repo
    else Промокод недійсний або прострочений
        Ctrl-->>UI: showPromoError("Промокод недійсний")
    end

    Ctrl-->>UI: updateCartSummary(discountAmount, finalAmount)
    deactivate Ctrl
    UI-->>Client: displayDiscountedTotal(finalAmount)
    deactivate UI

    Client->>UI: confirmPayment(orderId, addressId)
    activate UI
    UI->>Ctrl: submitOrder(userId, orderId, addressId)
    activate Ctrl

    Ctrl->>PG: chargePayment(orderId, finalAmount)
    activate PG
    PG-->>Ctrl: paymentResponse(status="APPROVED", txId)
    deactivate PG

    opt Оплата успішна
        Ctrl->>Repo: updateOrderStatus(orderId, "Paid")
        activate Repo
        Repo-->>Ctrl: order(status="Paid")
        deactivate Repo

        Ctrl->>Notif: notifyRestaurantManager(orderId)
        activate Notif
        Notif-->>Ctrl: managerNotified(true)
        deactivate Notif
    end

    Ctrl-->>UI: orderCompleted(orderId, status="Paid")
    deactivate Ctrl
    UI-->>Client: showOrderConfirmation(orderId)
    deactivate UI
```

---

## SC-03: Відмова платіжної системи під час оплати (Помилковий шлях А)

```mermaid
sequenceDiagram
    actor Client
    participant UI as CheckoutUI
    participant Ctrl as OrderController
    participant Repo as OrderRepository
    participant PG as PaymentGateway

    Client->>UI: confirmPayment(orderId, addressId)
    activate UI
    UI->>Ctrl: submitOrder(userId, orderId, addressId)
    activate Ctrl

    Ctrl->>Repo: findOrderWithItems(orderId)
    activate Repo
    Repo-->>Ctrl: order(status="Draft"), items[]
    deactivate Repo

    Ctrl->>PG: chargePayment(orderId, totalAmount)
    activate PG
    PG-->>Ctrl: paymentResponse(status="DECLINED", reason="INSUFFICIENT_FUNDS")
    deactivate PG

    alt Відмова платіжного шлюзу (DECLINED)
        Ctrl->>Repo: keepDraftAndPreserveCart(orderId)
        activate Repo
        Repo-->>Ctrl: order(status="Draft", itemsCount > 0)
        deactivate Repo
        Note over Ctrl: Менеджер ресторану НЕ сповіщується
    else Платіж успішний
        Ctrl->>Repo: updateOrderStatus(orderId, "Paid")
        activate Repo
        Repo-->>Ctrl: order(status="Paid")
        deactivate Repo
    end

    Ctrl-->>UI: paymentDeclined(orderId, status="Draft", retryAllowed=true)
    deactivate Ctrl
    UI-->>Client: showPaymentErrorWithRetry("Помилка оплати", "Повторити спробу")
    deactivate UI
```

---

## SC-04: Спроба оформлення замовлення без вказаної адреси доставки (Помилковий шлях Б)

```mermaid
sequenceDiagram
    actor Client
    participant UI as CheckoutUI
    participant Ctrl as OrderController
    participant Repo as OrderRepository

    Client->>UI: clickProceedToPayment(orderId, addressId=null)
    activate UI
    UI->>Ctrl: initiateCheckout(userId, orderId, addressId=null)
    activate Ctrl

    Ctrl->>Repo: getOrderDraft(orderId)
    activate Repo
    Repo-->>Ctrl: order(status="Draft", addressId=null, itemsCount > 0)
    deactivate Repo

    alt Адреса доставки порожня (addressId == null)
        Ctrl->>Ctrl: blockPaymentGatewayRedirect(orderId)
        Note over Ctrl: Запит до PaymentGateway не формується
    else Адреса доставки валідна
        Ctrl->>Repo: attachDeliveryAddress(orderId, addressId)
        activate Repo
        Repo-->>Ctrl: order(addressId)
        deactivate Repo
    end

    Ctrl-->>UI: validationFailed(field="addressId", message="Будь ласка, вкажіть адресу доставки")
    deactivate Ctrl
    UI-->>Client: highlightAddressField("Будь ласка, вкажіть адресу доставки")
    deactivate UI
```
