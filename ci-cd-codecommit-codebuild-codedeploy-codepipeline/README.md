## **AWS CI/CD: CodeCommit, CodePipeline, CodeBuild, CodeDeploy**

- AWS CodeCommit: Armazenar nosso código
- AWS CodePipeline: automatizar a esteira do código para o ElasticBeanstalk (?)
- AWS CodeBuild: construir e testar nosso código
- AWS CodeDeploy: deploy o código para o EC2 frotas (CloudFormation ou AWS Code Deploy) (não Beanstalk)

![image-20210511072315473](C:\Users\ared\AppData\Roaming\Typora\typora-user-images\image-20210511072315473.png)

## Codecommit

* Repositórios Git privados
* Sem limite de tamanho (se escala sozinho)
* Altamente gerenciável e disponível
* Código na conta da AWS Cloud -> aumentando sergurança e compliance
* Possui segurança (Encriptacação, Controle de Acesso, Autenticação e Autorização)
* Integrado com Jenkis / Code Build e outras CI

### Segurança CodeCommit

* Iterações realizadas utilizando o Git (padrão)
* Authenticação no Git:
  * SSH Keys: Usuários da AWS  pode usar as chaves de SSH em seus consoles IAM
  * HTTPS: Realizado através de Autenticação AWS CLI ou gerando credentials HTTPS
  * MFA: pode ser usado para segurança extra
* Autorização no Git:
  * IAM Policies para gerênciar usuários / direitos de acesso para o repositório
* Encriptação:
  * Repositório são automáticamente encriptados utilizando o KMS
  * Encriptação em transito (pode usar HTTPS ou SSH - ambos seguros)
* Acesso Cross Account:
  * Utiliza IAM Role em usa AWS Account e utiliza AWS STS (with **AssumeRole** API)

### CodeCommit vs GitHub

* Similaridades:
  * Ambos utilizam repositório git
  * Ambos suportam code review (pull requests)
  * GitHub e CodeCommid pode ser integrados com o AWS CodeBuild
  * Ambos suportam métodos HTTPS e SSH de autenticação
* Diferenças:
  * Segurança:
    * Administração de Usuários:	
      * GitHub: GitHub users
      * CodeCommit: AWS IAM user & roles
    * Hosted:
      * GitHub: hospedado por GitHub
      * GitHub Enterprise: gerenciado em seus servidores
      * CodeCommit: gerenciado e hospeado pela AWS
    * UI:
      * GitHub UI é altamente funcional
      * CodeCommit UI é minimo

### CodeCommit Notifications

* É possível disparar uma notificação no CodeCommit utilizando AWS SNS, AWS Lambda ou AWS CloudWatch Event Rules
* Utilizado nos casos para SNS / AWS Lambda notifications:
  * Exclusão de branches
  * Disparo de pushes que acontecem no master
  * Notificação de Build System externo
  * Disparar AWS Lambda function para análise de código
* Casos para uso do CloudWatch Event Rules:
  * Trigger para pull request updates (created/ updated/ deleted / commented)
  * Comentários em eventos de Commit 
  * CloudWatch Event Rules vai para um SNS Topic

## CodePipeline

* Integrar continua

* Visual Workflow

* Source: GitHub / CodeCommit / AWS S3

* Build: CodeBuild / Jenkis / Etc...

* Teste de Carga: ferramentas de 3drs

* Deploy: AWS CodeDeploy / Beanstalk/ CloudFormation / ECS

* Stages:

  * Cada stage pode ter **ações sequenciais e/ou paralelas
  * Exemplos: Build / Teste / Deploy / Load Test / etc..
  * Aprovação Manual pode ser definida em qualquer stage

  ### AWS CodePipeline Artifacts

  * Cada Stage da pipeline pode criar "artifacts"
  * Artifacts são armazenados no Amazon S3 e passado para o próximo Stage

  ### CodePipeline Troubleshooting

  * Alterações no Stages acontecem **AWS CloudWatch Events**, que podem retornar uma notificação SNS
    * Ex: pode criar eventos por falhar uma pipeline
    * Ex: pode criar eventos por cancelar um stage
  * Se um Stage falhar, a pipeline interrompe e você pode obter informações no console
  * AWS CloudTrail pode ser usado para auditar chamadas de AWS API
  * Se um Pipeline não pode executar uma ação, garanta que o "IAM Service Role" anexo tem as permições necessárias (IAM Policy)

## CodeBuild

* Serviço de "build" altamente gerenciável

* Alternativa para outras ferramentas de build, como o Jenkis

* Escalonamento continuo (sem servidor para gerênciar ou provisionar - sem queue build)

* Paga por uso: o tempo que leva para completar o build

* Deixar disponíbilizado Docker para reproduzir build

* Possibilidade de extender capacidade de sua própria imagem de Docker

* Segurança: Integração com o KMS para encriptação de artifatos de build, IAM para permissões de build, VPC para segurança de rede e CloudTrail para logar chamadas de API

* Instruções de build dedem ser definidas no código do diretório root (buildspec.yml)

* Saída de logs para CloudWatch e S3

* Métricas podem ser monitoradas no CodeBuild statics

* Uso de CloudWatch Alarms para detectar falhas no builds e disparar notificações (SNS incluída)

* CloudWatch Events / AWS Lambda e Glue

