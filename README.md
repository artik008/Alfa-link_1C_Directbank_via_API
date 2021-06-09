# Инструкция для настройки и тестирования Альфа-Линк - 1С:Директбанк через API

[TOC]

## 1. Скачивание архива со всеми необходимыми файлами

По [ссылке](https://github.com/artik008/Alfa-link_1C_Directbank_via_API/archive/refs/heads/main.zip) можно скачать архив с необходимыми для тестирования файлами.

**Архив содержит:**

- Данную инструкцию по подключению (README.md и в формате pdf)

- Файл с тестовым проектом для Postman (1C-DirectBank Тестирование.postman_collection.json)

- Папка с примерами документов в формате xml (examples)

## 2. Стандартный алгоритм взаимодействия

1. **POST /API/v1/directbank/Logon** - создание сессии для отправки и получения пакетов. В ответ возвращается xml документ с идентификатором сессии в поле SID. Время жизни сессии - 600 сек. Идентификатор сессии необходим при любых отправке/получении документов.

2. **POST /API/v1/directbank/SendPack** - отправка пакета документов. В хедере SID указывается идентификатор сессии. XML пакета помещается в body запроса. 

3. **GET /API/v1/directbank/GetPackList** - получить список пакетов. В ответ возвращается список идентификаторов непрочитанных пакетов пользователя. После отправки документа первым придёт пакет с результатом отправки («Принят»/Сообщение об ошибке)

4. **GET /API/v1/directbank/GetPack** - получить пакет по его идентификатору. Аргумент запроса id - идентификатор пакета из списка, полученного в пункте 3. 


## 3. Данные тестовой среды

**Данные для авторизации:**

| Параметр | Значение |
| ------------- |-------------|
| Сервер | https://grampus-int.alfabank.ru/API/v1/directbank |
| Логин | directBank |
| Пароль | 123456 |

**Реквизиты компании:**

| Параметр | Значение |
| ------------- |-------------|
| Краткое наименование | ООО "Тест Директ Банк"
| Полное наименование | Общество с ограниченной ответственностью "Тест Директ Банк" 
| ИНН | 0329156629
| КПП | 043360081
| Номер счета | 40702810701300009144
| Банк | АО "АЛЬФА-БАНК"
| Корсчет банка | 30101810200000000593 

**Реквизиты компании получателя (для тестирования платежей):**

| Параметр | Значение |
| ------------- |-------------|
| Краткое наименование | ООО "Тест Директ банк Получатель"
| Полное наименование | Общество с ограниченной ответственностью "Тест Директ банк Получатель"
| ИНН | 0329156630
| КПП | 770101999
| Номер счета | 40702810300000019796
| Банк | АО "АЛЬФА-БАНК"
| Корсчет банка | 30101810200000000593 


## 4. Описание заголовков 

| Заголовок |	Значение | Описание |	Комментарий |
| --------- | -------- | -------- | ----------- |
| Content-Type |	application/xml |	MIME тип передаваемого контента | Все пакеты в формате xlm |
| customerid | 40702810601300003716 | Id клиента | В качестве идентификатора клиента используется один из его счетов |
| apiversion | 2.2.2 | Версия формата данных | Версия формата данных, в которых осуществляется обмен с банком (текущая рабочая версия - 2.2.2) |
| Authorization | | Данные для авторизации | Используется логин и пароль для авторизации |
| SID	| | Идентификатор сессии | Не используется в методе Logon |


## 5. Правила формирования пакетов

Формат пакетов, используемых при обмене соответствует схемам даных 1С.

Ссылка на xsd-схемы: https://github.com/1C-Company/DirectBank/blob/master/doc/xsd-scheme/readme.md

Описание полей пакета:

| Параметр  |	Значение | Описание |
| --------- | -------- | -------- |
| Packet@id	| Новый id	| Уникальный идентификатор транспортного пакета |
| Packet@CreationDate	| dateTime	| Дата и время создания транспортного контейнера (системное время) |
| Packet@formatVersion	| 2.2.2	| Версия формата данных, в которых осуществляется обмен с банком (текущая рабочая версия - 2.2.2) |
| Packet@UserAgent	| Наименование системы	| Наименование системы, в рамках которых формируется документ |
| Packet.Document@id	| Новый id	| Уникальный идентификатор электронного документа |
| Packet.Document@dockind	| Тип документа	| Основные типы: 10 - платёжное поручение, 14 - запрос выписки |
| Packet.Document@formatVersion	| 2.2.2	| Версия формата данных, в которых осуществляется обмен с банком (текущая рабочая версия - 2.2.2) |
| Packet.Sender.Customer@id	| Id клиента	| В качестве идентификатора клиента используется один из его счетов. |
| Packet.Sender.Customer@Name	| Наименование организации	| Указывается полное наименование организации |
| Packet.Sender.Customer@inn	| ИНН организации	| |
| Packet.Recipent.Bank@bic	| БИК банка	| |
| Packet.Recipent.Bank@Name	| Наименование банка	| |
| Packet.Document.Data	| Данные электронного документа	| Документ кодируется в base64 |



## 6. Формирование документа в пакете на примере запроса выписки

Любой документ для отправки кодируется в строку формата base64 и помещается в поле Data пакета.

Схемы документов см. тут: https://github.com/1C-Company/DirectBank/blob/master/doc/xsd-scheme/readme.md

Описания полей запроса выписки:

| Параметр  |	Значение | Описание |
| --------- | -------- | -------- |
| StatementRequest@id	| Новый id	| Уникальный идентификатор электронного документа (должен совпадать с Packet.Document@id) |
| StatementRequest@CreationDate	| dateTime	| Дата и время создания выписки (системное время) |
| StatementRequest.Data.DateFrom	| dateTime	| Дата выписки «с» (включая указанный день) |
| StatementRequest.Data.DateTo	| dateTime	| Дата выписки «по» (включая указанный день) |
| StatementRequest@formatVersion	| 2.2.2	| Версия формата данных, в которых осуществляется обмен с банком (текущая рабочая версия - 2.2.2) |
| StatementRequest@UserAgent	| Наименование системы	| Наименование системы, в рамках которых формируется документ |
| StatementRequest.Sender@id	| Id клиента	| В качестве идентификатора клиента используется один из его счетов (должно совпадать с packet.Sender.Customer@id) |
| StatementRequest.Sender@Name	| Наименование организации	| Должно совпадать с Packet.Sender.Customer@Name |
| StatementRequest.Sender@inn	| ИНН организации	| Должно совпадать с Packet.Sender.Customer@inn |
| StatementRequest.Sender@kpp	| КПП организации| 	|
| StatementRequest.Recipent@bic	| БИК банка	| Должно совпадать с Packet.Sender.Recipent.Bank.bic |
| StatementRequest.Recipent@Name	| Наименование банка	| Должно совпадать с Packet.Sender.Recipent.Bank.Name |
| StatementRequest.Data.StatementType	| 0	| Тип выписки: 0 - Окончательная выписка; 1 - Промежуточная выписка; 2 - Текущий остаток на счете |
| StatementRequest.Data.Account	| Номер счета	| Номер счета, по которому запрашивается выписка (При тестировании необходимо использовать счёт, указанный в проекте Postman) |
| StatementRequest.Data.Bank@bic	| БИК банка	| Должно совпадать с Packet.Recipent.Bank@bic |
| StatementRequest.Data.Bank@name	| Наименование банка	| Должно совпадать с Packet.Recipent.Bank@Name |



## 7. Подписание документов

В зависимости от настроек обмена с банком документы могут иметь различные маршруты подписания (подробнее https://github.com/1C-Company/DirectBank/blob/master/doc/common-section/type-tables.md#edo-Settings_Data_CryptoParam[…]omerSignature_GroupSignatures)

На тестовом стенде по умолчанию устанавливаются следующие маршруты (в случае необходимости их можно изменить.):
- Платежное поручение – единоличная подпись/2 подписи;
- Выписка – без подписи;
- Выписка в банк – без подписи.

Электронная подпись формируется в формате PKCS #7 Detached. Подпись хранится в отдельных тегах схемы 1С (теги описаны в схеме XML-схема описания общих типов: 1C-Bank_Exch-Common.xsd). 

Для автоматического подписания документов необходимо установить и использовать следующие инструменты (**только, если нужно отправлять платежи!**):
- КриптоПро CSP с серверной лицензией (доступен пробный период 90 дней), 
- КриптоПро JavaCSP с серверной лицензией (доступен пробный период 90 дней), 
- Jdk. 

https://www.cryptopro.ru/products/csp/jcsp https://www.cryptopro.ru/products/csp/jcp/downloads 

Пример подписанного файла в payment_directbank.xml (в примерах) 

В качестве примера кода для подписи можно использовать файл PKCS7Example.java (в примерах, это официальный пример из пакета КриптоПро JavaCSP). 

Больше примеров можно найти в архиве samples-sources.jar самого пакета КриптоПро JavaCSP. 

## 8. Инструкция по работе с тестовым проектом Postman

1. Создать сессию через POST-запрос «Создать сессию». Для этого

   1.1. На вкладке «Auth» ввести логин/пароль клиента, который был выдан при подключении к тестовому стенду Альфа-Линк (по умолчанию можно тестировать на пользователе, который сохранен в проекте Postman)
 
   1.2. На вкладке «Headers» в поле «CustomerId» ввести номер счета клиента, с которым был настроен тестовый обмен с банком

   1.3. Нажать «Send» и скопировать ответ «SID»

2. Отправить документ через POST-запрос «Отправить документа». Для этого

   2.1. На вкладке «Auth» ввести логин/пароль клиента, который был выдан при подключении к тестовому стенду Альфа-Линк Линк (по умолчанию можно тестировать на пользователе, который сохранен в проекте Postman)

   2.2. На вкладке «Headers» в параметр «CustomerId» ввести номер счета клиента, в параметр «SID» = значение из 1.1

   2.3. На вкладке «Body» внести исходных xml документ (Платеж, Выписка, Ведомость в банк, Запрос статуса), созданных по схемам 1C

   2.4. Нажать «Send»

3. Получить список пакетов, которые не были получены клиентом через Get-запрос «Получить список пакетов»

   3.1. На вкладке «Auth» ввести логин/пароль клиента, который был выдан при подключении к тестовому стенду Альфа-Линк

   3.2. На вкладке «Headers» в параметр «CustomerId» ввести номер счета клиента, в параметр «SID» = значение из 1.1

   3.3. Нажать «Send» и скопировать значения идентификаторов пакетов

4. Получить значение пакета через Get-запрос «Получить значение пакета (извещения)»

   4.1. На вкладке «Auth» ввести логин/пароль клиента, который был выдан при подключении к тестовому стенду Альфа-Линк

   4.2. На вкладке «Headers» в параметр «CustomerId» ввести номер счета клиента, в параметр «SID» = значение из 1.1

   4.3. На вкладке «Params» ввести параметр из 3.3

   4.4. Нажать «Send» 
