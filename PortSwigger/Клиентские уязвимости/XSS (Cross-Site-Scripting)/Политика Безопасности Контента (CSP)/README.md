Политика безопасности контента
В этом разделе мы объясним, что такое политика безопасности контента, и опишем, как можно использовать CSP для предотвращения некоторых распространенных атак.

Что такое CSP (политика безопасности контента)?
CSP — это механизм безопасности браузера, который направлен на смягчение XSS и некоторых других атак. Он работает, ограничивая ресурсы (такие как скрипты и изображения), которые может загрузить страница, и ограничивая, может ли страница быть обрамлена другими страницами.

Чтобы включить CSP, ответ должен включать заголовок ответа HTTP, вызываемый Content-Security-Policyсо значением, содержащим политику. Сама политика состоит из одной или нескольких директив, разделенных точкой с запятой.

Смягчение XSS-атак с помощью CSP
Следующая директива позволит загружать скрипты только из того же источника, что и сама страница:

script-src 'self'
Следующая директива разрешит загрузку скриптов только с определенного домена:

script-src https://scripts.normal-website.com
Следует проявлять осторожность при разрешении скриптов с внешних доменов. Если у злоумышленника есть возможность контролировать контент, который обслуживается с внешнего домена, то он может осуществить атаку. Например, сети доставки контента (CDN), которые не используют URL-адреса для каждого клиента, такие как ajax.googleapis.com, не следует доверять, поскольку третьи лица могут размещать контент на своих доменах.

Помимо внесения в белый список определенных доменов, политика безопасности контента также предоставляет два других способа указания доверенных ресурсов: одноразовые номера и хэши:

Директива CSP может указывать nonce (случайное значение), и то же значение должно использоваться в теге, загружающем скрипт. Если значения не совпадают, скрипт не будет выполнен. Чтобы быть эффективным в качестве элемента управления, nonce должен безопасно генерироваться при каждой загрузке страницы и не поддаваться угадыванию злоумышленником.
Директива CSP может указывать хэш содержимого доверенного скрипта. Если хэш фактического скрипта не соответствует значению, указанному в директиве, то скрипт не будет выполнен. Если содержимое скрипта когда-либо изменится, то вам, конечно, нужно будет обновить хэш-значение, указанное в директиве.
Довольно часто CSP блокируют такие ресурсы, как script. Однако многие CSP разрешают запросы изображений. Это означает, что вы часто можете использовать imgэлементы для отправки запросов на внешние серверы, чтобы раскрыть токены CSRF, например.

Некоторые браузеры, такие как Chrome, имеют встроенную функцию предотвращения висячей разметки, которая блокирует запросы, содержащие определенные символы, такие как необработанные, незакодированные символы новой строки или угловые скобки.

Некоторые политики более строгие и запрещают все формы внешних запросов. Однако все еще возможно обойти эти ограничения, вызвав некоторое взаимодействие с пользователем . Чтобы обойти эту форму политики, вам нужно внедрить элемент HTML, который при щелчке будет сохранять и отправлять все, что заключено во внедренном элементе, на внешний сервер.
