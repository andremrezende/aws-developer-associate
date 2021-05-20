# RDS

* Usa serviço de banco de dados relacionais
* É serviço de gerênciamento para DB que utiliza SQL como linguagem de consulta
* Permite que crie banco de dados na nuvem e gerênciado pela AWS
  * Postgres
  * MySQL
  * MariaDB
  * Oracle
  * Microsoft SQL Server
  * Aurora (Banco de dados proprietário da AWS)

## Vantagens de usar RDS ao invés de DB no EC2

* RDS é serviço gerenciado:
  * Automação provisiona, aplicar patches de SO
  * Backup continuo e restauração por um timestamp especifico (Ponto no Tempo para Restaurar)
  * Dashboard de monitoramento
  * Replica de leitura para melhorar a performance
  * Multi AZ configurado para DR (Disaster Recovery)
  * Janelas de manutenção para upgrades
  * Capacidade de escalonamento (vertical e horizontal)
  * Armazenamento por EBS (gp2 ou io1)
* Mas não pode utilizar SSH para suas instâncias

## Backups

* São automaticamente habilitados 
* Automação de backups:
  * Backup completo diário do banco de dados (durante a janela de manutenção)
  * Transações são logados pelo RDS a cada 5 minutos
  * Habilidade de restauração de qualquer ponto no tempo (do mais velho para 5 minutos atrás)
  * 7 dias de retenção (pode ser aumento para 35 ou desabilitado)
* DB Snapshots:
  * Manualmente acionados pelo usuário
  * Retenção para o mais longo que você deseja

## Escalonamento de Auto Armazenamento

