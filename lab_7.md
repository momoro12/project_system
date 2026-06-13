# Лабораторная работа №7: Логическая модель БД
## ИС аэропорта (Вариант 20)
### Система: AirPortIS
---
## Выделяем основные сущности
На основе анализа предметной области, контекстных потоков данных и функциональных требований выделяем следующие хранимые сущности:
| Сущность | Почему она нужна | Ссылка на требования |
|----------|------------------|----------------------|
| **Department** (Отдел) | Работники административно относятся к своим отделам | ФТ-ПЕР-02, ФТ-ПЕР-04 |
| **Brigade** (Бригада) | Внутри отделов работники объединяются в бригады | ФТ-БРГ-01, ФТ-БРГ-05 |
| **Employee** (Работник) | Основная кадровая сущность аэропорта | ФТ-ПЕР-01, ФТ-ОТЧ-01 |
| **AircraftType** (Тип самолёта) | Нужен для классификации самолётов и расписания | ФТ-ВС-01, ФТ-РЕЙ-09 |
| **Aircraft** (Самолёт) | Учёт приписанных к аэропорту самолётов, их состояния и эксплуатации | ФТ-ВС-01, ФТ-ОТЧ-04 |
| **Route** (Маршрут) | Описывает направление рейса: пункт отправления, прибытия, пересадки | ФТ-РЕЙ-03, ФТ-ОТЧ-06 |
| **FlightCategory** (Категория рейса) | Внутренний, международный, чартерный, грузовой, специальный | ФТ-РЕЙ-02, ФТ-ОТЧ-10 |
| **Flight** (Рейс) | Ключевая операционная сущность аэропорта | ФТ-РЕЙ-01, ФТ-ОТЧ-06 |
| **FlightSchedule** (Расписание рейсов) | Хранит дни вылета, время вылета/прилёта и тариф | ФТ-РЕЙ-03 |
| **Passenger** (Пассажир) | Учёт пассажиров, их документов и характеристик | ФТ-ПАС-01, ФТ-ОТЧ-11 |
| **Ticket** (Билет) | Продажа, бронирование и возврат билетов | ФТ-БИЛ-01, ФТ-ОТЧ-13 |
| **BaggageRecord** (Багаж) | Учёт сдачи вещей в багажное отделение | ФТ-ПАС-05, ФТ-ОТЧ-11 |
| **MedicalCheck** (Медосмотр пилота) | Контроль ежегодных медосмотров пилотов | ФТ-МЕД-01, ФТ-МЕД-03 |
| **TechnicalInspection** (Техосмотр самолёта) | Журнал технических осмотров самолётов | ФТ-ВС-04, ФТ-ПОД-01 |
| **RepairRecord** (Ремонт самолёта) | Фиксация отправки самолёта в ремонт и его результатов | ФТ-ВС-05, ФТ-ОТЧ-05 |
| **FlightPreparation** (Подготовка к рейсу) | Подготовка самолёта к вылету по технической и сервисной части | ФТ-ПОД-01, ФТ-ПОД-05 |
| **Agency** (Агентство) | Нужна для чартерных рейсов, билеты на которые распространяет организующее агентство | ФТ-БИЛ-05 |
**Почему выделяем именно эти сущности?**
- Они напрямую следуют из предметной области аэропорта.
- Каждая сущность имеет собственные атрибуты и жизненный цикл.
- Они покрывают кадровый, технический, пассажирский и рейсовый контур.
- Они позволяют реализовать все запросы, указанные в постановке задачи.
---
## Промежуточные (связующие) сущности
| Сущность | Зачем нужна | Тип связи |
|----------|-------------|-----------|
| **BrigadeMember** | Связывает работников с бригадами | M:N (Employee ↔ Brigade) |
| **AircraftBrigadeAssignment** | Закрепляет бригады за самолётом по виду обслуживания | M:N (Aircraft ↔ Brigade) с атрибутами |
| **FlightCrewAssignment** | Назначает работников/бригады на конкретный рейс | M:N (Flight ↔ Employee / Brigade) с атрибутами |
### Почему нужны именно эти промежуточные таблицы?
1. **BrigadeMember**: работник может переводиться между бригадами, а бригада содержит нескольких работников. Нужна история включения и выбытия.
2. **AircraftBrigadeAssignment**: по условию за самолётом закрепляются бригады пилотов, техников и обслуживающего персонала. Это не простая ссылка в самолёте, а отдельная связь с ролью и периодом действия.
3. **FlightCrewAssignment**: на конкретный рейс могут быть назначены разные работники и/или бригады. Нужно хранить роль в обслуживании рейса и факт участия.
### Почему НЕ нужны некоторые другие промежуточные таблицы?
- **Flight ↔ Route / AircraftType / FlightCategory**: это связи M:1, реализуются через FK в таблице `Flight`.
- **Ticket** уже связывает пассажира и рейс и содержит собственные атрибуты.
- **MedicalCheck** связан напрямую с работником-пилотом, а не через отдельную таблицу.
---
## Атрибуты сущностей
### Department (Отдел)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| name | VARCHAR(255) | NOT NULL, UNIQUE |
| chief_id | CHAR(36) | FK → Employee.id, NULLABLE |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
**Обоснование:** у отдела должно быть уникальное название. Начальник отдела задаётся отдельно, поскольку может быть назначен позже создания отдела.
### Brigade (Бригада)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| department_id | CHAR(36) | FK → Department.id, NOT NULL |
| name | VARCHAR(255) | NOT NULL |
| brigade_type | VARCHAR(100) | NULLABLE |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
### Employee (Работник)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| department_id | CHAR(36) | FK → Department.id, NOT NULL |
| first_name | VARCHAR(100) | NOT NULL |
| last_name | VARCHAR(100) | NOT NULL |
| middle_name | VARCHAR(100) | NULLABLE |
| gender | ENUM('MALE','FEMALE') | NOT NULL |
| birth_date | DATE | NOT NULL |
| hire_date | DATE | NOT NULL |
| salary | DECIMAL(12,2) | NOT NULL |
| children_count | INT | DEFAULT 0 |
| employee_type | ENUM('PILOT','DISPATCHER','TECHNICIAN','CASHIER','SECURITY','INFO','OTHER') | NOT NULL |
| position_title | VARCHAR(150) | NOT NULL |
| special_attrs | JSON | NULLABLE |
| is_active | BOOLEAN | DEFAULT TRUE |
**Почему `special_attrs` JSON?** У разных категорий работников разные профессиональные атрибуты: у пилота — налёт и класс, у техника — специализация, у диспетчера — зона ответственности. Это полиморфные атрибуты.
### AircraftType (Тип самолёта)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| name | VARCHAR(100) | NOT NULL, UNIQUE |
| passenger_capacity | INT | NULLABLE |
| cargo_capacity | DECIMAL(12,2) | NULLABLE |
| range_km | INT | NULLABLE |
### Aircraft (Самолёт)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| aircraft_type_id | CHAR(36) | FK → AircraftType.id, NOT NULL |
| tail_number | VARCHAR(50) | NOT NULL, UNIQUE |
| manufactured_at | DATE | NULLABLE |
| commissioned_at | DATE | NULLABLE |
| status | ENUM('ACTIVE','MAINTENANCE','REPAIR','OUT_OF_SERVICE') | DEFAULT 'ACTIVE' |
| total_flights_count | INT | DEFAULT 0 |
| current_airport | VARCHAR(255) | NULLABLE |
| arrived_at | DATETIME | NULLABLE |
| departed_at | DATETIME | NULLABLE |
### Route (Маршрут)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| origin_city | VARCHAR(150) | NOT NULL |
| destination_city | VARCHAR(150) | NOT NULL |
| transfer_city | VARCHAR(150) | NULLABLE |
| is_international | BOOLEAN | DEFAULT FALSE |
### FlightCategory (Категория рейса)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| name | VARCHAR(100) | NOT NULL, UNIQUE |
| code | VARCHAR(50) | NOT NULL, UNIQUE |
### Agency (Агентство)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| name | VARCHAR(255) | NOT NULL, UNIQUE |
| contact_info | VARCHAR(255) | NULLABLE |
### Flight (Рейс)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| flight_number | VARCHAR(50) | NOT NULL, UNIQUE |
| route_id | CHAR(36) | FK → Route.id, NOT NULL |
| aircraft_type_id | CHAR(36) | FK → AircraftType.id, NOT NULL |
| flight_category_id | CHAR(36) | FK → FlightCategory.id, NOT NULL |
| agency_id | CHAR(36) | FK → Agency.id, NULLABLE |
| duration_minutes | INT | NULLABLE |
| base_ticket_price | DECIMAL(12,2) | NULLABLE |
| status | ENUM('PLANNED','DELAYED','CANCELLED','COMPLETED','BOARDING') | DEFAULT 'PLANNED' |
| delay_reason | VARCHAR(255) | NULLABLE |
| cancel_reason | VARCHAR(255) | NULLABLE |
| min_tickets_required | INT | NULLABLE |
**Обоснование:** `agency_id` нужно только для чартерных рейсов, поэтому поле допускает `NULL`.
### FlightSchedule (Расписание рейсов)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| flight_id | CHAR(36) | FK → Flight.id, NOT NULL |
| day_of_week | TINYINT | NOT NULL |
| departure_time | TIME | NOT NULL |
| arrival_time | TIME | NOT NULL |
| effective_from | DATE | NULLABLE |
| effective_to | DATE | NULLABLE |
| ticket_price | DECIMAL(12,2) | NOT NULL |
### Passenger (Пассажир)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| first_name | VARCHAR(100) | NOT NULL |
| last_name | VARCHAR(100) | NOT NULL |
| middle_name | VARCHAR(100) | NULLABLE |
| gender | ENUM('MALE','FEMALE') | NOT NULL |
| birth_date | DATE | NOT NULL |
| passport_number | VARCHAR(50) | NOT NULL |
| international_passport_number | VARCHAR(50) | NULLABLE |
| phone | VARCHAR(50) | NULLABLE |
| email | VARCHAR(150) | NULLABLE |
### Ticket (Билет)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| flight_id | CHAR(36) | FK → Flight.id, NOT NULL |
| passenger_id | CHAR(36) | FK → Passenger.id, NOT NULL |
| seat_number | VARCHAR(20) | NULLABLE |
| ticket_status | ENUM('BOOKED','PAID','RETURNED','USED','CANCELLED') | NOT NULL |
| booked_at | DATETIME | NULLABLE |
| purchased_at | DATETIME | NULLABLE |
| returned_at | DATETIME | NULLABLE |
| price_paid | DECIMAL(12,2) | NOT NULL |
| sale_channel | ENUM('AIRPORT_CASHDESK','AGENCY','ONLINE') | NOT NULL |
**Почему отдельная сущность?** Билет содержит собственный жизненный цикл: бронь, покупка, возврат, использование.
### BaggageRecord (Багаж)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| ticket_id | CHAR(36) | FK → Ticket.id, NOT NULL |
| pieces_count | INT | DEFAULT 1 |
| total_weight_kg | DECIMAL(8,2) | NULLABLE |
| checked_in_at | DATETIME | NULLABLE |
### MedicalCheck (Медосмотр пилота)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| employee_id | CHAR(36) | FK → Employee.id, NOT NULL |
| check_year | YEAR | NOT NULL |
| check_date | DATE | NOT NULL |
| result_status | ENUM('PASSED','FAILED') | NOT NULL |
| valid_until | DATE | NULLABLE |
| notes | TEXT | NULLABLE |
### TechnicalInspection (Техосмотр самолёта)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| aircraft_id | CHAR(36) | FK → Aircraft.id, NOT NULL |
| inspection_date | DATETIME | NOT NULL |
| technician_id | CHAR(36) | FK → Employee.id, NULLABLE |
| result_status | ENUM('PASSED','FAILED','REQUIRES_REPAIR') | NOT NULL |
| notes | TEXT | NULLABLE |
### RepairRecord (Ремонт самолёта)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| aircraft_id | CHAR(36) | FK → Aircraft.id, NOT NULL |
| started_at | DATETIME | NOT NULL |
| finished_at | DATETIME | NULLABLE |
| reason | VARCHAR(255) | NOT NULL |
| description | TEXT | NULLABLE |
| flights_before_repair | INT | NULLABLE |
### FlightPreparation (Подготовка к рейсу)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| flight_id | CHAR(36) | FK → Flight.id, NOT NULL |
| aircraft_id | CHAR(36) | FK → Aircraft.id, NOT NULL |
| technical_ready | BOOLEAN | DEFAULT FALSE |
| service_ready | BOOLEAN | DEFAULT FALSE |
| fuel_amount_liters | DECIMAL(12,2) | NULLABLE |
| cleaning_completed | BOOLEAN | DEFAULT FALSE |
| food_stock_completed | BOOLEAN | DEFAULT FALSE |
| prepared_at | DATETIME | NULLABLE |
| notes | TEXT | NULLABLE |
---
## Промежуточные таблицы M:N
### BrigadeMember
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| brigade_id | CHAR(36) | FK → Brigade.id, NOT NULL |
| employee_id | CHAR(36) | FK → Employee.id, NOT NULL |
| joined_at | DATE | NOT NULL |
| left_at | DATE | NULLABLE |
UNIQUE: `(brigade_id, employee_id, joined_at)`
### AircraftBrigadeAssignment
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| aircraft_id | CHAR(36) | FK → Aircraft.id, NOT NULL |
| brigade_id | CHAR(36) | FK → Brigade.id, NOT NULL |
| assignment_role | ENUM('PILOT','TECHNICAL','SERVICE') | NOT NULL |
| assigned_from | DATE | NOT NULL |
| assigned_to | DATE | NULLABLE |
### FlightCrewAssignment
| Поле | Тип | Ограничения |
|------|-----|-------------|
| id | CHAR(36) | PK |
| flight_id | CHAR(36) | FK → Flight.id, NOT NULL |
| employee_id | CHAR(36) | FK → Employee.id, NULLABLE |
| brigade_id | CHAR(36) | FK → Brigade.id, NULLABLE |
| assignment_role | VARCHAR(100) | NOT NULL |
| assigned_at | DATETIME | NULLABLE |
**Ограничение:** хотя бы одно из полей `employee_id` или `brigade_id` должно быть заполнено.
---
## Ключевые моменты по связям
1. **Employee → Department (M:1)**: каждый работник относится к одному отделу.
2. **Employee ↔ Brigade (M:N)**: через `BrigadeMember`, поскольку работник может переводиться между бригадами.
3. **Aircraft ↔ Brigade (M:N)**: через `AircraftBrigadeAssignment`, так как за самолётом закрепляются разные бригады по типу обслуживания.
4. **Flight → Route / AircraftType / FlightCategory (M:1)**: рейс имеет один маршрут, один тип самолёта и одну категорию.
5. **Flight ↔ Passenger**: через `Ticket`, так как билет фиксирует участие пассажира в рейсе.
6. **Ticket → BaggageRecord (1:M)**: по одному билету может быть несколько записей багажа, если модель это допускает.
7. **Employee → MedicalCheck (1:M)**: только для работников типа `PILOT`.
8. **Aircraft → TechnicalInspection / RepairRecord (1:M)**: самолёт проходит много осмотров и ремонтов.
9. **Flight → FlightPreparation (1:M или 1:1 по вылету)**: подготовка к конкретному выполнению рейса.
---
## Ограничения целостности
### Первичные и внешние ключи
| Таблица | PK | Важные FK | Правило ON DELETE |
|---------|----|-----------|-------------------|
| Employee | id | department_id | RESTRICT |
| Brigade | id | department_id | RESTRICT |
| BrigadeMember | id | brigade_id, employee_id | CASCADE для brigade, RESTRICT для employee |
| Aircraft | id | aircraft_type_id | RESTRICT |
| Flight | id | route_id, aircraft_type_id, flight_category_id, agency_id | RESTRICT |
| FlightSchedule | id | flight_id | CASCADE |
| Ticket | id | flight_id, passenger_id | RESTRICT |
| BaggageRecord | id | ticket_id | CASCADE |
| MedicalCheck | id | employee_id | CASCADE |
| TechnicalInspection | id | aircraft_id, technician_id | CASCADE для aircraft, SET NULL для technician |
| RepairRecord | id | aircraft_id | CASCADE |
| FlightPreparation | id | flight_id, aircraft_id | CASCADE для flight, RESTRICT для aircraft |
### Уникальность и обязательность
- `UNIQUE (department.name)` — название отдела не дублируется
- `UNIQUE (aircraft.tail_number)` — бортовой номер уникален
- `UNIQUE (flight.flight_number)` — номер рейса уникален
- `UNIQUE (flight_category.name)` — название категории уникально
- `UNIQUE (flight_category.code)` — код категории уникален
- `UNIQUE (agency.name)` — название агентства уникально
- `CHECK (salary >= 0)` — зарплата неотрицательна
- `CHECK (children_count >= 0)` — количество детей неотрицательно
- `CHECK (duration_minutes > 0)` — длительность положительна
- `CHECK (base_ticket_price >= 0)` — базовая цена неотрицательна
- `CHECK (ticket_price >= 0)` — цена билета в расписании неотрицательна
- `CHECK (price_paid >= 0)` — уплаченная сумма неотрицательна
- `CHECK (pieces_count > 0)` — количество мест багажа положительно
- `CHECK (total_weight_kg >= 0)` — вес багажа неотрицателен
- `CHECK (finished_at IS NULL OR finished_at >= started_at)` — ремонт не может закончиться раньше начала
- `CHECK (valid_until IS NULL OR valid_until >= check_date)` — медсправка не может истечь раньше выдачи
- `CHECK ((employee_id IS NOT NULL AND brigade_id IS NULL) OR (employee_id IS NULL AND brigade_id IS NOT NULL))` — в назначении на рейс указан либо работник, либо бригада
### Бизнес-правила на уровне БД
1. **Работник принадлежит одному отделу**: `department_id NOT NULL` в `Employee`.
2. **Пилот должен иметь медосмотр**: контролируется связью `MedicalCheck` и прикладной логикой назначения на рейс.
3. **На грузовые и специальные рейсы билеты не продаются**: должно контролироваться прикладной логикой и ограничениями бизнес-уровня.
4. **Для международного рейса нужен загранпаспорт**: проверяется при оформлении билета/регистрации.
5. **Чартерный рейс может быть привязан к агентству**: `agency_id` в `Flight`.
6. **Спецрейсы не обязаны иметь расписание**: `FlightSchedule` создаётся только для рейсов, где оно применимо.
---
## Нормализация до 3НФ
### 1НФ: Все атрибуты атомарны
- Все поля содержат одно значение.
- Профессиональные особенности работников вынесены в `special_attrs` как осознанное решение для полиморфных атрибутов.
### 2НФ: Все неключевые атрибуты зависят от полного ключа
- В `Ticket` атрибуты билета зависят от самого билета.
- В `BrigadeMember` даты участия зависят от связи работника и бригады.
- В `AircraftBrigadeAssignment` роль зависит от пары самолёт–бригада и периода назначения.
### 3НФ: Нет транзитивных зависимостей
- Данные отдела не дублируются в работнике.
- Данные маршрута не дублируются в рейсе, а выносятся в `Route`.
- Данные типа самолёта вынесены в `AircraftType`, чтобы не повторять характеристики у каждого самолёта.
