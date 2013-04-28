Перенос сайта на LiveStreet на другой домен
===========================================
При переносе сайта на LiveStreet на другой домен необходимо поправить ссылки и пути к изображениям в базе данных, например через phpMyAdmin выполнив следующие команды, не забыв заменить все `http://old.ru` и `http://new.ru` на свои.
~~~
UPDATE `prefix_topic_content` SET `topic_text` = REPLACE(`topic_text`, "http://old.ru", "http://new.ru")

UPDATE `prefix_topic_content` SET `topic_text_short` = REPLACE(`topic_text_short`, "http://old.ru", "http://new.ru")

UPDATE `prefix_topic_content` SET `topic_text_source` = REPLACE(`topic_text_source`, "http://old.ru", "http://new.ru")

UPDATE `prefix_topic_comment` SET `comment_text` = REPLACE(`comment_text`, "http://old.ru", "http://new.ru")
~~~
