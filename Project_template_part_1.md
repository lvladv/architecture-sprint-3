Это шаблон для решения **первой части** проектной работы. Структура этого файла повторяет структуру заданий. Заполняйте его по мере работы над решением.

# Задание 1. Анализ и планирование

Чтобы составить документ с описанием текущей архитектуры приложения, можно часть информации взять из описания компании условия задания. Это нормально.

### 1. Описание функциональности монолитного приложения

**Управление отоплением:**

- Пользователи могу управлять отоплением удаленно 
- Система поддерживает управление от сервера к датчику

**Мониторинг температуры:**

- Пользователи могут просматривать текущую температуру в своих домах через веб-интерфейс.
- Система поддерживает данные о температуре с датчиков, установленных в домах.

### 2. Анализ архитектуры монолитного приложения

- Язык программирования: Java
- База данных: PostgreSQL
- Архитектура: Монолитная, все компоненты системы (обработка запросов, бизнес-логика, работа с данными) находятся в рамках одного приложения.
- Взаимодействие: Синхронное, запросы обрабатываются последовательно.
- Масштабируемость: Ограничена, так как монолит сложно масштабировать по частям.
- Развёртывание: Требует остановки всего приложения.

### 3. Определение доменов и границы контекстов

- Домен: Управление устройствами
    - поддомен: упраления светом
        - контекст: отключение/включение света
        - контекст: данные с пирибора
    - поддомен: упраления отоплением
        - контекст: отключение/включение отопление
        - контекст: данные по температуре
    - поддомен: упраления воротами
        - контекст: открытие/закрытие ворот
        - контекст: статус (открыто/закрыто)
    - поддомен: видеонаблюдения
        - контекст: включение/выключение видеонаблюдения
        - контекст: упраление устройствами

- Домен: Подключение/отключение устройсвта
    - поддомен: подключение устройства
        - контекст: самостоятельное подключение приборов
        - контекст: статус подключения приборов
    - поддомен: программирование модулей
  
### **4. Проблемы монолитного решения**

- Плохо поддоется расширению
- Менее надежно. При падении, все приложение будет не доступно
- Самостоятельно подключить свой датчик к системе пользователь не может, что требуется в новых условиях
- Задержки при запросах, из-за отсутствия асинхронности
- Каждая установка сопровождается выездом специалиста по подключению системы, а требуется самообслуживание

### 5. Визуализация контекста системы — диаграмма С4

Добавьте сюда диаграмму контекста в модели C4.

Чтобы добавить ссылку в файл Readme.md, нужно использовать синтаксис Markdown. Это делают так:

```markdown
[Context](https://github.com/lvladv/architecture-sprint-3/blob/sprint_3/Context.puml)
```

# Задание 2. Проектирование микросервисной архитектуры

В этом задании вам нужно предоставить только диаграммы в модели C4. Мы не просим вас отдельно описывать получившиеся микросервисы и то, как вы определили взаимодействия между компонентами To-Be системы. Если вы правильно подготовите диаграммы C4, они и так это покажут.

**Диаграмма контейнеров (Containers)**

@startuml
title "Теплый дом" Container Diagram

top to bottom direction

!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Component.puml

Person(user, "Пользователь", "Пользователь системы")
System(WarmHouse, "Теплый дом", "Система подключения и управления устройствами")

Container_Boundary(WarmHouse, "FitLife System") {
  Container(WebApp, "Web Application", "React", "Управление через веб-приложение")
  Container(MobileApp, "Mobile Application", "ReactNative", "Упраление через мобильное приложение")
  Container(Gateway, "API Gateway", "Java,Spring")
  Container(UserDevicesService, "Api Server Application", "Java,Spring", "Микросервис по подключению/отключению устройств")
  Container(DeviceManagementService, "Api Server Application", "Java,Spring", "Микросервис по управлению устройствами")
  Container(UserDevicesDatabase, "Database", "PostgreSQL", "Хранит данные о пользователе и имеющихся у него устройствах")
  Container(DeviceManagementDatabase, "Database", "PostgreSQL", "Хранит данные подключеных устройствах пользователя и их статусы")
}

