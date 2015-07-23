# Капитанские советы БЭМ

Не программист и не фронтендер, но захотелось попробовать написать свой проект,
используя БЭМ. В процессе собрался набор очевидных рекомендаций, которые по началу
казались вовсе не очевидными.
Надеюсь, они помогут новичкам быстро сориентироваться, что к чему.

    «Нужно писать код правильно, не писать неправильно.»
    © veged@

`<cut>`

1. Про `JS`, `BEMHTML` и `CSS`. Если код элемента или модификатора описывается несколькими строками,
то можно всё держать в одном файле, например в блоке или элементе для модификатора элемента.
Это делает код более читаемым. При разрастании кода, выносим в
[отдельные файлы](https://ru.bem.info/method/filesystem/#Опциональные-элементы-и-модификаторы-выносятся-в-отдельные-файлы),
соответствующей [БЭМ-сущности](https://ru.bem.info/method/definitions/#БЭМ-сущность).

1. Про `BEMHTML`. Очень удобно сначала писать BEMJSON, а потом начинать на него писать шаблоны.
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

1. Про `JS` и `BEMHTML`. Везде, где необходимо написать JS с использованием методов из
модулей `i-bem` и `i-bem__dom`, в шаблоне указываем `js()(true)`, что будет означать,
что для блока сгенерируется необходимая разметка, включающая параметры блока и микс
с блоком `i-bem`. В HTML-теге появится атрибут `data-bem`, а из JS возможность
использовать BEM и BEMDOM-методы библиотеки [bem-core](https://ru.bem.info/libs/bem-core/v2.6.0/desktop/i-bem/jsdoc/)
при [соответствующей декларации](https://ru.bem.info/libs/bem-core/v2.6.0/desktop/i-bem/#Синтаксис-декларации).

1. Про `JS`, `BEMHTML` и `CSS`. При статической сборке из BEMJSON если в нём описать используемые
элементы и модификаторы блоков, то зависимости будут сгенерированы из BEMJSON,
всё будет работать без `deps.js` файлов в блоках.
Но если блок разрабатывается без гарантии, что все необходимые элементы и модификаторы будут
объявлены в BEMJSON, например когда сущности объявляются в шаблонах или шаблоны генерируются
в рантайме, тогда эти сущности нужно явно указывать в файлах `deps.js`.
Для этого, всё, что выносится из файла блока в отдельный файл сущности нужно тут-же,
осмысленно, упомянуть эти сущности в `deps.js`.
По началу часто забываешь, и вспоминаешь об этом, когда что-то не доехало в браузер.

1. Про `BEMHTML`. Везде, где используем BEMHTML, нужно добавить зависимость от `i-bem`
в `deps.js` файл БЭМ-сущности. Это подключит базовые шаблоны.

    ```JS
    [{
        mustDeps: { block : 'i-bem', elems : 'dom' }
    },{
        // зависимости блока
    }]
    ```


1. Про `CSS`. Если возникла необходимость модифицировать какой-то один блок
(или другую БЭМ-сущность), то лучше это сделать
[булевым модификатором](https://ru.bem.info/method/naming-convention/#Имя-модификатора).
Если такая, однотипная модификация, требуется для нескольких различных блоков,
то лучше её вынести в блок и [подключать миксом](https://ru.bem.info/technology/bemjson/v2/bemjson/#Представление-БЭМ-сущностей).

1. Про `JS`, `BEMHTML` и `CSS`. Инициализацию и функциональность элемента или модификатора
выносим в отдельный файл (или объявление) элемента или модификатора. Это делает блок
независимым от кода данной сущности. Немного про [декларацию в JS](https://ru.bem.info/libs/bem-core/v2.6.0/desktop/i-bem/docs/#Декларация).

    ```JS
    modules.define('block1', ['i-bem__dom'], function(provide, BEMDOM) {
    provide(BEMDOM.decl(this.name, {}, {}));
    });

    modules.define('block1', ['i-bem__dom'], function(provide, BEMDOM, Block1) {
    provide(Block1.decl({block: 'block1', elem: 'elem1'}, {
        onSetMod : {
            'js' : {
                'inited' : function() {
                        this.__base.apply(this, arguments);
                        ...
                    });
                }
            }
        }
    }))
    
    });
    ```

1. Про `JS`. Поиск BEMDOM одного из нескольких одноименных блоков внутри родительского блока
с помощью обёртки в элемент. Иногда нужно навесить БЭМ-событие на один из нескольких блоков
(например `link`) внутри родительского блока. Самый простой способ, обернуть или смиксовать нужный блок
элементом и найти его BEMDOM методом [findBlockInside([elem], block)](https://ru.bem.info/libs/bem-core/v2.6.0/desktop/i-bem/jsdoc/#findBlockInside-1).

    ```JS
    var link = this.findBlockInside('elem1', 'link');
    ```

1. Про `BEMHTML`. Иногда возникает необходимость быстро обернуть элемент другим элементом.
Для этого можно воспользоваться модой [default](https://ru.bem.info/technology/bemhtml/v2/reference/#default)
и методом [applyCtx](https://ru.bem.info/technology/bemhtml/v2/templating/#applyctx)
для модификации контента, перед тем, как применется шаблон элемента (или другой БЭМ-сущности).

    Например есть элемент `image` в шаблоне блока:

    ```JS
    block('block1')(
        elem('image')(
            tag()('img'),
            attrs()(function() { return { src: this.ctx.url }; })
        )
    );
    ```

    Оборачиваем элемент `image` тэгом `<span>` с помощью элемента `wrapper`

    ```JS
    block('block1')(
        elem('image')(
            def()(function(){ applyCtx([{ elem: 'wrapper', content: this.ctx }]); }),
            tag()('img'),
            attrs()(function() { return { src: this.ctx.url }; })
        )
    );
    block('block1').elem('wrapper').tag('span');
    ```

    Но так лучше не оставлять, а отрефакторить при первой же возможности.

1. Про `BEMHTML`. Чтобы в блоке (или другой БЭМ-сущности) расположить последовательно несколько
других элементов или блоков, нужно их передать в `content()` с помощью функции в формате BEMJSON:

    ```JS
    block('blok1')(
        content()(function() {
            return [
                { elem: 'elem1', content: 'BOO' },
                { block: 'block2', content: 'FOO' }
            ];
        })
    );
    ```

    Это же свойство можно использовать для создания более сложной HTML-структуры блока:

    ```JS
        content()(function() {
            return [
                {
                    elem: 'elem1',
                    content: [
                        { elem: 'elem2', content: 'Bla' }
                        { block: 'block3', content: 'Bla Bla' },
                        'BOO'
                    ]
                },
                { block: 'block2', content: 'FOO' }
            ];
        })
    ```

1. Про `BEMHTML`. Чтобы вложить в элемент другой элемент (или другую БЭМ-сущность), можно воспользоваться
предыдущим советом, с отличаем, что для проброса содержимого элемента нужно использовать
[applyNext()](https://ru.bem.info/technology/bemhtml/v2/templating/#applynext):

    ```JS
    block('block1').elem('elem1')(
        content()(function(){
            return {
                elem: 'elem2',
                content: applyNext()
            };
        }),
    );
    ```

1. Про `BEMHTML`. Можно добавлять элемент (или другую БЭМ-сущность) к существующему контенту в начало:

    ```JS
    block('block1').elem('elem1')(
        content()(function(){
            return [
                { elem: 'elem2', content: 'BOO' },
                applyNext()
            ];
        }),
    );
    ```

    Или конец:

    ```JS
    block('block1').elem('elem1')(
        content()(function(){
            return [
                applyNext(),
                { elem: 'elem2', content: 'FOO' }
            ];
        }),
    );
    ```

1. Про `BEMHTML` и `JS`.
Очень удобно использовать [произвольные поля](https://ru.bem.info/technology/bemjson/v2/bemjson/#Произвольные-поля)
в сочетании с [произвольными условиями](https://ru.bem.info/technology/bemhtml/v2/templating/#Произвольное-условие).
Это можно использовать для [шаблонизации на клиенте](https://ru.bem.info/forum/476/) данных
в формате JSON вместо BEMJSON с минимальной модификацией на клиенте. Давайте разберем пример,
выведем список пользователей и их изображения, входящие в организацию BEM на github.com.

    Чтоб протестировать блок `git-members`, используем структуру, приближенную к Github API
    [Members list](https://developer.github.com/v3/orgs/members/#members-list):

    ```JS
    {
        block: 'git-members',
        members: [
            { 'login': 'IronCat', 'avatar_url': 'https://octodex.github.com/images/ironcat.jpg' },
            { 'login': 'Spidertocat', 'avatar_url': 'https://octodex.github.com/images/spidertocat.png' },
            { 'login': 'Okal-Eltocat', 'avatar_url': 'https://octodex.github.com/images/okal-eltocat.jpg' }
        ]
    }
    ```

    Шаблон блока `git-members` подробно опишу:

    ```JS
    block('git-members')(
        tag()('div'), // По умолчанию пустой `<div>`
        js()(true), // Включаем JS

        match(this.ctx.members)( // Если передали в шаблон блока произвольное поле `members`,
            tag()('ul'), // переопределяем тег на список

            content()(function() {  // В контент передаем список `members`, 
                                    // модифицируя каждый элемент массива полем `elem: 'user'`
                                    // Массив превращается в список элементов `user`
                return this.ctx.members.map(function(member) {
                    member.elem = 'user';
                    return member;
                });
            })
        ),

        elem('user')( // Здесь матчимся на то самое поле `elem: 'user'`
            tag()('li'),

            match(this.ctx.avatar_url)( // Если есть произвольное поле `avatar_url`,
                                        // вставляем в контент списка картинку
                content()(function() { return { block: 'image', url: this.ctx.avatar_url } })
            ),
                          
            match(this.ctx.login)(  // Если есть произвольное поле `login`,
                                    // добавляем за картинкой элемент `username` с содержимым `login`
                content()(function() { return [ applyNext(), { elem: 'username', content: this.ctx.login } ] })
            )
        ),

        elem('username').tag()('span') // доопределяем шаблон элемента `username` тегом `span`
    );
    ```

    Добавляем файл с зависимостями `git-members.deps.js` и можно проверить работу блока на тестовых данных:

    ```JS
    [{
        mustDeps: { block : 'i-bem', elems : 'dom' },
        shouldDeps: [
            { block: 'image' }
        ]
    }]
    ```

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

    Для того, чтоб BEMHTML блока `git-members` во время сборки попал в клиентски JS страницы
    необходимо это явно указать в файле с зависимостями `bemembers.deps.js`, что технология
    `tech: 'js'` зависит от технологии `tech: 'bemhtml'` блока `git-members`:

    ```JS
    [{
        mustDeps: { block : 'i-bem', elems : 'dom' }
    }, {
        tech: 'js',
        mustDeps: [
            {
                block: 'git-members',
                tech: 'bemhtml'
            }
        ]
    }]

    ```

    Меняем в BEMJSON объявление блока `git-members` на блок `bemembers` без параметров и проверяем работу.

Предлагаю накапливать капитанские советы, пишите в комментариях, будем расширять список.
