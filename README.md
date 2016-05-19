# Техническое описание интеграции с веб магазинами.

Данный материал предназначен для демонстрации сценариев интеграции платёжной системы (ПС) AllPay, содержит описание сценариев взаимодействия, описание протоколов и примеры реализации на языках Java, PHP.

### СОДЕРЖАНИЕ

1. [ЦЕЛЬ](#ЦЕЛЬ)
1. [СЦЕНАРИЙ ПЛАТЕЖА](#СЦЕНАРИЙ-ПЛАТЕЖА)
1. [ПРИНЦИПЫ ВЗАИМОДЕЙСТВИЯ МЕЖДУ МАГАЗИНОМ И ПС ALLPAY](#ПРИНЦИПЫ-ВЗАИМОДЕЙСТВИЯ-МЕЖДУ-МАГАЗИНОМ-И-ПС-ALLPAY)
1. [ОПИСАНИЕ ФОРМАТА СООБЩЕНИЙ](#ОПИСАНИЕ-ФОРМАТА-СООБЩЕНИЙ)
 2. [WesbshopRequest](#WebshopRequest)
 2. [WebshopResponse](#WebshopResponse)
1. [ТЕХНИЧЕСКИЕ ДЕТАЛИ ВЗАИМОДЕЙСТВИЯ](#ТЕХНИЧЕСКИЕ-ДЕТАЛИ-ВЗАИМОДЕЙСТВИЯ)
1. [СЕРВИСЫ МАНИПУЛИРОВАНИЯ ТРАНЗАКЦИЯМИ](#СЕРВИСЫ-МАНИПУЛИРОВАНИЯ-ТРАНЗАКЦИЯМИ)
2. [ИСПОЛЬЗОВАНИЕ ТЕСТОВОЙ СИСТЕМЫ ALLPAY](#ИСПОЛЬЗОВАНИЕ-ТЕСТВОЙ-СИСТЕМЫ-ALLPAY)
2. [ПРИМЕРЫ ДЕМО РЕАЛИЗАЦИЙ](#ПРИМЕРЫ-РЕАЛИЗАЦИЙ)
 3. [НА ЯЗЫКЕ JAVA](#РЕАЛИЗАЦИЯ-НА-ЯЗЫКЕ-JAVA)
 3. [НА ЯЗЫКЕ PHP](#РЕАЛИЗАЦИЯ-НА-ЯЗЫКЕ-PHP)


### ЦЕЛЬ

Сервисы Allpay предназначены для того, чтобы дать возможность онлайн магазинам принимать платежи через систему Allpay. Далее описываются общие принципы взаимодействия между магазином и платежной системой (ПС) Allpay.

### СЦЕНАРИЙ ПЛАТЕЖА

1. Покупатель формирует заказ на сайте продавца
2. Покупатель выбирает систему Allpay в качестве способа платежа
3. Покупатель попадает на портал Allpay для авторизации платежа
4. Покупатель подтверждает платеж
5. Сумма электронных денег за вычетом предусмотренных комиссий переводятся с электронного кошелька Покупателя на электронный кошелек Мерчанта 
6. Allpay сообщает магазину о результате платежа посредством вызова заранее определённого веб-сервиса (URL на сайте продавца)

### ПРИНЦИПЫ ВЗАИМОДЕЙСТВИЯ МЕЖДУ МАГАЗИНОМ И ПС ALLPAY

Обмен информацией происходит через браузер пользователя и веб-сервисы. Для обмена данными используется формат XML, для всех видов передаваемых сообщений определена [соответствующая схема XSD](https://github.com/allpaykz/webshop-service-examples/tree/master/webshop-integration-keypair/src/main/resources/xsd/1.0.0). Для безопасного обмена сообщениями все XML подписываются обеими сторонами. Подпись реализована по стандарту W3C описанными в `XSD` схеме [XmlDSig](https://www.w3.org/TR/xmldsig-core/).
Для запроса на проведение платежа используется XSD схема `WebShopRequest.xsd`, для передачи результата о проведенном платеже используется XSD схема WebShopResponse.xsd. Каждое сообщение должно проходить валидацию по соответствующей XSD схеме, все обязательные поля должны быть заполнены по формату.
Используемая кодировка – `UTF-8`

### ОПИСАНИЕ ФОРМАТА СООБЩЕНИЙ

#### WebShopRequest

Пример подписанной XML приведён в файле WebShopRequest.xml. Далее будет приведено описание и назначение каждого поля.

1. `WSReq:ShopName` Название магазина, которое будет видеть Покупатель на странице подтверждения платежей. Обязательное для заполнения
2. `WSReq:InvoiceNumber` Номер заказа, поле должно быть уникальным в базе данных магазина. Обязательное для заполнения
3. `WSReq:Merchant` Составное поле, описывающее аккаунт магазина в ПС Allpay. Обязательное для заполнения 
 * `WSReq:MerchantID` Идентификатор магазина в ПС. Выдается в момент заключения договора. Обязательное для заполнения
 * `WSReq:WalletID` Идентификатор счета в ПС. На этот счет будут переводится деньги от Покупателя. Выдается в момент заключения договора. Обязательное для заполнения
4. `WSReq:Cart` Составное поле, описывающее товары в корзине Покупателя.
 * `WSReq:Items` Составное поле. Описание одной товарной позиции
 * `WSReq:Code` Код товара 
 * `WSReq:Name` Название товара 
 * `WSReq:Description` Описание товара
 * `WSReq:Quantity` Количество товара в соответствующей мере измерения
 * `WSReq:Cost` Стоимость товара
5. `WSReq:TotalCost` Общая сумма к оплате. Обязательное для заполнения
6. `WSReq:TimeOutInSeconds` Таймаут транзакции. Минимальное значение 600, максимальное 86400. Обязательное для заполнения
6. `WSReq:SuccessLink` URL Ссылка магазина, на которую попадёт Покупатель после удачного платежа. Обязательное для заполнения
7. `WSReq:FailureLink` URL Ссылка магазина, на которую попадёт Покупатель в случае неудачного платежа. Обязательное для заполнения
8. `WSReq:AutoTransaction` Автономное проведение транзакции. По умолчанию значение `false`. Если значение `true`, то ПС сама завершает транзакцию. Если значение `false`, то ПС замораживает деньги на счету Покупателя. Для завершение или отмена транзакции ПС предоставляет сервисы манупулирования транзакциями. Описание сервисов [здесь](https://github.com/allpaykz/documentation/tree/master/transaction-management-soap])
8. `Signature` Подпись. Обязательное для заполнения

#### WebShopResponse
1. `WSResp:InvoiceNumber` Номер заказа магазина. Может отсутствовать в случае ошибок в WebshopRequest полученном от магазина
2. `WSResp:Transaction` Составное поле. Описывает транзакцию
 3. `WSResp:TransactionId` Номер транзакции в ПС Allpay. По этому номеру можно проверить статус транзакции в веб интерфейсе и проводить манипуляции через [веб сервис](https://github.com/allpaykz/documentation/tree/master/transaction-management-soap]). Обязательное для заполнения
 4. `WSResp:Status` Статус транзакции. Обязательное для заполнения
 5. `WSResp:StatusDescription` Описание статуса транзакции. Обязательное для заполнения
2. `Signature` Подпись. Обязательное для заполнения

### ТЕХНИЧЕСКИЕ ДЕТАЛИ ВЗАИМОДЕЙСТВИЯ

Схема взаимодействия онлайн магазина и ПС Allpay

1. Покупатель формирует корзину на сайте магазина
2. Покупатель выбирает ПС Allpay в качестве способа оплаты 
3. Сайт магазина формирует HTTP POST запрос, и перенаправляет Покупателя на портал ПС Allpay https://mfs.all-pay.kz/mfs/. В параметры `HTTP POST` запроса должно быть указано два значения:
 * webshopResponse – Запрос в ПС от магазина, XML сообщение WebShopRequest.
 * merchantId – Уникальный номер магазина, выдается в момент заключения договора между магазином и ПС.
4. Покупатель авторизуется на портале ПС Allpay.
6. Проверяется подпись `XML`.
7. По полученным XML данным заполняется форма подтверждения платежа.
8. Покупатель подтверждает платеж.
10. ПС отправляет подписанное сообщение XML WebShopResponse на веб-сервис магазина при помощи `HTTP POST` запроса.
11. Покупатель перенаправляется на веб-страницу магазина, указанную в сообщении WebShopRequest
Со стороны магазина для интеграции требуется два веб-сервиса 1) для выдачи запросов WebShopRequest и 2) для получения WebShopResponse.
Ключи для подписи и верификации будут выдаваться ПС Allpay после подписания договора.

### СЕРВИСЫ МАНИПУЛИРОВАНИЯ ТРАНЗАКЦИЯМИ

Описание находится в [соответствующем](https://github.com/allpaykz/documentation/tree/master/transaction-management-soap) разделе.

### ИСПОЛЬЗОВАНИЕ ТЕСТОВОЙ СИСТЕМЫ ALLPAY

Для тестирования сервисов можно подключится к тестовой инфраструктуре ПС Allpay. Запрос на получение доступа к тестовому серверу отправляйте на почту techsupport@allpay.kz

### ПРИМЕРЫ РЕАЛИЗАЦИЙ

Исходный код демо проектов находится [здесь](https://github.com/allpaykz/webshop-service-examples)

##### РЕАЛИЗАЦИЯ НА ЯЗЫКЕ JAVA

Исходный код проекта [здесь](https://github.com/allpaykz/webshop-service-examples/tree/master/webshop-integration-demo). Для запуска проекта нужно склонировать репозиторий:

    $ git clone https://github.com/allpaykz/webshop-service-examples

Для сборки проекта используется `maven 3`. После успешной сборки проекта появится `war` файл `webshop-service-examples/webshop-integration-demo/target/webshop-integration-rest.war`.

##### РЕАЛИЗАЦИЯ НА ЯЗЫКЕ PHP

Исходный код проекта [здесь](https://github.com/allpaykz/webshop-service-examples/tree/master/webshop-integration-php-demo)

