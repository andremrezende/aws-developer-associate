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