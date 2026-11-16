# Privacy Policy — Tickets Helper

**Last updated:** November 2026

## Summary

Tickets Helper is a browser extension that helps support agents quickly find and fill in a ready-made reply template based on the text of a customer request, and (optionally) open the related client card in the company's backoffice. **The text of a request is processed locally, on the user's own device, and is never sent anywhere.**

The extension sends data outside the browser in only three cases, all described below: (1) a plain download of the template list, (2) a read-only client check in the company's backoffice when the user clicks "Process + open", and (3) **anonymous-style usage statistics** — a short record that a button was clicked, together with the user's backoffice login name — sent to a Google Sheet owned and controlled by the extension's administrator inside the organization's Google Workspace. None of this goes to any advertising or analytics service.

## What data the extension handles, and where it goes

| Data | What happens to it |
|---|---|
| Text pasted into the "Request" field (may contain a client ID, email, card number, deposit amount) | Processed entirely in the browser (locally) to detect a matching template and to extract values for substitution into that template. **Never sent to the developer, to the statistics sheet, or to any third party.** Cleared when the popup closes; nothing is logged or retained beyond the current session. The only value taken from it that is ever sent anywhere is the client ID, and only for the backoffice check described below. |
| Email typed into the "Email masking" field | Masked locally in the browser. **Never sent anywhere.** Only the fact that the "Hide" button was clicked is counted in usage statistics (see below), not the email itself. |
| Client ID | Used locally for template substitution. If the user clicks "Process + open" on the backoffice site, it is also sent to the company's backoffice API (`backofficewebadmin.betconstruct.com`) as part of a read-only lookup. It is never sent to the developer or to the statistics sheet. |
| Backoffice session token | Read by the extension inside the backoffice page itself (from that page's own local storage) only at the moment of the client check, and used only for that single request to the backoffice API, exactly as the backoffice page does it. It is never passed to the extension popup, never stored, logged, or transmitted anywhere else. |
| Backoffice login name (`UserName`) | Read inside the backoffice page from the page's own `userData` local-storage entry. **Only the `UserName` field is read; no other field of that entry and no token is read or returned to the extension.** The name is checked against a strict pattern (letters, digits, `.`, `-`, `_`) and, if it is an email address, only the part before "@" is kept. It is stored locally in the extension's storage (name + time, no token) for up to 7 days so that clicks made on other tabs can still be attributed, and it is included in usage statistics events (see below). If no login can be determined, the value "невідомий користувач" ("unknown user") is used instead. |
| Usage statistics events | When the user clicks "Process", "Process + open" or "Hide" (with non-empty input), the extension records one short event and sends it to the statistics sheet described below. An event contains **only**: a random event ID (to avoid counting the same click twice), the button type, whether the "Header" option was on (for the two processing buttons), the user's backoffice login name (or "unknown user"), the extension version, and the time of the click. It contains **no** request text, client ID, email, template name, token, or other client data. Events that could not be sent (for example, while offline) wait in a local queue (up to 200 events, no longer than 14 days) and are sent later. |
| Address (URL) of the active tab | Read when the popup is opened, only to determine whether the current tab is the company's backoffice (so the "Process + open" button is shown only there, and the login name can be read there). Not stored or transmitted. The extension does not read or monitor other tabs or browsing activity. |
| Result of the client check | Only a minimal result (client found / not found / session expired) is returned to the popup and shown to the user. Client details from the backoffice response are not displayed, copied, or stored. |
| Template library | Stored locally in the browser's extension storage (`chrome.storage.local`), on the user's own device only. Not synced to a Google account, not accessible to other extensions or websites. |
| User preference (e.g. the "Header" toggle) | Stored locally the same way, purely to remember the user's UI preference between sessions. |
| Clipboard | The extension writes the finished reply text to the system clipboard only when the user clicks "Copy" (or when a template is selected and copied). No clipboard content is read, stored, or transmitted. |

## Network access

The extension makes **three types of network requests**:

1. **Template library update.** When the popup is opened, and at most once every 4 hours, the extension checks a single static file (a plain list of reply templates) hosted at a URL controlled by the developer (`raw.githubusercontent.com`), to keep the template library up to date. This request:
   - sends no personal data, request text, or identifying information — it is a plain file download;
   - does not execute any remote code — only text/JSON data is read and displayed;
   - fails silently if there is no network connection; the extension continues working with its last cached copy.

2. **Client check in the backoffice.** Only when the user clicks "Process + open" on the company's backoffice site, the extension sends one read-only (GET) request per click to the company's backoffice API to check whether the client ID from the request exists. The client ID is part of the request address, and the user's existing, already signed-in backoffice session is used for authorization. This request goes to the company's own backoffice system (the same system the user is working in), not to the developer. The extension only determines from the response whether the client exists; it does not read out, display, or store client details. It does not execute any remote code.

3. **Usage statistics.** After a counted button click, the extension sends the event described above (one or a few events at a time) to a Google Apps Script web application (`script.google.com`) that writes it into a Google Sheet. The script and the sheet belong to the extension's administrator within the organization's Google Workspace; access to the sheet is limited to the administrator, and the script only **writes** events — it cannot be used to read the sheet. The script validates every event and discards malformed ones. To restrict writing to members of the organization, the request carries the user's Google session cookies for `script.google.com`; the extension does not read, store, or forward these cookies — the browser attaches them. The statistics are used only to understand how actively the tool is used (counts per button, per user and per day) and to build summary charts. They are not sent to advertising, analytics, or other third-party services, and the extension executes no remote code.

No other network requests are made. The extension does not use analytics, tracking pixels, advertising SDKs, or any third-party telemetry service.

## Permissions and why they are needed

| Permission | Purpose |
|---|---|
| `clipboardWrite` | To copy the prepared reply text to the clipboard when the user clicks "Copy". The clipboard is never read. |
| `storage` | To keep the template library, UI settings, the short-lived login-name cache, and the queue of unsent statistics events locally on the user's device (`chrome.storage.local`). |
| Host access to `raw.githubusercontent.com` | To download the static template list described above. |
| Host access to `script.google.com` and `script.googleusercontent.com` | To send usage statistics events to the administrator's Google Apps Script web application described above, and to confirm that they were received. |
| `activeTab` | To read the address of the active tab when the popup is opened, so the "Process + open" button is shown only on the company's backoffice, and to allow the functions described below to run in that tab. |
| `scripting` | To run fixed functions (bundled in the extension; no remote code) in the active backoffice tab: the client check after the user clicks "Process + open", and reading the login name (`UserName` only) for usage statistics. |

## Data retention and deletion

Data stored locally by the extension lives only in the user's browser storage: nothing from the client check (client ID, token, response) is retained, the login-name cache is kept for at most 7 days, and unsent statistics events for at most 14 days. Removing the extension, or clearing its data via the browser's extension settings, deletes all local data immediately.

Usage statistics events that have been sent are stored in the administrator's Google Sheet for as long as the administrator keeps them to analyze usage; the administrator can delete them at any time. The developer has no other server-side copy of any user's data.

## Contact

Questions about this policy, or requests to delete a user's usage statistics records, can be directed to the extension's developer/administrator within the organization that distributed it.

---

# Політика приватності — Tickets Helper (українською)

**Востаннє оновлено:** листопад 2026

## Коротко

Tickets Helper — розширення браузера, яке допомагає операторам підтримки швидко підібрати й заповнити готовий шаблон відповіді за текстом запиту клієнта та (за бажанням) відкрити відповідну картку клієнта в бекофісі компанії. **Текст запиту обробляється локально, на пристрої користувача, і нікуди не надсилається.**

За межі браузера розширення надсилає дані лише в трьох випадках, описаних нижче: (1) звичайне завантаження списку шаблонів, (2) перевірка клієнта в бекофісі компанії лише на читання, коли користувач натискає «Обробити + відкрити», та (3) **статистика використання** — короткий запис про натискання кнопки разом із логіном користувача в бекофісі — у Google Таблицю, якою володіє й яку контролює адміністратор розширення в межах Google Workspace організації. Жодних рекламних чи аналітичних сервісів не використовується.

## Які дані обробляються і що з ними відбувається

| Дані | Що з ними відбувається |
|---|---|
| Текст, вставлений у поле «Запит» (може містити ID клієнта, email, номер картки, суму депозиту) | Обробляється повністю в браузері (локально) для підбору шаблону та витягування значень для підстановки. **Ніколи не надсилається розробнику, у таблицю статистики чи третім сторонам.** Очищується при закритті попапу, нічого не логується й не зберігається довше поточної сесії. Єдине значення з нього, яке взагалі кудись надсилається, — ID клієнта, і лише для перевірки в бекофісі, описаної нижче. |
| Email, введений у поле «Маскування Email» | Маскується локально в браузері. **Ніколи нікуди не надсилається.** У статистиці використання (див. нижче) рахується лише факт натискання кнопки «Приховати», а не сам email. |
| ID клієнта | Використовується локально для підстановки в шаблон. Якщо користувач натискає «Обробити + відкрити» на сайті бекофісу, ID також надсилається до API бекофісу компанії (`backofficewebadmin.betconstruct.com`) у вигляді запиту лише на читання. Розробнику й у таблицю статистики він ніколи не надсилається. |
| Токен сесії бекофісу | Зчитується розширенням усередині самої сторінки бекофісу (з її власного локального сховища) лише в момент перевірки клієнта й використовується лише для цього одного запиту до API бекофісу — так само, як це робить сама сторінка бекофісу. Ніколи не передається в попап розширення, не зберігається, не логується й не надсилається в жодне інше місце. |
| Логін в бекофісі (`UserName`) | Зчитується всередині сторінки бекофісу з її власного запису `userData` у локальному сховищі. **Читається лише поле `UserName`; жодні інші поля цього запису й жодний токен не читаються й не передаються в розширення.** Ім'я перевіряється за суворим шаблоном (літери, цифри, `.`, `-`, `_`); якщо це email, зберігається лише частина до «@». Воно зберігається локально в сховищі розширення (ім'я + час, без токена) до 7 діб, щоб натискання на інших вкладках теж можна було віднести до користувача, і входить у події статистики використання (див. нижче). Якщо логін визначити не вдалося, використовується значення «невідомий користувач». |
| Події статистики використання | Коли користувач натискає «Обробити», «Обробити + відкрити» чи «Приховати» (за непорожнього вводу), розширення записує одну коротку подію й надсилає її в таблицю статистики, описану нижче. Подія містить **лише**: випадковий ID події (щоб не порахувати один клік двічі), тип кнопки, чи було ввімкнено «Шапку» (для двох кнопок обробки), логін користувача в бекофісі (або «невідомий користувач»), версію розширення та час натискання. Вона **не** містить тексту запиту, ID клієнта, email, назви шаблону, токена чи інших даних клієнта. Події, які не вдалося відправити (наприклад, без інтернету), чекають у локальній черзі (до 200 подій, не довше 14 днів) і надсилаються пізніше. |
| Адреса (URL) активної вкладки | Зчитується при відкритті попапу, лише щоб визначити, чи поточна вкладка — бекофіс компанії (щоб кнопка «Обробити + відкрити» показувалась лише там, а логін можна було прочитати саме там). Не зберігається й не передається. Розширення не зчитує й не відстежує інші вкладки чи активність у браузері. |
| Результат перевірки клієнта | У попап повертається лише мінімальний результат (клієнта знайдено / не знайдено / сесію завершено) і показується користувачу. Дані клієнта з відповіді бекофісу не відображаються, не копіюються й не зберігаються. |
| Бібліотека шаблонів | Зберігається локально у сховищі розширення (`chrome.storage.local`), лише на пристрої користувача. Не синхронізується через Google-акаунт, недоступна іншим розширенням чи сайтам. |
| Налаштування користувача (напр. перемикач «Шапка») | Зберігається так само локально, лише щоб запам'ятати вибір інтерфейсу між сесіями. |
| Буфер обміну | Розширення записує готовий текст відповіді в буфер обміну лише за натисканням кнопки «Скопіювати» (або коли шаблон вибрано й скопійовано). Вміст буфера ніколи не зчитується, не зберігається і не передається. |

