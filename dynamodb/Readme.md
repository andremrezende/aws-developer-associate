# DynamoDB

## NoSQL Database

* NoSQL Banco de dados são não relacionais e são **distribuidos**
* NoSQL Banco de dados incluem MongoDB, DynamoDB, etc...
* NoSQL banco de dados não suportam *Joins*
* Todos dados para uma consulta/*query* estão presentes em uma linha
* **NoSQL** banco de dados podem ser escalados Horinzontalmente
* Não existe "certo" ou "errado" para NoSQL vs SQL, eles são modelos de dados diferentes e usam *queries* de forma diferente

## DynamoDB

* Altamente gerencialvel, alta disponibilidade com replicação através das 3s AZs
* NoSQL banco de dados - não relacional
* Escala com trabalhos massivos de forma distribuida
* Milhões de requisições por segundo, trilhões de linhas, 100s de TB de armazenamento
* Rápido e consistente em performance (baixa latência para recuperar dados)
* Integrado com IAM por segurança, autorização e administração
* Ativa a programação orientada a eventos com DynamoDB Stream
* Baixo custo e capacidade auto escalonamento

### Basics

* Trabalha com tabelas
* Cada tabela tem uma PK (deve ser decidida ao criar uma tabela)
* Cada tabela por ter infinito número de linhas (=row)
* Cada item tem atributos (Adicionado ao longo do tempo ou podem ser vazios/*nulos*)
* Tamanho máximo de cada item é de 400KB
* Tipos de dados suportados
  * Scalar Types: String, Number, Binary, Boolean, Null
  * Document Types: List, Map
  * Set Types, String Set, Number Set, Binary Set

### Primary Keys

* Opção 1: Partion Key somente (HASH)

  * Deve ser unico para cada item

  * Deve ser "diverse" para que o dado possa ser distribuido

    **Exemplo:** user_id para una tabela de usuarios

* Opção 2: Partion Key + Sorte Key

  * A combinação deve ser única

  * Dados são agrupados por Partion Key

  * Sorte Key == Range key

    **Exemplo:** tabela jogos_usuario

    * id_usuario para Partion Key
    * id_jogo para Sort Key

## DynamoDB - Taxa de transferência provisionada

* Tabela deve ter provisionamento de capacidade de unidades para leitura e escrita
* Unidade de Capacida de Leitura (RCU): taxa de transferência para leitura
* Unidade de Capacidade de Escrita (WCU): taxa de transferência para escrita
* Opção para configurar o escalonamento automático da taxa de transferência para atender à demanda
* O rendimento pode ser excedido temporariamente usando "crédito estourado/burst credit"
* Se o crédito estourado/burst credit estiver vazio, acontecerá um erro de "ProvisionedThroughputException"
* Para esse caso de "ProvisionedThroughputException" é aconcelhável realizar uma nova tentativa com exponentilal back-off retry

### WCU

* Uma unidade que reprecidade a capacidade de uma escrita por segundos de um item é de tamanho de 1KB
* Se um item é maior do que 1KB, mais WCU serão consumidas
  * **Exemplo 1:** escrever 10 objetos por segundo de 1KB cada, item é > 2, então 2 * 10 = **20 WCU** serão necessárias
  * **Exemplo 2:** escrever 6 objetos por segundo de 4.5 KB cada, item é > 6 * 5 (***arredondado 4.5***) = **30 WCU** serão necessárias
  * **Exemplo 3:** escrever 120 objetos por minuto de 2 KB cada, (120/60) = 2 * 2 = **4 WCU** serão necessários

### Leitura Fortmente Consistente  VS Eventualmente Consistente

* **Eventualmente Consistente**: Se nós formos ler após escrita, possivelmente será retornado algo inesperado, devido a replicação
* **Fortemente Consistente:** Se nós formmos ler após escrita, irá obter o dado correto
* **Por padrão:** DynamoDB utiliza de ***Consistencia Eventual***, porém ***GetItem***, ***Query & Scan*** provém um parametro ***"ConsistentRead"*** que poderá ser alterado para **True**.

### RCU

* Uma Capacidade de Unidade de Leitura (RCU) representa **uma** ***leitura fortemente consistente*** por segundo, or **duas** ***leituras eventualmente consistentes*** por segundo, para um item de tamanho de ***4KB***.
* Se um item tem mais do que ***4KB***, mais RCU serão consumidas
* **Exemplo 1:** 10 strongly consistent reads per second of 4 KB each > 10 * (4/4) = **10 RCU** 
* **Exemplo 2:** 16 eventually consistent reads per second of 12 KB each > (16 * (12/4)) / 2 = (16 * 3) / 2 =  **24 RCU**
* **Exemplo 3:** 10 strongly consistent reads per second of 6 KB each > 10 * (6/4) = 10 * 2 (rounded) = **20 RCU**

## Partições internas

* Dado é divido em partições
* Chave das partições passam por um algoritmo de hashing interno que conhece para qual partição o item deverá ser inserido
* Para caclular o número de partições
  * Por capacidade: (TOTAL RCU / 3000) + (TOTAL WCU / 1000)
  * Por tamanho: Tamanho Total / 10 GB
  * Total de partições = CEILING(MAX(Capacidade, Tamanho))
* **WCU  e RCU são espalhados claramento por partições**

## Throttling

* Se excedermos RCU ou WCU será gerado erro **ProvisionedThroughputExceededExceptions**
* Razões:
  * Partições Quentes
  * Item muito grande: RCU e WCU depende do tamanho do item
  * Chaves Quentes: Uma chave de partição está sendo lido muitas vezes (itens populares)
* Soluções:
  * Exponential back-off quando uma exception for encontradada (pronto na SDK)
  * Distribuir chave de partições tanto for possível
  * Se for erro de RCU, pode-se utilizar o DynamoDB Accelerator (DAX)

# API Básica

## Escrevendo

* **PutItem** - Escreve um dado no DynamoDB (cria ou altera todos valores)

  * Consome WCU

* **UpdateItem** - Alterado um item no DynamoDB (alteração parcial de valores)

  * Possibilidade de uso de Contadores Atomicos e aumenta-los

* **Escrita Condicional**:

  * Aceita uma escrita / alteração somente se as condições forem respeitadas, caso contrário irá rejeitar
  * Ajuda no acesso concorrênte para itens
  * Sem impacto de performance

  

## Excluindo

* **DeleteItem**
  * Exclui uma linha individualmente
  * Possibilidade de adicionar exclusão condicional
* **DeleteTable**
  * Exclui toda a tabela e seus respectivos itens
  * Mais rápido que exclusão de DeleteItem sobre todos itens

## Excrita em Lote

* **BatchWriteItem**
  * Acima de 25 **PutItem** e/ou **DeleteItem** em uma chamada
  * Acima de 16MB de dados escritos
  * Acima de 400KB de dados por item
* Evita latência reduzindo o número de chamadas a API ao DynanoDB
* Operações são realizadas em paralelo para melhor eficiência
* É possível que parte dos itens falhe, nesse caso terá que realizar um retry aos itens falhados (utilizando o algoritimo de exponential back-off)

## Lendo Dados

* **GetItem**
  * Leitura baseado no Primary Key
  * PK = Hash ou HASH-RANGE
  * Consistência eventual por padrão
  * Opção de uso de leitura fortemente consistênte (mais RCU - pode demorar um pouco mais)
  * ***ProjectionExpresion*** pode ser usado para especificar para incluir somente certos atributos
* **BatchGetItem**:
  * Acima de 100 itens
  * Acima de 16MB de dados
  * Itens são obtidos em paralelo para minimizar a latência

## Query

* **Query** retorna items baseados em:
  * Valores de PartionKey (**deve possuir operador *=* **)
  * Valores de SortKey (=, <, <=, >, >=, Between, Bengin) - opcional
  * Para filtros futuros FilterExpression (filtro realizado no lado do cliente)
  * **Retornos**
    * Acima de 1MB de dados
    * Um número de items especificados no **Limit**
  * Possibilidade de paginar resultados
  * Pode realizar consultar(**query**) em tabelas, em index secundários locais, ou um indice secondario global

## Scan

* ***Scan*** ineficiente, pois irá retornar todos os valores da tabela
* Retorna acima de !MB de dados -> utilize paginação
* Consome muito do RCU
* Limitar utilizando **Limit** ou reduzindo o tamanho da página de resultado
* Para performance rápidas, utilize ***parallel scan**:
  * Multiplas instancias 'scan' multiplas partições ao mesmo tempo
  * Aumenta o "througput"  e consumo de RCU
  * Limita o impacto de "scans" paralelos
* Pode ser usar ***ProjectionExpression*** + ***FilterExpression*** (não altera RCU)

### LSI (Indice Secundário Local)

* Chave de intervalos alternativas para a sua tabela, com hash key
* Até **5 ** índices secundários locais por tabela
* A chave de classificação consiste exatamente em um atributo escalar
* O atributo que você escolher deve ser scalar String, Number ou Binary
* LSI deverá ser definido ao criar a tabela

### GSI (Indice Secundário Global)

* Para acelerar as consultas sobre atributos não-chave, use um GSI
* GSI = partion key + optional sort key
* O índice é uma nova "tabela" e podera projetar atributos nela
* A partion key e a sort key são sempre projetadas (KEYS_ONLY)
* Pode projetar atributos extras para proteger (INCLUDE)
* Pode utilizar todos os atributos da tabela principal (ALL)
* Deve definir RCU / WCU para o indice
* Possibilidade de adicionar / modificar GSI (não LSI)

## Índices e Throttling

### GSI:

* **Se a escrita for "throttled" na GSI, a tabela principal irá ser "throttled"**
* Mesmo se o WCU da tabela principal estiver correto
* Escolhar sua chave de partição GSI com cuidado!
* Atribuir sua capacidade de WCU com cuidado

### LSI:

* Utiliza de WCU e RCU da tabela principal
* Nenhuma consideração especial para thottling

## Concorrência

* DynamoDB tem uma funcionalidae especial chamada "Conditional Update / Delete"
* Que significa que você pode garantir que um item não mudou antes de altera-lo
* Isso torna o DynamoDB um banco de dados com ***optimistic locking / concurrency***

## DAX

* DAX = DynamoDB Accelerator

* Não é necessário reescrever a aplicação para cachear

* As gravações passam do DAX para o DynamoDB

* Latência de microssegundos para leitura e consultas em cache

* Resolve o problema de Hot Key(muitas leituras)

* 5 minutos de cache por padrão

* Até 10 nós no cluster

* Multi AZ (míimo de 3 nós recomendados para produção)

* Seguro (criptogratia em repouso com KMS, VPC, IAM, ClouTrail, ...)

  

### DAX  VS ElasticCache

* DAX é melhor localizado por objetos individuais no cache, Query / Scan cache
* ElasticCache é melhor para utilização avançadas, como Agregação de Resultados (Aggregation Results)

## Streams

* Alteração no DynamoDB (CUD) podem acabar em Stream
* Esse Stream pode ser lito por Lambda & instancias EC2:
  * Reações para alterar em tempo real (email de boas vindas para novos usuários)
  * Analytics
  * Inserir no ElasticSearch
  * Criar tabelas / views derivadas
* Pode implementar replicação por região cruzada
* Possui 24h de retenção
* Escolha as informações que serão gravadas no fluxo sempre que os dados na tabela forem modificados:
  * KEYS_ONLY - Somente os atributos chaves foram modificados
  * NEW_IMAGE - O item inteiro, ele aparecerá após estar modificado
  * OLD_IMAGE - O item inteiro, ele aparecera antes de estar modificado
  * NEW_AND_OLD_IMAGES - Ambos os novas e velhas images do item
* São feitos de shards, como o Kinesis Data Streams
* Você não provisiona shards, é realizado pela AWS
* ***Os registros não são populados retroativamente após estar habilitado***

### Streams & Lambda

* Precisa definir um ***Event Source Mapping*** para ler do Streams
* Precisa garantir que a função Lambda tem a permissão apropriada
* ***A Lambda é invocada sincroniamente***
* 