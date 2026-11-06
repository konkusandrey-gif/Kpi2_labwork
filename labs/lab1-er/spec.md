Spec: Lab 1 - ER-модель FoodDelivery

Намір
Змоделювати дані платформи для замовлення та доставки їжі як ER-модель: сутності, атрибути, зв'язки, кардинальності, первинні та зовнішні ключі. Домен обрано на основі реального процесу доставки їжі (з кошиком, відстеженням кур'єра та реферальною системою).

Сутності та атрибути:

User (Клієнт)

-id (PK, UUID)
-name
- email (unique)
-phone (unique)
-createdAt

Address (Адреса доставки)

-id (PK, UUID)
-userId (FK -> User.id)
-city
-street
-house
-isDefault (boolean, замовчуванням false)

Friendship (Реферальна система друзів)

-id (PK, UUID)
-inviterId (FK -> User.id)
-inviteeId (FK -> User.id)
-createdAt
-Unique: (inviterId, inviteeId) - не можна запросити ту саму людину двічі

Promotion (Акція)

-id (PK, UUID)
-name
-discountPercentage (int)
-activeUntil (datetime)
-MenuItem (Страва)
-id (PK, UUID)
-name
-description (nullable)
-price (float)
-promotionId (FK -> Promotion.id, nullable - страва може бути без акції)

Favorite (Обране)

-id (PK, UUID)
-userId (FK -> User.id)
-menuItemId (FK -> MenuItem.id)
-createdAt

Unique: (userId, menuItemId) - один лайк на страву від одного клієнта

-Courier (Кур'єр)
-id (PK, UUID)
-name
-phone (unique)
-vehicleType (string)

Order (Замовлення)

-id (PK, UUID)
-userId (FK -> User.id)
-addressId (FK -> Address.id, nullable для кошика)
-courierId (FK -> Courier.id, nullable поки не призначено)
-status (string: Draft, Paid, Cooking, Delivering, Delivered, Cancelled)
-totalAmount (float)
-createdAt

OrderItem (Позиція замовлення)

-id (PK, UUID)
-orderId (FK -> Order.id)
-menuItemId (FK -> MenuItem.id)
-quantity (int)
-priceAtPurchase (float - фіксує ціну на момент замовлення)
-Unique: (orderId, menuItemId) - однакові страви групуються через quantity

Зв'язки

-User - Address: one-to-many (один клієнт має багато збережених адрес).
-User - User: many-to-many через Friendship (self-referencing, асоціативна сутність для рефералів).
-Promotion - MenuItem: one-to-many (одна акція застосовується до багатьох страв).
-User - MenuItem: many-to-many через Favorite (асоціативна сутність).
-User - Order: one-to-many (один клієнт робить багато замовлень).
-Address - Order: one-to-many (на одну адресу оформлюється багато замовлень).
-Courier - Order: one-to-many (один кур'єр доставляє багато замовлень).
-Order - MenuItem: many-to-many через OrderItem (асоціативна сутність, містить кількість та ціну).

Критерії прийняття

-Кардинальності коректні (1:N). Немає хибних прямих зв'язків "багато-до-багатьох".
-M:N зв'язки розгорнуті виключно через асоціативні сутності (Friendship, Favorite, OrderItem) з власними атрибутами, бо чистий M:N без атрибутів — це зв'язок, а не сутність.
-Первинні ключі (PK) та зовнішні ключі (FK) чітко визначені для всіх сутностей.
-Модель нормалізована (3NF - немає транзитивних залежностей).
-ER-рендер збігається з описом у spec.
-Unique-обмеження зафіксовані для Friendship, Favorite та OrderItem.
-Unique на email та phone користувача і кур'єра.
-Усі Unique-обмеження (на email, phone, та композитні ключі у Favorite/Friendship) МАЮТЬ бути явно відображені на ER-діаграмі через коментарі або нотацію UK.
-Усі ідентифікатори (id) повинні мати тип UUID, а не string.
