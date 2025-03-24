### Задание 5. Проектирование GraphQL API
При развитии сервиса управления клиентскими данными (client-info) команда столкнулась с проблемой. Потребители данных (веб-приложения и сервиса core-app) в разных сценариях продажи и обслуживания страховок могут требовать абсолютно разные данные. При этом карточка клиента у сервиса достаточно объёмная: общий атрибутивный состав достигает 500 штук. Из-за высокой вариативности набора запрашиваемых данных команда приняла решение в своём REST API предоставить множество отдельных ресурсов, с помощью которых можно запрашивать отдельные объекты данных клиента: контакты, документы, родственники и так далее.

Однако такая реализация кратно увеличивает нагрузку и RPS сервиса client-info, поскольку в рамках одного сценария могут потребоваться сразу несколько объектов данных — их придётся запрашивать по отдельности. Предоставить один ресурс для получения абсолютно всех данных клиента не представляется возможным: объём передаваемых данных будет настолько большим, что замедлит скорость взаимодействия с сервисами (особенно с веб-приложением).

Вы обсудили проблему с командой и приняли решение перевести REST API сервиса client-info на GraphQL.


### Что нужно сделать
Спроектируйте GraphQL на основании существующего контракта сервиса:
1. Проанализируйте Swagger контракт client-inf. Оцените существующую структуру API, выделите ключевые ресурсы и операции.
   Контракт Swagger
    ```
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

Когда будете сдавать решение, загрузите схему в директорию Task5 в рамках пул-реквеста.

Проанализируйте получившееся решение и подумайте, как GraphQL помогает оптимизировать взаимодействие между потребителями и client-info. Описывать свои выводы в решении не нужно, это упражнение полезно выполнить для себя.

### Решение:
В существующем REST API есть три основных ресурса:
* Client (/clients/{id}) — основная сущность, содержит базовую информацию о клиенте.
* Document (/clients/{id}/documents) — список документов клиента.
* Relative (/clients/{id}/relatives) — список родственников клиента.

В GraphQL мы можем выразить эти сущности, как:
```
type Query {
  client(id: ID!): Client
}

type Client {
  id: ID!
  name: String!
  age: Int!
  documents: [Document]
  relatives: [Relative]
}

type Document {
  id: ID!
  type: String!
  number: String!
  issueDate: String!
  expiryDate: String!
}

type Relative {
  id: ID!
  relationType: String!
  name: String!
  age: Int!
}
```

### Примеры использования:
# REST API:
Допустим нам нужно получить информацию по клиенту, по его родственникам, а также по его документам:
мы сделаем 3 запроса на разные ручки:
1. /clients/{id} - забираем данные по клиенту: (id, name, age)
2. /clients/{id}/documents - забираем данные по документам: (id, type, number, issueDate, expiryDate)
3. /clients/{id}/relatives - забираем данные по родственникам: (id, relationType, name, age)

# GraphQL:
В случае с GraphQL мы можем забрать данные по трём сущностям за один запрос:
```
query {
  client(id: "123") {
    id
    name
    age
    documents {
      id
      type
      number
      issueDate
      expiryDate
    }
    relatives {
      id
      name
      relationType
      age
    }
  }
}
```

Также, преимущество в гибкости: при необходимости мы можем убрать из запроса любую сущность или конкретное поле,
получая только нужные данные и экономя трафик и вычислительные ресурсы сервера.