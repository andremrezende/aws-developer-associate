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

* Varáveis de estágio são como variáveis de ambiente para API Gateway
* Utilize para alterar com frequencia valores de configurações
* Elas são usadas em:
  * Funções lambda ARN
  * Endpoint HTTP
  * Templates de parametros de mapeamento
* Casos de Uso:
  * Configurar endpoints HTTP para seus estágios como(dev, test, homolog, prod, ...)
  * Passar parâmetros de configuração para o objeto "context" na Lambda

## Deploy Canary

* Possibilidade de habilitar deploys do tipo canar para qualquer estágio (normalmente produção)
* Escolher a % do tráfefo para canais que irão receber
* Logs & Métricas são separados (para melhor monitaramento)

## Tipos de Integração

* Tipo **MOCK**
  * API Gateway sempre retonar uma resposta sem enviar uma requisição para o backend
* Tipo **HTTP / AWS (Lambda & AWS Services) **
  * Você deve configurar em ambas na requislção de integração e da resposta de integração
  * Configurar o mapeamento de dados usando ***mapping templates*** para a requisição e resposta
* Tipo **AWS_PROXY (Lambda Proxy)**
  * Requisições encaminhadas de um cliente é entrada para Lambda
  * A função é responsável pela lógica de *"request / response"*
  * **Sem template de mapeamendo, cabeçalho, parametros string na query... são passados como argumentos **
* Tipo **HTTP_PROXY**
  * Sem template de mapemando
  * A requisição HTTP é passado para o backend
  * A resposta HTTP do backend é repassado por API Gateway

## Templates de Mapeamento (AWS & Integração HTTP)

* Mapeamento são usados para modificar requisições e repostas
* Renomear / Modificar parametros strings da query
* Moficar conteúdo do corpo
* Adicionar **"Headers"**
* Usado para Velocity Template Language (VTL): para iterações, if, etc...
* Filtrar saída de resultados (remover dados desnecessários)

## API Gateway Swagger / Open API spec

* 