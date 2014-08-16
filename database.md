Работа с базой данных
=====================

Мапперы (Mapper)
----------------
Для повышения уровня абстракции для запросов к БД в LiveStreet этого предусмотрены специальные объекты `Mapper`, реализующие паттерн Data Mapper. Это программная прослойка, разделяющая модуль и хранилище (БД). Его обязанность — пересылать данные между ними и изолировать их друг от друга. При использовании Data Mapper'а объекты не нуждаются в знании о существовании БД. Они не нуждаются в SQL-коде, и (естественно) в информации о структуре БД. Так как Data Mapper - это разновидность паттерна Mapper, сам объект-Mapper неизвестен объекту.

 Вся задача маппера сводится в выполнению запроса к базе данных и возвращения результата в модуль. Мапперы находятся в подкаталоге /mapper/ в каталоге с модулем, например, `/classes/modules/user/mapper/`. Название класса состоит из двух частей `Module[Name]_Mapper[NameMapper]`, где `Name` - это название модуля, а `NameMapper` - это название маппера(как обычно первая буква заглавная, остальные в нижнем регистре), например, `ModuleUser_MapperTest`. Зачастую название маппера совпадает с названием модуля. Класс маппера должен наследоваться от класса `Mapper`. Файл маппера называется по шаблону `[NameMapper].mapper.class.php`, например, `Test.mapper.class.php`.

Пример простого маппера (`/classes/modules/user/mapper/Test.mapper.class.php`):

~~~
<?php
class ModuleUser_MapperTest extends Mapper {
        /**
         * Получает пользователя по его логину из таблицы user
         *
         * @param string $sLogin
         * @return unknown
         */
        public function GetUserByLogin($sLogin) {               
                $sql = "SELECT 
                                *  
                        FROM 
                                ".Config::Get('db.table.user')."
                        WHERE 
                                user_login = ? ";
                if ($aRow=$this->oDb->selectRow($sql,$sLogin)) {
                        return Engine::GetEntity('User',$aRow);;
                }
                return null;
        }
}
~~~

В данном примере метод `GetUserByLogin()` выполняет запрос к базе данных для получения пользователя по логину из таблицы `user` и возвращает объект сущности пользователя. Непосредственно сам запрос выполняется через свойство маппера `$oDb`, в котором содержится объект для работы с БД. Таблица для запроса получается из конфига, где определен список таблиц и их префикс в БД (`/config/config.php`):

~~~
<?php
/**
 * Настройка таблиц базы данных
 */
$config['db']['table']['prefix'] = 'prefix_';

$config['db']['table']['user'] = '___db.table.prefix___user';

$config['db']['tables']['engine'] = 'InnoDB';  // InnoDB или MyISAM
~~~

Пример модуля с использованием маппера:

~~~
<?php
class ModuleUser extends Module {
        /**
         * Объект маппера
         *
         * @var Mapper
         */
        protected $oMapper=null;
        /**
         * Инициализация
         *
         */
        public function Init() {
                /**
                 * Получаем маппер по его имени
                 */
                $this->oMapper=Engine::GetMapper(__CLASS__,'Test');
        }
        /**
         * Получает пользователя по его логину
         *
         * @param string $sLogin
         * @return unknown
         */
        public function GetUserByLogin($sLogin) {
                return $this->oMapper->GetUserByLogin($sLogin);
        }
}
~~~

Объект маппера получается через статический метод ядра: `Engine::GetMapper([ClassModule],[NameMapper]);`, где `ClassModule` - это им класса модуля к которому относится маппер, а `NameMapper` - это название маппера, например, `Engine::GetMapper('ModuleUser','Test');`. 

Сущности (Entity)
-----------------

При запросе к базе данных удобно возвращать не просто массив данных, а данные в виде специального объекта - `Entity`. Основные методы такого объекта делятся на два вида: get-методы и set-методы. Первые получают свойство объекта по его имени, а вторые устанавливают. Сущности находятся в подкаталоге `/entity/` в каталоге с модулем, например, `/classes/modules/user/entity/`. Название класса состоит из двух частей `Module[Name]_Entity[NameEntity]`, где `Name` - это название модуля, а `NameEntity` - это название сущности, например, `ModuleUser_EntityUser, ModuleUser_EntitySession` или `ModuleUser_EntityUserVote`. Класс сущности должен наследоваться от класса `Entity`. Файл сущности называется по шаблону `[NameEntity].entity.class.php`, например, `UserVote.entity.class.php`.

Пример сущности (`/classes/modules/user/entity/User.entity.class.php`):

~~~
<?php
class ModuleUser_EntityUser extends Entity {
        
        public function getLogin() {
                return $this->_aData['user_login'];
        }      
        
