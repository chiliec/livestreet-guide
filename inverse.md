Обратный порядок комментариев
=============================
Для изменения порядка комментариев в LiveStreet (последние - в начале) необходимо в файле `/classes/modules/comment/mapper/Comment.mapper.class.php` найти функцию 
~~~
public function GetCommentsByTargetId($sId,$sTargetType)
~~~
далее в строке
~~~
ORDER by comment_id asc;
~~~
поменять `asc` на `desc`. 
