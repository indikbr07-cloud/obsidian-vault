# Задача: настроить git-синхронизацию Obsidian между Windows-ПК, iPad и iPhone

**Ответственный:** Алан или Армен
**Заказчик:** Ибрагим
**Оценка:** ~1.5–2 часа
**Приоритет:** высокий (блокирует автоматизацию «Директорский журнал»)

---

## Что должно получиться в итоге

Ибрагим открывает Obsidian на Windows-ПК, iPad, iPhone — везде один и тот же vault. Изменения, сделанные на одном устройстве, появляются на других. Vault лежит в GitHub-репозитории `indikbr07-cloud/obsidian-vault` и является **единственным источником правды**.

## Почему именно git, а не Obsidian Sync

В этот же репозиторий Claude Code пишет «Директорский журнал» — файлы с контекстом предыдущих сессий. Если использовать Obsidian Sync, получится два параллельных хранилища (Obsidian Sync cloud + git), которые будут расходиться. Через git — одно хранилище на всё: и заметки Ибрагима, и служебные файлы автоматизации.

## Предпосылки

- Репозиторий: `https://github.com/indikbr07-cloud/obsidian-vault`
- Основная ветка для ежедневной синхронизации: `main`
- У Ибрагима есть GitHub-аккаунт с доступом к репо (он владелец)
- Устройства, которые нужно синхронизировать: Windows-ПК (рабочий компьютер Ибрагима), iPad, iPhone

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

   # Windows
   Thumbs.db
   Desktop.ini
   ~$*

   # macOS (для Алана/Армена, если они работают с Mac)
   .DS_Store

   # iOS
   .Trash/
   ```

3. Закоммитить и запушить в `main`.

---

## Шаг 2. Настроить Windows-ПК (25 мин)

1. **Установить Git for Windows** с `https://git-scm.com/download/win`. В мастере установки — все настройки по умолчанию, но убедиться, что включён **Git Bash** (он понадобится для ssh-keygen и git clone).
2. **Установить Obsidian** с `https://obsidian.md/download`. Выбрать «Windows Installer (64-bit)».
3. **Сгенерировать SSH-ключ** в Git Bash (открыть из меню «Пуск» → «Git Bash»):
   ```bash
   ssh-keygen -t ed25519 -C "ibragim-windows"
   # Нажать Enter на все вопросы (путь по умолчанию, без пароля)
   cat ~/.ssh/id_ed25519.pub
   ```
   Скопировать вывод (начинается с `ssh-ed25519 AAAA...`). Добавить в GitHub: Settings → SSH and GPG keys → New SSH key → название «Ibragim Windows» → вставить ключ.
4. **Склонировать репозиторий** (путь согласовать с Ибрагимом, пример — `C:\Users\<Имя>\Documents\Obsidian\vault`). В Git Bash:
   ```bash
   git clone git@github.com:indikbr07-cloud/obsidian-vault.git "/c/Users/$USER/Documents/Obsidian/vault"
   ```
   При первом подключении Git спросит «Are you sure you want to continue connecting?» — ответить `yes`.
5. **Открыть vault в Obsidian:** «Open folder as vault» → перейти к `C:\Users\<Имя>\Documents\Obsidian\vault` → «Open».
6. **Установить плагин Obsidian Git:**
   Settings → Community plugins → Turn on community plugins → Browse → найти «Obsidian Git» (автор Vinzent) → Install → Enable.
7. **Настроить плагин** (Settings → Obsidian Git):
   - **Vault backup interval (minutes):** `5`
   - **Auto pull interval (minutes):** `5`
   - **Auto pull on startup:** `enabled`
   - **Auto push:** `enabled`
   - **Commit message:** `vault: {{date}}`
   - **Pull updates on startup:** `enabled`
8. **Настроить git user** (если не настроено, плагин будет ругаться):
   ```bash
   git config --global user.name "Ibragim"
   git config --global user.email "<email-ибрагима-в-github>"
   ```
9. **Проверить:** создать в Obsidian тестовую заметку, подождать 5 минут, проверить на GitHub что файл появился в `main`.

**Частая проблема на Windows:** Obsidian Git не видит установленный git. Решение — в настройках плагина указать путь вручную: `C:\Program Files\Git\bin\git.exe`.

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

1. **Windows → iPad:** создать на ПК в Obsidian заметку `тест-синхронизации.md`. Подождать 5 минут. На iPad открыть Working Copy → Fetch. Открыть Obsidian — файл должен появиться.
2. **iPad → Windows:** на iPad отредактировать заметку, добавить строку. В Working Copy → Commit + Push. На ПК через 5 минут (или через Obsidian Git → Pull) изменения должны подтянуться.
3. **iPhone → Windows и iPad:** аналогично, с iPhone.

После успешного теста удалить `тест-синхронизации.md` и запушить удаление.

---

## Шаг 6. Короткая инструкция для Ибрагима

Передать текстом, например в личку YouGile:

> **Как теперь работает Obsidian:**
>
> **На Windows-ПК** — ничего не делай, всё синхронизируется само каждые 5 минут, пока Obsidian открыт.
>
> **На iPad и iPhone:**
> - Перед началом редактирования заметок открой Working Copy, нажми Fetch — подтянутся изменения с ПК.
> - После того как закончил редактировать — открой Working Copy, нажми Commit + Push (5 секунд).
>
> Это единственный ручной шаг. Если забудешь сделать Push — изменения останутся на устройстве и не улетят на ПК, но ничего не потеряется.

---

## Возможные проблемы и как их обойти

- **Конфликты правки.** Если Ибрагим правит одну заметку одновременно на ПК и iPad — будет merge-конфликт. Working Copy показывает его понятно: нужно выбрать какую версию оставить. Объяснить Ибрагиму.
- **Obsidian Git не работает, если Obsidian закрыт.** На Windows синхронизация идёт только пока открыт Obsidian. Если Ибрагим выключил ПК с незапушенными изменениями — они улетят на GitHub только при следующем открытии Obsidian.
- **Кириллица в именах папок** (`Директорский журнал/`, `Задачи команде/`). Git обрабатывает нормально, но при настройке убедиться, что Obsidian на iOS корректно показывает кириллицу. Если кракозябры — проблема в кодировке имени файла (редкая).
- **Большие бинарные файлы** (PDF, изображения). Git не любит большие бинарники (>50 MB). Если Ибрагим будет хранить такие в vault — подключить Git LFS отдельной задачей.
- **Authentication failure на iOS.** Самая частая проблема — Working Copy не принимает GitHub-пароль. Решение: использовать SSH-ключ (генерируется внутри Working Copy) или Personal Access Token вместо пароля.

---

## Что делать после того, как всё работает

Написать Ибрагиму короткий отчёт в YouGile:
1. Синхронизация настроена на Windows / iPad / iPhone.
2. Тест прошёл успешно (перечислить три проверки).
3. Инструкция по ежедневному использованию передана.

И пинговать Claude Code (в любой сессии) — чтобы Claude протестировал чтение и запись в vault из своей стороны.
