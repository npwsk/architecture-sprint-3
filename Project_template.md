# Project_template

# Задание 1. Анализ и планирование

### 1. Описание функциональности монолитного приложения

**Управление отоплением:**

- Пользователи могут удалённо включать/выключать отопление в своих домах.

**Мониторинг температуры:**

- Пользователи могут просматривать текущую температуру в своих домах через веб-интерфейс.
- Система получает данные о температуре с датчиков, установленных в домах.

### 2. Анализ архитектуры монолитного приложения

- Язык программирования: Java
- База данных: PostgreSQL
- Архитектура: Монолитная, все компоненты системы (обработка запросов, бизнес-логика, работа с данными) находятся в рамках одного приложения.
- Взаимодействие: Синхронное, запросы обрабатываются последовательно.
- Масштабируемость: Ограничена, так как монолит сложно масштабировать по частям.
- Развёртывание: Требует остановки всего приложения.

### 3. Определение доменов и границы контекстов

Домены:
- Управление устройствами
  -  Отправка команд управления отоплением, освещением, воротами и т.п.
  -  Контроль состояния устройств.
- Телеметрия и мониторинг
  - Получение данных от датчиков (температура, движение, состояние устройств).
  - Просмотр истории и текущих значений.
- Модульность и сценарии
  -  Выбор и настройка модулей умного дома.
  -  Конфигурация пользовательских сценариев (автоматизация поведения устройств).
- Подключение устройств
  -  Привязка новых устройств к системе.
  -  Поддержка самостоятельной установки и настройки пользователем.
- Управление пользователями и доступом
  -  Регистрация, авторизация, многопользовательский доступ к дому.
  -  Управление правами и ролями в пределах одного дома или аккаунта.

### **4. Проблемы монолитного решения**

- Синхронная архитектура: Отсутствие асинхронных вызовов и реактивного взаимодействия приводит к низкой производительности при увеличении нагрузки и затрудняет масштабирование.
- Зависимость от выезда специалистов: Подключение датчиков требует физического присутствия специалиста, что является дорогостоящим и неэффективным процессом.
- Ограниченное количество подключений: Текущая система поддерживает только 100 веб-клиентов и 100 модулей управления отоплением. Это недостаточно для целевой экосистемы, которая должна поддерживать множество устройств и пользователей.
- Отсутствие стандартных протоколов: Неподдержка стандартных протоколов усложняет подключение устройств от сторонних производителей, ограничивая выбор пользователей и возможности расширения экосистемы.

### 5. Визуализация контекста системы — диаграмма С4

```mermaid

C4Context
    title Контекстная диаграмма As-Is системы «Тёплый дом»
    
    Person(Пользователь, "Пользователь", "Управляет отоплением через веб-интерфейс")
    
    System_Ext(Устройства, "Датчики и реле отопления", "Устройства, устанавливаемые в домах пользователей")
    System(ТеплыйДом, "Система «Тёплый дом»", "Монолитное Java-приложение для управления отоплением")

    Rel(Пользователь, ТеплыйДом, "Управление отоплением и просмотр температуры")
    Rel(ТеплыйДом, Устройства, "Синхронное взаимодействие для получения данных и управления")

    UpdateLayoutConfig($c4ShapeInRow="2", $c4BoundaryInRow="1")
```

# Задание 2. Проектирование микросервисной архитектуры

В этом задании вам нужно предоставить только диаграммы в модели C4. Мы не просим вас отдельно описывать получившиеся микросервисы и то, как вы определили взаимодействия между компонентами To-Be системы. Если вы правильно подготовите диаграммы C4, они и так это покажут.

## **Диаграмма контейнеров (Containers)**

