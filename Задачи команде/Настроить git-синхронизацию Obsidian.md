# Задача: настроить git-синхронизацию Obsidian между Mac, iPad и iPhone

**Ответственный:** Алан или Армен
**Заказчик:** Ибрагим
**Оценка:** ~1.5 часа
**Приоритет:** высокий (блокирует автоматизацию «Директорский журнал»)

---

## Что должно получиться в итоге

Ибрагим открывает Obsidian на Mac, iPad, iPhone — везде один и тот же vault. Изменения, сделанные на одном устройстве, появляются на других. Vault лежит в GitHub-репозитории `indikbr07-cloud/obsidian-vault` и является **единственным источником правды**.

## Почему именно git, а не Obsidian Sync

В этот же репозиторий Claude Code пишет «Директорский журнал» — файлы с контекстом предыдущих сессий. Если использовать Obsidian Sync, получится два параллельных хранилища (Obsidian Sync cloud + git), которые будут расходиться. Через git — одно хранилище на всё: и заметки Ибрагима, и служебные файлы автоматизации.

## Предпосылки

- Репозиторий: `https://github.com/indikbr07-cloud/obsidian-vault`
- Основная ветка для ежедневной синхронизации: `main`
- У Ибрагима есть GitHub-аккаунт с доступом к репо (он владелец)
- Устройства, которые нужно синхронизировать: Mac, iPad, iPhone

---

## Шаг 1. Подготовить репозиторий (5 мин)

Сейчас рабочая ветка — `claude/automation-discussion-08ppN`. Для синхронизации устройства должны подтягиваться из `main`.

1. Смерджить `claude/automation-discussion-08ppN` в `main` через Pull Request (или локально, если удобнее).
2. Создать `.gitignore` в корне репозитория со следующим содержимым:

   ```gitignore
   # Obsidian — служебные файлы, не нужны в синхронизации
   .obsidian/workspace.json
   .obsidian/workspace-mobile.json
   .obsidian/cache
   .obsidian/plugins/*/data.json

   # macOS
   .DS_Store

   # iOS
   .Trash/
   ```

3. Закоммитить и запушить в `main`.

---

## Шаг 2. Настроить Mac (20 мин)

1. Установить Obsidian (desktop) с `https://obsidian.md`, если не установлен.
2. Сгенерировать SSH-ключ на Mac (если нет):
   ```bash
   ssh-keygen -t ed25519 -C "ibragim-mac"
   cat ~/.ssh/id_ed25519.pub
   ```
   Добавить публичный ключ в GitHub: Settings → SSH and GPG keys → New SSH key.
3. Склонировать репозиторий в удобную папку (путь согласовать с Ибрагимом, пример — `~/Documents/Obsidian/vault`):
   ```bash
   git clone git@github.com:indikbr07-cloud/obsidian-vault.git ~/Documents/Obsidian/vault
   ```
4. В Obsidian: «Open folder as vault» → выбрать эту папку.
5. Установить плагин **Obsidian Git**:
   Settings → Community plugins → Browse → найти «Obsidian Git» (автор Vinzent) → Install → Enable.
6. Настроить плагин (Settings → Obsidian Git):
   - **Vault backup interval (minutes):** `5`
   - **Auto pull interval (minutes):** `5`
   - **Auto pull on startup:** `enabled`
   - **Auto push:** `enabled`
   - **Commit message:** `vault: {{date}}`
   - **Pull updates on startup:** `enabled`
7. Проверить: создать тестовую заметку, подождать 5 минут, проверить на GitHub что файл появился в `main`.

---

## Шаг 3. Настроить iPad (30 мин)

1. Установить **Working Copy** из App Store. Купить Pro-версию (~$21 one-time) — она нужна, чтобы экспортировать репозиторий в приложение Files.
2. Установить **Obsidian** из App Store (бесплатно).
3. В Working Copy:
   - Repositories → «+» → Clone → вставить URL: `git@github.com:indikbr07-cloud/obsidian-vault.git`
   - Авторизация: Working Copy предложит добавить SSH-ключ или войти через GitHub. Рекомендую SSH — создать ключ прямо в приложении, добавить публичную часть в GitHub.
   - Дождаться завершения клонирования.
