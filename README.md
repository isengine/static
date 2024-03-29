# Среда разработки

SCS - сокращенно от Static Content Server, сервер управления статическоим контентом.

Это сервер, построенный на базе Node.js и фреймворка Express, который может отдавать статический контент, такой как изображения, шрифты, файлы скриптов и стилей, **html** страницы.

Данный сервер поддерживает динамический рендеринг **html** страниц через шаблонизатор.

# Оглавление

- [Быстрый старт](#быстрый-старт)
- [О проекте](#о-проекте)
  - [Состав](#состав)
  - [Структура](#структура)
- [Роутинг](#роутинг)
- [Статический контент](#статический-контент)
- [Шаблонизатор](#шаблонизатор)
  - [Шаблоны](#шаблоны)
  - [Макеты](#макеты)
  - [Блоки](#блоки)
  - [Текущая страница](#текущая страница)
  - [Запросы](#запросы)
  - [Расширения](#расширения)
  - [Смена шаблонизатора](#смена-шаблонизатора)
- [Языки](#языки)
  - [Определение](#определение)
  - [Языковые библиотеки](#языковые-библиотеки)
  - [Настройка](#настройка)
  - [Вызов в шаблоне](#вызов-в-шаблоне)
- [Обработка ошибок](#обработка-ошибок)
- [Расширение функционала](#расширение-функционала)
  - [Установка зависимостей](#установка-зависимостей)
  - [Подключение библиотек](#подключение-библиотек)

# Быстрый старт

Быстрый запуск в 3 простых шага.

**1**. Выполните клонирование данного репозитория в текущий каталог backend части вашего проекта.

```shell script
git clone https://github.com/isengine/scs .
```

**2**. Выполните компиляцию

Рекомендуем использовать **yarn**.

```shell script
yarn
```

**3**. Соберите проект

В режиме **production**:

```shell script
yarn start
```

В режиме разработки **development**:

```shell script
yarn dev
```

Вывод проекта через node.js на 8080 порт: http://localhost:8080

> В случае необходимости изменить конфигурацию сервера, возникновения ошибок и прочих вопросов, смотрите полное руководство.

[^ к оглавлению](#оглавление)

# О проекте

## Состав

Backend использует следующие технологии:

- express
- ejs
- i18n
- nodejs
- yarn

[^ к оглавлению](#оглавление)

## Структура

```
scs
├── i18n
│   └── LANG
├── server
│   ├── compression.js
│   ├── ejs.js
│   ├── error.js
│   ├── i18next.js
│   └── routes.js
├── static
├── view
│   └── default
├── .env
└── server.js
```

[^ к оглавлению](#оглавление)

# Роутинг

Пути задаются в файле **routes.js**.

Чтобы задать новый путь, например, **contacts**, вам нужно добавить следующий код:

```
router.route('/contacts').get((req, res) => {
  res.render('contacts')
})
```

Этот код по запросу пути **/contacts** вызовет рендер сущности **contacts**, которая, согласно настройкам, перенаправит рендер на страницу с тем же именем, но расширением **html** (**contacts.html**) в папке текущего шаблона.

[^ к оглавлению](#оглавление)

# Статический контент

Вся статика должна лежать в папке **static**.

Оттуда идет перенаправление сервером по всем запросам.

Например запрос

```
http://your-domain.com/img/icons/icon.png
```

будет переведен в папку проекта

```
./static/img/icons/icon.png
```

[^ к оглавлению](#оглавление)

# Шаблонизатор

Данный сервер предназначен в первую очередь для выдачи статического контента.

Однако в состав включен шаблонизатор **Embedded JavaScript templates (ejs)**.

Он был выбран по причине того, что с его помощью можно отдавать чисто статические **html** страницы.

Однако вы также можете использовать его мощность.

Подробная документация:

https://ejs.co/

## Шаблоны

Шаблонизатор настроен таким образом, что он читает файлы из папки шаблона, заданной в переменной окружения **TEMPLATE**.

По-умолчанию, полный путь выглядит так:

```
./view/default/
```

По-умолчанию файлы шаблона имеют расширение **ejs**, однако мы настроили шаблонизатор на чтение **html** файлов.

Мы немного расширили функционал шаблонов при помощи middleware.

Для начала нужно его подключить в файле **routes.js**:

```
import render from '#server/ejs'
```

А затем использовать:

```
router.route('/page').get(render)
```

## Макеты

Шаблонизатор поддерживает макеты, хранящиеся в папке

```
./view/default/layouts/
```

По-умолчанию используется макет **default**.

Поменять макет можно в роутере, указав его четвертым аргументом в middleware-функции **render**:

```
router.route('/page').get((req, res, next) => {
  render(req, res, next, 'layout')
})
```

Такой вызов подключит файл

```
./view/default/layouts/layout.html
```

## Блоки

Также шаблонизатор настроен на чтение блоков из папки

```
./view/default/blocks/
```

Разрешено вызывать блоки с любой вложенностью.

Например, такая конструкция в коде html страницы:

```
<%- include(block('footer/main')) %>
```

Подключит файл

```
./view/default/blocks/footer/main.html
```

## Текущая страница

Страницы шаблона располагаются в папке

```
./view/default/inner/
```

Чтобы подключить вызов текущей страницы в макете, нужно написать такой код:

```
<%- include(page) %>
```

Не забывайте, что страницы должны быть заданы в файле **routes.js**

[^ к оглавлению](#оглавление)

## Запросы

Предположим, вам нужно обработать запросы при вызове страницы с параметрами. Например:

```
/page?first=1&second=2
```

Для этого в шаблонизаторе можно использовать переменную **query** - именно туда попадают все запросы.

Пример вывода данных из **query**:

```
<ul>
  <% for (const [key, value] of Object.entries(query)) { %>
    <li>
      <strong><%= key %></strong>: <%= value %>
    </li>
  <% } %>
</ul>
```

Другой пример вывода данных из **query**:

```
<ul>
  <% for (key in query) { %>
    <li>
      <strong><%= key %></strong>: <%= query[key] %>
    </li>
  <% } %>
</ul>
```

[^ к оглавлению](#оглавление)

## Расширения

Если вам не хватает текущего функционала, вы можете самостоятельно расширить шаблонизатор.

Для этого важно знать следующее:

- мы используем сервер **node.js** с пакетным менеджером **yarn**,
- сервер работает на фреймворке **express.js**,
- в качестве шаблонизатора используется библиотека **ejs** для **express.js** фреймворка.

[^ к оглавлению](#оглавление)

## Смена шаблонизатора

Мы не ограничиваем вас в использовании любого другого шаблонизатора. Но вам придется самостоятельно установить его и настроить запуск в коде файла **server.js**.

```
.set('view engine', 'html')
.engine('html', ejs.__express)
```

В этих двух строках шаблонизатор подключается к проекту.

Чтобы запустить другой шаблонизатор, например, популярный шаблонизатор **pug**, вам понадобится изменить этот код на следующий:

```
.set('view engine', 'pug')
```

[^ к оглавлению](#оглавление)

# Языки

Сервер поддерживает мультиязычность при помощи библиотеки **i18next**.

Все необходимые настройки заданы в файле **i18next.js**.

Вывод языков назначается свойству **lang** локального объекта сервера.

```
server.locals.lang = i18nextLang
```

[^ к оглавлению](#оглавление)

## Определение

Язык определяется тремя способами:

- из cookie,
- из query запроса,
- из заголовка запроса.

Исходя из этого, можно достаточно легко реализовать переключение языков.

[^ к оглавлению](#оглавление)

## Языковые библиотеки

Библиотеки хранятся в папке **i18n**, в подпапках с кодом языка.

Мы сразу предоставляем несколько библиотек для русского и английского языка:

- agency - информация о вашем агенстве или студии разработки,
- common - общие слова и фразы, которые наиболее часто употребляются на сайтах,
- counter - перевод числительных,
- country - перевод названий языков и стран,
- currency - денежные единицы,
- datetime - все, что касается даты и времени,
- default - базовая информация для использования на сайте,
- nav - разделы меню,
- social - социальные сети.

Как видите, библиотек более, чем достаточно для использования.

[^ к оглавлению](#оглавление)

## Настройка

Языковые настройки хранятся в файле переменных окружения **.env**:

- LANG - задает язык по-умолчанию,
- LANGS - массив используемых языков в виде массива json. Если будет пустой или окажется задан некорректно, то автоматически преобразуется в один используемых язык, заданный по-умолчанию,
- LANG_FILE - языковая библиотека, используемая по-умолчанию,
- LANG_FILES - массив языковых библиотек в виде массива json. Если будет пустой или окажется задан некорректно, то автоматически преобразуется в одну используемую библиотеку, заданную по-умолчанию.

Внося изменения, вы можете расширить библиотеки и языки.

Чтобы добавить языки, например, французский, немецкий и испанский, вам нужно изменить переменную **LANGS**:

```
# было
LANGS = ["ru", "en"]
# стало
LANGS = ["ru", "en", "fr", "de", "es"]
```

Например, вы хотите добавить еще одну библиотеку **custom**. Вам нужно переменная **LANG_FILES**:

```
# было
LANG_FILES = ["agency", "common", "counter", "country", "currency", "datetime", "default", "nav", "social"]
# стало
LANG_FILES = ["agency", "common", "counter", "country", "currency", "datetime", "default", "nav", "social", "custom"]
```

[^ к оглавлению](#оглавление)

## Вызов в шаблоне

Чтобы сделать вызов из языковой библиотеки, нужно использовать в шаблоне следующую конструкцию:

```
<%= lang('title') %>
```

Это вызовет значение **title** из библиотеки по-умолчанию.

Если же вы хотите сделать вызов из другой библиотеки, например **about** из библиотеки **nav**, вам нужно использовать:

```
<%= lang('nav:about') %>
```

Некоторые записи представляют собой массивы или объекты. Например, чтобы вызвать первый email-адрес из списка **email**, вам нужно сделать так:

```
<%= lang('email.0') %>
```

А вызвать класс соцсети whatsapp, заданный в библиотеке **social**, нужно таким образом:

```
<%= lang('social:whatsapp.class') %>
```

[^ к оглавлению](#оглавление)

# Обработка ошибок

При возникновении ошибки, например, если не найден запрошенный **url**, по-умолчанию вызывается страница **error.html** текущего шаблона.

[^ к оглавлению](#оглавление)

# Расширение функционала

## Установка зависимостей

Вы можете расширять возможности сервера при помощи библиотек. Такие библиотеки в контексте установки в проект называются зависимостями.

Установить библиотеку:

```shell script
yarn add LIBRARY
```

Удалить библиотеку:

```shell script
yarn remove LIBRARY
```

Также можно устанавливать и удалять несколько библиотек сразу, перечислив их через пробел:

```shell script
yarn add LIBRARY1 LIBRARY2 LIBRARY3
```

[^ к оглавлению](#оглавление)

## Подключение библиотек

Подключать библиотеки к проекту нужно в каждом **js** файле, где вы будете их использовать.

Подключить библиотеку к проекту:

```
import LIBRARY from 'LIBRARY'
```

Подключить определенные константы, объекты или функции:

```
import { OBJECT1, OBJECT2 } from 'LIBRARY'
```

Подключить все ресурсы библиотеки:

```
import * from 'LIBRARY'
```

[^ к оглавлению](#оглавление)