Rel(user, WebApp, "Uses the system")
Rel(user, MobileApp, "Uses the system")
Rel(WebApp,Gateway,"Reads/Writes user data")
Rel(MobileApp,Gateway,"Reads/Writes user data")
Rel(Gateway,UserDevicesService,"запрос к сервису по устройсвам")
Rel(Gateway,DeviceManagementService,"измение статуса ус-ва (например вкл/выкл)")
Rel(UserDevicesService,UserDevicesDatabase,"Подключене/удаление устройства")
Rel(DeviceManagementService,DeviceManagementDatabase,"Запись данных об измении статуса устройства")
@enduml

**Диаграмма компонентов (Components)**

@startuml
title WarmHouse DeviceManagementService Component Diagram

top to bottom direction

!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Component.puml

Container_Boundary(WarmHouse, "Система управления устройствами") {
  Container(DeviceManagementService, "Api Server Application", "Java,Spring", "Микросервис по управлению устройствами")
  Container(DeviceManagementDatabase, "Database", "PostgreSQL", "Хранит данные подключеных устройствах пользователя и их статусы")
}

Container(DeviceManagementService, "Web Application", "Java, Spring") {
  Component(LightController, "LightController", "Управление светом")
  Component(HeatingController, "HeatingController", "Управление отоплением")
  Component(GatesController, "GatesController", "Управление воротами")
  Component(SurveillanceController, "SurveillanceController", "Управление наблюдлением")
  Component(ServiceLayer, "Service Layer", "Business logic")
  Component(RepositoryLayer, "Repository Layer", "Data access logic")

}

Rel(LightController,ServiceLayer,"Calls business logic")
Rel(HeatingController,ServiceLayer,"Calls business logic")
Rel(GatesController,ServiceLayer,"Calls business logic")
Rel(SurveillanceController,ServiceLayer,"Calls business logic")
Rel(ServiceLayer,RepositoryLayer,"Reads/Writes data")
Rel(RepositoryLayer, DeviceManagementDatabase, "Записываем и получаем статус устройства в базу")
@enduml

**Диаграмма кода (Code)**

@startuml
title Device Management Code Diagram

top to bottom direction

!includeurl https://raw.githubusercontent.com/RicardoNiepel/C4-PlantUML/master/C4_Component.puml

class User {
  +String name
  +String email
  +List<Membership> membership
  +void register()
  +void login()
}

class Membership {
  +String type
  +String status
  +Date startDate
  +Date endDate
  +void control()
  +void status()
}

class Device {
  +Date date
  +String activity
  +void control()
  +void getStatus()
}

User "1" -- "0..*" Membership : has
Membership "1" -- "0..*" Device : includes

@enduml

# Задание 3. Разработка ER-диаграммы

Добавьте сюда ER-диаграмму. Она должна отражать ключевые сущности системы, их атрибуты и тип связей между ними.

@startuml

entity "User" {
  * user_id : NUMBER
  --
  username : VARCHAR
  email : VARCHAR
  password : VARCHAR
  created_at: DATETIME
  updeted_at: DATETIME
}

entity "HOME" {
  * home_id : NUMBER
  --
  name : VARCHAR
  description: STRING
  created_at: DATETIME
  updeted_at: DATETIME
}

entity "Device" {
  * device_id : NUMBER
  --
  activation_date: DATE
  name: STRING
  type_id: NUMBER
  telemetry_id: NUMBER
  module_id: NUMBER
  serial_number: STRING
}

entity "TelemetryData" {
  * telemetry_id : NUMBER
  --
  created_at: DATETIME
  name: STRING
  status: STRING
}

entity "DeviceType" {
  * type_id : NUMBER
  --
  updeted_at: DATETIME
  name: STRING
}

entity "Module" {
  * module_id : NUMBER
  --
  created_at: DATETIME
  updeted_at: DATETIME
  name: STRING
}



User ||--o{ HOME
HOME ||--o{ Device
Device ||--o{ TelemetryData
Device ||--o{ Module
Device ||--o{ DeviceType
  
@enduml
