# БЭМ проект от git init до Deploy

Может эта статья покажется капитанской, но часто звучит вопрос: «как БЭМ проект, состоящий из множества папок и файлов, выложить в production?».

Если кратко, то цели сборки находятся в папках `./*.bundles/<pages>/`, это технологии html, css, js, изображения и может что-то ещё, что входило в состав блоков как клиентская технология.  По сути, файлы (цели сборки) нужно перенести на веб-сервер, а пути (ссылки внутри технологий) разрулить при помощи `borschik` или/и при помощи `rewrite` в конфиге nginx.

<habracut text="Давайте разберём всё подробно" />

## Легенда

Нам дано условное техническое задание — сверстать две статические страницы:

  - индексная страница с единственной картинкой отцентрованной по вертикали и горизонтали
  - страница ошибки 404 без подключения css и js
  - необходимо подключить счетчик сбора статистики посещаемости страницы
  - проект выкладывается в OpenSource, и по этому не должен содержать приватной информации
  - ожидается высокая посещаемость страницы, поэтому необходимо оптимизировать выдачу
  - дизайнер прислал "макеты", картинку и фавиконку

В IT нет ничего более временного, чем постоянное. И к нашему техническому заданию добавятся пункты:

  - сборка страниц для мобильных устройств и планшетов, и выдача страницы в зависимости от типа устройства
  - показать на странице динамические данные, которые должны приезжать из REST ручки

Предполагаю, что вы знакомы c инструментами сборки и процессом создания БЭМ-проекта используя [bem-tools][bem-tools] или [enb][enb], [bemhtml][bemhtml] или [bh][bh], [project-stub][project-stub] или [generator-bem-stub][generator-bem-stub]. Для более глубокого понимания происходящего, все действия связанные с деплоем будем выполнять вручную. Способы автоматизации деплоя оставим за рамками этой статьи.

Договоримся о именовании путей. Далее относительные пути (начинаются с `./`) будут использоваться для указания директорий и файлов проекта, а абсолютные (начинаются с `/`) будут использоваться для директорий и файлов на сервере. Пути для сервера могут отличаться от вашего случая в зависимости от типа, версии и настройки операционной системы.

Для подготовки статьи использовал Ubuntu 14.04 LTS с предустановленными пакетами `nginx`, `git` и `nodejs` версии `0.10.32`.

<!-- Установка пакетов `nginx`, `git` и `nodejs-0.10.*`:

```bash
sudo apt-get install git nginx curl
curl -sL https://deb.nodesource.com/setup | sudo bash -
sudo apt-get install nodejs
sudo service nginx start
```
-->

## Новый проект

Искушенные БЭМ-ом разработчики могут перейти к следующему разделу.

Для создания нового проекта, назовём его `hypnotoad-www`, воспользуемся [generator-bem-stub][generator-bem-stub]. На основные вопросы ответим следующим образом:

> **?** Choose a toolkit to build the project: **ENB**
> **?** Specify additional libraries if needed: **bem-components**
> **?** Choose target platforms: **desktop**
> **?** Choose technologies to be used in the project: **BEMJSON**
> **?** Choose a template engine: **BEMHTML**
> **?** Do you want to build static HTML? **Yes**
> **?** Choose types of files to be minimized: **css**

Запускаем разработческий сервер, открываем страницу `http://localhost:8080/desktop.bundles/index/index.html` и видим приветствие `Hello, World!`.

### Страница 

У нас есть изображение `hypnotoad.gif`, как отцентровать его на экране мы знаем, блок с изображением мы так и назовём `hypnotoad`. Создадим директорию `./common.blocks/hypnotoad`, перенесем туда файл с изображением, добавим шаблон `./common.blocks/hypnotoad/hypnotoad.bemhtml`:

```js
block('hypnotoad').content()({
    block: 'image',
    alt: 'Hypnotoad',
    url: '../../common.blocks/hypnotoad/hypnotoad.gif'
})
```

файл с зависимостями `./common.blocks/hypnotoad/hypnotoad.deps.js`:

