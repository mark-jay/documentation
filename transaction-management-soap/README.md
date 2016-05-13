FIXME: add more stuff

Запрос для получения данных о транзакции по ее номеру
-----------------------------------------------------

Запрос:

	# request
    #!/bin/bash
	curl -XPOST -H "Content-Type: text/xml;charset=UTF-8" -d'@getTransactionRequest.xml' 'http://127.0.0.1:8080/mfs-public-soap/transaction-management'

Ответ будет следующим:

```xml
<env:Envelope xmlns:env='http://schemas.xmlsoap.org/soap/envelope/'>
<env:Header></env:Header>
<env:Body>
    <ns3:getTransactionResponse xmlns:ns2="http://www.w3.org/2000/09/xmldsig"
                                xmlns:ns3="http://www.allpay.kz/mfs/soap/TransactionManagement">
        <return>
            <errorInfo>OK</errorInfo>
            <result>
                <amount>1</amount>
                <blocked>false</blocked>
                <fromAccountNumber>fromAcc</fromAccountNumber>
                <fromUserLoginName>fromUser</fromUserLoginName>
                <status>COMPLETED</status>
                <toAccountNumber>toAcc</toAccountNumber>
                <toUserLoginName>toUser</toUserLoginName>
                <transactionNumber>31337</transactionNumber>
            </result>
        </return>
    </ns3:getTransactionResponse>
</env:Body>
</env:Envelope>
```
описание полей:

    <amount>1</amount>                               - сумма перевода
    <blocked>false</blocked>                         - заблокирована ли сумма у отправителя
    <fromAccountNumber>fromAcc</fromAccountNumber>   - номер счета отправителя
    <fromUserLoginName>fromUser</fromUserLoginName>  - логин(номер телефона) отправителя
    <status>COMPLETED</status>                       - статус транзакции
    <toAccountNumber>toAcc</toAccountNumber>         - номер счета получателя
    <toUserLoginName>toUser</toUserLoginName>        - логин(номер телефона) получателя
    <transactionNumber>31337</transactionNumber>     - номер транзакции в системе allpay


Запрос для завершения и отклонения транзакции
---------------------------------------------

Запрос:

	# request
    #!/bin/bash
	curl -XPOST -H "Content-Type: text/xml;charset=UTF-8" -d'@completeTransactionRequest.xml' 'http://127.0.0.1:8080/mfs-public-soap/transaction-management'

Ответ будет следующим:

```xml
<env:Envelope xmlns:env='http://schemas.xmlsoap.org/soap/envelope/'>
<env:Header></env:Header>
<env:Body>
    <ns3:completeTransactionResponse xmlns:ns2="http://www.w3.org/2000/09/xmldsig"
                                     xmlns:ns3="http://www.allpay.kz/mfs/soap/TransactionManagement">
        <return>
            <errorInfo>OK</errorInfo>
            <result>
                <amount>1</amount>
                <blocked>false</blocked>
                <fromAccountNumber>fromAcc</fromAccountNumber>
                <fromUserLoginName>fromUser</fromUserLoginName>
                <status>COMPLETED</status>
                <toAccountNumber>toAcc</toAccountNumber>
                <toUserLoginName>toUser</toUserLoginName>
                <transactionNumber>31337</transactionNumber>
            </result>
        </return>
    </ns3:completeTransactionResponse>
</env:Body>
</env:Envelope>
```
Описание полей аналогично запросу getTransactionRequest.xml
