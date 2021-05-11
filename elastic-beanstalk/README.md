## **AWS Elastic Beanstalk**

## All at once

* Deploy rápido
* Aplicação tem downtime
* Bom para iterações rápida em ambiente de desenvolvimento
* Sem custo Adicional

![image-20210511075305268](C:\Users\ared\AppData\Roaming\Typora\typora-user-images\image-20210511075305268.png)

## Rolling

* Aplicação executa abaixo da capacidade
* Pode ser configurado o tamanho do bucket
* Aplicação executa ambas versões simultaneamente
* Sem custo adicional
* Deploy longo

![image-20210511075515635](C:\Users\ared\AppData\Roaming\Typora\typora-user-images\image-20210511075515635.png)

## Rolling with additional batches

* Aplicação executa com a capacidade

* Pode configurar o tamanho do bucket

* Aplicação executa ambas as versões simultaneamente

* Custo adicional pequeno

* O batch adicional é removida ao final do deploy

* Deploy longo

* Bom para prod

  ![image-20210511075715180](C:\Users\ared\AppData\Roaming\Typora\typora-user-images\image-20210511075715180.png)

## Immutable

* Zero downtime
* Novo código é deployado para novas instância com um ASG temporário
* Custo alto, duplica a capacidade
* Deploy longo
* Rápido rollback em caso de falhas (somente termina o novo ASG)
* Ótimo para prod

![image-20210511075909970](C:\Users\ared\AppData\Roaming\Typora\typora-user-images\image-20210511075909970.png)

## Blue / Green

* Não é uma funcionalidade direta do Elastic Beanstalk
* Zero downtime e facilidade de release
* Cria um novo ambiente "stage" e deploy a v2 ali
* O novo ambniente (green) pode ser validado independete e rollback em caso de problema
* Route 53 pode ser condigurado para politicas de dimensionamento de redirecionamento de um pouco de tráfefo para o novo ambiente "stage"
* Usando Beanstalk, "altera as URLs" quando finalizar com o ambiente de test

![image-20210511080211991](C:\Users\ared\AppData\Roaming\Typora\typora-user-images\image-20210511080211991.png)