## **AWS Elastic Beanstalk**

## All at once

* Deploy rápido
* Aplicação tem downtime
* Bom para iterações rápida em ambiente de desenvolvimento
* Sem custo Adicional

![image](https://user-images.githubusercontent.com/4062447/117805618-8cc72300-b22f-11eb-9069-106b4e40ee91.png)


## Rolling

* Aplicação executa abaixo da capacidade
* Pode ser configurado o tamanho do bucket
* Aplicação executa ambas versões simultaneamente
* Sem custo adicional
* Deploy longo

![image](https://user-images.githubusercontent.com/4062447/117805655-98b2e500-b22f-11eb-8b9e-f159f4526fa1.png)

## Rolling with additional batches

* Aplicação executa com a capacidade
* Pode configurar o tamanho do bucket
* Aplicação executa ambas as versões simultaneamente
* Custo adicional pequeno
* O batch adicional é removida ao final do deploy
* Deploy longo
* Bom para prod

![image](https://user-images.githubusercontent.com/4062447/117805724-af593c00-b22f-11eb-9bd8-b19f0347f3f2.png)

## Immutable

* Zero downtime
* Novo código é deployado para novas instância com um ASG temporário
* Custo alto, duplica a capacidade
* Deploy longo
* Rápido rollback em caso de falhas (somente termina o novo ASG)
* Ótimo para prod

![image](https://user-images.githubusercontent.com/4062447/117805881-e2033480-b22f-11eb-931c-84e138a17669.png)

## Blue / Green

* Não é uma funcionalidade direta do Elastic Beanstalk
* Zero downtime e facilidade de release
* Cria um novo ambiente "stage" e deploy a v2 ali
* O novo ambniente (green) pode ser validado independete e rollback em caso de problema
* Route 53 pode ser condigurado para politicas de dimensionamento de redirecionamento de um pouco de tráfefo para o novo ambiente "stage"
* Usando Beanstalk, "altera as URLs" quando finalizar com o ambiente de test

![image](https://user-images.githubusercontent.com/4062447/117805918-efb8ba00-b22f-11eb-9460-9572eff7037b.png)