## Мережевий доступ

Розширення робить **три типи мережевих запитів**:

1. **Оновлення бібліотеки шаблонів.** Під час відкриття попапу, не частіше ніж раз на 4 години, розширення перевіряє один статичний файл (звичайний список шаблонів відповідей), розміщений за посиланням, яке контролює розробник (`raw.githubusercontent.com`), — щоб підтримувати бібліотеку шаблонів актуальною. Цей запит:
   - не передає жодних персональних даних, тексту запитів чи ідентифікуючої інформації — це просте завантаження файлу;
   - не виконує жодного віддаленого коду — читаються й відображаються лише текстові/JSON-дані;
   - мовчки завершується невдало за відсутності мережі; розширення продовжує працювати з останньою збереженою копією.

2. **Перевірка клієнта в бекофісі.** Лише коли користувач натискає «Обробити + відкрити» на сайті бекофісу компанії, розширення надсилає один запит лише на читання (GET) за кожен клік до API бекофісу компанії, щоб перевірити, чи існує клієнт із ID із запиту. ID клієнта міститься в адресі запиту, а для авторизації використовується вже наявна авторизована сесія користувача в бекофісі. Цей запит іде до власної системи бекофісу компанії (тієї самої, у якій працює користувач), а не до розробника. Із відповіді розширення лише визначає, чи існує клієнт; воно не зчитує, не показує й не зберігає дані клієнта. Віддалений код не виконується.

