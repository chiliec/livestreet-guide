#Как отключить создание персонального блога
Чтобы отключить создание персонального блога, необходимо в методе `Add()` [модуля User](https://github.com/livestreet/livestreet/blob/1.0.3/classes/modules/user/User.class.php#L414) закомментировать строчку:
~~~
$this->Blog_CreatePersonalBlog($oUser);
~~~
Теперь при регистрации не будет создаваться персональных блогов. 

Чтобы убрать возможность выбора уже отсутствующего персонального блога в шаблоне нужно удалить или закомментировать в `/templates/skin/названиеактивногошаблона/actions/ActionTopic/add.tpl` строку:
~~~
<option value="0">{$aLang.topic_create_blog_personal}</option>
~~~
То же самое необходимо проделать с файлом `add.tpl` в каталогах `ActionQuestion`, `ActionLink` и `ActionPhotoset`.