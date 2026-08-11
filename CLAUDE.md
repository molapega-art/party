# PartySpeakers

Веб-приложение: превращает несколько телефонов в синхронную сеть колонок. Один хост управляет
плейлистом, остальные телефоны («колонки») играют один трек в такт. Синхронизация — по серверным
часам Firebase (общее время у всех устройств) с коррекцией дрейфа.

Весь код — в одном файле **`index.html`** (без сборки, чистая статика).

## Как это работает (кратко)
- **Хост** создаёт вечеринку → код из 4 цифр + QR. Заливает MP3, управляет play/pause/next/prev.
- **Колонка** заходит по коду (или по QR-ссылке `?p=КОД`), качает трек, декодирует в AudioBuffer, играет.
- **Синхрон**: `.info/serverTimeOffset` → общие часы; отложенный старт (`START_DELAY_MS`);
  коррекция дрейфа (мягкая — скоростью ±0.5%, жёсткая — перезапуск); ручная подстройка `nudge`; фейды.

## Деплой / инфраструктура

### GitHub
- Репозиторий: **`molapega-art/party`** (публичный) — `https://github.com/molapega-art/party`
- Аккаунт GitHub: `molapega-art`. Подключение: `gh auth login -h github.com` (device flow в браузере).
- Клон: `gh repo clone molapega-art/party`

### Хостинг — GitHub Pages
- **URL: https://molapega-art.github.io/party/** (ветка `main`, корень; HTTPS принудительно).
- Деплой = просто `git push` в `main`. Настройка Pages: `gh api repos/molapega-art/party/pages`.
- HTTPS обязателен (Web Audio / wake lock / autoplay-разлочка требуют secure context).

### Firebase — проект `party-speakers`
- Управление через CLI: `firebase ... --project party-speakers` (CLI уже залогинен).
- **Realtime Database** (europe-west1): `https://party-speakers-default-rtdb.europe-west1.firebasedatabase.app`
  - Хранит: `parties/<код>/{meta,state,speakers,tracks,trackdata}`.
  - Правила — в `database.rules.json`, деплой: `firebase deploy --only database --project party-speakers`.
  - Правила scoped: писать/читать можно только под `parties/<4 цифры>`, с валидацией полей и размеров
    (раньше было полностью открыто `{".read":true,".write":true}`).
- **Storage** (`party-speakers.firebasestorage.app`): **пока НЕ подключён**.
  - Чтобы включить: консоль → Storage → Get Started (вероятно потребует план Blaze).
  - Правила уже готовы в `storage.rules`, деплой: `firebase deploy --only storage --project party-speakers`.
  - Нужен для миграции музыки из БД в Storage (сейчас аудио лежит в RTDB в base64 — дорого и упирается в лимиты).
- Веб-конфиг Firebase зашит в `index.html` (`FIREBASE_CONFIG`). `apiKey` тут **публичный по дизайну**
  (не секрет) — безопасность держится на правилах БД/Storage, а не на сокрытии ключа.
- Получить актуальный веб-конфиг: `firebase apps:sdkconfig WEB --project party-speakers`.

### Что стоит сделать дальше (hardening)
- Включить **Anonymous Auth** в консоли и ужесточить правила БД/Storage до `auth != null`.
- Подключить **Storage** и перенести аудио туда (или перейти на WebRTC-раздачу между устройствами).

## Секреты
- Настоящих секретов в репозитории нет (Firebase web apiKey — публичный).
- Если появятся приватные ключи (напр. service account) — только в `.env` (в `.gitignore`), не в коде/заметках.