```js
[{
    shouldDeps : { block : 'image' },
    mustDeps : { block : 'i-bem' }
}]
```

> Блок `i-bem` всегда подключаем в зависимости через `mustDeps`, если в блоке используем технологию JS или BEMHTML, чтоб базовые шаблоны оказались выше во время сборки.

и стили `./common.blocks/hypnotoad/hypnotoad.css`:
```css
.hypnotoad {
    position:absolute;
    top:50%;
    left:50%;
    width:500px;
    height:350px;
    margin-left: -250px;
    margin-top: -175px;
}
```

В `./desktop.bundles/index/index.bemjson.js` меняем `Hello, World!` на декларацию блока `hypnotoad`:

```js
        { block: 'hypnotoad' }
```

Обновляем страницу и видим изображение там, где его ожидаем. В шаблоне нам пришлось указать относительный путь к файлу изображения `../../common.blocks/hypnotoad/hypnotoad.gif` для того, чтоб разработческий веб-сервер смог найти к нему путь относительно расположения индексной страницы `./desktop.bundles/index/index.html`.

<!-- TODO: раздел про настройку сборки, где рассказать как убрать этот костыль -->

Осталось поправить цвет фона. Для этого добавим в блок `page` модификатор с темой `hypnotoad`. Создадим в проекте директорию `./common.blocks/page/_theme` и добавим в нее файл со стилем `./common.blocks/page/_theme/page_theme_hypnotoad.css`:

```css
.page_theme_hypnotoad {
    background: #fcfefb;
}
```

К нашей странице тема подключается декларацией модификатора блока `page` в `./desktop.bundles/index/index.bemjson.js`:

```js
    mods: { theme: 'hypnotoad' },
```

И снова проверяем результат.

<!-- TODO: вынести отдельно, в рефакторинг
Сначала иконку и картинку ложим в бандл страницы
потом раздел «Наведём порядок»
-->
Для красивого завершения примера добавим фавиконку на проект. Для этого в блок `page` положим файл с иконкой в терминах БЭМ `./common.blocks/page/__favicon/page__favicon.ico`, и добавим переопределение шаблона страницы `./common.blocks/page/page.bemhtml`:

```js
block('page').def().match(!this.ctx.favicon)(function() {
    this.ctx.favicon='../../common.blocks/page/__favicon/page__favicon.ico';
    applyNext();
})
```

Это не канонический пример, правильно было сделать шаблон в виде элемента или модификатора, подключаемого в BEMJSON. Но захотелось подключить иконку сразу для всех страниц проекта, по этому мы переопределили шаблон блока `page`.

<!-- TODO: рассказать про `.def().match()` -->

### Страница 404

Эта страница нужна для демонстрации настроек веб-сервера. Как пример гибкости шаблонизатора BEMHTML, максимально приблизим HTML-код страницы к странице ошибки которую отдает по умолчанию nginx 1.7.4. Без особых подробностей приведу код блока и страницы:

`./common.blocks/error/error.bemhtml`:

```js
block('error')(
    tag()(false),
    content()(function() {
        var ctx = this.ctx;
        return [
            {
                tag: 'center',
                content: {
                    tag: 'h1',
                    content: ctx.title
                }
            }, {
                tag: 'hr'
            }, {
                tag: 'center',
                content: ctx.content
            }
        ];
    })
)
```

<!-- TODO: Почему через функцию ????
функция callback ктоторая позволяет отложить выполнение js-частей внутри блока, до момента когда инициализируется js в dev-режиме -->

`./common.blocks/page/_error/page_error_on.bemhtml`:

```js
block('page').mod('error', 'on')(
    attrs()({ bgcolor: 'white' })
)
```

`./desktop.bundles/404/404.bemjson.js`:

```js
({
    block: 'page',
    title: '404 Not Found',
    mods: { error: 'on' },
    content: [
       {
           block: 'error',
           title: '404 Not Found',
           content: 'bender/4.0.4'
       }
    ]
})
```

