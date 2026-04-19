# Модель данных (ERD)

Высокоуровневая схема сущностей. Детализируется после того, как PO опишет процессы по каждому модулю — чтобы не изобретать лишних полей.

## Основные сущности

### Справочники
- `users` — сотрудники (id, full_name, login, pin_hash, password_hash, is_active)
- `roles`, `user_roles` — роли и связка many-to-many
- `suppliers` — поставщики (ИНН, название, контакты, канал: ЭДО / бумага)
- `locations` — локации склада (warehouse_raw, kitchen, line, …)
- `units` — единицы измерения (кг, г, л, мл, шт)
- `ingredients` — ингредиенты (название, unit, категория, last_purchase_price, cost_method)
- `dishes` — блюда (название, категория, sale_price, is_active, is_weight)
- `modifiers` — модификаторы (соус, размер)
- `ttk` — тех. карты (dish_id, version, is_current)
- `ttk_items` — состав карты (ttk_id, ingredient_id OR sub_dish_id, gross_qty, net_qty, loss_pct)
- `supplier_mapping` — `supplier_id + supplier_sku → ingredient_id` (для ЭДО-матчинга)

### Склад и движения
- `stock_movements` — единая таблица движений (location_id, item_type: ingredient|dish, item_id, qty, sign, doc_id, doc_type, created_at, user_id)
- `documents` — общая таблица документов (type: receiving | production | transfer | sale | inventory | write_off | procurement_issue | procurement_return, status: draft | posted, created_at, user_id, shift_id)
- `document_lines` — строки документов (doc_id, item_type, item_id, qty, price, from_location, to_location)

### Производство и продажи
- `production_plan` — план производства на день (date, dish_id, planned_qty)
- `sales` — чеки (shift_id, total, paid_cash, paid_card, user_id, created_at)
- `sale_lines` — строки чека (sale_id, dish_id, qty, price, modifiers jsonb)

### Смены
- `shifts` — открытая/закрытая смена (user_id, workplace: pos|kitchen|warehouse, opened_at, closed_at, opening_cash, closing_cash, z_report jsonb)

### Приёмка
- `receiving_drafts` — черновики прихода от OCR / ЭДО (source: edo|ocr|manual, raw_data jsonb, parsed_at, status)

### Закупки (PWA)
- `procurement_issues` — выдачи наличных закупщику (procurer_id, amount, issued_at, returned_at)
- `procurement_purchases` — покупки (issue_id, ingredient_id, qty, price, photo_url, geo, synced_at)

## Принципы

1. **Единая таблица движений `stock_movements`** — любое изменение остатка это запись сюда. Остаток = `SUM(qty * sign)` по фильтру. Легко считать, легко аудировать.
2. **Документы первичны, движения вторичны.** Удаление/откат документа → откат его движений.
3. **Версионирование ТТК.** Продажа запоминает `ttk_version_id` — расчёт себестоимости продажи не меняется задним числом.
4. **Soft delete** везде, кроме движений (они immutable).
5. **Аудит**: `created_by`, `created_at`, `updated_by`, `updated_at` на всех основных сущностях.

## Следующий шаг
После интервью с PO по каждому модулю — детализируем поля, пишем Alembic-миграции, генерируем начальные SQLAlchemy-модели.