```mermaid
C4Container
title Контейнерная диаграмма системы «Тёплый дом» (группировка по доменам)

Person(Пользователь, "Пользователь системы")

System_Boundary(ТеплыйДом, "Экосистема 'Тёплый дом'") {

  Container(WebApp, "Веб-приложение", "Next.js / SPA", "UI для управления домом, устройствами и сценариями")
  Container(API_Gateway, "API Gateway", "Node.js / Spring Cloud Gateway", "Точка входа. Аутентификация, маршрутизация")

  Container_Boundary(DeviceManagement, "Управление устройствами") {
    Container(DeviceControl, "Device Control Service", "Java/Kotlin", "Управление устройствами: команды, состояние")
    ContainerDb(DeviceControlDB, "Device Control DB", "PostgreSQL", "Состояние и история команд")
  }

  Container_Boundary(TelemetryDomain, "Телеметрия и мониторинг") {
    Container(Telemetry, "Telemetry Service", "Java/Kotlin", "Сбор и хранение телеметрии")
    ContainerDb(TelemetryDB, "Telemetry DB", "TimescaleDB/PostgreSQL", "Временные ряды телеметрии")
  }

  Container_Boundary(AutomationDomain, "Модульность и сценарии") {
    Container(Automation, "Automation Service", "Node.js/Java", "Сценарии автоматизации")
    ContainerDb(AutomationDB, "Automation DB", "PostgreSQL", "Правила и сценарии")
  }

  Container_Boundary(DeviceOnboarding, "Подключение устройств") {
    Container(DeviceProvisioning, "Device Provisioning Service", "Python/Go", "Регистрация и настройка устройств")
    ContainerDb(DeviceProvisioningDB, "Provisioning DB", "PostgreSQL", "Данные устройств и привязки")
    Container(DeviceConnectors, "Device Connector Service", "Go", "Протоколы: MQTT, Z-Wave и пр.")
  }

  Container_Boundary(AuthDomain, "Пользователи и доступ") {
    Container(Identity, "Identity & Access Service", "Keycloak/Auth0", "Аутентификация и авторизация")
    ContainerDb(IdentityDB, "Identity DB", "PostgreSQL", "Пользователи, роли, права")
  }
}

System_Ext(Device, "Физические устройства", "Датчики, реле и др.")

Rel(Пользователь, WebApp, "Использует")
Rel(WebApp, API_Gateway, "Запросы")
Rel(API_Gateway, Identity, "Аутентификация")
Rel(API_Gateway, DeviceControl, "Команды устройств")
Rel(API_Gateway, Telemetry, "Запрос телеметрии")
Rel(API_Gateway, Automation, "Управление сценариями")
Rel(API_Gateway, DeviceProvisioning, "Регистрация устройств")

Rel(DeviceControl, DeviceControlDB, "Чтение/запись")
Rel(Telemetry, TelemetryDB, "Чтение/запись")
Rel(Automation, AutomationDB, "Чтение/запись")
Rel(DeviceProvisioning, DeviceProvisioningDB, "Чтение/запись")
Rel(Identity, IdentityDB, "Чтение/запись")

Rel(DeviceControl, DeviceConnectors, "Передаёт команды")
Rel(DeviceConnectors, Device, "Управляет устройствами")
Rel(Device, DeviceConnectors, "Отправляет телеметрию")
Rel(DeviceConnectors, Telemetry, "Публикует телеметрию")

Rel(Telemetry, Automation, "Публикация событий")
Rel(DeviceControl, Automation, "События состояния")
Rel(Automation, DeviceControl, "Запуск действий")

Rel(DeviceProvisioning, DeviceConnectors, "Активация адаптера")

UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="5")

```

## **Диаграмма компонентов (Components)**

### Микросервис управления устройствами (Device Control Service)

