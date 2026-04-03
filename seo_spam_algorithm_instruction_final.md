# Алгоритм SEO-спама с подменой релевантного URL
## Готовая инструкция по сборке сайта под глубокий релевантный URL

---

## 1. Цель схемы

Задача схемы — сделать один заранее выбранный глубокий URL главным адресом документа на всех уровнях одновременно:

- в HTML;
- в Open Graph;
- в JSON-LD;
- в breadcrumb;
- в sitemap;
- в серверной маршрутизации;
- в путях к статическим ресурсам.

Если хотя бы один из этих слоёв указывает на другой адрес, схема теряет целостность.

---

## 2. Базовое правило

Вся конфигурация должна быть завязана на один и тот же URL:

```text
https://riobet.62138.casino/ru/ru/ru/ru/ru/ru/ru/ru/
```

Это эталонный адрес. Его нужно использовать без отклонений:

- одинаковое количество сегментов `ru` во всех файлах;
- одинаковый протокол и домен;
- одинаковый завершающий `/`;
- одинаковый URL во всех мета-сигналах и серверных правилах.

Для этой схемы рабочая глубина — **ровно 8 сегментов `ru`**.

Правильный путь:

```text
/ru/ru/ru/ru/ru/ru/ru/ru/
```

Неправильные варианты:

```text
/ru/ru/ru/ru/ru/ru/ru/
/ru/ru/ru/ru/ru/ru/ru/ru/ru/
/ru/ru/ru/ru/ru/ru/ru/ru
```

---

## 3. Правила для `index.html`

### 3.1 `canonical`

В `<head>` должен стоять только один `canonical`, и он обязан указывать на глубокий URL:

```html
<link rel="canonical" href="https://riobet.62138.casino/ru/ru/ru/ru/ru/ru/ru/ru/">
```

Нельзя:

- оставлять `canonical` на корень `/`;
- дублировать второй `canonical`;
- указывать путь с другой глубиной;
- использовать версию без завершающего `/`, если в остальных файлах путь со слэшем.

---

### 3.2 `og:url`

Open Graph URL должен совпадать с `canonical` один в один:

```html
<meta property="og:url" content="https://riobet.62138.casino/ru/ru/ru/ru/ru/ru/ru/ru/">
```

---

### 3.3 JSON-LD

Структурированные данные должны объявлять главным именно глубокий URL.

Минимальный ориентир:

```json
{
  "@context": "https://schema.org",
  "@type": "WebPage",
  "@id": "https://riobet.62138.casino/ru/ru/ru/ru/ru/ru/ru/ru/#webpage",
  "url": "https://riobet.62138.casino/ru/ru/ru/ru/ru/ru/ru/ru/",
  "name": "Riobet казино — официальный сайт, вход и регистрация"
}
```

Если используется `BreadcrumbList`, конечный элемент также обязан ссылаться на тот же URL:

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "Главная",
      "item": "https://riobet.62138.casino/"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "Riobet казино",
      "item": "https://riobet.62138.casino/ru/ru/ru/ru/ru/ru/ru/ru/"
    }
  ]
}
```

Главное правило: `WebPage.url`, `WebPage.@id`, `og:url`, `canonical` и конечный breadcrumb должны быть синхронизированы.

---

## 4. Правила для статических ресурсов

После переноса документа на глубокий путь все ссылки на ассеты должны быть корневыми абсолютными.

Правильно:

```html
<link rel="stylesheet" href="/css/app.css">
<script src="/js/app.js"></script>
<img src="/images/4.svg.gz" alt="">
<link rel="icon" href="/favicon.ico">
```

Неправильно:

```html
<link rel="stylesheet" href="css/app.css">
<script src="js/app.js"></script>
<img src="images/4.svg.gz" alt="">
<link rel="icon" href="favicon.ico">
```

Если оставить относительные пути, страница на глубоком URL начнёт искать ресурсы внутри `/ru/ru/ru/.../`, и часть ассетов перестанет загружаться.

Нужно перевести на абсолютные пути:

- CSS;
- JS;
- изображения;
- иконки;
- манифесты;
- preload/prefetch;
- любые внутренние URL ресурсов в HTML.

---

## 5. Правила для `sitemap.xml`

В карте сайта должен быть только главный глубокий URL.

Рабочий шаблон:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://riobet.62138.casino/ru/ru/ru/ru/ru/ru/ru/ru/</loc>
    <lastmod>2026-04-03</lastmod>
    <changefreq>daily</changefreq>
    <priority>1.0</priority>
  </url>
</urlset>
```