3. **Статистика використання.** Після врахованого натискання кнопки розширення надсилає описану вище подію (одну чи кілька за раз) до веб-застосунку Google Apps Script (`script.google.com`), який записує її в Google Таблицю. Скрипт і таблиця належать адміністратору розширення в межах Google Workspace організації; доступ до таблиці має лише адміністратор, а скрипт лише **записує** події — через нього таблицю прочитати неможливо. Скрипт перевіряє кожну подію й відкидає некоректні. Щоб записувати могли лише співробітники організації, запит містить cookies Google-сесії користувача для `script.google.com`; розширення їх не читає, не зберігає й не пересилає — їх додає сам браузер. Статистика використовується лише для розуміння того, наскільки активно застосовується інструмент (кількість натискань за кнопками, користувачами й днями), та для побудови зведених діаграм. Вона не передається в рекламні, аналітичні чи інші сторонні сервіси, а розширення не виконує віддаленого коду.

Жодних інших мережевих запитів не робиться. Розширення не використовує аналітику, пікселі відстеження, рекламні SDK чи будь-які сторонні сервіси телеметрії.

## Дозволи та навіщо вони потрібні

| Дозвіл | Призначення |
|---|---|
| `clipboardWrite` | Щоб копіювати підготовлений текст відповіді в буфер обміну за натисканням «Скопіювати». Буфер ніколи не зчитується. |
| `storage` | Щоб зберігати бібліотеку шаблонів, налаштування інтерфейсу, короткочасний кеш логіна та чергу невідправлених подій статистики локально на пристрої користувача (`chrome.storage.local`). |
| Доступ до хоста `raw.githubusercontent.com` | Щоб завантажувати описаний вище статичний список шаблонів. |
| Доступ до хостів `script.google.com` і `script.googleusercontent.com` | Щоб надсилати події статистики використання до веб-застосунку Google Apps Script адміністратора, описаного вище, і підтверджувати їх отримання. |
| `activeTab` | Щоб при відкритті попапу прочитати адресу активної вкладки, аби кнопка «Обробити + відкрити» показувалась лише на бекофісі компанії, і щоб описані нижче функції могли виконатися в цій вкладці. |
| `scripting` | Щоб в активній вкладці бекофісу виконувати фіксовані функції (вшиті в розширення, без віддаленого коду): перевірку клієнта після натискання «Обробити + відкрити» та зчитування логіна (лише `UserName`) для статистики використання. |

## Зберігання та видалення даних

Дані, які розширення зберігає локально, існують лише в локальному сховищі браузера користувача: нічого з перевірки клієнта (ID клієнта, токен, відповідь) не зберігається, кеш логіна живе не довше 7 діб, а невідправлені події статистики — не довше 14 днів. Видалення розширення чи очищення його даних через налаштування браузера миттєво видаляє все локальне.

Відправлені події статистики використання зберігаються в Google Таблиці адміністратора, доки той їх тримає для аналізу використання; адміністратор може видалити їх будь-коли. Інших серверних копій даних користувача розробник не має.

## Контакти

Питання щодо цієї політики, а також запити на видалення записів статистики конкретного користувача можна направляти адміністратору розширення у вашій організації.
