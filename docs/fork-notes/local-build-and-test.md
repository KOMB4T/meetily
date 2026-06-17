# Local Build And Test

## Проверка remotes

Из корня репозитория:

```powershell
git status
git branch -vv
git remote -v
git config --get remote.upstream.pushurl
```

Ожидаемое состояние:

- текущая ветка: `evg/sa-meeting-protocol`
- `origin` указывает на `https://github.com/KOMB4T/meetily.git`
- `upstream` указывает на `https://github.com/Zackriya-Solutions/meetily.git`
- `remote.upstream.pushurl = DISABLED`

## Запуск тестов

Фронтенд находится в `frontend/`.

Установка зависимостей:

```powershell
cd frontend
corepack pnpm install
```

Минимальные проверки:

```powershell
corepack pnpm build
```

Примечания:

- `corepack pnpm lint` в текущем upstream может уйти в интерактивную настройку `next lint`, если ESLint ещё не инициализирован. Это не признак ошибки в доработке профиля.
- Rust-проверки вида `cargo test -p meetily --lib ...` и `cargo check -p meetily --lib` требуют `libclang.dll` для `whisper-rs-sys`. Если `libclang` не установлен, сборка остановится до выполнения тестов summary-модуля.

## Локальный запуск приложения

Требования для Windows:

- Node.js
- Rust
- Visual Studio Build Tools с workload `Desktop development with C++`
- CMake
- `libclang.dll` в системе или настроенный `LIBCLANG_PATH` для Rust-сборки

Запуск:

```powershell
cd frontend
corepack pnpm tauri:dev
```

Сборка без упаковки инсталлятора отдельной автоматизацией:

```powershell
cd frontend
corepack pnpm tauri:build
```

## Настройки LLM в Meetily

В настройках summary model:

- Provider: `custom-openai`
- Base URL: `http://127.0.0.1:8080/v1`
- API Key: `local-dev-key`
- Model: `qwen3-14b-local`

Endpoint поднимается отдельно и не хардкодится профилем.

## Где выбрать SA Meeting Protocol

1. Открыть страницу встречи.
2. В блоке summary нажать `Template`.
3. Выбрать `SA Meeting Protocol`.
4. Нажать `Generate Summary` или `Regenerate Summary`.

Что проверить в UI:

- профиль `SA Meeting Protocol` есть в dropdown;
- профиль выбирается без ошибок;
- выбранный template сохраняется и восстанавливается при повторном открытии страницы;
- генерация summary уходит с `templateId = sa_meeting_protocol`;
- результат содержит разделы:
  - `Контекст`
  - `Краткий итог`
  - `Договорённости`
  - `Дальнейшие действия`
  - `Решения`
  - `Требования и изменения`
  - `Открытые вопросы`
  - `Риски и спорные моменты`
  - `Что нужно проверить вручную`

## Как обновляться от upstream/main

Из корня репозитория:

```powershell
git fetch upstream
git checkout main
git merge --ff-only upstream/main
git push origin main
git checkout evg/sa-meeting-protocol
git rebase main
git push --force-with-lease origin evg/sa-meeting-protocol
```

Если в вашем форке `main` не нужен, можно ребейзить рабочую ветку напрямую на `upstream/main`:

```powershell
git fetch upstream
git checkout evg/sa-meeting-protocol
git rebase upstream/main
git push --force-with-lease origin evg/sa-meeting-protocol
```

Перед ребейзом рабочее дерево должно быть чистым.