```mermaid

C4Component
title Компонентная диаграмма — Device Control Service

Container_Boundary(DeviceControl, "Device Control Service") {
  Component(DeviceControlAPI, "DeviceControlAPI", "REST/gRPC Controller", "Принимает команды управления от клиентов и маршрутизирует их во внутренние компоненты")
  Component(CommandHandler, "CommandHandler", "Service", "Обрабатывает команды и отправляет их устройствам через коннекторы")
  Component(DeviceStateManager, "DeviceStateManager", "Service", "Отслеживает и обновляет состояние устройств")
  Component(CommandLogRepository, "CommandLogRepository", "Repository", "Сохраняет историю команд и их статусы")
  Component(DeviceRegistryClient, "DeviceRegistryClient", "Client", "Получает информацию об устройствах из Provisioning Service")
  Component(EventPublisher, "EventPublisher", "Adapter", "Публикует события об изменениях состояния или результатах команд")
  ContainerDb(DeviceControlDB, "Device Control DB", "PostgreSQL", "Хранит историю команд и состояния устройств")

  Rel(DeviceControlAPI, CommandHandler, "Делегирует выполнение команд")
  Rel(CommandHandler, DeviceRegistryClient, "Проверяет устройство и права")
  Rel(CommandHandler, DeviceStateManager, "Получает и обновляет состояние")
  Rel(CommandHandler, CommandLogRepository, "Логирует команду")
  Rel(CommandHandler, EventPublisher, "Публикует результат выполнения")
  Rel(DeviceStateManager, EventPublisher, "Публикует события о смене состояния")
  Rel(CommandLogRepository, DeviceControlDB, "Чтение/запись")
  Rel(DeviceStateManager, DeviceControlDB, "Чтение/запись")
}

Container(API_Gateway, "API Gateway", "Node.js / Spring Cloud Gateway", "Маршрутизатор внешних запросов")
Container(DeviceConnectors, "Device Connector Service", "Go", "Обмен данными с физическими устройствами")
Container(Automation, "Automation Service", "Node.js/Java", "Сценарии автоматизации")
Container(DeviceProvisioning, "Device Provisioning Service", "Python/Go", "Регистрация и настройка устройств")

Rel(API_Gateway, DeviceControlAPI, "Команды управления устройствами")
Rel(CommandHandler, DeviceConnectors, "Передаёт команды на исполнение")
Rel(DeviceRegistryClient, DeviceProvisioning, "Получает данные об устройствах")
Rel(EventPublisher, Automation, "Публикует события о действиях и состояниях")

```

### Микросервис телеметрии и мониторинга (Telemetry Service)

```mermaid
C4Component
title Компонентная диаграмма — Telemetry Service

Container_Boundary(Telemetry, "Telemetry Service") {
  Component(TelemetryAPI, "TelemetryAPI", "REST/gRPC Controller", "Принимает запросы на получение телеметрии")
  Component(TelemetryProcessor, "TelemetryProcessor", "Service", "Обрабатывает и нормализует телеметрические данные")
  Component(TelemetryDataRepository, "TelemetryDataRepository", "Repository", "Сохраняет и извлекает данные из базы данных")
  Component(EventPublisher, "EventPublisher", "Adapter", "Публикует события о телеметрии в другие сервисы")
  Component(TelemetryDataCache, "TelemetryDataCache", "Cache", "Кэширует последние данные для быстрого доступа")

  ContainerDb(TelemetryDB, "Telemetry DB", "TimescaleDB/PostgreSQL", "Хранение временных рядов телеметрии")

  Rel(TelemetryAPI, TelemetryProcessor, "Отправляет данные для обработки")
  Rel(TelemetryProcessor, TelemetryDataRepository, "Записывает обработанные данные")
  Rel(TelemetryProcessor, TelemetryDataCache, "Обновляет кэшированные данные")
  Rel(TelemetryProcessor, EventPublisher, "Публикует события телеметрии")

  Rel(TelemetryDataRepository, TelemetryDB, "Чтение/запись")
  Rel(TelemetryDataCache, TelemetryDB, "Чтение")
}

```

### Микросервис модульности и сценариев (Automation Service)