Проверяем работу страницы `http://localhost:8080/desktop.bundles/404/404.html`.

### Добавляем метрику

Следующая задача, подключить на страницы счетчик с метрикой. Предположим, мы имеем счетчик [Яндекс.Метрики][Яндекс.Метрика].

По условиям Метрики JS-код должен быть включен в HTML-код страницы и не должен подгружаться из вне. Добавить произвольный HTML код в БЭМ терминах на нашу страницу очень просто, для этого произвольный код передаем в метод `content()('<strong>Произвольный HTML код</strong>')`.

Создаем новый блок `./common.blocks/metrika/metrika.bemhtml`:

```js
block('metrika')(
    tag()(false),
    content()('<script type="text/javascript"> ... </script> ... ')
)
```

Указываем `tag()(false)` для того, чтоб код метрики не оборачивался в тэг `div`. JS-код метрики приводить не стал, так как для каждого случая он индивидуальный и включает персональный `id`.

> подключаем на страницу

## Выкладываем страницы в продакшен

И вот, наши страницы готовы, пора их предьявить миру.

### Настраиваем веб-сервер

<!-- На свежеустановленной системе nginx должен отдавать страницу по умолчанию. -->

Убедитесь, что в секции `http` конфига `/etc/nginx/nginx.conf` подключаются 

```
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
```

...

<!-- копируем проект как есть проект в `/var/www/` -->


Перезапускаем nginx командой `sudo service nginx reload` и на этот раз, для проверки, в браузере вбиваем имя хоста сервера. Получаем удовольствие, делемся радостью с окружающими.

Подводя промежуточный итог, скопируем nginx-конфиг `/etc/nginx/sites-available/10-hypnotoad.conf` в папку проекта, в `./nginx/sites-available/10-hypnotoad.conf`. Все последующие изменения рекомендую вносить в конфигурационные файлы в проекте, после переносить файлы в `/etc` сервера.

### Заморозка статики 

> задача

> почему фриз

Преимущество фриза в том, что в случае внесения каких-либо изменений в подгружаемые на страницу файлы будет изменено имя файла. Это исключит использование на новой версии страницы ресурсов из кэша от старой страницы.

Ещё один плюс: сохраненная страница у пользователя даже через несколько лет вытянет с сервера со статикой правильные версии css и js, и удивит своей работоспособностью.

Для некоторых сервисов важно, чтоб параллельно работали несколько версий сайта. Например, когда новая версия страницы выкатывается на какую-то часть аудитории. Фриз позволяет старой и новой страницам не конфликтовать из-за внесенных изменений в css или js.

Если ваш сервис запущен на десятках или сотнях серверов, то выкатка новой версии делается два этапа:

   - сначала выкладывается на сервера статика (js,css,медиа), добавляя новые файлы, не заменяя старые
   - потом обновляются страницы на серверах на новую версию

Процес обновления страниц на большом кол-ве серверов занимает какое-то время. Так как балансер распределят обращения за страницами и ресурсами между всеми серверами, такой подход позволяет исключить ошибку 404 при обращении за ресурсами к серверу, до которого ещё не дошла очередь обновления данных.


> Как происходит/работает заморозка?

> код

Борщик по умолчанию фризит лишь вот такой набор файлов: https://github.com/bem/borschik/blob/master/lib/freeze.js#L512-L513
Соответственно, чтобы фризить priv.js, нужно запускать сборку с переменной окружения
BORSCHIK_FREEZABLE_EXTS='jpg jpeg gif ico png swf svg ttf eot otf woff css js'

### Оптимизация

> Чтоб облегчить жизнь серверу, можно сжать gzip-ом статические файлы в директории с зафрижеными файлами.
> Для этого в папке `./_` запускаем команду `gzip -fk *.css *.js`. При обращении браузера к серверу за статическими
> файлами nginx будет сразу отдавать заархивированный файл. 

### ~~Добавляем новый уровень переопределения с конфигурацией~~

## Конфигурация сборки (development, testing, production)

<!-- TODO -->

