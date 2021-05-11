## **AWS CI/CD: CodeCommit, CodePipeline, CodeBuild, CodeDeploy**

- AWS CodeCommit: Armazenar nosso código
- AWS CodePipeline: automatizar a esteira do código para o ElasticBeanstalk (?)
- AWS CodeBuild: construir e testar nosso código
- AWS CodeDeploy: deploy o código para o EC2 frotas (CloudFormation ou AWS Code Deploy) (não Beanstalk)

![image-20210511072315473](C:\Users\ared\AppData\Roaming\Typora\typora-user-images\image-20210511072315473.png)

## AWS CodeDeploy

* Deve estar com o CodeDeploy Agent executando
* Envia o arquivo *appspec.yml*
* Aplicações são baixadas do Github ou do S3
* EC2 executará as instruções de deploy
* CodeDeploy Agent reportará sucesso ou falha no deploy da instância
* Pode ser agrupado por grupos (dev / test / prod)
* **Pode ser usado Blue / Green somente em instâncias EC2 (não on premise)**
* Suporte para deploys de AWS Lambda
* Não provêm recursos, utiliza os existêntes
  * ***AWS CodeDeply Componentes Primarios:***
    * Application: nome único
    * Compute platform: EC2/ On Premises ou Lambda
    * Deployment configuration: Regras de deploy para sucesso ou falha
      * *EC2/On Premises: você pode especificar o número minimo de instâncias healthy para deploy*
      * *AWS Lambda: especificar como o trafico é roteado para atualização de versões de Lambda*
    * Deployment group: agrupar por instâncias "taguiadas" (permite realizar deploy gradualmente)
    * Deployment type: Deploy in-place or Blue/green deploy
    * IAM instance profile: precisa de permissão de EC2 para baixar do S3 ou do GitHub
    * Application Revision: código aplicação + appspec.yml
    * Service Role: Regra para CodeDeploy performar o que é necessário
    * Target revision: Destino do deploy da versão da aplicação

## AppSpec

* File section: como copiar da origem do S3 / Github para o filesystem
* Hooks: lista de instruções para deploy uma nova versão (pode ter timeouts). A ordem é:
  * ApplicationStop -> DownloadBundle -> BeforeInstall -> Install -> AfterInstall -> ApplicationStart -> **ValidateService** -> BeforeAllowTraffice -> AllowTraffic -> AfterAllowTrafic

