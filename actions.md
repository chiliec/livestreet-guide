#Экшены (Action)
**Экшены (Action)** в LiveStreet с точки зрения *MVC* являются контроллерами. Название экшена является первой частью URL. Например, при запросе `http://site.com/action_name/event_name/` будет запущен экшен с именем `action_name`. Соответствие имени экшена и его класса определяется в конфиге роутинга, например, так:
~~~
/**
 * При запросе УРЛ http://site.com/action_name
 * будет вызван /classes/actions/ActionExample.class.php
 */
$config['router']['page']['action_name'] = 'ActionExample';
~~~
Классы экшенов расположены в каталоге `/classes/actions/` и наследуются от базового класса Action([/framework/classes/engine/Action.class.php](https://github.com/livestreet/livestreet-framework/blob/master/classes/engine/Action.class.php)). Название файла экшена должно соответствовать названию класса, например, для класса `ActionExample` файл должен называться `ActionExample.class.php`. Эвент представляет из себя обычный метод класса, которому определено соответствие с именем эвента, в нашем примере это `event_name`. При запуске экшен пытается найти необходимый эвент по `event_name` и если находит, то запускает его. Пример простого экшена:
~~~
<?php
// Класс должен иметь такое же название, как и имя файла (до .class.php)
class ActionExample extends Action {
        /**
         * Инициализация экшена
         *
         */
        public function Init() {
			// Если экшен запущен без указания эвента - будет вызван эвент index
			$this->SetDefaultEvent('index');
			// 
        }
        /**
         * Регистрация эвентов
         *
         */
        protected function RegisterEvent() {
			// Соответствия между названиями эвентов и их методами
			$this->AddEvent('index','EventIndex');
			$this->AddEvent('edit','EventEdit');            
        }
        /**
         * Метод эвента index
         *
         */
        protected function EventIndex() {
			// Устанавливает путь до шаблона относительно общего каталога шаблонов
			$this->SetTemplate('index'); // /templates/skin/skin_name/index.tpl
        }
        /**
         * Метод эвента edit
         *
         */
        protected function EventEdit() {
			// Передаст в шаблон переменную $iSomeNumber равную 12345
			$this->Viewer_Assign('iSomeNumber', 12345); // будет доступна как {$iSomeNumber} в шаблоне
			// Устанавливает какой шаблон выводить относительно каталога шаблонов экшена
			$this->SetTemplateAction('some_template'); // /templates/skin/skin_name/actions/Action_name/some_template.tpl
        }       
        /**
         * Вызывается автоматически после выполнения эвента
         *
         */
        public function EventShutdown() {
			$this->Viewer_Assign('sText','Event complete');
        }
}
~~~
В данном примере мы создаем экшен ActionTest с двумя эвентами: `index` и `edit`. Обязательные методы, которые должны присутствовать в любом экшене, это `Init()` и `RegisterEvent()`: в первом происходит инициализация экшена, например, установка дефолтного эвента, а во втором регистрация всех используемых эвентов в данном экшене. Также предусмотрены два дополнительных метода, которые определять не обязательно, это `EventShutdown()` и `EventNotFound()`. Первый автоматически вызывается после выполнения текущего эвента, а второй в том случае, если запрашиваемый эвент не найден. По умолчанию `EventNotFound()` переадресует на экшен `error`, который показывает ошибку `404`:
~~~
protected function EventNotFound() {
	return Router::Action('error','404');
}
~~~
##Эвенты (Event)
Регистрация евентов происходит в методе `RegisterEvent()`. Существует два способа зарегистрировать евент: `AddEvent()` и `AddEventPreg()` (второй использует для этого регулярные выражения). Пример регистрации эвентов:
~~~
<?php

// Стандартный способ
$this->AddEvent('event_name','EventMethod');

// Через регулярные выражение
/**
* соответствует site.com/action/123.html и site.com/action/123.html/blabla/
*/
$this->AddEventPreg('/^(\d+)\.html$/i','EventMethod');

/**
* соответствует site.com/action/123.html, но не соответствует site.com/action/123.html/blabla/
*/
$this->AddEventPreg('/^(\d+)\.html$/i','/^$/i','EventMethod');

/**
* соответствует site.com/action/123/, site.com/action/123/page1/ ,
* но не соответствует site.com/action/123/blabla/
*/
$this->AddEventPreg('/^(\d+)$/i','/^(page\d+)?$/i','/^$/i','EventMethod');
~~~
В примерах выше при запросе правильного URL будет вызван метод `EventMethod()`, в противном случае метод `EventNotFound()`. В `AddEvent('event_name','EventMethod')` первый аргумент определяет имя эвента, а второй — метод, который будет вызван. В `AddEventPreg('/^(\d+)\.html$/i','/^$/i','EventMethod')` первый параметр определят регулярное выражение для имени эвента, а второй и последующие (кроме последнего) — регулярное выражение для параметров.  Последний параметр определят название метода для вызова. После выполнения эвента в браузер выводится шаблон, по умолчанию совпадающий с именем эвента. Например, для эвента `AddEvent('event_name','EventMethod')` будет использоваться шаблон `event_name.tpl`, расположенный в `/templates/skin/skin_name/actions/ActionTest/`, где `skin_name` — название скина (общего шаблона), а `ActionTest` — название класса экшена в котором зарегистрирован эвент. Шаблон этот можно переопределить используя метод `SetTemplateAction('some_tamplate')`, в таком случаи будет использован шаблон `/templates/skin/skin_name/actions/ActionTest/some_tamplate.tpl`. Также есть возможность из одного эвента передать управление на другой (внутренний реврайт), или даже на другой экшен. Делается это с помощью роутера:
~~~
<?php

protected function EventEdit() {
        // некоторый код
        //.....

        return Router::Action('test','edit');
}
~~~
В данном примере передается управление в экшен `test` с эвентом `edit`, т.е. происходит эмуляция URL `http://site.com/test/edit/`. Для передачи в шаблон каких-либо данный используется метод `Assign($sName,$value)` системного модуля `Viewer`, где `$sName` — будущее имя переменной в шаблоне, а $value её значение:
~~~
<?php

$this->Viewer_Assign('iSomeNumber',12345); // будет доступна как {$iSomeNumber} в шаблоне
~~~
##Доступные методы Action
* `AddEvent($sEventName,$sEventFunction)` - регистрация эвента по имени
* `AddEventPreg($sEventPregName,..,$sParamPregN,..,$sEventFunction)` - регистрация евента через регулярные выражения
* `SetDefaultEvent($sEvent)` - устанавливает эвент по умолчанию, т.е. тот который будет вызван при URL http://site.com/action/. Данный метод должен вызываться из метода `Init()`
* `GetDefaultEvent()` - возвращает эвент по умолчанию, например, `index`
* `GetEventMatch($iItem=null)` - возвращает элемент совпадения евента по регулярному выражению. Например, если для евента `AddEventPreg('/^(page(\d+))$/i','EventMethod')` вызвать метод `GetEventMatch(2)`, то он вернет числовое значение равное номеру страницы
* `GetParamEventMatch($iParamNum,$iItem=null)` - возвращает элемент совпадения по регулярному выражению для параметров евента. Например, если для евента `AddEventPreg('/^blog$/i','/^new$/','/^page(\d+)$/i','EventMethod')` вызвать метод `GetParamEventMatch(1,1)`, то он вернет числовое значение равное номеру страницы
* `GetParam($iOffset,$default=null)` - возвращает параметр по его номеру/смещению, `$default` определяет дефолтное значение в случаи если параметра не существует. Например, для http://site.com/user/ort/edit/fast/ `GetParam(0)` вернет значение `edit`
* `GetParams()` - возвращает массив текущих параметров, которые были переданные через URL. Например, для http://site.com/user/ort/edit/fast/ вернет значение array('edit','fast')
* `SetParam($iOffset,$value)` - устанавливает значение параметра, например, `SetParam(0,'add')`
* `SetTemplate($sTemplate)` - устанавливает шаблон для вывода, используя путь относительно общего каталога шаблонов. Например, `SetTemplate('index.tpl')` указывает на шаблон `/templates/skin/skin_name/index.tpl`
* `SetTemplateAction($sTemplate)` - устанавливает шаблон для вывода, используя путь относительно каталога шаблонов экшена. Например, `SetTemplateAction('index')` указывает на шаблон `/templates/skin/skin_name/actions/ActionTest/index.tpl`. Внимание! При указании шаблона использовать расширение .tpl не нужно!
* `GetTemplate()` - возвращает пусть до шаблона относительно общего каталога шаблонов, например, `actions/ActionTest/index.tpl`
* `GetActionClass()` - возвращает имя класса экшена, например, `ActionTest`





*При написании этой страницы использована информация с сайта <a href="http://kotoblog.pp.ua">kotoblog.pp.ua</a>.*