        public function setLogin($data) {
                $this->_aData['user_login']=$data;
        }
}
~~~

Данные у сущности хранятся в ассоциативном массиве `$this->_aData`. Описывать все поля сущности нет необходимости, если использовать camel-style при наименовании методов. Например, следующие две сущности эквивалентны:

~~~
<?php
/**
 * Первый вариант
 *
 */
class ModuleUser_EntityUser extends Entity {

        public function getLogin() {
                return $this->_aData['login'];
        }
        public function getProfileName() {
                return $this->_aData['profile_name'];
        }

        public function setLogin($data) {
                $this->_aData['login']=$data;
        }
        public function setProfileName($data) {
                $this->_aData['profile_name']=$data;
        }
}

/**
 * Второй вариант
 *
 */
class ModuleUser_EntityUser extends Entity {

}
~~~

Во втором варианте обращение к методам сущности автоматически преобразуется в ключ ассоциативного массива. Например, `OneTwoThreeFour` преобразуется в `one_two_three_four`. Также доступны дополнительные методы:

~~~
<?php
/**
 * Устанавливает значения сущности из ассоциативного массива $aData  
 *
 */
function _setData($aData)

/**
 * Возвращает ассоциативный массив значений сущности. 
 * Если передан параметр $aKeys, то вернуться только те значения, ключи которых перечислены в этом параметре.   
 */
function _getData($aKeys=array())
~~~

Создание объектов сущностей происходит через статический метод ядра:

~~~
<?php
/**
 * Возвращает пустой (без данных) объект сущности Session модуля User
 */
$oSession=Engine::GetEntity('User_Session');
/**
 * Возвращает объект сущности Session модуля User c заполненными даными user_id и key
 */
$oSession=Engine::GetEntity('User_Session',array('user_id'=>123,'key'=>'qwerty'));
/**
 * Возвращает объект сущности User модуля User.
 * Если название сущности совпадает с названием модуля, то допускается короткая запись:  User_User = User
 */
$oUser=Engine::GetEntity('User',$aData);
~~~

Модуль Database
---------------
Работа с базой данных осуществляется через внешнюю библиотеку от Дмитрия Котерова - [DbSimple](https://github.com/livestreet/livestreet-framework/tree/master/libs/vendor/DbSimple). Задача модуля Database сводится к получению экземпляра объекта DbSimple, используя параметры подключения к базе данных.

~~~
<?php
/**
 * Получает объект DbSimple для работы с базой данных.
 * $aConfig - содержит настройки БД(хост,логин,пароль,тип БД,имя БД), если не передан, то берутся настройки из конфига
 */
function GetConnect($aConfig=null)

/**
 * Получает статистику работы с БД в виде ассоциативного массива
 */
function GetStats()
~~~

Дефолтные настройки коннекта к БД хранятся в конфиге:

~~~
<?php
/**
 * Настройка базы данных
 */
$config['db']['params']['host']   = 'localhost';
$config['db']['params']['port']   = '3306';
$config['db']['params']['user']   = 'root';
$config['db']['params']['pass']   = '';
$config['db']['params']['type']   = 'mysql';
$config['db']['params']['dbname'] = 'social';
~~~

Также в конфиге можно указать настройки логирования SQL запросов:

~~~
<?php
/**
 * Логировать или нет все SQL запросы
 */
$config['sys']['logs']['sql_query'] = false;
/**
 * Файл лога SQL запросов относительно каталога /logs/
 */
$config['sys']['logs']['sql_query_file'] = 'sql_query.log';
/**
 * Логировать или нет ошибки при SQL запросах
 */
$config['sys']['logs']['sql_error'] = true;
/**
 * Файл лога ошибок SQL запросов относительно каталога /logs/
 */
$config['sys']['logs']['sql_error_file'] = 'sql_error.log';
~~~

В мапперах объект DbSimple содержится в свойстве `$this->oDb`. Пример запроса к БД из маппера:

~~~
<?php
class ModuleUser_MapperTest extends Mapper {
        /**
         * Получает пользователя по его логину из таблицы user
         *
         * @param string $sLogin
         * @return unknown
         */
        public function GetUserByLogin($sLogin) {               
                $sql = "SELECT 
                                *  
                        FROM 
                                ".Config::Get('db.table.user')."
                        WHERE 
                                user_login = ? ";
                if ($aRow=$this->oDb->selectRow($sql,$sLogin)) {
                        return Engine::GetEntity('User',$aRow);;
                }
                return null;
        }
}
~~~
Более подробно ознакомиться с DbSimple и синтаксисом плейсхолдеров в SQL запросах можно [на странице документации к DbSimple](http://dklab.ru/lib/DbSimple/). 
