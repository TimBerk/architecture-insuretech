# Задание 5. Проектирование GraphQL API

При развитии сервиса управления клиентскими данными (client-info) команда столкнулась с проблемой. Потребители данных (веб-приложения и сервиса core-app) в разных сценариях продажи и обслуживания страховок могут требовать абсолютно разные данные. При этом карточка клиента у сервиса достаточно объёмная: общий атрибутивный состав достигает 500 штук. Из-за высокой вариативности набора запрашиваемых данных команда приняла решение в своём REST API предоставить множество отдельных ресурсов, с помощью которых можно запрашивать отдельные объекты данных клиента: контакты, документы, родственники и так далее.

Однако такая реализация кратно увеличивает нагрузку и RPS сервиса client-info, поскольку в рамках одного сценария могут потребоваться сразу несколько объектов данных — их придётся запрашивать по отдельности. Предоставить один ресурс для получения абсолютно всех данных клиента не представляется возможным: объём передаваемых данных будет настолько большим, что замедлит скорость взаимодействия с сервисами (особенно с веб-приложением).

Вы обсудили проблему с командой и приняли решение перевести REST API сервиса client-info на GraphQL.

## Что нужно сделать

Спроектируйте GraphQL на основании существующего контракта сервиса:

1. Проанализируйте Swagger контракт client-inf. Оцените существующую структуру API, выделите ключевые ресурсы и операции.

```yaml
swagger: '2.0'
info:
 description: API сервиса управления клиентскими данными
 version: 1.0.0
 title: Клиентский Сервис
host: api.client-service.com
basePath: /v1
schemes:
 - https
paths:
 /clients/{id}:
   get:
     tags:
       - Клиент
     summary: Получить информацию о клиенте по ID
     description: Возвращает информацию о клиенте.
     produces:
       - application/json
     parameters:
       - name: id
         in: path
         description: ID клиента
         required: true
         type: string
     responses:
       '200':
         description: Успешный ответ
         schema:
           $ref: '#/definitions/Client'
 /clients/{id}/documents:
   get:
     tags:
       - Документы
     summary: Список документов клиента
     description: Возвращает список документов клиента по ID.
     produces:
       - application/json
     parameters:
       - name: id
         in: path
         description: ID клиента для поиска его документов
         required: true
         type: string
     responses:
       '200':
         description: Успешный ответ
         schema:
           type: array
           items:
             $ref: '#/definitions/Document'
 /clients/{id}/relatives:
   get:
     tags:
       - Родственники
     summary: Информация о родственниках клиента
     description: Возвращает информацию о родственниках клиента по ID.
     produces:
       - application/json
     parameters:
       - name: id
         in: path
         description: ID клиента
         required: true
         type: string
     responses:
       '200':
         description: Успешный ответ
         schema:
           type: array
           items:
             $ref: '#/definitions/Relative'
definitions:
 Client:
   type: object
   properties:
     id:
       type: string
     name:
       type: string
     age:
       type: integer
 Document:
   type: object
   properties:
     id:
       type: string
     type:
       type: string
     number:
       type: string
     issueDate:
       type: string
     expiryDate:
       type: string
 Relative:
   type: object
   properties:
     id:
       type: string
     relationType:
       type: string
     name:
       type: string
     age:
       type: integer
```
2. На основе анализа REST API разработайте эквивалентную схему GraphQL, которая позволит избежать дублирования за счёт гибкости в выборе запрашиваемых данных.
3. Определите сущности, их поля, а также необходимые запросы (queries), которые покроют все операции REST API.

## Решение


1) Анализ текущего REST (что покрываем)
Swagger даёт 3 операции чтения:

- `GET /clients/{id}` → `Client { id, name, age }`
- `GET /clients/{id}/documents` → `[Document]`
- `GET /clients/{id}/relatives` → `[Relative]`

Ключевые сущности: `Client`, `Document`, `Relative` и связи:
- `Client` 1—N `documents`
- `Client` 1—N `relatives`

1) Эквивалентная GraphQL схема (SDL)

```graphql
"""
Корневые запросы сервиса client-info.
Эквивалентно REST:
- GET /clients/{id}
- GET /clients/{id}/documents
- GET /clients/{id}/relatives
"""
type Query {
  """
  Получить клиента по ID (эквивалент GET /clients/{id}).
  """
  client(id: ID!): Client

  """
  Получить документы клиента по ID (эквивалент GET /clients/{id}/documents).
  Полезно, если нужны только документы без базового профиля.
  """
  clientDocuments(clientId: ID!): [Document!]!

  """
  Получить родственников клиента по ID (эквивалент GET /clients/{id}/relatives).
  Полезно, если нужны только родственники без базового профиля.
  """
  clientRelatives(clientId: ID!): [Relative!]!
}

"""
Клиент. В реальном сервисе у него сотни атрибутов, но GraphQL позволяет
выбирать только нужные поля.
"""
type Client {
  id: ID!
  name: String
  age: Int

  """
  Документы клиента (эквивалент GET /clients/{id}/documents),
  но через вложенность можно забрать вместе с базовыми данными клиента.
  """
  documents: [Document!]!

  """
  Родственники клиента (эквивалент GET /clients/{id}/relatives).
  """
  relatives: [Relative!]!
}

type Document {
  id: ID!
  type: String
  number: String
  issueDate: String
  expiryDate: String
}

type Relative {
  id: ID!
  relationType: String
  name: String
  age: Int
}
```

3) Примеры запросов (для самопроверки)

### Эквивалент `GET /clients/{id}`
```graphql
query ($id: ID!) {
  client(id: $id) { id name age }
}
```

### Эквивалент `GET /clients/{id}/documents`
```graphql
query ($id: ID!) {
  clientDocuments(clientId: $id) { id type number }
}
```

### Всё в одном запросе (что и решает проблему “много REST вызовов”)
```graphql
query ($id: ID!) {
  client(id: $id) {
    id
    name
    documents { id type number }
    relatives { id relationType name }
  }
}
```
