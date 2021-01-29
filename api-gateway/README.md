# API Gateway

* AWS Lambda + API Gateway: Sem gerenciamento de infraestrtutura
* Suporte para Protocolo Websocket
* Tratamento de verionamento de API (v1, v2, ...)
* Tratamento para diferentes ambientes (dev, homolog, prod, ...)
* Tratamento de segurança (Autenticação e Autorização)
* Criar chaves de API, tratamento de throttling de requisições
* Importação de Swagger / Open API  para definição de APIs
* Transformação de validação de requisições e respostas
* Gerador de SDK e especificações de API
* Repostas de API cacheadas

## Alto nível de integrações

* Funções Lambda
  * Invocar função lambda
  * Fácil para expor a API REST por AWS Lambda
* HTTP
  * Export endpoints HTTP
  * Exemplo: HTTP API interno on premises, Load Balancer de Aplicação, ...
  * Porque? Adicionar taxa limite , cacheamento, autenticação de usuários, Chave de API, etc...
* Serviço AWS
  * Expoem qualquer API da AWS através de um API Gateway
  * Exemplo: iniciar uma função workflow AWS Step, postar uma mensagem para SQS
  * Porque? Adicionar autenticação, publicação, controle de taxa, etc...

## Tipos de Endpoints

* Edge-Optimized(padrão): Para clientes Globais
  * Requisições são roteadas atrvés do CloudFront Edge Locations (melhora latência)
  * A API Gateway ainda funciona somente em uma região
* Regional
  * Para clientes dentro da mesma região
  * Nuvem manualmente combinada com CloudFront (mais controle sobre as estratégias de cache e distribuição)
* Privado
  * Pode ser acessado somente dentro da sua VPC usando um interface de endpoint de VPC (ENI)
  * Usa uma politica de recursos para definir o acesso

## Estágios de Deploy

* Alterações na API Gateway não significa que estão funcionando
* É necessário realizar um "deploy" para que funcione as novas alterações
* Alterações são implantadas para "Stages" (como tanto for necessário)
* Cada estágio tem sua própria configurações de parâmetros
* Estágios pode sobre "rollback" com o histórico de "deploys" mantidos

### Variáveis de estágio