```mermaid
C4Component
title Компонентная диаграмма — Automation Service

Container_Boundary(Automation, "Automation Service") {
  Component(AutomationAPI, "AutomationAPI", "REST/gRPC Controller", "Принимает запросы на создание, изменение или удаление сценариев автоматизации")
  Component(AutomationEngine, "AutomationEngine", "Service", "Исполняет сценарии автоматизации на основе входных событий")
  Component(AutomationRuleRepository, "AutomationRuleRepository", "Repository", "Сохраняет и извлекает правила автоматизации")
  Component(EventSubscriber, "EventSubscriber", "Service", "Подписывается на события из других сервисов и инициирует действия")
  Component(ActionExecutor, "ActionExecutor", "Service", "Исполняет действия согласно правилам автоматизации")
  ContainerDb(AutomationDB, "Automation DB", "PostgreSQL", "Хранение сценариев и конфигураций автоматизации")

  Rel(AutomationAPI, AutomationEngine, "Создаёт и изменяет сценарии автоматизации")
  Rel(AutomationEngine, ActionExecutor, "Передаёт действия для исполнения")
  Rel(AutomationEngine, AutomationRuleRepository, "Извлекает правила автоматизации")
  Rel(EventSubscriber, AutomationEngine, "Отправляет события в Automation Engine для исполнения")
  Rel(ActionExecutor, DeviceControlService, "Отправляет команды управления устройствами")
  Rel(ActionExecutor, TelemetryService, "Отправляет команды по телеметрии")
  Rel(AutomationRuleRepository, AutomationDB, "Чтение/запись")
}

Container(DeviceControlService, "Device Control Service", "Java", "Управляет устройствами через их API, включая отопление, свет и другие модули умного дома")
Container(TelemetryService, "Telemetry Service", "Java", "Сбор, обработка и хранение данных телеметрии")

```

### Микросервис подключения устройств (Device Control Service)

```mermaid
C4Component
title Компонентная диаграмма — Device Control Service

Container_Boundary(DeviceControlService, "Device Control Service") {
  Component(DeviceControlAPI, "DeviceControlAPI", "REST/gRPC Controller", "Принимает запросы на управление устройствами от внешних сервисов")
  Component(DeviceManager, "DeviceManager", "Service", "Управляет состоянием подключённых устройств")
  Component(DeviceCommandExecutor, "DeviceCommandExecutor", "Service", "Исполняет команды управления устройствами")
  Component(DeviceRepository, "DeviceRepository", "Repository", "Сохраняет и извлекает данные о подключённых устройствах")
  Component(EventPublisher, "EventPublisher", "Adapter", "Публикует события о состоянии устройств в другие сервисы")

  ContainerDb(DeviceControlDB, "Device Control DB", "PostgreSQL", "Хранение информации о устройствах и их состояниях")
  
  Rel(DeviceControlAPI, DeviceManager, "Отправляет запросы на управление состоянием устройств")
  Rel(DeviceManager, DeviceCommandExecutor, "Передаёт команды для исполнения")
  Rel(DeviceManager, DeviceRepository, "Извлекает и сохраняет информацию об устройствах")
  Rel(DeviceCommandExecutor, DeviceRepository, "Обновляет информацию о выполненных действиях с устройствами")
  Rel(DeviceManager, EventPublisher, "Публикует события об изменениях состояния устройств")
  Rel(DeviceRepository, DeviceControlDB, "Чтение/запись")
}

UpdateLayoutConfig($c4ShapeInRow="2", $c4BoundaryInRow="1")
```

### Микросервис управления пользователями и доступом (User & Access Management Service)

```mermaid
C4Component
title Компонентная диаграмма — User & Access Management Service

Container_Boundary(UserAccess, "User & Access Management Service") {
  Component(UserAPI, "UserAPI", "REST/gRPC Controller", "Обрабатывает запросы на регистрацию, вход и управление профилем пользователя")
  Component(AuthenticationService, "AuthenticationService", "Service", "Обрабатывает аутентификацию пользователей и управление сессиями")
  Component(AuthorizationService, "AuthorizationService", "Service", "Управляет правами доступа пользователей к ресурсам")
  Component(UserRepository, "UserRepository", "Repository", "Сохраняет информацию о пользователях, их профилях и данных для аутентификации")
  Component(RoleRepository, "RoleRepository", "Repository", "Управляет ролями пользователей и их правами доступа")
  Component(EventPublisher, "EventPublisher", "Adapter", "Публикует события о действиях пользователей, связанных с безопасностью и доступом")

  Rel(UserAPI, AuthenticationService, "Обрабатывает запросы на аутентификацию")
  Rel(UserAPI, AuthorizationService, "Запрашивает права доступа для пользователей")
  Rel(AuthenticationService, UserRepository, "Проверяет и хранит данные о пользователях")
  Rel(AuthorizationService, RoleRepository, "Проверяет права доступа на основе ролей")
  Rel(EventPublisher, UserRepository, "Публикует события об изменениях в профилях пользователей")
  Rel(UserRepository, UserAccessDB, "Чтение/запись")
  Rel(RoleRepository, UserAccessDB, "Чтение/запись")

  ContainerDb(UserAccessDB, "User Access DB", "PostgreSQL", "Хранение информации о пользователях, их ролях и правах доступа")
}

```

