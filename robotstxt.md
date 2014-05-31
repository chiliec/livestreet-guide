Robots.txt для LiveStreet
========================
Файл **robots.txt** – это текстовый файл, находящийся в корневой директории сайта, в котором записываются специальные инструкции для поисковых роботов. Эти инструкции могут запрещать к индексации некоторые разделы или страницы на сайте, указывать на правильное «зеркалирование» домена, рекомендовать поисковому роботу соблюдать определенный временной интервал между скачиванием документов с сервера и т.д. 
Имя файла должно быть в нижнем регистре (ROBOTS.TXT, Robots.txt — неправильно).

Каждый уважающий себя веб-мастер должен разобраться в директивах **robots.txt**. Не нужно слепо копировать **robots.txt** других сайтов - это может наоборот повредить индексации вашего сайта или вообще запретить её, что может привести к довольно печальным последствиям. 

Создание **robots.txt** рассматривается на многих ресурсах. Основными из них являются:

  1. [Справка Яндекса](http://help.yandex.ru/webmaster/?id=996567)
  2. [Справка Гугла](http://support.google.com/webmasters/bin/answer.py?hl=ru&answer=156449)
  3. [Robotstxt.org.ru](http://robotstxt.org.ru/)
  
Для *LiveStreet CMS* существуют специальный плагин, упрощающий процесс создание и редактирование файла **robots.txt**:
<iframe src="http://livestreetcms.com/api/addons/list/frame/?width=550&addon_action=1&addons=513" height="180" width="570" style="border:0 none;margin:0;padding:0;"></iframe>

Ну и напоследок **пример robots.txt для LiveStreet CMS**:
~~~
User-agent: *
Disallow: /profile/
Disallow: /my/
Disallow: /search/
Disallow: /people/
Disallow: /top/ 
Disallow: /stream/
Disallow: /feed/
Disallow: /talk/
Disallow: /settings/
Disallow: /registration/
Disallow: /login/
Disallow: /tag/
Sitemap: http://livestreet.pro/sitemap.xml #Полный URL до файла sitemap

User-agent: Yandex
Disallow: /profile/
Disallow: /my/
Disallow: /search/
Disallow: /people/
Disallow: /top/
Disallow: /stream/
Disallow: /feed/
Disallow: /talk/
Disallow: /settings/
Disallow: /registration/
Disallow: /login/
Disallow: /tag/
Host: livestreet.net #Директива Host есть только у Яндекса. Указывается только имя домена

User-agent: Mediapartners-Google* #Если вы используете Google Adsense
Disallow:
~~~
