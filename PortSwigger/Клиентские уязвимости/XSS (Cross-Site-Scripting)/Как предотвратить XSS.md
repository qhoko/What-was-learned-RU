## 🛡️ Как предотвратить XSS

В этом разделе мы опишем общие принципы предотвращения уязвимостей межсайтового скриптинга (XSS) и обсудим, как использовать различные технологии для защиты от XSS-атак.

### Два уровня защиты

Предотвращение XSS обычно достигается с помощью двух уровней защиты:

1. **Кодирование данных на выходе**
2. **Проверка входных данных**

Вы можете использовать **Burp Scanner** для сканирования ваших веб-сайтов на предмет многочисленных уязвимостей, включая XSS. Продвинутая логика сканирования Burp воспроизводит действия опытного злоумышленника и достигает высокого покрытия уязвимостей XSS. Используйте Burp Scanner, чтобы убедиться в эффективности вашей защиты от атак XSS.

---

### 1. Кодирование данных на выходе

Кодирование должно применяться непосредственно перед записью пользовательских данных на страницу, так как контекст, в который вы записываете, определяет тип необходимого кодирования. Например, значения внутри строки JavaScript требуют другого кодирования, чем в контексте HTML.

- **HTML-контекст:** Преобразуйте ненадежные значения в HTML-сущности:
  - `<` становится `&lt;`
  - `>` становится `&gt;`

- **JavaScript-контекст строки:** Небуквенно-цифровые значения должны быть закодированы в Unicode:
  - `<` становится `\u003c`
  - `>` становится `\u003e`

Иногда требуется несколько слоёв кодирования в правильном порядке. Например, чтобы безопасно встроить пользовательский ввод в обработчик событий, сначала закодируйте ввод в Unicode, затем в HTML:

```html
<a href="#" onclick="x='Эта строка требует двух уровней кодирования'">тест</a>
```

---

### 2. Проверка входных данных

Кодирование является важной линией защиты от XSS, но этого недостаточно для всех контекстов. Также следует строго проверять входные данные при их получении от пользователя.

Примеры проверки входных данных:

- Если пользователь отправляет URL, убедитесь, что он начинается с безопасного протокола, например, `http` или `https`.
- Если пользователь вводит значение, ожидаемое как числовое, проверьте, что оно содержит только целые числа.
- Ограничьте входные данные ожидаемыми символами.

#### Белый список против чёрного списка

Проверка входных данных должна использовать белые списки вместо чёрных. Например, вместо попытки перечислить все вредоносные протоколы (например, `javascript`, `data`), разрешите только безопасные протоколы (`http`, `https`). Это гарантирует, что защита останется эффективной при появлении новых векторов атак.

---

### Разрешение "безопасного" HTML

Избегайте разрешения пользователям публиковать HTML-разметку, если это возможно. Если это необходимо, используйте такие библиотеки, как **DOMPurify**, для фильтрации и кодирования HTML в браузере. Альтернативно, используйте Markdown для ввода пользователем контента, преобразуя его в HTML серверной стороной. Обновляйте библиотеки для устранения уязвимостей XSS.

> **Примечание:** Помимо JavaScript, вредоносным может быть CSS и обычный HTML.

---

### Предотвращение XSS с использованием шаблонизаторов

Современные веб-приложения часто используют серверные шаблонизаторы, такие как Twig или Freemarker, для встраивания динамического контента в HTML. Обычно они имеют встроенные механизмы экранирования. Например, в Twig:

```twig
{{ user.firstname | e('html') }}
```

Фреймворки, такие как Jinja или React, автоматически экранируют динамический контент, эффективно предотвращая большинство случаев XSS. Изучите функции экранирования используемого шаблонизатора или фреймворка.

> **Примечание:** Прямое объединение пользовательского ввода в строки шаблона может привести к внедрению шаблона на стороне сервера, что часто опаснее, чем XSS.

---

### Предотвращение XSS в PHP

В PHP используйте встроенную функцию `htmlentities` для кодирования сущностей в HTML-контексте. Вызывайте её с тремя аргументами:

1. Входная строка.
2. `ENT_QUOTES` для кодирования всех кавычек.
3. Набор символов, обычно `UTF-8`.

Пример:

```php
<?php echo htmlentities($input, ENT_QUOTES, 'UTF-8'); ?>
```

Для строк JavaScript кодируйте ввод в Unicode, так как PHP не предоставляет встроенный API для этого. Используйте пользовательскую функцию:

```php
function jsEscape($str) {
    $output = '';
    foreach (str_split($str) as $char) {
        $chrNum = ord($char);
        if (!ctype_alnum($char)) {
            $output .= sprintf('\\u%04x', $chrNum);
        } else {
            $output .= $char;
        }
    }
    return $output;
}
```

Пример использования:

```php
<script>x = '<?php echo jsEscape($_GET['x']) ?>';</script>
```

---

### Предотвращение XSS на стороне клиента в JavaScript

Чтобы кодировать пользовательский ввод в HTML-контексте в JavaScript, используйте пользовательский HTML-кодировщик:

```javascript
function htmlEncode(str) {
    return String(str).replace(/[^\w. ]/gi, c => `&#${c.charCodeAt(0)};`);
}
```

Для строк JavaScript выполняйте кодирование в Unicode:

```javascript
function jsEscape(str) {
    return String(str).replace(/[^\w. ]/gi, c => `\\u${('0000' + c.charCodeAt(0).toString(16)).slice(-4)}`);
}
```

Пример использования:

```javascript
<script>document.body.innerHTML = htmlEncode(untrustedValue);</script>
```

---

### Смягчение последствий XSS с помощью политики безопасности контента (CSP)

CSP является последней линией защиты от XSS. Включите заголовок HTTP-ответа `Content-Security-Policy` со следующей политикой:

```
default-src 'self'; script-src 'self'; object-src 'none'; frame-src 'none'; base-uri 'none';
```

Эта политика ограничивает загрузку ресурсов только с того же источника, что и главная страница. Даже если злоумышленник внедрит полезную нагрузку XSS, его возможности будут существенно ограничены.
