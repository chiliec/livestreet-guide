#Редакторы разметки
В стандартной поставке LiveStreet CMS имеется два редактора: [MarkItUp!](http://markitup.jaysalvat.com/home/) и [TinyMCE](http://www.tinymce.com/).

*MarkItUp!* — это легкий, функциональный и очень гибкий редактор, основанный на JQuery. Подходит для опытных пользователей, поскольку оперерирует тегами. Используется по умолчанию.

*TinyMCE* — WYSIWYG-редактор (от англ. What You See Is What You Get, «что видишь, то и получишь»). Подходит для использования неопытными пользователями, поскольку процесс работы не сильно отличается от работы с офисными пакетами.
Включить его можно, добавив в конфиг:
~~~
$config['view']['tinymce']         = true;  // использовать или нет визуальный редактор TinyMCE
~~~
Добавить свои кнопки в редактор MarkItUp! можно через javascript:
~~~
/* Получим текущие настройки редактора */
var newMarkitupSettings = ls.settings.getMarkitup();

ls.settings.getMarkitup = function() {
    newMarkitupSettings.markupSet = newMarkitupSettings.markupSet.concat([ /* Объединяем текущие настройки с добавленными ниже */
        {separator: '---------------'},
		{name: 'Align Text', className: 'mAlign',
			dropMenu: [ /* Эти элементы будут выпадающими при выделении */
				{name: 'Centre', className: 'mCentre', openWith: '<p style="text-align: center;">', closeWith: '</p>'},
				{name: 'Justify', className: 'mJustify', openWith: '<p style="text-align: justify;">', closeWith: '</p>'},
				{name: 'Left', className: 'mLeft', openWith: '<p style="text-align: left;">', closeWith: '</p>'},
				{name: 'Right', className: 'mRight', openWith: '<p style="text-align: right;">', closeWith: '</p>'}
			]
		},
    ]);

    return newMarkitupSettings;
};
~~~
Обратите внимание, что добавленные теги должны быть внесены в список разрешенных в конфиге Jevix!
