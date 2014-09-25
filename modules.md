#Модули (Module)
**Модуль (Module)** в LiveStreet с точки зрения *MVC* являются моделью. Модули предназначены для объединения часто используемого функционала, т.е. это некие аналоги внутренних библиотек, которые можно вызывать в любом месте. Например, модуль может содержать методы для обработки запросов к базе данных, подсчета некой статистики, работы с системой кеширования. Модули бывают двух типов — системные (модули ядра) и пользовательские (обычные модули). Отличие в них только одно — системные находятся в каталоге фреймворка [/framework/classes/modules](https://github.com/livestreet/livestreet-framework/tree/master/classes/modules), а пользовательские в каталоге приложения [/application/classes/modules](https://github.com/livestreet/livestreet/tree/master/application/classes/modules). Название класса модуля строится по шаблону `Module[Name]`, где `Name` — это название модуля, первая буква должна быть заглавной, остальные в нижнем регистре. Например, `ModuleUser`. Класс модуля должен наследоваться от базового класса `Module` ([/framework/classes/engine/Module.class.php](https://github.com/livestreet/livestreet-framework/blob/master/classes/engine/Module.class.php)). Каждый модуль расположен в отдельном каталоге, название которого совпадает с именем модуля, но в нижнем регистре, например, `/classes/modules/user/`. Сам файл модуля также содержит его название — `User.class.php`, первая буква в верхнем регистре, остальные в нижнем. Пример простого модуля (`/classes/modules/user/User.class.php`):
~~~
<?php

class ModuleUser extends Module {
	/**
	 * Инициализация
	 *
	 */
	public function Init() {
			// здесь какие либо действи при инициализации модуля 
	}

	/**
	 * Некий метод модуля
	 *
	 */
	public function SomeFunction() {

	}     

	/**
	 * Вызывается автоматически при завершении работы модуля
	 *
	 */
	public function Shutdown() {

	}
}
~~~
Обязательным методом для модуля является `Init()` — он автоматически выполняется **один раз** при инициализации модуля. Инициализация происходит при первом обращении к модулю, либо при инициализации ядра, если модуль добавлен в автозагрузку через конфиг:
~~~
$config["module"]["autoLoad"] = array("Hook","Cache","User");
~~~
Здесь в массиве перечисляются модули, которые должны загружаться вместе с ядром. Метод `Shutdown()` выполняется автоматически при завершении модуля. Процедура завершения всех активных (инициализированных) модулей запускается ядром при его завершении. Обращение к методам модуля в большинстве случаев (в экшенах, хуках, модулях, блоках, сущностях, плагинах) осуществляется через `$this->Module_Method()`, например, `$this->User_SomeFunction()`. Но также можно обратиться к модулю через ядро: `Engine::getInstance()->User_SomeFunction()`

##Системные модули

###Cache
Модуль кеширования. Для реализации кеширования используетс библиотека Zend_Cache с двумя бэкэндами File и Memcache. Т.к. в memcache нет встроенной поддержки тегирования при кешировании, то для реализации тегов используется враппер для `Zend_Cache_Backend_Memcached` — [Dklab_Cache_Backend_TagEmuWrapper](http://dklab.ru/lib/Dklab_Cache/). Настройки кеширования устанавливаются в конфиге:
~~~
<?php
/**
 * Использовать кеширование или нет
 */
$config['sys']['cache']['use'] = true;
/**
 * Тип кеширования. Возмоные значения: file, memory 
 */
$config['sys']['cache']['type'] = 'file';
/**
 * Каталог для хранения файлового (file) кеша, также используется для временных картинок.
 */
$config['sys']['cache']['dir'] = '___path.root.server___/tmp/';
/**
 * Префикс кеширования, чтоб можно было на одном сервере держать несколько сайтов с общим кешевым хранилищем
 */
$config['sys']['cache']['prefix'] = 'livestreet_cache';
/**
 * Уровень вложенности директорий файлового кеша
 */
$config['sys']['cache']['directory_level'] = 1;

/**
 * Настройка серверов memcache
 */
$config['memcache']['servers'][0]['host'] = 'localhost';
$config['memcache']['servers'][0]['port'] = '11211';
$config['memcache']['servers'][0]['persistent'] = true;
$config['memcache']['compression'] = true;
~~~
Доступные методы модуля кеширования:
~~~
<?php
/**
 * Получает значение из кеша по имени.
 * В качестве имени может быть передан массив имен, тогда метод вернет ассоциативный массив значений.
 * $sName - имя (ключ) кеша 
 */
function Get($sName)

/**
 * Записывает значение в кеш.
 * $data - данные
 * $sName - имя кеша
 * $aTags - список тегов
 * $iTimeLife - время изни кеша в секундах  
 */
function Set($data,$sName,$aTags=array(),$iTimeLife=false)

/**
 * Удаляет значение из кеша по имени
 * $sName - имя (ключ) кеша 
 */
function Delete($sName)

/**
 * Очищает кеш.
 * $cMode - режим очистки кеша. 
 * Zend_Cache::CLEANING_MODE_ALL - удаляет весь кеш
 * Zend_Cache::CLEANING_MODE_OLD - удаляет просроченный кеш
 * Zend_Cache::CLEANING_MODE_MATCHING_TAG - удаляет кеш с определенными тегами
 * $aTags - список тегов, используется при $cMode = Zend_Cache::CLEANING_MODE_MATCHING_TAG
 */
function Clean($cMode = Zend_Cache::CLEANING_MODE_ALL, $aTags = array())

/**
 * Получает статистику использовани кеша в виде ассоциативного массива
 */
function GetStats()
~~~
Пример использования кеширования:
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
		/**
		 * Пытаемся получить значение из кеша
		 */
		if (false === ($oUser = $this->Cache_Get("user_login_{$sLogin}"))) {
			/**
			 * Если значение из кеша получить не удалось, то обращаемся к базе данных
			 */
			$oUser = $this->oMapper->GetUserByLogin($sLogin);
			/**
			 * Записываем значение в кеш
			 */
			$this->Cache_Set($oUser, "user_login_{$sLogin}", array(), 60*60*24*5);
		}
		return $oUser;
	}
	/**
	 * Обновляет пользовател в БД
	 */
	public function UpdateUser($oUser) {
		/**
		 * Удаляем кеш конкретного пользователя
		 */
		$this->Cache_Delete("user_login_{$oUser->getLogin()}");
		/**
		 * Удалем кеш со списком всех пользователей 
		 */
		$this->Cache_Clean(Zend_Cache::CLEANING_MODE_MATCHING_TAG,array('user_update'));
		/**
		 * Обновлем пользовател в базе данных
		 */
		return $this->oMapper->UpdateUser($oUser);
	}
	/**
	 * Получает список всех пользователей
	 */
	public function GetUsers() {
		/**
		 * Пытаемся получить значение из кеша
		 */
		if (false === ($aUserList = $this->Cache_Get("users"))) {
			/**
			 * Если значение из кеша получить не удалось, то обращаемся к базе данных
			 */
			$aUserList = $this->oMapper->GetUsers();
			/**
			 * Записываем значение в кеш
			 */
			$this->Cache_Set($oUser, "users", array('user_update'), 60*60*24*5);
		}
		return $aUserList;
	}
}
~~~

