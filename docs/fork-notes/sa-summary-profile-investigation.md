# SA Summary Profile Investigation

## Что изучено

- `frontend/src-tauri/templates/standard_meeting.json`
  Содержит встроенный шаблон `Standard Meeting Notes`.
- `frontend/src-tauri/src/summary/templates/defaults.rs`
  Регистрирует встроенные template id и связывает их с JSON-файлами.
- `frontend/src-tauri/src/summary/templates/loader.rs`
  Загружает built-in, bundled и custom templates; `list_templates()` отдает список для UI.
- `frontend/src-tauri/src/summary/template_commands.rs`
  Экспортирует `api_list_templates` и `api_get_template_details` для фронтенда.
- `frontend/src/hooks/meeting-details/useTemplates.ts`
  Получает список профилей через `api_list_templates`, хранит выбранный `templateId`, передает его в UI.
- `frontend/src/components/MeetingDetails/SummaryGeneratorButtonGroup.tsx`
  Рендерит dropdown выбора шаблона в UI.
- `frontend/src/hooks/meeting-details/useSummaryGeneration.ts`
  Передает выбранный `templateId` в `api_process_transcript`.
- `frontend/src-tauri/src/summary/commands.rs`
  Принимает `template_id` и запускает background summary processing.
- `frontend/src-tauri/src/summary/service.rs`
  Загружает template по `template_id` и вызывает summary processor.
- `frontend/src-tauri/src/summary/processor.rs`
  Формирует chunk/combine/final prompt, передает transcript/summary text в LLM, выполняет language post-processing.
- `frontend/src-tauri/src/summary/llm_client.rs`
  Отправляет `system_prompt` и `user_prompt` в OpenAI-compatible, Ollama и другие провайдеры.

## Где что находится

- Список готовых профилей: `frontend/src-tauri/src/summary/templates/defaults.rs`
- Шаблон `Standard Meeting Notes`: `frontend/src-tauri/templates/standard_meeting.json`
- Генерация final prompt: `frontend/src-tauri/src/summary/processor.rs`
- Передача transcript в LLM: `frontend/src-tauri/src/summary/processor.rs` -> `frontend/src-tauri/src/summary/llm_client.rs`
- UI-список профилей: `frontend/src/hooks/meeting-details/useTemplates.ts` + `frontend/src/components/MeetingDetails/SummaryGeneratorButtonGroup.tsx`

## Выбранный минимальный способ

Выбран минимальный путь без миграций и без изменения pipeline записи/транскрибации:

1. Добавить новый built-in JSON template `sa_meeting_protocol.json`.
2. Зарегистрировать его в `defaults.rs`, чтобы он автоматически попал в `api_list_templates`.
3. Минимально расширить структуру template optional-полями:
   - `markdown_structure` для точного Markdown-каркаса;
   - `final_system_prompt` для адресного override final prompt;
   - `bypass_language_postprocessing` для возврата уже-русского результата без повторного перевода.
4. Использовать эти optional-поля только для `SA Meeting Protocol`, не меняя поведение существующих профилей.
5. Сохранить выбранный `templateId` в `localStorage`, чтобы выбор профиля в UI сохранялся между открытиями страницы.

## Почему не выбран более широкий рефакторинг

- Полноценный редактор prompt не нужен по ТЗ.
- Изменение провайдера, Whisper pipeline, схемы БД и summary service не требуется.
- Изменение только template-layer и final prompt builder остается локальным и простым для поддержки при обновлении upstream.