## **Диаграмма кода (Code)**

Добавьте одну диаграмму или несколько.

```mermaid
classDiagram
    class Device {
        +String id
        +String name
        +String type
        +String state
        +String config
        +turnOn()
        +turnOff()
        +getStatus()
        +updateConfig(config: String)
    }

    class DeviceManager {
        +Map<String, Device> devices
        +getDevice(deviceId: String): Device
        +addDevice(device: Device)
        +removeDevice(deviceId: String)
        +updateDeviceState(deviceId: String, state: String)
    }

    class DeviceCommandExecutor {
        +executeCommand(device: Device, command: String)
        +turnOnDevice(device: Device)
        +turnOffDevice(device: Device)
        +sendCommandToDevice(device: Device, command: String)
    }

    class DeviceRepository {
        +saveDevice(device: Device)
        +getDeviceById(deviceId: String): Device
        +deleteDevice(deviceId: String)
    }

    class EventPublisher {
        +publishEvent(device: Device, event: String)
        +notifyOtherServices(event: String)
    }

    DeviceManager --> Device : Управляет
    DeviceCommandExecutor --> Device : Исполняет команды
    DeviceRepository --> Device : Хранит
    EventPublisher --> Device : Публикует события

    DeviceManager --> DeviceRepository : Читает/Записывает
    DeviceManager --> EventPublisher : Генерирует события

```

# Задание 3. Разработка ER-диаграммы

```mermaid
erDiagram
    User {
        string id
        string email
        string password_hash
        string first_name
        string last_name
        string role_id
        string created_at
        string updated_at
    }
    
    House {
        string id
        string user_id
        string address
        string name
        string created_at
        string updated_at
    }

    Device {
        string id
        string type_id
        string house_id
        string serial_number
        string status
        string name
        string config
        string created_at
        string updated_at
    }

    DeviceType {
        string id
        string name
        string description
        string created_at
        string updated_at
    }

    Module {
        string id
        string house_id
        string name
        string type
        string status
        string created_at
        string updated_at
    }

    TelemetryData {
        string id
        string device_id
        string timestamp
        string data
        string unit
        string created_at
    }

    DeviceCommand {
        string id
        string device_id
        string command
        string timestamp
        string status
    }

    Role {
        string id
        string name
        string permissions
        string created_at
        string updated_at
    }

    SystemSettings {
        string id
        string temperature_threshold
        string timezone
        string notifications_enabled
        string created_at
        string updated_at
    }

    %% Связи
    User }|--o{ House : "владеет"
    House ||--o| Device : "содержит"
    DeviceType ||--o| Device : "определяет"
    Device ||--o| TelemetryData : "генерирует"
    Device ||--o| DeviceCommand : "принимает"
    Role ||--o| User : "назначена"
    House ||--o| Module : "включает"
    Module ||--o| Device : "управляет"
    User ||--o| SystemSettings : "имеет"

```

- User: Пользователь имеет связь с домами, которые ему принадлежат, и с настройками системы.

- House: Дом содержит устройства и модули.

- Device: Устройство связано с типом устройства, домом, может генерировать данные телеметрии и принимать команды.

- DeviceType: Тип устройства может быть связан с множеством устройств.

- Module: Модуль связан с домом и может управлять несколькими устройствами.

- TelemetryData: Записи телеметрии генерируются устройствами.

- DeviceCommand: Устройство может получать множество команд.

- Role: Роль пользователя может быть присвоена нескольким пользователям.

- SystemSettings: Каждый пользователь может иметь свои настройки системы.



# ❌ Задание 4. Создание и документирование API

### 1. Тип API

Укажите, какой тип API вы будете использовать для взаимодействия микросервисов. Объясните своё решение.

### 2. Документация API

Здесь приложите ссылки на документацию API для микросервисов, которые вы спроектировали в первой части проектной работы. Для документирования используйте Swagger/OpenAPI или AsyncAPI.