* É possível executar o CodeBuild na máquina local para investigar erros

* Build podem ser definidos dentro de CodePipeline or dentro do próprio CodeBuild

* **CodeBuild suporta muitos ambiente nativamente, mas em caso de ambiente não suportados, é possível gerar uma imagem Docker e executar em qualquer ambiente que deseje**

  

  ### CodeBuild BuildSpec

  * Deve ser especificado o arquivo buildspec.yml no diretório root
  * Definir as variáveis de ambiente:
    * Plaintext variables
    * Secure Secrets: use SSM Parameter Store
  * Phases:
    * Install: instala todas dependencias que você deseja para construir seu build
    * Pre_build: executa comandos antes de iniciar o build
    * **Build**: comandos de build
    * Pos_build: finaliza o build (compactação de zip por exemplo)
  * Artifacts: O que for enviado para o S3 (encriptado por KMS)
  * Cache: Arquivos para cache (normalmente dependencias) para o S3 para aumentar a velocidade build

  ### CodeBuild Local Build

  * No caso de investigar problemas de logs é possível executa-lo localmente no seu desktop
    * Necessário ter Docker e CodeBuild Agent instalados

  ### CodeBuild na VPC

  * Pode padrão, CodeBuild containers são lançados forada da VPC. Então, não pode acessar recursos de uma VPC
  * É possível especificar uma configuração de VPC (VPC ID, Subnet ID, Security Group ID), e então acessar recursos da VPC
  * Casos de uso: testes de integração, consulta a dados, load balancers internos

  ### Segurança CodeBuild

  * Para acessar recursos na sua VPC, especificar uma configuração de VPC no seu CodeBuild
  * Segurança no CodeBuild:
    * Variáveis de ambiente podem referenciar parâmetros armazenados como parâmetros
    * Variáveis de ambiente podem referenciar parâmetros no SSM

## AWS CodeDeploy

* Deve estar com o CodeDeploy Agent executando

* Envia o arquivo *appspec.yml*

* Aplicações são baixadas do Github ou do S3

* EC2 executará as instruções de deploy

* CodeDeploy Agent reportará sucesso ou falha no deploy da instância

* Pode ser agrupado por grupos (dev / test / prod)

* **Pode ser usado Blue / Green somente em instâncias EC2 (não on premise)**

* Suporte para deploys de AWS Lambda

* Não provêm recursos, utiliza os existentes
  * ***AWS CodeDeploy Componentes Primários:***
    * Application: nome único
    * Compute platform: EC2/ On Premises ou Lambda
    * Deployment configuration: Regras de deploy para sucesso ou falha
      * *EC2/On Premises: você pode especificar o número mínimo de instâncias healthy para deploy*
      * *AWS Lambda: especificar como o trafico é roteado para atualização de versões de Lambda*
    * Deployment group: agrupar por instâncias "taguiadas" (permite realizar deploy gradualmente)
    * Deployment type: Deploy in-place or Blue/green deploy
    * IAM instance profile: precisa de permissão de EC2 para baixar do S3 ou do GitHub
    * Application Revision: código aplicação + appspec.yml
    * Service Role: Regra para CodeDeploy performar o que é necessário
    * Target revision: Destino do deploy da versão da aplicação
  
  
  
  ### AppSpec
  
* File section: como copiar da origem do S3 / Github para o filesystem

* Hooks: lista de instruções para deploy uma nova versão (pode ter timeouts). A ordem é:
  * ApplicationStop -> DownloadBundle -> BeforeInstall -> Install -> AfterInstall -> ApplicationStart -> **ValidateService** -> BeforeAllowTraffice -> AllowTraffic -> AfterAllowTrafic

  ### Deployment Config

  * Configs:

    * One at a time: uma instância por um tempo, se uma instância falhar, deployment é interrompido

    * Half at a time: 50%

    * All at once: rápido mas não é "saudável", downtime. Bom para ambiente DEV

    * Custom: min de host saudável = 75%

    * Canary:

      * Falhas:
        * Instancias ficam no "estado de falha"
        * Novos deployments irão ser deployados primeiros nas instâncias "falhadas"
        * Rollback: redeploy versões velhas ou habilitar rollbacks automáticos para falhas
      * Destino de Deploys:
        * Lista de instâncias EC2 com tags
        * Diretamente no ASG
        * Mix de ASG / Tags podendo deployar por segmentos
        * Customização por scripts na variável DEPLOYMENT_GROUP_NAME

      ### Rollbacks

      * Pode ser especificado opções de automação de rollback
      * Roll back quando um deployment falha
      * Roll back quando um alarme é encontrado
      * Disable rollbacks -> Não realiza rollbacks para este deployment

  ## CodeStar

  * É uma solução integrada para reagroupar: GitHub, CodeCommit, CodeBuild, CodeDeploy, CloudFormation, CodePipeline, CloudWatch
  * Ajuda a rapidamente criar um CI/CD para projetos do EC2, Lambda, Beanstalk
  * Suporta linguagens: C#, Go, HTML5, Java, Node.js, PHP, Phyton, Ruby
  * Integração com ferramentas de controle: JIRA / GitHub Issues
  * Possui integração com Cloud9 para obter uma Web IDE (não para todas regiões)
  * Um dashboard para visualizar todos os componentes
  * Serviço grátis, paga somente por serviços e recursos
  * Customização Limitada

