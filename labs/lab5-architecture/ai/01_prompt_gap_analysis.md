# Промпт 1: Аналіз прогалин та уточнення специфікації архітектури (`spec.md`)

Ти — системний архітектор. Проаналізуй мою початкову специфікацію компонентної архітектури системи `FoodDelivery` (`labs/lab5-architecture/spec.md`) на узгодженість із попередніми артефактами проєкту:
- модель даних `er_v2.mmd` (9 сутностей: `User`, `Address`, `Courier`, `Order`, `OrderItem`, `MenuItem`, `Promotion`, `Friendship`, `Favorite`);
- вимоги та прецеденти Завдання 2 (`FR-01`–`FR-08`, `NFR-01`–`NFR-05`, сценарії `SC-01`–`SC-04`, а також `UC_Login`, `UC_ViewMenu`, `UC_Track`, `UC_TakeDelivery`, `UC_UpdateStatus`, `UC_ManageMenu`, `UC_ConfirmOrder`);
- класи та взаємодії Завдань 3 і 4 (`CheckoutUI`, `OrderController`, `OrderRepository`, `PromoService`, `NotificationService`, зовнішні системи `PaymentGateway` і `RestaurantManager`).

Не генеруй діаграму одразу. Виконай 3 кроки:
1. Знайди 3 головні архітектурні прогалини або неоднозначності у `spec.md` (щодо нотації, деталізації компонентів/інтерфейсів та меж системи).
2. Постав уточнювальні запитання за кожною прогалиною.
3. Запропонуй по 2 конкретні варіанти вирішення для кожного пункту, щоб я міг зафіксувати рішення в оновленому `spec.md` та `ADR`.
