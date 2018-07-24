### API

Формат ответа:

```json
{
  "success": "true|false",
  "error": "Код ошибки если success = false",
  "data": "Отсутствует или объект/массив данных если success = true"
}
```

### Аутентификация

```
JSON /api/auth/
```

В первом запросе отправляем сертификат пользователя и простую подпись этого сертификата в форматах base64:

```json
{
  "cert": "base64",
  "sign": "base64"
}
```

Сервис проверяет сертификат, корректность подписи и возвращает объект с данными для подтверждения аутентификации:

```json
{
  "success": true,
  "data": {
    "nonce": "bOx8uVvyKAKo4f/cBXHPeEE2SFyqkMjutU2z5Q34nFM=",
    "token": "99b082c6-979b-4646-9e85-60f30b1c37a7"
  }
}
```

#### Подтверждение аутентификации

```
JSON /api/confirmAuth/
```

Для подтверждения надо подписать nonce простой подписью и отправить второй запрос:

```json
{
  "token": "99b082c6-979b-4646-9e85-60f30b1c37a7",
  "sign": "BRZjB74opJptKg4Mv5DYCmot+xdtiWRcH/rgSx+WViypiicgfV3HaZ5iI9561LpbikFGTUfZmpFBwYIlAmyCcg=="
}
```

Ответ:

```json
{
  "success": true,
  "data": {
    "needRegisterPhone": true
  }
}
```

### Регистрация номера телефона

```
JSON /api/registryPhone/
```

Если после успешной аутентификации свойство `needRegisterPhone = true` необходимо зарегистрировать номер мобильного телефона с подтверждением кодом из СМС.

```json
{
  "token": "99b082c6-979b-4646-9e85-60f30b1c37a7",
  "phone": "+7 (123) 123-45-67"
}
```

В ответ получаем идентификатор код подтверждения:

```json
{
  "success": true,
  "data": {
    "id": 1234
  }
}
```

#### Подтверждение телефона

```
JSON /api/confirmPhone/
```

```json
{
  "token": "99b082c6-979b-4646-9e85-60f30b1c37a7",
  "id": 1234,
  "code": 987654
}
```

### Профиль

```
JSON /api/profile/
```

Запрос:

```json
{
  "token": "99b082c6-979b-4646-9e85-60f30b1c37a7"
}
```

Ответ:

```json
{
  "success": true,
  "data": {
    "pid": 451,
    "cid": 32,
    "type": 1,
    "name": "ФИО",
    "company": "Название организации|Индивидуальный предприниматель",
    "title": "Должность",
    "inn": "770123456789",
    "ogrn": "1234567890123",
    "snils": "12345678901",
    "phone": "79991234567",
    "photo": "base64:AAAAA=",
    "billing": {
      "balance": 299.00
    },
    "certificates": [{
      "id": 22,
      "cert": "base64"
    }]
  }
}
```

### Контакты

```
JSON /api/contacts/
```

Получение списка контактов или поиск по переданным номерам `phones`. Если в результате поиска контакта он уже добавлен, то будет передан идентификатор `chatId` и объект `key` с ключом шифрования для данного контакта.

Запрос:

```json
{
  "token": "99b082c6-979b-4646-9e85-60f30b1c37a7",
  "phones": ["79991234567"]
}
```

Ответ:

```json
{
  "success": true,
  "data": [{
    "uid": 29,
    "pid": 33,
    "user": "ФИО",
    "name": "Название организации|Индивидуальный предприниматель|ФИО",
    "title": "Должность",
    "phone": "79991234567",
    "photo": "base64:AAAAA=",
    "type": 1,
    "certificates": [{
      "id": 22,
      "cert": "base64"
    }],
    "chatId": "91ec6ed7-aced-4251-bb01-55debd90ddb6",
    "chatType": 0,
    "chatName": "Название группы или канала",
    "key": {
      "id": 123,
      "key": "base64",
      "import": false,
      "cert": "base64"
    }
  }]
}
```

После получения списка контактов в случае поиска и если `chatId` и `key` равны `null` (контакт не добавлен в чат) то создаем сессионный ключ для контакта. Экспортируем ключ на все сертификаты пользователя и сертификаты контакта.

#### Новый чат (контакт)

```
JSON /api/createChat/
```

Новый чат создается перед первой отправкой сообщения и возвращает полный объект созданного чата. Чаты бывают трех типов: личный - 0, группа - 1, канал - 2.

`cid` - идентификатор сертификата, `pid` - идентификатор профиля, `key` - зашифрованный (экспортированный) ключ

```json
{
  "type": 0,
  "name": "Название группы или канала",
  "keys": [{
    "cid": 123,
    "pid": 456,
    "key": "base64"
  }]
}
```

После успешного создания чата возращает полный объект Контакта.

### Отправка сообщения

```
JSON /api/sendMessage/
```
