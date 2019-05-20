# Пользовательские турбо-компоненты

Репозиторий с кодом пользовательских компонентов, которые могут быть использованы на Турбо-страницах.

## Содержание
* [Начало разработки](#начало-разработки)
    * [Клонирование репозитория](#клонирование-репозитория)
* [Создание своего компонента](#создание-своего-компонента)
    * [Ограничения](#ограничения)
    * [Написание кода](#написание-кода)
    * [Отладка кода](#отладка-кода)
* [Процесс разработки](#процесс-разработки)
* [Релизный процесс (когда я окажусь в `production`?)](#релизный-процесс-когда-я-окажусь-в-production)
* [Работа с линтерами и CI](#работа-с-линтерами-и-ci)
* [Вопросы и проблемы](#вопросы-и-проблемы)


## Начало разработки

### Клонирование репозитория

Для начала необходимо выполнить команды:

```sh
git clone git@github.com:turboext/components.git components
cd components
npm i
npm start
```

Проверяем работу репозитория: переходим на [`https://localhost:8443/render/ext-exchange-rates/default`](https://localhost:8443/render/ext-exchange-rates/default).

## Создание своего компонента
После того, как мы настроили среду, можно приступить к написанию своего компонента.

### Структура директории с блоками

Для его создания необходимо соблюдать соглашение по наименованию:

```
components/
└── ExtFancyButton
    ├── ExtFancyButton.examples
    │   └── default.xml
    ├── ExtFancyButton.scss
    └── ExtFancyButton.tsx
```

То, есть все кастомные блоки лежат в корневой директории `components`.

Отдельный блок имеет свою директорию, начинается с `Ext` и используют `pascal case` для наименования.

> Например, в данном случае:`ExtFancyButton` — директория с блоком.

Компонент называется так же, как и директория.

> Например, в данном случае:`ExtFancyButton.tsx` — файл входа в блок (будет вызван с пришедшими данными на сервере и гидрирован на клиенте).

`<Component-Name>.examples` - директория с примерами данных для отображения блока. Используется для отладки.

> Например, в данном случае:`ExtFancyButton.examples` — директория с примерами к блоку.

Примеры именуются свободно, и имеют расширение `.xml`

> Например, в данном случае: `default.xml` — единственный пример для блока.

### Написание своего примера

Все примеры описываются в xml формате, как содержимое `rss`-файла. Пример:

```html
<ExtExchangeRates data-rates='["USD","EUR","CHF","GBP","JPY","AUD","CZK"]'></ExtExchangeRates>
```

Данные можно передавать с помощью `data-`аттрибутов, в компонент они придут как пропсы с такими же
названиями, но сериализованные для удобства:

- Если в качестве значения передана строка, которая сериализуется в число (например, `"123"` или `"123.123"`),
то в качестве аргумента в компонент будет передано число
- Если в качестве значения передана строка, которая сериализуется в объект (не примитив), то в качестве
аргумента будет передан объект (например, `'{"a": "b"}'` или `'["USD","EUR"]'`).
- В иных случаях в качестве значения будет передана строка.

### Ограничения

Если вы хотите делать ajax запросы, то их возможно совершать только к API, находящемуся на одном домене с вашим сайтом.
Например: для сайта [rozhdestvenskiy.ru](https://rozhdestvenskiy.ru) можно совершать ajax запросы только к домену rozhdestvenskiy.ru
Запросы, например, к [aws.com](https://aws.com) будут заблокированы политиками [CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
При этом надо использовать [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) и нельзя передавать Cookie yandex.ru домена.

### Написание кода

Все блоки пишутся строго на `typescript`. Точка входя для блока должна экспортировать class или простую (не стрелочную) функцию, названные так же, как и компонент.

**Валидные записи**:

```javascript
export class ExtExchangeRates extends React.PureComponent<IProps> {}
```

```javascript
export function ExtExchangeRates () {}
```

**Невалидные записи**:

```javascript
export const ExtExchangeRates = () => {}
```

```javascript
export default function ExtExchangeRates () {}
```

При импорте он будет вызван как реакт-компонент, а в качестве пропсов ему будут переданы данные из rss.
Внутри компонента можно импортировать стили, которые приедут вместе с ним.

### Отладка кода

Код компонента доступен по ссылке `/render/<component-name>/<example-name>`. Список всех примеров можно
посмотреть на заглавной странице.

## Процесс разработки

1. Для разработки вы создаете [форк](https://help.github.com/en/articles/fork-a-repo) проекта.
1. Вы создаете `pull-request` в репозиторий
2. Проходят все линтеры. Если линтеры не проходят, пулреквест не рассматривается. Если есть проблемы, связанные с линтингом файлов (например, конфликтующие линтеры или ошибки во время линтинга кода), необходимо создать issue.
3. После того, как прошли все линтеры, пулреквест получает два апрува от команды.
4. Пулреквест вливается в репозиторий.

## Релизный процесс (когда я окажусь в `production`?):

1. Отведение новой релизной ветки происходит автоматически в пятницу вечером.
2. За выходные все кастомные блоки тестируются.
3. Если все хорошо, то выпускаем новую версию пакета компонентов. Если есть проблемы с отдельными блоками, то эти блоки не будут включены в релиз.
4. Релиз с компонентами попадают на страницы в понедельник.
5. Изменения будут появляться в [CHANGELOG.md](CHANGELOG.md)

## Работа с линтерами и CI

Мы используем много самых разнообразных линтеров для контроля качества кода.

- Для контроля стилей (`.scss`) мы используем stylelint.
- Для контроля наименования мы используем [линтер файловой структуры](tools/lint-filesystem.ts)
- Для контроля типо мы используем `tsc`.
- Для контроля качества кода мы используем [eslint](https://eslint.org/) с рядом плагинов.

Часть из них запускается перед коммитом, часть — перед пушем в удаленный репозиторий.

Если линтер падает на прекомите, можно сначала попробовать поправить изменения автоматически с помощью команды:
`npm run lint:fix`.

## Вопросы и проблемы

В случае каких-либо проблем или вопросов вы можете создать `issue`.
