# Личная напоминалка — Bot specification

**Archetype:** workflow

**Voice:** warm and concise — write every user-facing message, button label, error, and empty state in this voice.

Личный бот-напоминалка для одного пользователя: добавляйте напоминания с расписанием (каждый день в указанное время), получайте их в текущем чате, управляйте через команды /add, /list, /remove, /help.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- один пользователь для личных дел

## Success criteria

- пользователь добавляет напоминание и получает его в указанное время в текущем чате
- список напоминаний отображается корректно
- удаление напоминания по ID работает

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Открыть главное меню
- **/add** (command, actor: user, command: /add) — Добавить новое напоминание
  - inputs: текст напоминания, время (например, 09:00)
  - outputs: подтверждение добавления, ID напоминания
- **/list** (command, actor: user, command: /list) — Показать список всех напоминаний
  - outputs: список напоминаний с ID, текстом и временем
- **/remove** (command, actor: user, command: /remove) — Удалить напоминание по ID
  - inputs: ID напоминания
  - outputs: подтверждение удаления или сообщение об ошибке
- **/help** (command, actor: user, command: /help) — Показать справку по командам
  - outputs: список доступных команд и их описание

## Flows

### Добавление напоминания
_Trigger:_ /add

1. пользователь вводит текст напоминания
2. пользователь вводит время (например, 09:00)
3. бот сохраняет напоминание в базу данных
4. бот отправляет подтверждение с ID

_Data touched:_ reminders

### Просмотр списка
_Trigger:_ /list

1. бот извлекает все активные напоминания из базы данных
2. бот отправляет список с ID, текстом и временем

_Data touched:_ reminders

### Удаление напоминания
_Trigger:_ /remove

1. пользователь вводит ID напоминания
2. бот проверяет существование напоминания
3. бот удаляет напоминание из базы данных
4. бот отправляет подтверждение

_Data touched:_ reminders

### Отправка напоминания
_Trigger:_ расписание

1. бот проверяет активные напоминания с текущим временем
2. бот отправляет текст напоминания в текущий чат

_Data touched:_ reminders

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **reminders** _(retention: persistent)_ — Напоминания пользователя
  - fields: id, text, time, active

## Integrations

- **Telegram** (required) — Bot API messaging
- **SQLite** (required) — Хранение напоминаний
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- добавление напоминаний
- просмотр списка
- удаление напоминаний
- активация/деактивация напоминаний

## Notifications

- напоминания отправляются в текущий чат с ботом

## Permissions & privacy

- данные хранятся только на устройстве пользователя
- нет передачи данных третьим лицам
- нет сбора статистики

## Edge cases

- напоминание с уже существующим ID
- напоминание с некорректным временем
- напоминание с пустым текстом
- удаление несуществующего напоминания

## Required tests

- добавление напоминания и проверка сохранения
- просмотр списка напоминаний
- удаление напоминания по ID
- отправка напоминания в указанное время
- обработка ошибок (некорректные данные)

## Assumptions

- поддерживается только расписание 'каждый день в указанное время'
- управление только через команды (без кнопок меню)
- хранение в SQLite файле reminders.db
- разработка на Python + aiogram