###Database
Модуль для работы с базой данных. Работа с базой данных осуществляется через внешнюю библиотеку [DbSimple](https://github.com/livestreet/livestreet-framework/tree/master/libs/vendor/DbSimple). Задача модуля Database сводится к получению экземпляра объекта *DbSimple*, используя параметры подключения к базе данных.
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
В мапперах объект *DbSimple* содержится в свойстве `$this->oDb`. Пример запроса к БД из маппера:
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
		$sql = "SELECT *  
			FROM ".Config::Get('db.table.user')."
			WHERE user_login = ? ";
		if ($aRow=$this--->oDb->selectRow($sql,$sLogin)) {
			return Engine::GetEntity('User',$aRow);
		}
		return null;
	}
}
~~~
Более подробно ознакомиться с `DbSimple` и синтаксисом плейсхолдеров в SQL запросах можно на [странице документации к DbSimple](http://dklab.ru/lib/DbSimple/manual.html).

###Image
Модуль для обработки изображений, до версии LiveStreet 2.0 использовавший  библиотеку LiveImage. В версии 2.0 она была [заменена на библиотеку Imagine](https://github.com/livestreet/livestreet-framework/commit/a1e83e2aaac77468a2d48a95c334689000faef4a). Настройки и методы будут добавлены сюда после финального релиза LiveStreet 2.0.

###Lang

Модуль для работы с языковыми текстами, позволяющий реализовать мультиязычный интерфейс на сайте. Языковые тексты хранятся в языковых файлах, название файла соответствует языку текста содержащегося в нем, например, `russian.php`.
~~~
<?php
/**
 * Пример русского языкового файла russian.php  
 */
return array(   
        'blogs' => 'Блоги',     
        'blogs_rating' => 'Рейтинг',
        'blogs_owner' => 'Смотритель: %%login%% ',
);      
~~~
Структура расположения языковых файлов следующая:
~~~
/**
 * Основные языковые файлы
 */
/templates/language/
					russian.php
					english.php

/**
 * Дополнительные языковые файлы модулей, название подкаталога (name_module) не обязательно должно совпадать с названием модуля
 */
/templates/language/modules/
							name_module/
										russian.php
										english.php

/**
 * Дополнительные языковые файлы шаблонов
 */
/templates/language/skin/name_skin/settings/language/
													russian.php
													english.php
~~~
Настройки модуля:
~~~
<?php
/**
 * Текущий язык текстовок 
 */
$config['lang']['current'] = 'russian';
/**
 * Язык текстовок по умолчанию. Если в текущем языке текстовка не найдена, то она берется из дефолтного языка
 */
$config['lang']['default'] = 'russian';
/**
 * Путь до каталога с основными языковыми файлами
 */
$config['lang']['path']    = '___path.root.server___/templates/language';
/**
 * Если установлена true, то модуль будет автоматически удалять из языковых конструкций переменные вида %%var%%, по которым не была произведена замена
 */
$config['module']['lang']['delete_undefined'] = true;
~~~
При использовании кеширования с memcache, текстовки кешируются в памяти на 1 час. Основные методы модуля:
~~~
<?php
/**
 * Устанавливает текущий язык текстовок
 * $sLang - язык текстовок, например, 'russian'
 */
function SetLang($sLang)

/**
 * Возвращает текущий язык текстовок
 */
function GetLang()

/**
 * Возвращает дефолтный язык текстовок
 */
function GetLangDefault()

/**
 * Возвращает полный список загруженных текстовок
 */
function GetLangMsg()

/**
 * Возвращает текстовку по её имени
 * $sName - имя текстовки
 * $aReplace - массив для замены шаблонов в текстовке. Например, array('login'=-->'Ivan') заменит %%login%% на 'Ivan'
 */
function Get($sName,$aReplace=array())

/**
 * Добавляет к текстовках новые тексты из массива
 * $aMessages - массив текстовок, например, array('new_text'=>'new text')
 */
function AddMessages($aMessages)

/**
 * Добавлет новую текстовку
 * $sKey - имя текстовки
 * $sMessage - значение текстовки, т.е. сам текст
 */
function AddMessage($sKey, $sMessage)
~~~

###Logger
Модуль логирования. Позволяет вести логи каких-либо действий. Все логи хранятся в каталоге `/logs/`. В конфиге определяется имя файла лога по умолчанию:
~~~
<?php
/**
 * Файл лога относительно каталога /logs/
 */
$config['sys']['logs']['file'] = 'log.log';
~~~
Логгер поддерживает 3 уровня логирования по возрастающему приоритету: `DEBUG`, `NOTICE` и `ERROR`. Логирование происходит только тогда, кода установленный уровень логирования **больше либо равен** уровню текущей записи в лог. Основные методы модуля:
~~~
<?php
/**
 * Устанавливает уровень логирования
 * $level - число от 0 до 2, либо название уровня логирования: 'DEBUG','NOTICE','ERROR'
 */
function SetWriteLevel($level)

/**
 * Возвращет уровень логирования в виде числа: 0 - 'DEBUG', 1 - 'NOTICE', 2 - 'ERROR'
 */
function GetWriteLevel()

/**
 * Использовать трассировку или нет. Позволяет опредлить место вызова записи в лог. 
 * $bool - true | false
 */
function SetUseTrace($bool)

/**
 * Возвращает используется трассировка или нет
 */
function GetUseTrace()

/**
 * Использовать ротацию логов или нет. При превышении максимальное размера файл лога переименовывается в 'filename.1.log'  
 * $bool - true | false
 */
function SetUseRotate($bool)

/**
 * Возвращет используется ротаци логов или нет
 */
function GetUseRotate()

/**
 * Устанавливает файл лога
 * $sFile - имя файла относительно каталога /logs/
 */
function SetFileName($sFile)

/**
 * Возвращет имя файл лога
 */
function GetFileName()

/**
 * Производит запись в лог с уровнем логирования 'DEBUG'
 */
function Debug($msg)

/**
 * Производит запись в лог с уровнем логирования 'ERROR'
 */
function Error($msg)

/**
 * Производит запись в лог с уровнем логирования 'NOTICE'
 */
function Notice($msg)
~~~

###Mail
Модуль для отправки почты на e-mail адреса. Для реализации отправки почты модуль использует библиотеку [phpMailer](https://github.com/livestreet/livestreet-framework/tree/master/libs/vendor/phpMailer). Настройки модуля в конфиге:
~~~
<?php
/**
 * Тип отправки e-mail. Значения: "mail", "sendmail" или "smtp"
 */
$config['sys']['mail']['type'] = 'mail';
/**
 * Адрес с которого отправлять уведомления
 */
$config['sys']['mail']['from_email'] = 'rus.engine@gmail.com';
/**
 * Имя  с которого отправлять уведомления
 */
$config['sys']['mail']['from_name'] = 'Почтовик LiveStreet';
/**
 * Кодировка писем
 */
$config['sys']['mail']['charset'] = 'UTF-8';
/**
 * Настроки SMTP типа отправки почты
 */
$config['sys']['mail']['smtp']['host'] = 'localhost';
$config['sys']['mail']['smtp']['port'] = 25;
$config['sys']['mail']['smtp']['user'] = '';
$config['sys']['mail']['smtp']['password'] = '';
$config['sys']['mail']['smtp']['auth'] = true;
~~~
Основные методы модуля:
~~~
<?php
/**
 * Устанавливает тему письма
 * $sText - текст темы письма
 */
function SetSubject($sText)

/**
 * Устанавливает текст письма
 * $sText - текст содержания письма
 */
function SetBody($sText)

/**
 * Добавлет к получателям новый адрес
 * $sMail - e-mail адрес получателя
 * $sName - имя получателя 
 */
function AddAdress($sMail,$sName=null)

/**
 * Устанавливает одного получателя
 * $sMail - e-mail адрес получателя
 * $sName - имя получателя 
 */
function SetAdress($sMail,$sName=null)

/**
 * Очищает список всех получателей
 */
function ClearAddresses()

/**
 * Устанавливает режим отправки письма как HTML
 */
function setHTML()

/**
 * Устанавливает режим отправки письма как plain text
 */
function setPlain()

/**
 * Отправляет письмо
 * Возвращает true в случае успешной отправки
 */
function Send()
~~~
Пример отправки почты:
~~~
<?php
$this->Mail_SetAdress('user@domain.com','Ivan');
$this->Mail_SetSubject('Test mail');
$this->Mail_SetBody('Hello, <b>Ivan!</b>');
$this->Mail_setHTML();
$this->Mail_Send();
~~~

###Message
Модуль обработки системных сообщений. Необходим для отображения пользователю каких-либо информационных сообщений. Поддерживается две очереди сообщений — сообщения об ошибке и информационные сообщения. Сообщения могут отображаться как в текущем сеансе, так и при следующем обращении к сайту (использование сессионных сообщений). При завершении работы модуля он передает массивы (очереди) сообщений в шаблон: `$aMsgError` и `$aMsgNotice`. Основные методы модуля:
~~~
<?php
/**
 * Добавляет в очередь новое сообщение об ошибке
 * $sMsg - текст сообщения
 * $sTitle - заголовок сообщения
 * $bUseSession - если true, то сообщение будет сохранено в сессии и показанно пользователю только при следующем обращении к сайту
 */
function AddError($sMsg,$sTitle=null,$bUseSession=false)

/**
 * Добавляет новое сообщение об ошибке, предварительно очищает всю очередь
 * $sMsg - текст сообщения
 * $sTitle - заголовок сообщения
 * $bUseSession - сохранять в сессии или нет
 */
function AddErrorSingle($sMsg,$sTitle=null,$bUseSession=false)

/**
 * Добавляет в очередь новое сообщение
 * $sMsg - текст сообщения
 * $sTitle - заголовок сообщения
 * $bUseSession - сохранять в сессии или нет
 */
function AddNotice($sMsg,$sTitle=null,$bUseSession=false)

/**
 * Добавляет новое сообщение, предварительно очищает всю очередь
 * $sMsg - текст сообщения
 * $sTitle - заголовок сообщения
 * $bUseSession - сохранять в сессии или нет
 */
function AddNoticeSingle($sMsg,$sTitle=null,$bUseSession=false)

/**
 * Возвращает очередь (массив) сообщений об ошибке
 */
function GetError()

/**
 * Возвращает очередь (массив) сообщений
 */
function GetNotice() 

/**
 * Возвращает очередь (массив) сообщений об ошибке сохраненых в сессии
 */
function GetErrorSession()

/**
 * Возвращает очередь (массив) сообщений сохраненых в сессии
 */
function GetNoticeSession()
~~~

###Security
Модуль безопасности. На данный момент проверяет только корректность отправленных данных серверу. Для этого перед обработкой данных необходимо вызвать метод:
~~~
<?php
/**
 * Проводит валидацию отправленных данных
 */
function ValidateSendForm()
~~~
Этот метод вернет `true` только в том случае, если при отправке данных был передан правильный ключ (GET/POST параметр `security_ls_key`). Данный параметр (ключ) автоматически прогружается в шаблон с именем `$LIVESTREET_SECURITY_KEY`. Данный параметр уникальный для каждой PHP сессии, т.е. фактически уникальный для каждого пользователя. В конфиге определяется "соль" для генерации ключа:
~~~
<?php
/**
 * "Соль" к строке, хешируемой в качестве security ключа
 */
$config['module']['security']['hash']  = "livestreet_security_key";
~~~
Использование данного модуля предотвращает возможность "фоновой" отправки форм/данных при посещении пользователем злонамеренных сайтов.

###Session
Модуль для работы с PHP-сессиями. Поддерживает два механизма работы: через стандартный механизм PHP-сессий и через собственный. Второй вариант до конца не протестирован, поэтому строго рекомендуется использовать первый, т.е. через стандартные сессии. Настройки модуля в конфиге:
~~~
<?php
/**
 * Использовать или нет стандартный механизм сессий
 */
$config['sys']['session']['standart'] = true;
/**
 * Название сессии
 */
$config['sys']['session']['name'] = 'PHPSESSID';
/**
 * Тайм-аут сессии в секундах
 */
$config['sys']['session']['timeout'] = null;
/**
 * Хост сессии в куках
 */
$config['sys']['session']['host'] = '___sys.cookie.host___';
/**
 * Путь сессии в куках
 */
$config['sys']['session']['path'] = '___sys.cookie.path___';
~~~
Основные методы модуля:
~~~
<?php
/**
 * Возвращает идентификатор сессии
 */
function GetId()

/**
 * Возвращает значение из сессии
 * $sName - имя параметра
 */
function Get($sName)

/**
 * Устанавливает значение параметра в сессии
 * $sName - имя параметра
 * $data - данные параметра
 */
function Set($sName,$data)

/**
 * Удаляет значение параметра из сессии
 * $sName - имя параметра
 */
function Drop($sName)

/**
 * Возвращает сразу все данные сессии в виде ассоциативного массива
 */
function GetData()

/**
 * Удаляет все данные из сессии и завершает сессию
 */
function DropSession()
~~~

###Text
Модуль обработки и типографирования текста. Модуль использует библиотеку типографа [Jevix](https://github.com/livestreet/livestreet-framework/blob/master/libs/vendor/Jevix/jevix.class.php). Основная задача модуля - это удаление из текста недопустимых HTML тегов и фильтрация на вставку JavaScript. Основные методы модуля:
~~~
<?php
/**
 * Парсит текст и возвращает результат
 * $sText - исходный текст
 */
function Parser($sText)

/**
 * Парсит текст используя только Jevix
 * $sText - исходный текст
 */
function JevixParser($sText, $aError=null)

/**
* Парсинг текста на предмет видео
* Находит теги <pre><video></video></pre> и реобразовываетих в видео
*
* @param string $sText Исходный текст
* @return string
*/
function VideoParser($sText)

/**
* Подсветка исходного кода
*
* @param string $sText Исходный текст
* @return mixed
*/
function CodeSourceParser($sText) 

/**
* Производить резрезание текста по тегу cut.
*
* $sText Исходный текст
*/
public function Cut($sText)
~~~

	
	

*При написании этой страницы использована информация с сайта <a href="http://kotoblog.pp.ua">kotoblog.pp.ua</a>.*
