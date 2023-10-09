![1CVK](https://github.com/Bayselonarrend/VKEnterprise/raw/main/logo_small.png)
# VKEnterprise
Библиотека интеграции 1С и ВКонтакте

В данной библиотеке реализованы базовые методы VK API для управления группой ВКонтакте из 1С  

**Особенности:**
- Не нужно ничего устанавливать: создайте 2 пустых модуля в базе и скопируйте код из репозитория туда
- Не нужно разбираться в документации VK API: методы уже выполняют конкретные действия, которые подойдут большинству пользователей, вроде СоздатьПост() или СделатьРепост()
- Основная разработка с тестированием велись на режиме своместимости версии 8.3.6. Возможно, будет работать и на более ранних версиях.

Документация VK API:<br>
https://dev.vk.com/ru/method

Схожая библиотека, но для Telegram:<br>
[https://github.com/Bayselonarrend/TelegramEnterprise](https://github.com/Bayselonarrend/TelegramEnterprise)

## Для начала использования библиотеки достаточно пройти два простых шага: ##
<details>
<summary>Установка библиотеки</summary>
<br>
Библиотека представляет из себя всего два общих модуля

  
- **Инструменты** - содержит вспомогательные методы, вроде отправки http запросов, чтения JSON и пр.
- **Действия**    - непосредственно сами методы работы с VK
  
Эти модули необходимо добавить в свою конфигурацию (модули серверные). При переименовании модуля **Инструменты** необходимо будет провести рефакторинг в модуле **Действия**. Модуль же **Действия** можно переименовывать без изменений.

Если вы уже используете бибилотеку [TelegramEnterprise](https://github.com/Bayselonarrend/TelegramEnterprise) для интеграции с Telegram, то модули Инструменты совместимы и дублировать их не нужно, однако стоит проверить, не изменилось ли что-нибудь с выходами новых версий библиотек.

После установки можно вызывать нужные методы из модуля **Действия**
</details>

<details>
<summary>Получение необходимых данных</summary>
<br>
	
Перед началом работы необходимо получить некоторые параметры для VK API. Их перечень содержится в функции *ПолучитьСтандартныеПараметры()*

<br><br>
Полученные параметры можно будет вставить в структуру внутри **ПолучитьСтандартныеПараметры()** и все будет работать. Если же вы хотите передавать их каждый раз при вызове (например разный набор для нескольких сообществ или аккаунтов), то это можно сделать через параметр **Параметры**, который есть у каждого метода.
При вызове, параметры заполняются из функции *ПолучитьСтандартныеПараметры()*, после чего заменяются по ключам значениями, переданными в параметр **Параметры**, если таковые существуют
<br><br>
 
  ```
  _Параметры = Новый Структура;

  _Параметры.Вставить("v"	              , "5.131");
  _Параметры.Вставить("from_group"      , "1");
_Параметры.Вставить("group_id"        , "123456789");
  _Параметры.Вставить("owner_id"        , "-123456789");
  _Параметры.Вставить("app_id"          , "87654321");
_Параметры.Вставить("access_token"    , "vk1.a.E-byuFeG1qcN7...");
	
  ```

Рассмотрим получение каждого значения:

**1. v**
   
   Параметр v означает версию VK API. Тестирование проводилось на 5.131, рекомендуется его таким и оставить

**2. from_group**

   От лица группы. Должен быть 1

**3. group_id и owner_id**

   ID группы. Если у вас стандартный адрес группы, то id можно найти в URL. В противном случае он будет на вкладке "Управление" в группе, под полем Адрес. owner_id - тоже самое, но со знаком '-' впереди

**4. app_id**

   app_id - ID приложения. Для создания приложения необходимо:
   
  * Перейти по адресу https://vk.com/apps?act=manage, авторизоваться и нажать "Создать"
  * Выбрать название и пункт Standalone-приложение
  * После создания, перейти в редактирование на вкладку Настройки, забрать оттуда ID приложения (и есть app_id), и переключить статус на "Приложение включено и видно всем"
  * Сохранить изменения

**5. access_token**

  acess_token можно получить при помощи одного из методов модуля **Действия**:
  
  * Выполнить _СоздатьСсылкуПолученияТокена(app_id)_, Передав туда ID приложения из пункта 5
  * Метод вернет URL, по нему необходимо перейти в браузере
  * Авторизоваться через ВК и подтвердить
  * Забрать токен из параметра URL в адресной строке

**Дополнительно communitytoken**
  Некоторые методы, например отправка сообщений чат-ботом сообщества, принимают в качестве параметра communitytoken - в этих методах он заменяет access_token. Для его получения необходимо:

  * Зайти в раздел "Управление" в группе ВК
  * Найти вкладку "Работа с API"
  * Нажать "Создать ключ" и забрать его

  Пока вам не нужно использовать такие методы, получать communitytoken не обязательно


</details>

## Реализованные методы: ##
*Все методы возвращают ответ от ВК как структуру*
<details>
<summary> Для управления контентом в группе</summary>
<br>
Эти методы предназначены для создания/удаления/редактирования контента в сообществе

* __Создание поста | Метод: СоздатьПост()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | Текст | Строка | Непосредственно текст поста |
  | МассивКартинок | Массив строк (путей к файлам), Массив Двоичных данных | Массив картинок, которые необходимо прикрепить к посту |
  | Рекламный | Булево (По умолчанию Ложь) | Пометить пост как рекламу |
  | СсылкаПодЗаписью | Строка (По умолчанию "") | Позволяет прикрепить ссылку к посту |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных) |

___

* __Удаление поста | Метод: УдалитьПост()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | ID | Строка/Число | ID поста (из URL адреса поста или ответа СоздатьПост() |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

___

* __Создание опроса | Метод: СоздатьОпрос()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | Вопрос | Строка | Вопрос в опросе |
  | МассивОтветов | Массив строк | Набор ответов на вопрос опроса |
  | Картинка | Строка (Путь к файлу)/ДвоичныеДанные (по умолчанию "") | Картинка фона опроса* |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

 ***Вероятно, из-за бага ВК картинка в опрос на 09.2023 не добавляется. Разбирали запрос вместе с поддержкой - ошибок не нашли, но и решить ничего не смогли***
___

* __Создание альбома для картинок | Метод: СоздатьАльбом()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | Наименование | Строка | Название альбома |
  | Описание | Строка | Описание альбома |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

___

* __Сохранить картинку в альбом | Метод: СохранитьКартинкуВАльбом()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | IDАльбома | Строка/Число | ID альбома со стены или из ответа метода *СоздатьАльбом()* |
  | Картинка | Строка (Путь к файлу)/ДвоичныеДанные | Картинка для сохранения |
  | Описание | Строка (По умолчанию "") | Описание картинки |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

___

* __Удалить картинку из группы | Метод: УдалитьКартинку()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | IDКартинки | Строка/Число | ID картинки из URL в браузере или из ответа метода *СохранитьКартинкуВАльбом()* |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

___

* __Добавить историю (storie) с картинкой в группу | Метод: СоздатьИсторию()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | Картинка | Строка (Путь к файлу)/ДвоичныеДанные | Картинка истории |
  | URL | Строка (по умолчанию "") | Добавляет кнопку с переходом по ссылке внизу истории, если введено |

___

</details>


<details>
<summary>Для работы с обсуждениями</summary>
<br>
Эти методы предназначены для работы с обсуждениями в группе

* __Создать обсуждение | Метод: СоздатьОбсуждение()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | Наименование | Строка | Название темы |
  | ТекстПервогоСообщения | Строка | Первое сообщение в теме |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

___

* __Закрыть или удалить обсуждение  | Метод: ЗакрытьОбсуждение()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | IDОбсуждения | Строка/Число | ID обсуждение из URL или ответа метода *СоздатьОбсуждение()* |
  | УдалитьПолностью | Булево (по умолчанию Ложь) | Истина - удаляет обсуждение, Ложь - закрывает обсуждение для новых сообщений |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

___

* __Открыть ранее закрытое обсуждение  | Метод: ОткрытьОбсуждение()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | IDОбсуждения | Строка/Число | ID обсуждение из URL или ответа метода *СоздатьОбсуждение()* |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

___

* __Написать в обсуждение | Метод: НаписатьВОбсуждение()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | IDОбсуждения | Строка/Число | ID обсуждение из URL или ответа метода *СоздатьОбсуждение()* |
  | Текст | Строка | Текст сообщения в теме |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

___

</details>


<details>
<summary>Для получения статистики по группе</summary>
<br>
Эти методы предназначены для получения статистических данных сообщества

* __Получение общей статистики за период | Метод: ПолучитьСтатистику()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | ДатаНачала | Дата | Начало периода |
  | ДатаОкончания | Дата | Конец периода |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

___

* __Получение статистики постов | Метод: ПолучитьСтатистикуПостов()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | МассивIDПостов | Массив строк/чисел | Массив номеров постов |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

___

</details>



<details>
<summary>Для работы с рекламным кабинетом</summary>
<br>
Эти методы предназначены для работы с кампаниями и объявлениями в https://vk.com/ads


* __Создание новой рекламной кампании | Метод: СоздатьРекламнуюКампанию()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | IDКабинета | Строка/Число | Номер кабинета из настроек https://vk.com/ads?act=settings |
  | Наименование | Строка | Наименование кампании |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

___

* __Создание нового рекламного объявления на основе существующего поста | Метод: СоздатьРекламноеОбъявление()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | НомерКампании | Строка/Число | Номер кампании из кабинета или ответа метода *СоздатьРекламнуюКампанию()* |
  | ДневнойЛимит | Строка/Число | Дневной лимит затрат записи в рублях |
  | НомерКатегории | Строка/Число | Номер категории тематики ВК |
  | IDПоста | Строка/Число | ID поста, который нужно рекламировать |
  | IDКабинета | Строка/Число | Номер рекламного кабинета из настроек https://vk.com/ads?act=settings|
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

___

* __Приостанавливает выполнение рекламного объявления | Метод: ПриостановитьРекламноеОбъявление()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | IDКабинета | Строка/Число | Номер кабинета из настроек https://vk.com/ads?act=settings |
  | IDОбъявления | Строка/Число | ID рекламного объявления из кабинета или ответа метода *СоздатьРекламноеОбъявление()* |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

___

* __Заменяет рекламируемый пост в рекламном объявлении | Метод: ИзменитьЗаписьРекламногоОбъявления()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | IDПоста | Строка/Число | ID поста, который необходимо рекламировать |
  | IDКабинета | Строка/Число | Номер кабинета из настроек https://vk.com/ads?act=settings |
  | IDОбъявления | Строка/Число | ID рекламного объявления из кабинета или ответа метода *СоздатьРекламноеОбъявление()* |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

___

</details>



<details>
<summary>Интерактивные действия</summary>
<br>
Эти методы предназначены для выполнения действий с объектами ВК


* __Лайк | Метод: ПоставитьЛайк()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | IDПоста | Строка/Число | Номер поста, на который нужно поставить лайк |
  | IDСтены | Строка/Число (по умолчанию группа из стандартных настроек) | Группа или профиль, на котором находится пост (ID стены сообщества пишется со знаком '-' !) |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|
  
___

* __Репост | Метод: СделатьРепост()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | IDПоста | Строка/Число | Номер поста, который нужно репостнуть |
  | IDСтены | Строка/Число (по умолчанию группа из стандартных настроек) | Группа или профиль, на котором находится пост (ID стены сообщества пишется со знаком '-' !) |
  | ЦелеваяСтена | Строка/Число | Стена, на которую нужно репостнуть запись. Если пусто - репостит на стену текущего пользователя |
  | Рекламный | Булево | Пометить как рекламу |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|
  
___


* __Написать комментарий от имени сообщества | Метод: НаписатьКомментарий()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | IDПоста | Строка/Число | Номер поста, который нужно репостнуть |
  | IDСтены | Строка/Число (по умолчанию группа из стандартных настроек) | Группа или профиль, на котором находится пост (ID стены сообщества пишется со знаком '-' !) |
  | Текст | Строка | Текст сообщения |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

  ***Попытка отправить комментарий от недавно созданной или малочисленной группы может привести к ошибке "Access denied: could not access to this community"***
___

* __Отправить сообщение пользователю от бота | Метод: НаписатьСообщение()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | Текст | Строка | Текст сообщения |
  | IDПользователя | Строка/Число | ID пользователя, который может быть получен при приеме сообщения от пользователя боту* |
  | communitytoken | Строка | access_token сообщества (см. Получение необходимых данных) |
  | Клавиатура | Строка | Описание кнопок для бота. Может быть получена методом *СформироватьКлавиатуру()*  |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

  ***Боты могут отправлять сообщения пользователю только если тот написал первым***
___

* __Сформировать простейшую клавиатуру из массива кнопок | Метод: СформироватьКлавиатуру()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | МассивКнопок | Массив строк* | Массив строк, каждый элемент которого обозначает надпись на кнопке в интерфейсе чат-бота |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|

  ***Нажатия на клавиатуру можно будет идентифицировать по этим же строкам***
___
</details>




<details>
<summary>Сервисные методы</summary>
<br>
Небольшие методы-инструменты от ВК


* __Сокращение ссылки | Метод: СократитьСсылку()__
  
  | Параметр | Тип | Назначение |
  |-|-|-|
  | URL | Строка | Адрес для сокращения |
  | Параметры | Структура (по умолчанию нет) | Параметры / перезапись стандартных параметров (см. Получение необходимых данных)|
  
</details>

___


>В проекте используется механизм распаковки zip и gzip vbondarevsky/Connector
>
>Copyright 2017-2023 Vladimir Bondarevskiy
>под Apache License, Version 2.0
>
>https://github.com/vbondarevsky/Connector/
>
>Остальной проект распространяется под лицензией MIT
>Модуль Инструменты данной библиотеки совместим с подобным модулем билиотеки TelegramEnterprise для интеграции с Telegram <br><br>
>https://github.com/Bayselonarrend/TelegramEnterprise


<br>

>![Infostart](https://infostart.ru/bitrix/templates/sandbox_empty/assets/tpl/abo/img/logo.svg)
>
>Статья на Инфостарте: [https://infostart.ru/1c/articles/1946543/](https://infostart.ru/1c/articles/1946543/)