Правила:

- не оставлять корневой URL в том же sitemap;
- не добавлять параллельно второй путь с другой глубиной;
- `loc` должен полностью совпадать с `canonical`;
- если обновляется дата, обновлять только реальное поле `lastmod`, а не плодить новые URL.

---

## 6. Правила для `robots.txt`

`robots.txt` не должен блокировать основной глубокий URL и ресурсы, которые ему нужны.

Рабочий минимальный вариант:

```txt
User-agent: *
Allow: /

Sitemap: https://riobet.62138.casino/sitemap.xml
```

Если используются отдельные ограничения, нельзя запрещать:

- `/ru/ru/ru/ru/ru/ru/ru/ru/`
- `/css/`
- `/js/`
- `/images/`
- favicon, manifest и другие обязательные ресурсы.

---

## 7. Правила серверной маршрутизации

Сервер обязан делать три вещи:

1. перенаправлять корень на нужный глубокий URL;
2. отдавать `index.html` только на точный целевой путь;
3. возвращать `404` для остальных путей под `/ru`, если они не совпадают с эталоном.

---

### 7.1 `Apache` — `.htaccess`

Использовать такую схему:

```apache
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /

  # Корень -> глубокий URL
  RewriteRule ^$ /ru/ru/ru/ru/ru/ru/ru/ru/ [R=301,L]

  # Реальные файлы и папки не трогать
  RewriteCond %{REQUEST_FILENAME} -f [OR]
  RewriteCond %{REQUEST_FILENAME} -d
  RewriteRule ^ - [L]

  # Точный глубокий путь -> index.html
  RewriteRule ^ru/ru/ru/ru/ru/ru/ru/ru/?$ index.html [L]

  # Любой другой путь под /ru -> 404
  RewriteCond %{REQUEST_URI} ^/ru [NC]
  RewriteCond %{REQUEST_URI} !^/ru/ru/ru/ru/ru/ru/ru/ru/?$ [NC]
  RewriteRule .* - [R=404,L]

  # Остальные URL -> index.html
  RewriteRule ^ index.html [L]
</IfModule>
```

---

### 7.2 `Netlify` — `_redirects`

Если используется `_redirects`, нужен такой вариант:

```text
/  /ru/ru/ru/ru/ru/ru/ru/ru/  301!
/ru/ru/ru/ru/ru/ru/ru/ru/  /index.html  200
/ru/*  /404.html  404
/*  /index.html  200
```

Обязательное условие: в корне проекта должен существовать `404.html`, иначе правило на `404` будет ссылаться в пустоту.

---

### 7.3 `Vercel` — `vercel.json`

Для восьми сегментов `ru` шаблон должен быть именно таким:

```json
{
  "routes": [
    { "src": "^/ru(?:/ru){7}/?$", "dest": "/index.html" },
    { "src": "^/ru", "dest": "/404.html", "status": 404 },
    { "handle": "filesystem" },
    { "src": "^/(.*)$", "dest": "/index.html" }
  ]
}
```

Ключевой момент: выражение `^/ru(?:/ru){7}/?$` даёт **ровно 8 сегментов `ru`**.

Нельзя использовать:

```json
"^/ru(?:/ru){8}/?$"
```

Этот вариант уже соответствует **9 сегментам**.

Если нужен отдельный 301 с корня, его надо добавлять отдельным правилом под конкретный режим деплоя. Главное — не ломать правило точного совпадения для целевого пути.

