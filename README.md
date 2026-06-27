# Monolith

[![Build Status](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bugayovdeni/ad700eae77d7761e803d9d80c93745ce/raw/badge.json)](https://github.com/bugayovdeni/monolith/actions)

**Monolith** — desktop-приложение для промышленного мониторинга данных в реальном времени.

Проект построен на **Tauri 2**, где:

* **Frontend** написан на TypeScript, HTML и CSS;
* **Backend** написан на Rust;
* обмен между frontend и backend идет через Tauri commands/events;
* данные принимаются через Serial Port;
* ASCII-поток разбирается на телеметрию;
* данные могут отображаться на графиках и сохраняться/читатьcя через CSV.

---

## Назначение

Monolith предназначен для локального промышленного мониторинга:

* подключение к последовательному порту;
* прием ASCII-данных от внешнего устройства;
* парсинг телеметрии;
* отображение live-данных;
* работа с графиками;
* загрузка и обработка CSV-файлов;
* управление интерфейсом через нативные возможности Tauri.

Проект не является веб-сайтом.
Это desktop-приложение с Rust-бэкендом и TypeScript-интерфейсом.

---

## Стек

### Frontend

* TypeScript
* Vite
* HTML
* CSS
* ECharts
* Tauri API

### Backend

* Rust
* Tauri 2
* Tokio
* Serde
* Chrono
* UUID
* CSV
* SerialPort

---

## Архитектура

Общая схема проекта:

```text
┌──────────────────────────┐
│        Frontend          │
│ TypeScript / HTML / CSS  │
└─────────────┬────────────┘
              │
              │ Tauri invoke / events
              ▼
┌──────────────────────────┐
│       Tauri Backend      │
│           Rust           │
└─────────────┬────────────┘
              │
              ├── Serial Port
              ├── ASCII Parser
              ├── Telemetry Store
              ├── CSV Manager
              └── Native App Services
```

---

## Основные модули

### Serial Port

Модуль отвечает за:

* поиск доступных COM/TTY портов;
* открытие диалога выбора порта;
* подключение к выбранному порту;
* запуск и остановку serial-потока.

Поток работы:

```text
Frontend
  ↓ invoke("get_serial_ports")
Tauri command
  ↓
serial_service
  ↓
serial_scanner
  ↓
OS serial ports
```

---

### ASCII Parser

Модуль отвечает за разбор входящих строк ASCII-формата.

Назначение:

* принять сырые данные;
* распарсить строку;
* преобразовать ее в структурированную телеметрию;
* передать результат дальше в приложение.

---

### Telemetry

Модуль телеметрии хранит и передает актуальные live-данные.

Он связывает backend-поток данных с frontend-отображением.

---

### CSV

CSV-модуль отвечает за работу с сохраненными данными:

* чтение CSV;
* преобразование CSV-строк в доменные записи;
* подготовка данных для отображения;
* отладочная сериализация.

---

### Charts

Frontend-модуль графиков отвечает за:

* конфигурацию графиков;
* управление сериями;
* оси;
* состояние отображения;
* передачу данных в ECharts.

---

## Структура проекта

```text
monolith
├── docs
│   └── serial-port-flow.md
│
├── src
│   ├── assets
│   │   └── monolith.svg
│   │
│   ├── services
│   │   └── csv
│   │       └── csv-manager.ts
│   │
│   ├── types
│   │   └── csv.ts
│   │
│   └── windows
│       ├── charts
│       │   ├── chart-config.ts
│       │   ├── chart-manager.ts
│       │   └── axis
│       │       ├── axis-state.ts
│       │       └── axis-types.ts
│       │
│       ├── data
│       │   ├── ascii
│       │   │   └── ascii-record.mapper.ts
│       │   │
│       │   └── live
│       │       └── live-data.controller.ts
│       │
│       ├── main
│       │   ├── app.ts
│       │   ├── main.ts
│       │   ├── styles.css
│       │   └── styles
│       │       ├── base.css
│       │       ├── sidebar.css
│       │       ├── titlebar.css
│       │       └── components
│       │
│       └── modules
│           ├── axis-settings
│           ├── seria-port
│           ├── series-panel
│           └── sidebar
│
├── src-tauri
│   ├── capabilities
│   │   └── default.json
│   │
│   ├── crates
│   │   ├── monolith-ascii
│   │   ├── monolith-domain
│   │   └── monolith-serial
│   │
│   ├── icons
│   │
│   ├── src
│   │   ├── app
│   │   ├── commands
│   │   ├── domain
│   │   ├── services
│   │   ├── lib.rs
│   │   └── main.rs
│   │
│   ├── Cargo.toml
│   ├── Cargo.lock
│   ├── build.rs
│   └── tauri.conf.json
│
├── index.html
├── package.json
├── package-lock.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

---

## Frontend structure

### `src/windows/main`

Главная точка сборки интерфейса.

Содержит:

* инициализацию приложения;
* основные стили;
* базовую компоновку окна;
* стили titlebar/sidebar/buttons/check boxes.

---

### `src/windows/modules`

UI-модули приложения.

#### `seria-port`

Модуль выбора последовательного порта.

Содержит:

* `dialog.ts` — логика модального окна;
* `template.ts` — HTML-шаблон;
* `styles.css` — стили окна выбора порта.

> Примечание: папка называется `seria-port`. Если это не намеренно, лучше позже переименовать в `serial-port`.

#### `axis-settings`

Модуль настройки осей графика.

#### `series-panel`

Модуль управления сериями графика.

#### `sidebar`

Контроллер боковой панели.

---

### `src/windows/charts`

Модуль графиков.

Содержит:

* конфигурацию графиков;
* менеджер графика;
* состояние осей;
* типы осей.

---

### `src/windows/data`

Модуль подготовки данных для отображения.

#### `ascii`

Преобразование ASCII-записей в формат frontend.

#### `live`

Контроллер live-данных.

---

## Backend structure

### `src-tauri/src`

Основной Rust-код Tauri-приложения.

```text
src-tauri/src
├── app
├── commands
├── domain
├── services
├── lib.rs
└── main.rs
```

---

### `commands`

Слой Tauri-команд.

Команды вызываются из frontend через `invoke(...)`.

Основные группы:

```text
commands
├── csv
├── serial_port
└── telemetry
```

Назначение слоя:

* принять вызов из frontend;
* проверить входные данные;
* вызвать нужный сервис;
* вернуть результат обратно во frontend.

---

### `services`

Сервисный слой backend.

Содержит бизнес-операции и работу с внешними источниками:

* serial port;
* telemetry;
* CSV.

Сюда должна попадать логика, которая не является напрямую Tauri-командой.

---

### `domain`

Доменный слой backend.

Содержит основные модели, value objects и доменные структуры.

Здесь должны находиться типы, которые описывают предметную область проекта, а не детали UI или Tauri.

---

### `app`

Слой настройки приложения.

Отвечает за:

* меню;
* обработчики окна;
* lifecycle;
* системные события приложения.

---

## Internal crates

Проект использует отдельные Rust-крейты внутри `src-tauri/crates`.

```text
src-tauri/crates
├── monolith-ascii
├── monolith-domain
└── monolith-serial
```

### `monolith-ascii`

Крейт для работы с ASCII-данными.

Содержит:

* parser;
* listener;
* ошибки ASCII-слоя.

### `monolith-domain`

Крейт доменной модели.

Содержит общие типы, которые могут использоваться разными частями backend.

### `monolith-serial`

Крейт для работы с последовательным портом.

Отвечает за низкоуровневый serial-слой.

---

## Поток Serial Port

```text
Пользователь
  ↓
Открывает окно выбора порта
  ↓
Frontend вызывает open_port_dialog
  ↓
Rust отправляет событие show-port-dialog
  ↓
Frontend открывает модальное окно
  ↓
Frontend вызывает get_serial_ports
  ↓
Rust сканирует доступные порты
  ↓
Frontend отображает список
  ↓
Пользователь выбирает порт
  ↓
Frontend вызывает connect_port
  ↓
Backend подключается к порту
  ↓
Запускается чтение ASCII-потока
  ↓
Данные парсятся в телеметрию
  ↓
Frontend обновляет live-отображение и графики
```

---

## Установка

### Требования

* Node.js
* npm
* Rust
* Cargo
* Tauri prerequisites для целевой ОС

---

### Установка зависимостей

```bash
npm install
```

---

## Запуск в режиме разработки

```bash
npm run tauri dev
```

или отдельно frontend:

```bash
npm run dev
```

---

## Сборка frontend

```bash
npm run build
```

---

## Сборка desktop-приложения

```bash
npm run tauri build
```

---

## Проверка TypeScript

```bash
npm run build
```

Команда выполняет:

```bash
tsc && vite build
```

---

## Основные npm scripts

```json
{
  "dev": "vite",
  "build": "tsc && vite build",
  "preview": "vite preview",
  "tauri": "tauri",
  "lint": "eslint src/",
  "test": "vitest"
}
```

---

## Правила разработки

### Frontend

* UI-логика хранится в `src/windows`.
* Общие frontend-сервисы — в `src/services`.
* Типы — в `src/types`.
* Графики — в `src/windows/charts`.
* Live-данные — в `src/windows/data/live`.
* Преобразование ASCII-данных для UI — в `src/windows/data/ascii`.

---

### Backend

* Tauri-команды размещаются в `src-tauri/src/commands`.
* Бизнес-логика — в `src-tauri/src/services`.
* Доменные типы — в `src-tauri/src/domain` или `src-tauri/crates/monolith-domain`.
* Низкоуровневая работа с serial — в `monolith-serial`.
* Парсинг ASCII — в `monolith-ascii`.

---

## Как добавить новую Tauri-команду

1. Создать команду в `src-tauri/src/commands`.
2. Вынести логику в `services`, если команда делает больше одной простой операции.
3. Зарегистрировать команду в `invoke_handler` в `src-tauri/src/lib.rs`.
4. Вызвать команду из frontend через `invoke(...)`.

Пример направления:

```text
frontend
  ↓ invoke("command_name")
commands
  ↓
services
  ↓
domain / external device / file system
```

---

## Как добавить новый frontend-модуль

1. Создать папку в `src/windows/modules`.
2. Разделить:

   * controller;
   * template;
   * styles.
3. Подключить модуль в основном приложении.
4. Если модуль общается с Rust — использовать Tauri `invoke` или события.

---

## Документация

Дополнительные документы находятся в папке:

```text
docs
```

Сейчас доступен документ:

```text
docs/serial-port-flow.md
```

Он описывает последовательность вызовов между TypeScript frontend и Rust backend при работе с serial ports.

---

## Платформенные особенности Serial Port

### Windows

Порты обычно имеют вид:

```text
COM1
COM2
COM3
```

### Linux

Порты обычно имеют вид:

```text
/dev/ttyUSB0
/dev/ttyACM0
```

На Linux пользователю может потребоваться доступ к группе:

```text
dialout
```

### macOS

Порты обычно имеют вид:

```text
/dev/cu.usbserial-*
/dev/tty.usbserial-*
```

---

## Что не хранить в репозитории

В репозиторий не должны попадать:

```text
node_modules
dist
target
.vite
```

Эти папки являются результатом установки зависимостей или сборки.

Они должны быть исключены через `.gitignore`.

---

## Roadmap

Планируемые направления развития:

* стабильный serial streaming;
* улучшение ASCII parser;
* расширение telemetry store;
* сохранение и загрузка сессий;
* улучшение графиков;
* настройка осей и серий;
* экспорт данных;
* оформление релизов;
* автоматическая сборка через GitHub Actions;
* документация по протоколу входных данных.

---

## Статус проекта

Проект находится в активной разработке.

Текущий README описывает архитектуру и структуру проекта на момент версии `0.1.0`.

---

## License

Licensed under the [MIT License](LICENSE).
