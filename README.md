# Yandex Metrika Core

> Библиотека и каталог skills для работы с Яндекс.Метрикой в OpenCode

Core-навык предоставляет общий код для всех skills Яндекс.Метрики:
- OAuth авторизация
- API клиент
- Проверка безопасности (.gitignore)
- Экспорт в CSV/Markdown

## Доступные skills

| Skill | Описание | Репозиторий |
|-------|----------|-------------|
| yandex-metrika-search | Поисковые фразы с метриками | [github](https://github.com/prikotov/yandex-metrika-search) |

## Установка

### 1. Установите core

```bash
cd your-project
git clone https://github.com/prikotov/yandex-metrika-core.git .opencode/skills/yandex-metrika-core
```

### 2. Создайте OAuth-приложение Яндекс

1. Перейдите на https://oauth.yandex.ru/client/new
2. Заполните:
   - **Название**: `Metrika Stats`
   - **Платформа**: Веб-сервисы
   - **Redirect URI**: `https://oauth.yandex.ru/verification_code`
   - **Доступы**: Яндекс.Метрика → Чтение
3. Скопируйте:
   - **ClientID** → `client_id`
   - **Client secret** → `client_secret`

### 3. Узнайте номер счётчика

В настройках Яндекс.Метрики или в коде: `ym(XXXXXX, 'init'...)`

### 4. Создайте конфигурацию

```bash
cd .opencode/skills/yandex-metrika-core
cp metrika_config.example.json ../../../metrika_config.json
```

Заполните:
```json
{
    "client_id": "ваш_client_id",
    "client_secret": "ваш_client_secret",
    "counter_id": 12345678
}
```

### 5. Установите нужные skills

```bash
git clone https://github.com/prikotov/yandex-metrika-search.git .opencode/skills/yandex-metrika-search
```

## Структура

```
your-project/
├── metrika_config.json          # Общий конфиг (создаётся вручную в корне проекта)
├── yandex_token.json            # Создаётся автоматически при первом запуске
├── metrika_reports/             # Создаётся автоматически при запуске отчёта
└── .opencode/skills/
    ├── yandex-metrika-core/     # Библиотека
    │   ├── MetrikaClient.php
    │   └── SKILL.md
    └── yandex-metrika-search/   # Skill для поисковых фраз
```

## Безопасность

Все skills автоматически проверяют `.gitignore`:
- Создают если нет
- Добавляют недостающие записи

Защищаемые файлы: `metrika_config.json`, `yandex_token.json`, `metrika_reports/`

## Создание нового skill

1. Создайте репозиторий `yandex-metrika-XXX`
2. Подключите MetrikaClient:
```php
<?php
require_once __DIR__ . '/../yandex-metrika-core/MetrikaClient.php';

MetrikaClient::checkGitignore();
$config = MetrikaClient::loadConfig();

$client = new MetrikaClient(
    $config['client_id'],
    $config['client_secret'],
    $config['counter_id']
);

// Ваш код...
```

### MetrikaClient API

```php
// Загрузка конфигурации
$config = MetrikaClient::loadConfig();

// Создание клиента
$client = new MetrikaClient(
    $config['client_id'],
    $config['client_secret'],
    $config['counter_id']
);

// API запрос
$data = $client->request([
    'ids' => $client->getCounterId(),
    'metrics' => 'ym:s:visits',
    'dimensions' => 'ym:s:lastSearchPhrase',
    'date1' => '2026-01-01',
    'date2' => '2026-02-28'
]);
```

## Требования

- PHP 7.4+
- Расширение cURL

---

Постановка задач, архитектура, ревью — [Dmitry Prikotov](https://prikotov.pro/)

Реализация — [OpenCode](https://opencode.ai) + GLM-5
