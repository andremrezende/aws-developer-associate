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