---

### 7.4 `IIS` — `web.config`

Правильный шаблон для восьми сегментов:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="DeepRu8" stopProcessing="true">
          <match url="^ru(?:/ru){7}/?$" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
          </conditions>
          <action type="Rewrite" url="/index.html" />
        </rule>

        <rule name="RuPrefix404" stopProcessing="true">
          <match url="^ru" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
          </conditions>
          <action type="Rewrite" url="/404.html" />
        </rule>

        <rule name="SpaFallback" stopProcessing="true">
          <match url=".*" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
          </conditions>
          <action type="Rewrite" url="/index.html" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

Как и в случае с `_redirects`, файл `/404.html` должен реально существовать.

---

## 8. Что обязательно проверить перед публикацией

### 8.1 Сигналы документа

Проверить, что один и тот же глубокий URL стоит везде:

- `canonical`;
- `og:url`;
- `WebPage.url`;
- `WebPage.@id`;
- конечный breadcrumb;
- кнопки и внутренние ссылки, если они завязаны на self URL.

---

### 8.2 Роутинг

Проверить ответы сервера:

- `/` -> `301` на `/ru/ru/ru/ru/ru/ru/ru/ru/`
- `/ru/ru/ru/ru/ru/ru/ru/ru/` -> `200`
- `/ru/ru/ru/ru/ru/ru/ru/` -> `404`
- `/ru/ru/ru/ru/ru/ru/ru/ru/ru/` -> `404`
- любой случайный путь под `/ru/...` -> `404`

---

### 8.3 Ресурсы

Открыть страницу по глубокому URL и убедиться, что без ошибок грузятся:

- CSS;
- JS;
- изображения;
- favicon;
- любые preload/prefetch ресурсы.

Если что-то запрашивается как `/ru/ru/.../css/...`, значит в коде остались относительные пути.

---

### 8.4 Карта сайта

Проверить, что в `sitemap.xml`:

- только один основной URL;
- это именно глубокий URL;
- он совпадает с `canonical` посимвольно.

---

### 8.5 404-страница

Если конфиг ссылается на `/404.html`, файл обязан существовать физически в корне проекта.

Минимально допустимый шаблон:

```html
<!doctype html>
<html lang="ru">
<head>
  <meta charset="utf-8">
  <title>404</title>
  <meta name="robots" content="noindex, nofollow">
</head>
<body>404</body>
</html>
```

---

## 9. Критические ошибки, которые ломают схему

Нельзя допускать следующие ошибки:

1. `canonical` указывает на один URL, а `sitemap.xml` на другой.
2. `og:url` не совпадает с `canonical`.
3. `WebPage.url` и `WebPage.@id` указывают на другой путь.
4. В breadcrumb стоит другой конечный URL.
5. В конфиге сервера зашита глубина 9 сегментов вместо 8.
6. В HTML остались относительные ссылки на ассеты.
7. Конфиг отправляет в `/404.html`, но самого файла нет.
8. Корень не редиректит на главный глубокий URL.
9. Под `/ru/*` отдаются лишние `200`, хотя должен обслуживаться только один точный путь.
10. На одном сервере схема работает, а на другом нет, потому что правила маршрутизации написаны с разной глубиной.

---

## 10. Финальный регламент

Рабочая сборка считается корректной только если одновременно выполнены все условия:

- главный URL: `https://riobet.62138.casino/ru/ru/ru/ru/ru/ru/ru/ru/`
- глубина: ровно 8 сегментов `ru`
- `canonical` = `og:url` = `WebPage.url` = breadcrumb item = `sitemap loc`
- все ассеты переведены на корневые абсолютные пути
- корень редиректит на глубокий URL
- только целевой путь под `/ru/...` получает `200`
- остальные пути под `/ru` получают `404`
- `404.html` существует, если на него ссылаются конфиги
- все серверные конфиги используют одну и ту же глубину пути

Если хотя бы один пункт не выполнен, сборка считается несогласованной.
