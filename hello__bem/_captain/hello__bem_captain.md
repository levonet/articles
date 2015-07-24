# Капитанские советы БЭМ

> TODO: заменить все версии в url на current
> TODO: отсортировать по технологиям, возможно с заголовками

Не программист и не фронтендер, но захотелось попробовать написать свой проект,
используя БЭМ. В процессе собрался набор очевидных рекомендаций, которые поначалу
казались вовсе не очевидными.
Надеюсь, они помогут новичкам быстро сориентироваться что к чему.

    «Нужно писать код правильно, не писать неправильно.»
    © veged@

`<cut>`

1. Если код элемента или модификатора описывается несколькими строками,
то можно всё держать в одном файле, например в блоке или элементе для модификатора элемента.
Это делает код более читаемым. При разрастании кода выносим в
[отдельные файлы](https://ru.bem.info/method/filesystem/#Опциональные-элементы-и-модификаторы-выносятся-в-отдельные-файлы),
соответствующей [БЭМ-сущности](https://ru.bem.info/method/definitions/#БЭМ-сущность).

1. Про шаблонизаторы.
Удобно сначала писать BEMJSON, а потом шаблоны.
Потому, что шаблонизатор BEMHTML предполагает поведение по умолчанию. Если написать BEMJSON

    ```JS
    {
        block: 'block1',
        content: [
            {
                elem: 'elem1',
                content: 'Bla bla bla'
            }
        ]
    }
    ```

    и не написать никаких шаблонов, то в результате сгенерируем такой HTML:

    ```HTML
    <div class="block1"><div class="block1__elem1">Bla bla bla</div></div>
    ```

1. Про `JS` и `BEMHTML`.
Везде, где необходимо написать JS с использованием методов из
модулей `i-bem` и `i-bem__dom`, в шаблоне указываем `js()(true)`, что будет означать,
что для блока сгенерируется необходимая разметка, включающая параметры блока и микс
с блоком `i-bem`. В HTML-теге появится атрибут `data-bem`, а из JS возможность
использовать BEM и BEMDOM-методы библиотеки [bem-core](https://ru.bem.info/libs/bem-core/v2.6.0/desktop/i-bem/jsdoc/)
при [соответствующей декларации](https://ru.bem.info/libs/bem-core/v2.6.0/desktop/i-bem/#Синтаксис-декларации).

    ```JS
    block('block1').js()(true);
    ```

    В результате получим HTML:

    ```JS
    <div class="block1 i-bem" data-bem="{"block1":{}}"></div>
    ```

1. Про `JS`, `BEMHTML` и `CSS`.
При статической сборке из BEMJSON, если в нём описать используемые
элементы и модификаторы блоков, то зависимости будут сгенерированы из BEMJSON,
всё будет работать без `deps.js` файлов в блоках.
Но если блок разрабатывается без гарантии, что все необходимые элементы и модификаторы будут
объявлены в BEMJSON, например когда сущности объявляются в шаблонах или BEMJSON генерируются
в рантайме, тогда эти сущности нужно явно указывать в файлах `deps.js`.
Поначалу часто забываешь, и вспоминаешь об этом, когда что-то не доехало в браузер.

1. Про `BEMHTML`. Везде, где используем BEMHTML, нужно добавить зависимость от `i-bem`
в `deps.js` файл БЭМ-сущности. Это подключит базовые шаблоны.

    ```JS
    ({
        mustDeps: [ 'i-bem' ]
    }, {
        // Зависимости блока
    })
    ```

1. Про `CSS`.
Если возникла необходимость модифицировать какой-то один блок
(или другую БЭМ-сущность), то лучше это сделать
[булевым модификатором](https://ru.bem.info/method/naming-convention/#Имя-модификатора).
Если такая, однотипная модификация, требуется для нескольких различных блоков,
то лучше её вынести в блок и [подключать миксом](https://ru.bem.info/technology/bemjson/v2/bemjson/#Представление-БЭМ-сущностей).

  > __все равно не понимаю, почему именно булевым и лучше чем что?__
  > __расскажи подробнее, что ты хочешь донести, попробуем вместе сформулировать.__

1. Про `JS`, `BEMHTML` и `CSS`.
Инициализацию и функциональность элемента или модификатора
выносим в отдельный файл (или объявление) элемента или модификатора. Это делает блок
независимым от кода данной сущности. Немного про [декларацию в JS](https://ru.bem.info/libs/bem-core/v2.6.0/desktop/i-bem/docs/#Декларация).

    ```JS
    modules.define('block1', ['i-bem__dom'], function(provide, BEMDOM) {
    provide(BEMDOM.decl(this.name, {}, {}));
    });

    modules.define('block1', ['i-bem__dom'], function(provide, BEMDOM, Block1) {
    provide(Block1.decl({block: 'block1', modName: 'mod1', modVal: 'value'}, {
        onSetMod: {
            'js': {
                'inited': function() {
                        this.__base.apply(this, arguments);
                        // Код инициализации модификатора блока
                    });
                }
            }
        }
    }))
    
    });
    ```

1. Про `JS`.
Иногда нужно вызвать какой-то BEMDOM-метод или принудительно инициализировать один
из нескольких вложенных блоков внутрь родительского.
Например на один из нескольких вложенных блоков `link` нужно навесить `popup` используя метод `setAnchor()`.
Самый простой способ — обернуть или смиксовать нужный блок элементом и найти его BEMDOM методом
[findBlockInside([elem], block)](https://ru.bem.info/libs/bem-core/v2.6.0/desktop/i-bem/jsdoc/#findBlockInside-1).

    ```JS
    var link = this.findBlockInside('elem1', 'link'),
        popup = this.findBlockInside('elem1', 'popup')
                        .setAnchor(link);

        link.on('click', function() {
            popup.setMod('visible', true);
        });
    ```

1. Про `BEMHTML`.
Чтобы в блоке (или другой БЭМ-сущности) расположить последовательно несколько
других элементов или блоков, нужно их передать в `content()` в формате BEMJSON:

    ```JS
    block('block1')(
        content()([
            { elem: 'elem1', content: 'BOO' },
            { block: 'block2', content: 'FOO' }
        ])
    );
    ```

    Получим HTML:

    ```HTML
    <div class="block1">
      <div class="block1__elem1">BOO</div>
      <div class="block2">FOO</div>
    </div>
    ```

    Это же свойство можно использовать для создания более сложной HTML-структуры блока:

    ```JS
        content()([
            {
                elem: 'elem1',
                content: [
                    { elem: 'elem2', content: 'Bla' }
                    { block: 'block3', content: 'Bla Bla' },
                    'BOO'
                ]
            },
            { block: 'block2', content: 'FOO' }
        ])
    ```

1. Про `BEMHTML`.
Чтобы вложить в элемент другой элемент (или другую БЭМ-сущность), можно воспользоваться
предыдущим советом. Для проброса содержимого элемента нужно использовать
[applyNext()](https://ru.bem.info/technology/bemhtml/v2/templating/#applynext).

    Для дальнейших примеров используем BEMJSON:

    ```JS
    {
        block: 'block1',
        elem: 'elem1',
        content: 'BOO'
    }
    ```

    Важно! Так как в теле шаблона нельзя выполнить JS-метод `applyNext()`, в `content()` передадим
    функцию, которая вернет BEMJSON.

    ```JS
    block('block1').elem('elem1')(
        content()(function(){
            return {
                elem: 'elem2',
                content: applyNext()
            };
        })
    );
    ```

    Получим HTML:

    ```HTML
    <div> class="block1__elem1">
      <div class="block1__elem2">BOO</div>
    </div>
    ```


1. Про `BEMHTML`.
Можно добавлять элемент (или другую БЭМ-сущность) к существующему контенту в начало:

    ```JS
    block('block1').elem('elem1')(
        content()(function(){
            return [
                { elem: 'elem2', content: 'FOO' },
                applyNext()
            ];
        })
    );
    ```

    Получим HTML:

    ```HTML
    <div class="block1__elem1">
      <div class="block1__elem2">FOO</div>
      BOO
    </div>
    ```

    Или в конец:

    ```JS
    block('block1').elem('elem1')(
        content()(function(){
            return [
                applyNext(),
                { elem: 'elem2', content: 'FOO' }
            ];
        })
    );
    ```

1. Про `BEMHTML`.
Иногда возникает необходимость быстро обернуть элемент другим элементом.
Для этого можно воспользоваться модой [default](https://ru.bem.info/technology/bemhtml/v2/reference/#default)
и методом [applyCtx](https://ru.bem.info/technology/bemhtml/v2/templating/#applyctx)
для модификации контента, перед тем, как применятся шаблоны элемента (или другой БЭМ-сущности).

    Например есть шаблон элемента `elem1`:

    ```JS
    block('block1').elem('elem1')(
        tag()('span')
    );
    ```

    В нём необходимо обернуть `elem1` элементом `wrapper`:

    ```JS
    block('block1').elem('elem1')(
        def()(function(){ applyCtx([{ elem: 'wrapper', content: this.ctx }]); }),
        tag()('span')
    );
    ```

    Получим HTML:

    ```HTML
    <div class="block1__wrapper">
      <span class="block1__elem1">BOO</span>
    </div>
    ```

    Но так лучше не делать, это не БЭМ-путь, так как сущность не должна ничего знать и изменять во вне.

1. Про `BEMHTML`.
Очень удобно использовать [произвольные поля](https://ru.bem.info/technology/bemjson/v2/bemjson/#Произвольные-поля)
в сочетании с [произвольными условиями](https://ru.bem.info/technology/bemhtml/v2/templating/#Произвольное-условие).
Эти поля ещё называют кастомными.

    Чтоб протестировать блок `git-members`, используем структуру, приближенную к Github API
    [Members list](https://developer.github.com/v3/orgs/members/#members-list):

    ```JS
    {
        block: 'git-members',
        members: [
            { 'login': 'IronCat', 'avatar_url': 'https://octodex.github.com/images/ironcat.jpg' },
            { 'login': 'Spidertocat', 'avatar_url': 'https://octodex.github.com/images/spidertocat.png', 'site_admin': true },
            { 'login': 'Okal-Eltocat', 'avatar_url': 'https://octodex.github.com/images/okal-eltocat.jpg', 'site_admin': false }
        ]
    }
    ```

    Шаблон блока `git-members` подробно опишу комментариями:

    ```JS
    block('git-members')(
        // По умолчанию пустой `<div>`
        js()(true), // Включаем JS

        match(this.ctx.members)(    // Если передали в шаблон блока произвольное поле `members`,
            tag()('ul'),            // переопределяем тег на список

            content()(function() {  // В контент передаем список `members`, 
                                    // модифицируя каждый элемент массива полем `elem: 'user'`
                                    // Массив превращается в список элементов `user`
                return this.ctx.members.map(function(member) {
                    member.elem = 'user';
                    return member;
                });
            })
        ),

        elem('user')(   // Здесь матчимся на то самое поле `elem: 'user'`
            tag()('li'),

            // Если есть произвольное поле `avatar_url`,
            // вставляем в контент списка картинку.
            // Хоть в документации написано, что метод `match()` принимает в качестве аргумента
            // хэш, рекомендуется оборачивать его в функцию. Иначе в продакшен режиме, `this` будет
            // указывать на глобал и шаблон сломается.
            match(function() { return this.ctx.avatar_url; })(
                content()(function() { return { block: 'image', url: this.ctx.avatar_url } })
            ),

            // Если есть произвольное поле `login`
            match(function() { return this.ctx.login; })(

                content()(function() {      // добавляем в контент
                    return [ 
                        applyNext(),        // за картинкой
                        {
                            elem: 'login',  // элемент `login` с содержимым из `member.login`
                            content: this.ctx.login,
                                            // добавляем модификатор `admin`
                                            // в качестве значения передаем `member.site_admin`
                            elemMods: { admin: this.ctx.site_admin }
                        }
                    ]
                })
            )
        ),

        // Доопределяем шаблон элемента `login` тегом `span`
        elem('login').tag()('span')
    );
    ```

    Добавляем файл с зависимостями `git-members.deps.js` и можно проверить работу блока на тестовых данных:

    ```JS
    ({
        mustDeps: [ 'i-bem' ],
        shouldDeps: [
            { block: 'image' }
        ]
    })
    ```

    В результате получим HTML, класс со `Spidertocat` содержит модификатор `git-members__login_admin`:

    ```HTML
    <ul class="git-members i-bem" data-bem="{"git-members":{}}">
      <li class="git-members__user">
        <img class="image" role="img" src="https://octodex.github.com/images/ironcat.jpg"/>
        <span class="git-members__login">IronCat</span>
      </li>
      <li class="git-members__user">
        <img class="image" role="img" src="https://octodex.github.com/images/spidertocat.png"/>
        <span class="git-members__login git-members__login_admin">Spidertocat</span>
      </li>
      <li class="git-members__user">
        <img class="image" role="img" src="https://octodex.github.com/images/okal-eltocat.jpg"/>
        <span class="git-members__login">Okal-Eltocat</span>
      </li>
    </ul>
    ```

1. Про `BEMHTML` на клиенте.
Блоки можно [шаблонизировать на клиенте](https://ru.bem.info/forum/476/). В качестве примера можно
использовать блок `git-members` из предыдущего совета. Данные блоку нужно передавать в формате
BEMJSON, но в некоторых случаях можно использовать JSON с минимальной модификацией на клиенте.

    Выведем список пользователей и их изображения, входящие в организацию BEM на github.com.
    Для того, чтоб загрузить боевой JSON с Github, напишем ещё один блок `bemembers`.
    Его шаблон будет состоять всего из одной строчки, включающей JS:

    ```JS
    block('bemembers').js()(true);
    ```

    Добавим JS файл `bemembers.js`, который во время инициализации блока, заберёт данные в формате JSON
    и передаст их в поле `members` шаблона блока `git-members`. Далее шаблон с данными применяется
    внутри блока `bemembers`:

    ```JS
    modules.define('bemembers', ['i-bem__dom', 'BEMHTML', 'jquery'], function(provide, BEMDOM, BEMHTML, $) {

    provide(BEMDOM.decl(this.name, {
        onSetMod: {
            js: {
                inited: function() {
                    var bemMembers = this.domElem;

                    $.get('https://api.github.com/orgs/bem/members').then(function(members) {
                        BEMDOM.update(bemMembers, BEMHTML.apply({
                            block: 'git-members',
                            members: members
                        }));
                    });
                }
            }
        }
    }));

    });

    ``` 

    Для того, чтоб BEMHTML блока `git-members` во время сборки попал в клиентский JS страницы
    необходимо это явно указать в файле с зависимостями `bemembers.deps.js`, что технология
    `tech: 'js'` зависит от технологии `tech: 'bemhtml'` блока `git-members`:

    ```JS
    ({
        mustDeps: [ 'i-bem' ]
    }, {
        tech: 'js',
        mustDeps: [
            {
                block: 'git-members',
                tech: 'bemhtml'
            }
        ]
    })

    ```

    Меняем в BEMJSON объявление блока `git-members` на блок `bemembers` без параметров и проверяем работу.

1. Про `BEMHTML` на клиенте и `BEMTREE`.
Модифицировать данные — это очень очень плохая практика, и это не по БЭМ. Для того чтобы привести
JSON к BEMJSOM был придуман BEMTREE.

1. Про `BEMHTML`.
Чего нельзя делать в шаблоне:

  - Хотеть модифицировать шаблон ...


Предлагаю накапливать капитанские советы, пишите в комментариях, будем расширять список.