`./configs/[?common|development|production|testing]`
  `app.json`
  `borschik`

`ln -sf ../configs/current ./node_modules/configs`
`ln -sf $NODE_ENV ./configs/current`

```js
{
    "NODE_ENV": "production",
    "metrika": 1234567890
}
```

`YENV=production node_modules/.bin/enb make`

`./*.bundles/*/*.bemjson.js`

```js
        {
            block: 'metrika',
            id: '{METRIKA_ID}'
        }
```

`common.blocks/metrika/metrika.bemhtml`

```js
block('metrika')(
    match()(function () { return !this.ctx.id; }).def()(false),

    content()(function () {
        var counterId = this.ctx.id;

        return '<script type="text/javascript">(function (d, w, c) { (w[c] = w[c] || []).push(function () ' +
            '{ try { w["yaCounter' + counterId + '"] = new Ya.Metrika({id:' + counterId + ', enableAll: true,' +
            ' webvisor:true}); } catch(e) { } }); var n = d.getElementsByTagName("script")[0], s = ' +
            'd.createElement("script"), f = function () { n.parentNode.insertBefore(s, n); }; s.type = ' +
            '"text/javascript"; s.async = true; s.src = (d.location.protocol == "https:" ? "https:" : "http:") +' +
            ' "//mc.yandex.ru/metrika/watch.js"; if (w.opera == "[object Opera]") ' +
            '{ d.addEventListener("DOMContentLoaded", f); } else { f(); } })(document, window, ' +
            '"yandex_metrika_callbacks");</script><noscript><div><img src="//mc.yandex.ru/watch/' + counterId + '" ' +
            'style="position:absolute; left:-9999px;" alt="" /></div></noscript>';
    })
);
```

## Библиотеки

<!-- выносим в отдельную библиотеку блоки с метрикой, борщиком и error, так как их мы хотим реиспользовать в других проектах,
а также это даст возможность развивать и тестировать общие блоки [150505] -->

Осталось немного времени и мы позволим себе порефакторить конфигурацию сборки. В библиотеки принято выносить блоки или их части, общие для нескольких проектов. Но мы поступим наоборот. Мы вынесем из нашего проекта уровень с конфигурацией блоков в отдельную библиотеку, которая будет храниться в условно-приватном репозитории. Это нам позволит не вносить данные конфигурации каждый раз, когда мы склонировали репозиторий на новое место.

> Что такое библиотека? Это папочка с другими папочками, которые мы считаем блоками. Или папочка с папочками, которые мы считаем уровнями переопределения, а в них папочки, которые считаем блоками. Это весь секрет. Как подключаются библиотеки? Для этого нужно сделать два действия.
> Первое. Папка с блоками магическим образом должна появиться в вашем проекте во время сборки. Рекомендуемый способ, это установка библиотеке через `bower`. Для этого в `./bower.json` в `dependencies` достаточно добавить ссылку на вашу библиотеку в формате `"libname": "git url"` и запустить команду `bower i`. Библиотека клонируется в каталог `./libs`. Более подробно можно прочитать в документации [bower][bower].
> Второе. Подключить к сборке папку с блоками как уровень переопределения. Например, для инструмента сборки `enb`, как у нас, нужно в `./.enb/make.js` в функцию `getDesktops(config)` добавить строчку TODO (ссылаемся на доку и белем описание из неё).

<!-- выносим уровень с конфигом в условно-закрытый репозиторий и подключаем его сабмодулем -->

<!-- для общих библиотек рекомендуется использовать подключение через [bower][bower] -->

## Собираем отдельные страницы для мобильных устройств

Но тут пришел менеджер:

  — Всё отлично, то что нужно! И БЭМ — это очень интересно. Но на телефоне изображение открывается в стороне, нужно починить. Я прочитал в какой-то статье про БЭМ, что несложно сгенерировать страницы для планшетов и телефонов. А мы как раз хотели сделать другой дизайн для телефонов и планшетов.

