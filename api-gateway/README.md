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

* Maneira comum de definir APIs REST, usando definição de API como código
* Importação de Swagger / Especificação OpenAPI 3.0 para o API Gateway
  * Method
  * Method Request
  * Integration Request
  * Method Response
  * AWS extensions para API Gateway e configuração para uma opção simples
* Pode exportar a API como Swagger / OpenAPI spec
* Swagger pode ser lido por YAML ou JSON
* **Usar Swagger par gerar SDK para aplicações**

## Cacheando reposta de API

* Ajuda a reduzir o número de chamadas requisitadas para o backend
* O padrão de **TTL** são de 300 segundos (*min 0s, max: 3600s*)
* Caches são definidos por segundo
* Possibilidade de sobrescrever as configurações de cache **por método**
* Opção de encriptação
* Capacidade entre *0.5GB e 237GB*
* Cache é caro, utilizado em produção, podendo não fazer sentido em ambientes de dev ou test

### Invalidação de Cache

* Habilitar para descartar todo o cache (inválidando) imediatamente
* Clientes podem invalidar cache no header: **Cache-Control: max-age=0** (com a autorização do IAM)
* Você não pode impor uma politica de InvalidateCache (ou requisitar uma autorização por checkbox no console), poist qualquer cliente pode invalidar a API cache

## Planos de Uso & API Keys

Se você quiser monetizar sua API para seus clientes

* Planos de Uso:

  * Quem acessa ou o mais estágios de API e seus respectivos métodos
  * Quanto custa e quão rápido eles podem acessa-la
  * API Keys são usados para identificar e metrificar acessos
  * Configurar limites de estrangulamento e de cota que são forçados a utilizar para cada cliente individual

* API Keys:

  * String alfanumércia para distribuir ao seus clientes
  * Ex: WBjxaASDNQwebkiasdn4198Nkasdqweu859123FN
  * Pode ser usado para controle de acesso ao plano
  * Limites de estrangulamento podem ser aplicados para API keys
  * Limites de cotas são definidos como o número máximo total de requisições

  ### Ordem correta para API Keys

  * Para configurar um plano de uso:
    1. Configurar uma ou mais APIs, configurar os métodos que requerem o uso de chave e "deployar" para um estágio
    2. Gerar ou importar uma chave para distribuir para os desenvolvedores de aplicações (clientes) quem irão utilizar a API
    3. Criar um plano de uso como desejado definindo limites de estrangulamento e cota
    4. **Associdar os estágios de API e chaves com o plano de uso**
    5. ao chamcar a API deverá fornecer no cabeçalho da requisição o **x-api-key**

  ## Logging e Rastreamento

  * CloudWatch Logs:
    * 