# Yandex Metrika Core

> Библиотека и каталог skills для работы с Яндекс.Метрикой

Этот пакет содержит MetrikaClient — общий API-клиент для всех skills Яндекс.Метрики:
- OAuth авторизация
- Запросы к API Метрики
- Проверка безопасности (.gitignore)
- Экспорт в CSV/Markdown

## Skills на основе этого пакета

| Skill | Описание | Репозиторий |
|-------|----------|-------------|
| yandex-metrika-search | Поисковые фразы с метриками | [github.com/prikotov/yandex-metrika-search](https://github.com/prikotov/yandex-metrika-search) |
| yandex-metrika-visitors | Анализ посетителей по срезам | [github.com/prikotov/yandex-metrika-visitors](https://github.com/prikotov/yandex-metrika-visitors) |
| yandex-metrika-pages | Статистика по страницам | [github.com/prikotov/yandex-metrika-pages](https://github.com/prikotov/yandex-metrika-pages) |
| yandex-metrika-traffic | Источники трафика | [github.com/prikotov/yandex-metrika-traffic](https://github.com/prikotov/yandex-metrika-traffic) |

## Установка

Skills совместимы с различными AI-агентами. Примеры ниже даны для OpenCode — для других инструментов смотрите их документацию по установке skills.

### 1. Установите core

```bash
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
cp .opencode/skills/yandex-metrika-core/metrika_config.example.json ./metrika_config.json
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
git clone https://github.com/prikotov/yandex-metrika-visitors.git .opencode/skills/yandex-metrika-visitors
git clone https://github.com/prikotov/yandex-metrika-pages.git .opencode/skills/yandex-metrika-pages
git clone https://github.com/prikotov/yandex-metrika-traffic.git .opencode/skills/yandex-metrika-traffic
```

## Структура

```
your-project/
├── metrika_config.json          # Общий конфиг (создаётся вручную в корне проекта)
├── yandex_token.json            # Создаётся автоматически при первом запуске
├── metrika_reports/             # Создаётся автоматически при запуске отчёта
│   └── YYYY-MM-DD/              # Папка с отчётами за день
└── .opencode/skills/
    ├── yandex-metrika-core/     # Библиотека
    ├── yandex-metrika-search/   # Поисковые фразы
    ├── yandex-metrika-visitors/ # Посетители
    ├── yandex-metrika-pages/    # Страницы
    └── yandex-metrika-traffic/  # Источники трафика
```

## Безопасность

MetrikaClient автоматически защищает конфиденциальные данные от случайной публикации в git. При первом запуске он проверяет `.gitignore` и добавляет недостающие записи.

Защищаемые файлы:
- `metrika_config.json` — OAuth-данные приложения
- `yandex_token.json` — токен авторизации
- `metrika_reports/` — папка с отчётами

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

### Пример использования MetrikaClient API

Ниже — полный пример skill, который запрашивает поисковые фразы и сохраняет отчёт.

```php
<?php
require_once __DIR__ . '/../yandex-metrika-core/MetrikaClient.php';

// 1. Проверка .gitignore и загрузка конфига
MetrikaClient::checkGitignore();
$config = MetrikaClient::loadConfig();

// 2. Создание клиента
$client = new MetrikaClient(
    $config['client_id'],
    $config['client_secret'],
    $config['counter_id']
);

// 3. Запрос к API Метрики
// Документация параметров: https://yandex.ru/dev/metrika/doc/api/createdownstat/createdownstat.html
$data = $client->request([
    'ids' => $client->getCounterId(),
    'metrics' => 'ym:s:visits,ym:s:pageviews',      // что измеряем
    'dimensions' => 'ym:s:lastSearchPhrase',        // по чему группируем
    'date1' => '2026-01-01',
    'date2' => '2026-02-28',
    'limit' => 100
]);

// 4. Преобразование ответа в плоский массив
$rows = [];
foreach ($data['data'] as $item) {
    $rows[] = [
        'Фраза' => $item['dimensions'][0]['name'],
        'Визиты' => $item['metrics'][0],
        'Просмотры' => $item['metrics'][1]
    ];
}

// 5. Сохранение отчёта в CSV и Markdown
$reportDir = MetrikaClient::createReportDir();
MetrikaClient::saveCsv($rows, "$reportDir/report.csv");
MetrikaClient::saveMarkdown($rows, "$reportDir/report.md", 'Поисковые фразы', '2026-01-01', '2026-02-28');
```

## Требования

- PHP 7.4+
- Расширение cURL

---

> Постановка задач, архитектура, ревью — [Dmitry Prikotov](https://prikotov.pro/), реализация — GLM-5 в [OpenCode](https://opencode.ai)
