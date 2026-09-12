# Account core

```mermaid
flowchart TB
    subgraph Core
        direction TB
        C[User]
        G[Access]
        H[Notification]
        S[Storage]
        K[Audit]
    end

    subgraph Prod
        direction TB
        D[Product 1]
        E[Product 2]
        F[Product 3]
    end
    A[Frontend] -->|get, create, update| B[API Gateway]
    Z[Public API] -->|get, create, update| B[API Gateway]
    B --> C[User]
    B --> D[Product 1]
    B --> E[Product 2]
    B --> F[Product 3]
    C --> H[Notification]
    D --> G[Access]
    E --> G[Access]
    F --> G[Access]
    D --> C[User]
    E --> C[User]
    F --> C[User]
    D --> H[Notification]
    E --> H[Notification]
    F --> H[Notification]
    D --> S[Storage]
    E --> S[Storage]
    F --> S[Storage]
    G --> K[Audit]
    H --> K[Audit]
    S --> K[Audit]
    C --> K[Audit]
    D --> K[Audit]
    E --> K[Audit]
    F --> K[Audit]
```

## Компоненты

1. **User**:
    * Цель:
        * Управление жизненным циклом пользователя (регистрация, профиль)
        * Проверка учетных данных (логин/пароль)
        * Выпуск и валидация токенов сессии.
    * НЕ Цель:
        * Что юзер юзает
        * Какие у него права
    * Что хранит:
        * Учетные записи (ID, пароль, email, телефон и тд)
        * Активные сессии
2. **Access**:
    * Цель:
        * Хранение ролей и доступов на действия/ресурсы
    * НЕ Цель:
        * Не знает, на что конкрнкретно и какие это права
    * Что хранит:
        * Права доступа (Кто, На что, Роли, Доступы)
3. **Storage**:
    * Цель:
        * Хранение и отдача файлов
    * НЕ Цель:
        * Не знает, для чего эти файлы
    * Что хранит:
        * Файлы, метаданные
4. **Notification**:
    * Цель:
        * Создать сообщение
        * Отправить сообщение
    * НЕ Цель:
        * Не знает смысла уведомления (зачем оно отправляется)
        * Не принимает решения, нужно ли отправлять
    * Что хранит:
        * Шаблоны писем
        * Статусы уведомлений (доставлено, ошибка, ин прогресс)

## Карта владения

| Сущность / Данные        | Владелец | Кто использует (Читает/Модифицирует через API)       |
|--------------------------|----------|------------------------------------------------------|
| User (ID, логин, Пароль) | User     | Frontend, Продукты (только чтение)                   |
| Session (JWT Токены)     | User     | API Gateway (для пропуска запросов)                  |
| Permissions и Roles      | Access   | Продукты (для проверки допусков)                     |
| Resource permissions     | Access   | Продукты (передают в Access инфу, кто создал ресурс) |
| Product Data             | Product  | Frontend (через API конкретного продукта)            |
| Files                    | Storage  | Продукты, Frontend                                   |
| Audit Logs               | Audit    | Админы или *анал*итики (Security Team)               |

## Зависимости

```mermaid
flowchart TB
    A[API Gateway]
    U[User]
    P[Products]
    A -->|Валидация токенов при каждом запросе| U
    A -->|Маршрутизация запросов в соответствующие продукты| P
```

```mermaid
flowchart TB
    P[Products]
    A[Access]
    U[User]
    S[Storage]
    N[Notification]
    L[Audit]
    P -->|Получение профиля пользователя| U
    P -->|проверка прав перед выполнением| A
    P -->|Сохранение и чтение файлов| S
    P -->|Отправка уведомлений| N
    P -->|Логгирвоание| L
```

```mermaid
flowchart TB
    N[Notification]
    U[User]
    L[Audit]
    U -->|подтверждение регистрации, сброс пароля и тд| N
    U -->|вход с нового устройства, блокировка аккаунта, крч всякое, что нужно будет посомтреть| L
```

```mermaid
flowchart TB
    A[Access]
    L[Audit]
    A -->|Фиксация изменений в доступах| L
```

## Контракты

```mermaid
classDiagram
    class User {
        RegisterUser()
        Authenticate()
        CheckSession()
        KillSession()
        GetUserInfo()
    }

    class Access {
        CheckAccess()
        GrantAccess()
        DeleteAccess()
        GetUserPermissions()
    }

    class Notification {
        SendNotification()
    }

    class Storage {
        StartDownload()
        Upload()
        Delete()
        Copy()
    }
```

## Use cases

```mermaid
---
title: регистрация юзера
---
flowchart LR
    A[UI]
    B[API Gateway]
    C[User]
    D[Mailer]
    E[Audit]
    A --> B --> C -.-> D
    C -.-> E
```

```mermaid
---
title: вход пользователя
---
flowchart LR
    A[UI]
    B[API Gateway]
    C[User]
    D[Audit]
    A --> B --> C
    C -.-> D
```

```mermaid
---
title: обращение к продукту
---
flowchart LR
    A[UI]
    B[API Gateway]
    C[User]
    D[Product]
    A -->|Запрос с токеном| B
    B --> C
    C --> B
    B --> D
```

```mermaid
---
title: проверка доступа к ресурсу
---
flowchart LR
    A[UI]
    B[API Gateway]
    C[Product B]
    D[Access]
    A --> B --> C
    C --> D
    D --> C
```

## Как добавить новый продукт

1. Регистрируем политики и роли в **Access**
2. Пишем продукт
3. Добавить роутинг для продукта в **API Gateway**
4. Получаем деньги и идем купить бамбук

## Доработки (TODO)

* Core-компонент `Subscription` -- определяет, какие продукты есть (купил) у юзера

[//]: # (TODO)
* Из `User` вынести сессии, способы 