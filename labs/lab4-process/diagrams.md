# Діаграма активностей процесу оформлення та оплати замовлення (`UC_Checkout`)

```mermaid
flowchart TD
    subgraph Client ["Client"]
        Start(("Start")) --> FillCart["Сформувати кошик та натиснути «Оформити і оплатити»"]
        ShowCartErr["Повідомити про порожній кошик"] --> EndEmptyCart((("End")))
        ShowAddrErr["Підсвітити поле адреси («Вкажіть адресу доставки»)"] --> EndNoAddr((("End")))
        AcceptClientNotif[/"Прийняти підтвердження оплати"/]
        ShowPayErr["Відобразити помилку оплати та кнопку «Повторити спробу»"] --> EndPayFail((("End")))
    end

    subgraph RestaurantManager ["RestaurantManager"]
        AcceptMgrNotif[/"Прийняти деталі нового замовлення"/]
    end

    subgraph OrderSystem ["OrderSystem"]
        CheckCart{"Кошик не порожній?"}
        CheckCart -->|"Ні"| ShowCartErr
        CheckCart -->|"Так"| CheckAddr{"Адресу доставки вказано?"}
        CheckAddr -->|"Ні"| ShowAddrErr
        CheckAddr -->|"Так"| SaveAddr["Зберегти адресу доставки"]

        SaveAddr --> CheckPromoInput{"Промокод введено?"}
        CheckPromoInput -->|"Ні"| CalcNoPromo["Розрахувати базову суму замовлення"]
        CheckPromoInput -->|"Так"| ValidatePromo{"Промокод активний та реферальний зв'язок підтверджено?"}
        ValidatePromo -->|"Так"| CalcDiscount["Застосувати знижку 5% та перерахувати суму"]
        ValidatePromo -->|"Ні"| CalcInvalidPromo["Залишити базову суму без знижки"]

        CalcNoPromo --> MergeSum{" "}
        CalcDiscount --> MergeSum
        CalcInvalidPromo --> MergeSum

        MergeSum --> FixTotal["Зафіксувати підсумкову суму замовлення"]
        FixTotal --> SendPayReq>"Надіслати запит на оплату"]
        SendPayReq --> WaitRace{"Очікування відповіді шлюзу (до 15 с)"}

        WaitRace -->|"Таймер 15 с спрацював"| TimeoutEvent[/"Подія часу: вичерпано 15 с"/]
        WaitRace -->|"Відповідь надійшла"| AcceptPayResp[/"Прийняти відповідь платіжного шлюзу"/]
        AcceptPayResp --> CheckPayStatus{"Оплата успішна?"}

        CheckPayStatus -->|"Так (APPROVED)"| MarkPaid["Позначити замовлення оплаченим"]
        MarkPaid --> ForkNotif[["Fork"]]
        ForkNotif --> SendMgrNotif>"Надіслати сповіщення менеджеру"]
        ForkNotif --> SendClientNotif>"Надіслати чек клієнту"]

        JoinNotif[["Join"]] --> EndSuccess((("End")))

        CheckPayStatus -->|"Ні (DECLINED)"| KeepDraft["Залишити замовлення у статусі «Чернетка» та зберегти кошик"]
        TimeoutEvent --> KeepDraft
        KeepDraft --> ShowPayErr
    end

    subgraph PaymentGateway ["PaymentGateway"]
        AcceptPayReq[/"Прийняти запит на списання коштів"/] --> ProcessPay["Обробити платіжну транзакцію"]
        ProcessPay --> SendPayResp>"Повернути результат оплати"]
    end

    FillCart --> CheckCart
    SendPayReq -.-> AcceptPayReq
    SendPayResp -.-> AcceptPayResp
    SendMgrNotif -.-> AcceptMgrNotif
    SendClientNotif -.-> AcceptClientNotif
    AcceptMgrNotif --> JoinNotif
    AcceptClientNotif --> JoinNotif

    classDef swimlane fill:#111827,stroke:#374151,stroke-width:1.5px,color:#f9fafb;
    classDef action fill:#1f2937,stroke:#60a5fa,stroke-width:1.5px,color:#f3f4f6;
    classDef decision fill:#312e81,stroke:#a5b4fc,stroke-width:1.5px,color:#eef2ff;
    classDef signal fill:#064e3b,stroke:#34d399,stroke-width:1.5px,color:#ecfdf5;
    classDef sync fill:#3f1d2e,stroke:#f472b6,stroke-width:1.5px,color:#fdf2f8;
    classDef terminal fill:#1e293b,stroke:#94a3b8,stroke-width:2px,color:#f8fafc;

    class Client,OrderSystem,PaymentGateway,RestaurantManager swimlane;
    class FillCart,ShowCartErr,ShowAddrErr,ShowPayErr,SaveAddr,CalcNoPromo,CalcDiscount,CalcInvalidPromo,FixTotal,MarkPaid,KeepDraft,ProcessPay action;
    class CheckCart,CheckAddr,CheckPromoInput,ValidatePromo,MergeSum,WaitRace,CheckPayStatus decision;
    class AcceptClientNotif,SendPayReq,TimeoutEvent,AcceptPayResp,SendMgrNotif,SendClientNotif,AcceptPayReq,SendPayResp,AcceptMgrNotif signal;
    class ForkNotif,JoinNotif sync;
    class Start,EndEmptyCart,EndNoAddr,EndPayFail,EndSuccess terminal;