Действительно, библиотеки `bem-*` имеют уровни `touch-pad` и `touch-phone` для сборки специальных версий страниц для планшетов и телефонов. Хотя разработчики библиотек поддерживают нормальную работу тач устройств на уровне `desktop`.

Чтоб включить сборку страниц для мобильных устройств на нашем проекте можно снова запустить [generator-bem-stub][generator-bem-stub] и повторить конфигурацию проекта, как это делали при [создании нового проекта][#Новый проект] с единственным отличием:

> **?** Choose target platforms: **desktop**, **touch-pad**, **touch-phone**


> мы считаем, что у сайтов должна быть специальная тач версия (сделать которую помогают наши touch уровни), а десктопная версия должна просто работать в тачах (т.е. всякие события и т.п., без адаптации размеров)

### Настройка веб-сервера для выдачи страницы в зависимости от типа устройства

Появился новый вопрос: как загрузить в браузер необходимую страницу?

Поиск в интернет тут-же дал ответ, что в конфиге nginx для определения устройства можно воспользоваться информацией из заголовка `User-Agent`. Данные из заголовка запроса соответствуют переменной `$http_user_agent`.

Ещё немного поиска и мы получаем готовое решение. Часть конфига, которая включается только в секцию `http {}` конфигурации nginx. Это ограничение директивы [map][ngx_http_map].

```
# Add MAP for Default As desktop
# First match regex = touch-pad
# Second match regex = touch-phone
map $http_user_agent $ua_device {
    default desktop;
    ~*iPad|iPad.*Mobile|^.*Android.*Nexus(((?:(?!Mobile))|(?:(\s(7|10).+))).)*$|SAMSUNG.*Tablet|Galaxy.*Tab|SC-01C|GT-P1000|GT-P1010|GT-P6210|GT-P6800|GT-P6810|GT-P7100|GT-P7300|GT-P7310|GT-P7500|GT-P7510|SCH-I800|SCH-I815|SCH-I905|SGH-I957|SGH-I987|SGH-T849|SGH-T859|SGH-T869|SPH-P100|GT-P3100|GT-P3110|GT-P5100|GT-P5110|GT-P6200|GT-P7320|GT-P7511|GT-N8000|GT-P8510|SGH-I497|SPH-P500|SGH-T779|SCH-I705|SCH-I915|GT-N8013|GT-P3113|GT-P5113|GT-P8110|GT-N8010|GT-N8005|GT-N8020|GT-P1013|GT-P6201|GT-P6810|GT-P7501|Kindle|Silk.*Accelerated|xoom|sholest|MZ615|MZ605|MZ505|MZ601|MZ602|MZ603|MZ604|MZ606|MZ607|MZ608|MZ609|MZ615|MZ616|MZ617|Android.*\b(A100|A101|A110|A200|A210|A211|A500|A501|A510|A511|A700|A701|W500|W500P|W501|W501P|W510|W511|W700|G100|G100W|B1-A71)\b|Android.*(AT100|AT105|AT200|AT205|AT270|AT275|AT300|AT305|AT1S5|AT500|AT570|AT700|AT830)|Sony\ Tablet|Sony\ Tablet\ S|SGPT12|SGPT121|SGPT122|SGPT123|SGPT111|SGPT112|SGPT113|SGPT211|SGPT213|EBRD1101|EBRD1102|EBRD1201|MID1042|MID1045|MID1125|MID1126|MID7012|MID7014|MID7034|MID7035|MID7036|MID7042|MID7048|MID7127|MID8042|MID8048|MID8127|MID9042|MID9740|MID9742|MID7022|MID7010|MediaPad|IDEOS\ S7|S7-201c|S7-202u|S7-101|S7-103|S7-104|S7-105|S7-106|S7-201|S7-Slim|IQ310|Fly\ Vision|Android.*(K8GT|U9GT|U10GT|U16GT|U17GT|U18GT|U19GT|U20GT|U23GT|U30GT)|CUBE\ U8GT|Android.*(\bMID\b|MID-560|MTV-T1200|MTV-PND531|MTV-P1101|MTV-PND530)|\bL-06C|LG-V900|LG-V909\|Android.*(TAB210|TAB211|TAB224|TAB250|TAB260|TAB264|TAB310|TAB360|TAB364|TAB410|TAB411|TAB420|TAB424|TAB450|TAB460|TAB461|TAB464|TAB465|TAB467|TAB468)|Android.*\bOYO\b|LIFE.*(P9212|P9514|P9516|S9512)|LIFETAB|AN10G2|AN7bG3|AN7fG3|AN8G3|AN8cG3|AN7G3|AN9G3|AN7dG3|AN7dG3ST|AN7dG3ChildPad|AN10bG3|AN10bG3DT|Android.*ARCHOS|101G9|80G9|NOVO7|Novo7Aurora|Novo7Basic|NOVO7PALADIN|Transformer|TF101\|PlayBook|RIM\ Tablet|HTC\ Flyer|HTC\ Jetstream|HTC-P715a|HTC\ EVO\ View\ 4G|PG41200|Android.*Nook|NookColor|nook\ browser|BNTV250A|LogicPD\ Zoom2|Android.*(RK2818|RK2808A|RK2918|RK3066)|RK2738|RK2808A|bq.*(Elcano|Curie|Edison|Maxwell|Kepler|Pascal|Tesla|Hypatia|Platon|Newton|Livingstone|Cervantes|Avant)|Android.*\b97D\b|Tablet(?!.*PC)|ViewPad7|MID7015|BNTV250A|LogicPD\ Zoom2|\bA7EB\b|CatNova8|A1_07|CT704|CT1002|\bM721\b|hp-tablet|Playstation|TB07STA|TB10STA|TB07FTA|TB10FTA|z1000|Z99\ 2G|z99|z930|z999|z990|z909|Z919|z900|TOUCHPAD.*[78910]|Broncho.*(N701|N708|N802|a710)|Pantech.*P4100|\bN-06D|\bN-08D|T-Hub2|Android.*\bNabi|Playstation.*(Portable|Vita) touch-pad;
    ~*SM-N|Tapatalk|PDA|PPC|SAGEM|mmp|pocket|psp|symbian|Smartphone|smartfon|treo|up.browser|up.link|vodafone|wap|nokia|Series40|Series60|S60|SonyEricsson|N900|MAUI.*WAP.*Browser|LG-P500|iPhone.*Mobile|iPod|iTunes|BlackBerry|\bBB10\b|rim[0-9]+|HTC|HTC.*(Sensation|Evo|Vision|Explorer|6800|8100|8900|A7272|S510e|C110e|Legend|Desire|T8282)|APX515CKT|Qtek9090|APA9292KT|HD_mini|Sensation.*Z710e|PG86100|Z715e|Desire.*(A8181|HD)|ADR6200|ADR6425|001HT|Inspire\ 4G|Android.*\bEVO\b|Nexus\ One|Nexus\ S|Galaxy.*Nexus|Android.*Nexus.*Mobile|Dell.*Streak|Dell.*Aero|Dell.*Venue|DELL.*Venue\ Pro|Dell\ Flash|Dell\ Smoke|Dell\ Mini\ 3iX|XCD28|XCD35|\b001DL\b|\b101DL\b|\bGS01\b|sony|SonyEricsson|SonyEricssonLT15iv|LT18i|E10i|Asus.*Galaxy|PalmSource|Palm|Vertu|Vertu.*Ltd|Vertu.*Ascent|Vertu.*Ayxta|Vertu.*Constellation(F|Quest)?|Vertu.*Monika|Vertu.*Signature|IQ230|IQ444|IQ450|IQ440|IQ442|IQ441|IQ245|IQ256|IQ236|IQ255|IQ235|IQ245|IQ275|IQ240|IQ285|IQ280|IQ270|IQ260|IQ250|\b(SP-80|XT-930|SX-340|XT-930|SX-310|SP-360|SP60|SPT-800|SP-120|SPT-800|SP-140|SPX-5|SPX-8|SP-100|SPX-8|SPX-12)\b|PANTECH|IM-A|VEGA\ PTL21|PT003|P8010|ADR910L|P6030|P6020|P9070|P4100|P9060|P5000|CDM8992|TXT8045|ADR8995|IS11PT|P2030|P6010|P8000|PT002|IS06|CDM8999|P9050|PT001|TXT8040|P2020|P9020|P2000|P7040|P7000|C790|Samsung|BGT-S5230|GT-B2100|GT-B2700|GT-B2710|GT-B3210|GT-B3310|GT-B3410|GT-B3730|GT-B3740|GT-B5510|GT-B5512|GT-B5722|GT-B6520|GT-B7300|GT-B7320|GT-B7330|GT-B7350|GT-B7510|GT-B7722|GT-B7800|GT-C3|GT-C5010|GT-C5212|GT-C6620|GT-C6625|GT-C6712|GT-E|GT-I|GT-M3510|GT-M5650|GT-M7500|GT-M7600|GT-M7603|GT-M8800|GT-M8910|GT-N7000|GT-P6810|GT-P7100|GT-S|GT-S8530|GT-S8600|SCH-A310|SCH-A530|SCH-A570|SCH-A610|SCH-A630|SCH-A650|SCH-A790|SCH-A795|SCH-A850|SCH-A870|SCH-A890|SCH-A930|SCH-A950|SCH-A970|SCH-A990|SCH-I100|SCH-I110|SCH-I400|SCH-I405|SCH-I500|SCH-I510|SCH-I515|SCH-I600|SCH-I730|SCH-I760|SCH-I770|SCH-I830|SCH-I910|SCH-I920|SCH-LC11|SCH-N150|SCH-N300|SCH-R100|SCH-R300|SCH-R351|SCH-R4|SCH-T300|SCH-U|SCS-26UC|SGH-A|SGH-B|SGH-C|SGH-D307|SGH-D|SGH-D807|SGH-D980|SGH-E105|SGH-E200|SGH-E315|SGH-E316|SGH-E317|SGH-E335|SGH-E590|SGH-E635|SGH-E715|SGH-E890|SGH-F300|SGH-F480|SGH-I|SGH-J150|SGH-J200|SGH-L170|SGH-L700|SGH-M110|SGH-M150|SGH-M200|SGH-N|SGH-N7|SGH-P|SGH-Q105|SGH-R210|SGH-R220|SGH-R225|SGH-S105|SGH-S307|SGH-T|SGH-U|SGH-V|SGH-X|SGH-Z130|SGH-Z150|SGH-Z170|SGH-ZX10|SGH-ZX20|SHW-M110|SPH-A|SPH-D600|SPH-D700|SPH-D710|SPH-D720|SPH-I300|SPH-I325|SPH-I330|SPH-I350|SPH-I500|SPH-I600|SPH-I700|SPH-L700|SPH-M|SPH-N100|SPH-N200|SPH-N240|SPH-N300|SPH-N400|SPH-Z400|SWC-E100|SCH-i909|GT-N7100|GT-N8010|Motorola|\bDroid\b.*Build|DROIDX|Android.*Xoom|HRI39|MOT-|A1260|A1680|A555|A853|A855|A953|A955|A956|Motorola.*ELECTRIFY|Motorola.*i1|i867|i940|MB200|MB300|MB501|MB502|MB508|MB511|MB520|MB525|MB526|MB611|MB612|MB632|MB810|MB855|MB860|MB861|MB865|MB870|ME501|ME502|ME511|ME525|ME600|ME632|ME722|ME811|ME860|ME863|ME865|MT620|MT710|MT716|MT720|MT810|MT870|MT917|Motorola.*TITANIUM|WX435|WX445|XT3|XT502|XT530|XT531|XT532|XT535|XT6|XT7|XT8|XT9 touch-phone;
}
```

Подключаем это решение к нашей конфигурации. В моем случае сохраняю в файл `/etc/nginx/conf.d/ua_device.conf`.

Теперь у нас есть переменная `$ua_device` которая содержит одно из следующих значений в зависимости от типа устройства: `desktop`, `touch-pad`, `touch-phone`. Но структура хранения страниц на сервере плоским списком нам не очень подходит. Чтоб не связываться с переименованием страниц переносим файлы страниц из папок проекта `./*.bundles/` на сервер по папкам с соответствующими названием устройства в имени:

```
/var/www/hypnotoad/_:
    ...
/var/www/hypnotoad/desktop.pages:
    404.html
    index.html
/var/www/hypnotoad/touch-pad.pages:
    404.html
    index.html
/var/www/hypnotoad/touch-phone.pages:
    404.html
    index.html
```

А в конфиг `/etc/nginx/sites-available/10-hypnotoad.conf` добавляем строчку с `rewrite` с шаблоном удовлетворяющим поиску html-страниц указанием нового пути к странице и завершением обработки текущего набора директив:

```
    location  /404.html {
        internal;
    }

    rewrite ^/([\w-]+.html)?(\?.*)?$ /$ua_device.pages/$1$2 last;

    location /_/ {
        gzip_static on;
    }
```

Перезапускаем сервер nginx и проверяем результат, откройте сайт с различных устройств.

## Настоящий сервис

К нам снова пришел менеджер проектов и сообщил, что маркетологи предлагают добавить на страницу всплывающие надписи, которые будут оказывать больше гипнотического влияния на посетителей страницы. Текст для надписей храниться в условной базе данных компании.

Мы вправе выбрать для сервиса любой язык программирования и фреймворк, но полагаю, что node.js и express будет понятнее для демонстрации фронтенд-разработчикам.

Статические страницы и код сервиса конечно можно хранить в одном репозитории, но мне хочется всё разложить по полочкам. Создадим репозиторий отдельный `hypnotoad-api`. 



Вместо условной базы данных будем использовать генератор случайных чисел и массив со строчками.



### Разработческий и продакшеновый запуск

Наше приложение умеет работать в качестве веб-сервера и отдавать http

Но на сервере IP-порт 80 занят веб-сервером, таким как nginx, по этому мы должны настроить веб-сервер таким образом, чтоб он передавал приходящие запросы в наше приложение. Использование Unix-сокета, это один из самых распространенных способов общения веб-сервера и приложения.


<!-- TODO: пример с использованием ластер, нужно понять профит, кроме использования процессоров -->

## Summary

Мы узнали:

  - TODO: для чего все эти файлы после сборки проекта, и какие из них являются конечным результатом
  - где и как настраивать сборку проекта под текущие задачи
  - про то, как оптимизировать результаты сборки для публикации в production
  - nginx — это не сложновыговариваемое слово, а полезный, гибкий и простой web-сервер
  - ...



PS: Описанные в статье решения не являются единственно верными, просто автор их посчитал более наглядными для демонстрации конфигурации проекта.

### Материалы

Цель статьи показать спектр возможностей БЭМ, цель рассказать начинающим про неочивидные на первы взгляд вещи

[bem-tools]: https://bem.info/tools/bem/bem-tools/
[enb]: http://enb-make.info
[bemhtml]: https://bem.info/technology/bemhtml/v2/intro/
[bh]: https://ru.bem.info/technology/bh/
[project-stub]: https://bem.info/tutorials/project-stub/
[generator-bem-stub]: https://bem.info/tools/bem/bem-stub/
[ngx_http_map]: http://nginx.org/ru/docs/http/ngx_http_map_module.html
[ngx_http_rewrite]: http://nginx.org/ru/docs/http/ngx_http_rewrite_module.html
[Яндекс.Метрика]: https://metrika.yandex.ru
[bower]: http://bower.io

---

http://nosensus.com/mobilnaya-verstka/

Рассказываем по шаблону для каждой части:

  - проблема или задача
  - решение
  - преимущества, для чего применяется
  - как устроено, что происходит
  - пример реализации
  - пояснения, почему так
