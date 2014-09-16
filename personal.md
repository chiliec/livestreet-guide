#Как отключить создание персонального блога
Чтобы отключить создание персонального блога, необходимо в методе `Add()` [модуля User](https://github.com/livestreet/livestreet/blob/1.0.3/classes/modules/user/User.class.php#L414) закомментировать строчку:
~~~
$this->Blog_CreatePersonalBlog($oUser);
~~~