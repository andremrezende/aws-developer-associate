## EBS (Elastic Block Store) Volume

* É um drive de rede (não um driver físico) que pode ser anexo para suas instâncias em quanto elas rodam
* Permite as instâncias persistir dados, mesmo após termino das mesmas
* POode ser usado somente para montar por uma instância por vez (no nível de CCP)
* Elas são definidas em uma zona de disponibilidade especifica
* Pensar como um Pendrive(USB)
* Camada grátis: 30 GB de storage de EBS do tipo General Purpose (SSD) ou Magnetic por mês
* Usado para comunicação com a instância, que significa menos latência
* Pode ser desanexo de uma instância EC2 e anexa a outra
* Disponível somente em uma Zona de Disponibilidade (AZ)
  * Um Volume EBS em us-east-1a não pode ser anexo em us-east-1b
  * Para mover um volume, primeiro criar um snapshot dele
* Provém capacidade provisionada (tamanhos em GBs e IOPS)
  * É cobrado por toda capacidade provisionada
  * Pode aumentar a capacidade durante o tempo

### Atributo de Exclusão na Terminação

* Por padrão o volume root do EBS é excluído (atributo habilitado)
* Por padrão, qualquer volme de EBS não é excluído (atributo desabilitado)
* Pode ser controlado pelo console da AWS / AWS CLI
* **Caso de uso: preservar o volume root em caso de terminar a instância **

### Snapshots

* Realizar um backup do seu volume de EBS daquele ponto do tempo
* Não é necessário desanexar o volume para criar um snapshot, mas recomendado
* Pode copiar snpashost em diferentes regiões ou AZ

## AMI

* AMI = Amazon Machine Imae
* AMI são customizadas para uma instância EC2
  * Adicionar seu próprio software, configuração, sistema operação, monitoramento, ...
  * Boot rápido / tempo de configuração por causa de todos os softwares pré-empacotados
  * AMI são construídos para uma região especifica (e pode ser copiado através de regiões)
  * Pode executar instância EC2 de:
    * Um AMI Publico: Provido pela AWS
    * Sua própria AMI: você cria e mantém ela
    * Um AMI do AWS MArketplace: uma AMI que outra pessoa criou (e potencialmente vende)

## EC2 Instance Store

* Volumes EBS são drivers de rede e bons, mas limitados em performance
* Se existir necessidade de alta-performance de disco físico, utilize **EC2 Instance Store**
* Melhor performance de I/O
* Perde armazenamento ao pausar a instância (ephemeral)
* Bom para buffer/ cache/ scratch data/ conteúdo temporário
* Rico de perder os dados caso exista falhar de hardware
* Backups e Replicações ficam como sua responsabilidade

## Tipos de Volumes

* Volumes EBS possuem 6 tipos:
  * gp2 / gp3 (SSD): Propósito genérico de volume SSD que balanceia o custo e performance para vários tipos de cargas de trabalho
  * io1 / io2 (SSD): Alta performance de volume SSD para missão critica de baixa latência ou alta carga de throughput
  * st1 (HDD): Baixo custo de volume de HDD apropriado para acesso frequente, carga de througput intenso
  * sc1 (HDD): Baixo custo de volum de HDD para acesso infrquente de carga de trabalho
* Volumes EBS são caracterizados em Tamanho | Throughput | IOPS (I/O Ops Per Sec)
* Somente gp2/gp3 e io1/io2 podem ser usados com volumes de boot

### Casos de Uso

#### General Purpose SSD

* Efetividade de custo e baixa latência

* Boot de volume de sistema, virtual desktop, desenvolvimento e ambientes de testes

* 1 GiB - 16 TiB

* gp3: 

  * 3.000 IOPS e throughput de 125MiB/s
  * POde aumentar IOPS para 16.000 e throughput para 1000 MiB/s independetemente

* gp2:

  * Menos volume de gp2 e burst IOPS de 3.000
  * Tamanho do volume e IOPS são lincados, max IOPS é de 16.000
  * 3 IOPS per GB, significa 5.334 GB que são o máximo de IOPS

  #### IOPS Provisionado (PIOPS) SSD

  * Aplicação de missão críticas e sustentadas pela performance de IOPSS
  * Ou aplicação que precisão de mais do que 16.000 IOPS
  * Bom para cargas de trabalhos de banco de dados (armazenamento sensitivo e consistente)
  * io1/io2 (4GiB - 16TiB):
    * Max PIOS: 64,000 para instância EC2 Nitro & 32.000 para outras
    * Pode aumentar PIOPS independentemente do tamanho de armazenamento
    * io2 pode ter durabilidade e mais IOPS per GiB (pelo mesmo preço que o io1)
    * io2 Block Express (4 GiB - 64TiB):
    * Latência em submilisegundos
    * Max PIOPS: 256.000 com um IOPS: GiB de ratio de 1.000:I
    * Suporta EBS Multi-attach

  #### Disco Rígido (HDD)

  * Não podem ser volumes de boot
  * 125 MiB para 16 TiB
  * Throughput otimizado para HDD (st1)
    * Big Data, Data Warehouses, Processamento de Log
    * Max throughput: 500 MiB/s - max IOPS 500
  * Cold HDD (sc1):
    * Para dados que tem acesso infrequente
    * Cenários onde o menor curso é importante
    * Max throughput 250 MiB/s - max IOPS: 250

## EBS Multi-Attach - io1/io2 family

* Anexar o mesmo volume de EBS para multiplas instâncias de EC2 na mesma AZ
* Casa instância pode ter acesso completo de leitura/escrita para o volume
* Caso de Uso:
  * Arquivamento de alplicações de alta disponibilidade em um cluster Linux  (ex: Teradata)
  * Aplicação devem gerencias as operações concorrênte de escrita
* Deve usar um sistema arquivo cluster-aware (não XFS, EX4, etc...)