4. В Working Copy: открыть репозиторий → кнопка share (квадратик со стрелкой) → **Setup Folder Sync** → выбрать место в Files: `On My iPad/Obsidian-Vault`. Working Copy создаст зеркальную папку в Files, которая будет автоматически синхронизироваться с репо.
5. Открыть Obsidian: «Open folder as vault» → перейти в Files → `On My iPad/Obsidian-Vault` → выбрать папку.
6. В Working Copy → Settings → Automation:
   - **Fetch on app launch:** `enabled`
   - **Push after commit:** `enabled`

**Важно:** на iOS нет плагина Obsidian Git, поэтому коммит изменений делается вручную в Working Copy (открыть → Commit → Push, 5 секунд).

---

## Шаг 4. Настроить iPhone (30 мин)

Процесс идентичен Шагу 3 для iPad. Working Copy Pro — одна покупка на Apple ID, на iPhone уже куплено. Нужно только:
1. Установить Working Copy и Obsidian.
2. Склонировать репозиторий в Working Copy (SSH-ключ свой, не iPad-овский — создать новый).
3. Настроить Folder Sync в Files.
4. Открыть vault в Obsidian.

---

## Шаг 5. Тест синхронизации (10 мин)

Обязательный приёмочный тест. Не считать задачу выполненной, пока все три сценария не пройдут:

1. **Mac → iPad:** создать на Mac в Obsidian заметку `тест-синхронизации.md`. Подождать 5 минут. На iPad открыть Working Copy → Fetch. Открыть Obsidian — файл должен появиться.
2. **iPad → Mac:** на iPad отредактировать заметку, добавить строку. В Working Copy → Commit + Push. На Mac через 5 минут (или через Obsidian Git → Pull) изменения должны подтянуться.
3. **iPhone → Mac и iPad:** аналогично, с iPhone.

После успешного теста удалить `тест-синхронизации.md` и запушить удаление.

---

## Шаг 6. Короткая инструкция для Ибрагима

Передать текстом, например в личку YouGile:

> **Как теперь работает Obsidian:**
>
> **На Mac** — ничего не делай, всё синхронизируется само каждые 5 минут.
>
> **На iPad и iPhone:**
> - Перед началом редактирования заметок открой Working Copy, нажми Fetch — подтянутся изменения с Mac.
> - После того как закончил редактировать — открой Working Copy, нажми Commit + Push (5 секунд).
>
> Это единственный ручной шаг. Если забудешь сделать Push — изменения останутся на устройстве и не улетят на Mac, но ничего не потеряется.

---

## Возможные проблемы и как их обойти

- **Конфликты правки.** Если Ибрагим правит одну заметку одновременно на Mac и iPad — будет merge-конфликт. Working Copy показывает его понятно: нужно выбрать какую версию оставить. Объяснить Ибрагиму.
- **Кириллица в именах папок** (`Директорский журнал/`, `Задачи команде/`). Git обрабатывает нормально, но при настройке убедиться, что Obsidian на iOS корректно показывает кириллицу. Если кракозябры — проблема в кодировке имени файла (редкая).
- **Большие бинарные файлы** (PDF, изображения). Git не любит большие бинарники (>50 MB). Если Ибрагим будет хранить такие в vault — подключить Git LFS отдельной задачей.
- **Authentication failure на iOS.** Самая частая проблема — Working Copy не принимает GitHub-пароль. Решение: использовать SSH-ключ (генерируется внутри Working Copy) или Personal Access Token вместо пароля.

---

## Что делать после того, как всё работает

Написать Ибрагиму короткий отчёт в YouGile:
1. Синхронизация настроена на Mac / iPad / iPhone.
2. Тест прошёл успешно (перечислить три проверки).
3. Инструкция по ежедневному использованию передана.

И пинговать Claude Code (в любой сессии) — чтобы Claude протестировал чтение и запись в vault из своей стороны.
