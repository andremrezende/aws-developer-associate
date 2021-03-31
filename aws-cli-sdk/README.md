## **Desenvolvendo na AWS: AWS CLI, SDK, IAM Roles & Policies**

#### **AWS CLI**

* AWS CLI sobre instancias EC2: Nuca deixe suas credenciais no EC2, sempre utilize de IAM Roles anexas a ele
* Alguns comandos AWS CLI (não todos) contém a opção --dry-run para simular a chamada da API
* Quando uma chamda de API falhar, você pode visualizar a mensagem de erro, que pode ser decodificada usando o comando the **STS** em linha de comando(IAM user ou Role deve ter STS permissão de escrita: decode message):
  * sts **decode-authorization-message**
* Metadados da instância EC2:
  * Permite que instâncias do EC2 "aprendam sobre elas mesmas" sem o uso IAM Role para esse propósito
  * A URL é <http://169.254.169.254/latest/meta-data> (ex: `curl http://169.254.169.254/latest/meta-data`)
    * Você pode obter um nome de IAM Role do metadado, mas você ***não pode*** obter a IAM Policy
  * Metadado = informações sobre a instância do EC2
  * Userdata = script de lançamento da instância de EC2
* AWS Profile
  * Você pode usar `aws configure --profile` para configurar multiplas credenciais na mesma máquina
  * Quando um comando está em execução, eles continuarão execução no profile padrão, se você precisar um profile especifico você deverá finalizar o comando com `--profile {your-profile}`

#### **MFA com CLI**

* Para usar  MFA com o CLI, você deverá criar uma sessão temporaria. Para fazer, execute a chamada da API STS GetSessionToken
  * **aws sts -getsession-token** --serial-number arn-of-the-mfa-device --token-code code-from-token --duration-seconds 3600

#### AWS SDK

* Quando se deseja iteragir diretamente da aplicação sem usar o CLI, você pode usar uma SDK (Software Development KIT)
* Utiliza SDK quando chamar diretamente um serviço da AWS tal como DynamoDBB
* Se não especificar a região padrão, então us-east-1 será utilizada
* É recomendado usar a credencial padrão:
  * Funciona perfeitamente com credenciais em ~/.aws/credentials (somente na sua máquina / local)
  * Credenciais de perfil de instância com papéis IAM 
  * Variáveis de ambiente : AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY
  * Nunca armazene as credenciais da AWS no seu código fonte
* Qualquer API que falha por causa de muitas chamadas precisão tratar com Eponential Backoff
  * Aplicar para avaliar o limite de API
  * Mecanismo de retry incluso em chamadas de SDK API

### AWS Limites (Quotas)

* **API Rate Limits**
  * Descreve instancias de API para EC2 com um limite de 100 chamadas/s
  * GetObject no S3 tem um limite de 5500 GET/s
  * Para Intermittent Errors: implementar Exponential Backoff
  * Para Consistent Errors: requisitar aumento de limite de API thorttling
* **Service Quotas (Limites de Serviço)**
  * Executado sobre instância sobre demanda Standard: 1152vCPU
  * Pode ser requisitado para aumentar o limite com um ticket
  * Pode ser requisitado para aumentar a quota usando o Service Quotas API