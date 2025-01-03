## ❓ Что такое хранимый межсайтовый скриптинг (XSS)?

Хранимый межсайтовый скриптинг (также известный как XSS второго порядка или постоянный XSS) возникает, когда приложение получает данные из ненадежного источника и включает эти данные в свои последующие HTTP-ответы небезопасным способом.

Например, веб-сайт может позволять пользователям оставлять комментарии к записям в блоге, которые затем отображаются для других пользователей. Комментарии отправляются с помощью HTTP-запроса следующего вида:

```http
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Length: 100

postId=3&comment=Этот+пост+был+очень+полезен.&name=Карлос+Монтоя&email=carlos%40normal-user.net
```

После отправки этот комментарий может быть включен в ответ приложения следующим образом:

```html
<p>Этот пост был очень полезен.</p>
```

Если приложение не обрабатывает данные должным образом, злоумышленник может отправить вредоносный комментарий следующего вида:

```html
<script>/* Вредоносный код... */</script>
```

Запрос злоумышленника будет закодирован следующим образом:

```html
comment=%3Cscript%3E%2F*%2BВредоносный%2Bкод...%2B*%2F%3C%2Fscript%3E
```

Теперь любой пользователь, посетивший запись в блоге, получит следующий ответ приложения:

```html
<p><script>/* Вредоносный код... */</script></p>
```

Скрипт, предоставленный злоумышленником, будет выполнен в браузере пользователя-жертвы в контексте его сеанса работы с приложением.

---

## 📉 Влияние сохраненных XSS-атак

Если злоумышленник может контролировать скрипт, выполняемый в браузере жертвы, он может:

- Выполнить любые действия, которые пользователь может выполнить в приложении.
- Похитить конфиденциальные данные пользователя.
- Маскироваться под пользователя или использовать его учетные данные.

Ключевое отличие отраженного XSS от сохраненного заключается в следующем:

- **Отраженный XSS** требует, чтобы злоумышленник убедил пользователей выполнить определенные запросы, содержащие эксплойт.
- **Сохраненный XSS** автоматически внедряет эксплойт в приложение, срабатывая, когда пользователь сталкивается с ним.

Самодостаточность сохраненных XSS делает их особенно опасными, особенно если уязвимость затрагивает пользователей, которые вошли в систему. Злоумышленнику не нужно рассчитывать время атаки, как в случае с отраженным XSS. Эксплойт ждет, пока пользователь не столкнется с ним.

---

## 📂 Сохраненные XSS в разных контекстах

Существует множество разновидностей сохраненного XSS. Расположение сохраненных данных в ответе приложения определяет:

1. **Тип полезной нагрузки, необходимой для эксплуатации уязвимости.**
2. **Потенциальное воздействие уязвимости.**

Если приложение выполняет проверку или обработку данных перед их сохранением или включением в ответы, это также влияет на тип XSS-полезной нагрузки.

---

## 🔍 Как найти и протестировать сохраненные уязвимости XSS

Ручное тестирование уязвимостей XSS может быть сложным. Необходимо изучить все соответствующие "точки входа", через которые данные, контролируемые злоумышленником, могут попасть в приложение, и "точки выхода", где эти данные могут появиться в ответах приложения.

### Точки входа

Наиболее распространенные точки входа:

- Параметры или другие данные в строке запроса URL и теле сообщения.
- Путь URL.
- Заголовки HTTP-запросов (менее часто эксплуатируемы для отраженного XSS).
- Внеполосные маршруты, такие как:
  - Электронные письма, обрабатываемые приложением веб-почты.
  - Твиты сторонних пользователей, отображаемые в приложении социальных сетей.
  - Данные, агрегированные с других веб-сайтов.

### Точки выхода

Точками выхода для сохраненных XSS-атак являются все возможные HTTP-ответы, возвращаемые любому пользователю приложения в любом контексте.

### Шаги для тестирования сохраненных XSS

1. **Определите связи между точками входа и выхода**:
   - Найдите, как данные, отправленные в точку входа, могут появляться в точке выхода. Это может потребовать тестирования различных комбинаций ввода и анализа ответов.

2. **Учитывайте перезапись данных**:
   - Сохраненные данные часто могут быть перезаписаны другими действиями в приложении. Например, функция поиска может отображать последние запросы, которые заменяются новыми поисками.

3. **Сфокусируйтесь на ключевых функциях приложения**:
   - Особое внимание уделите таким функциям, как комментарии к сообщениям в блоге или профили пользователей.

4. **Тестируйте каждую связь между точками входа и выхода**:
   - Отправьте определенный ввод в точку входа и наблюдайте за соответствующим выводом в точке выхода. Проверьте, появляются ли данные в ответах.

5. **Определите контекст**:
   - Когда данные появляются в ответе, определите контекст (например, внутри HTML-тегов, JavaScript или атрибутов). Это поможет адаптировать XSS-полезную нагрузку.

6. **Проведите попытку эксплуатации**:
   - Используйте подходящие полезные нагрузки для проверки, возможно ли успешно эксплуатировать уязвимость.

Следуя этим шагам, вы сможете идентифицировать и подтвердить сохраненные уязвимости XSS, а также оценить их потенциальное воздействие на приложение.
