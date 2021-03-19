## AWS AppSync

* **AppSync** é um serviço gerenciado que utiliza de **GraphQL**
* **GraphQL** tornar fácil as aplicações obterem exatamento os dados que elas precisam
* Isso include dados combinados de um ou mais fontes(sources)
  * Armazenamento de dados NoSQL, Relational databases, HTTP APIs, etc...
  * Integra com DynamoDB, Aurora, Elasticearch & outros
  * Fontes customizadas com AWS Lambda
* Obtém dados em  **real-time com WebSocket ou MQTT no WebSocket**
* Para aplicações mobile: acesso ao local data & synchronização de dados
* Tudo começa com um upload de um esquema **GraphQL**

### Segurança - AppSync

* Existe quatro jeitos que pode autorizar as aplicações para iteragir com seu AWS AppSync GraphQL API
  * API_KEY
  * AWS_IAM: IAM users / roles / cross-account access
  * OPENID_CONNECT: OpenID Connect provider / JSON Web Token
  * AMAZON_COGNITO_USER_POOLS
* Para domininios customizados & HTTPS, utilizar CloudFront na frente do AppSync