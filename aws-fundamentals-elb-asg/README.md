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