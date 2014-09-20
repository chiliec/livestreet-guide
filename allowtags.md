#Как разрешить все теги для администратора
Jevix предназначен для контроля перечня допустимых тегов и аттрибутов и предотвращения возможных XSS-атак. Именно он вырезает все теги, кроме разрешенных явно. Отключать его для пользователей крайне не рекомендуется, но почему бы не отключить для администратора?

Чтобы отключить Jevix (разрешить все html-теги и любую другую разметку) для администратора, нужно лишь добавить соответствующее условие в методе `Parser` модуля `Text` ([здесь][1]):

~~~
$sResult=$this->JevixParser($sResult);
~~~
заменить на:
~~~
if( !$this->User_GetUserCurrent()->isAdministrator() ) {
	$sResult=$this->JevixParser($sResult);
}
~~~
Чтобы отключить Jevix и другим доверенным пользователям, можно сделать, например, так:
~~~
$aTrustedUsers = array('ort','PSNet','otherTrustedUser'); // массив доверенных пользователей
if( !in_array($this->User_GetUserCurrent()->getLogin(),$aTrustedUsers) ) {
	$sResult=$this->JevixParser($sResult);
}
~~~
В новой версии LS (2.0) для этого можно будет использовать [RBAC][2].

[1]: https://github.com/livestreet/livestreet/blob/1.0.3-replication/engine/modules/text/Text.class.php#L149
[2]: http://livestreet.ru/blog/dev_documentation/17378.html