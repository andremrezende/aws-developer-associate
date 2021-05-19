# Escalabilidade & Alta disponibilidade

* Escalabilidade significa que uma aplicação / sistemas pode trabalhar com grandes volumes se adaptando
* Existem dois tipos de escalabilidade:
  * Vertical
  * Horizontal (= elasticidade)
  * **Escalabilidade é lincado mas diferente de Alta disponibilidade **

## Escalabilidade Vertical

* Significa aumentar o tamanho de uma instância
* Por exemplo, uma aplicação que executa em uma t2.micro ser aumentada para uma t2.large
* Escalabilidade vertical é muito comum para sistemas não distribuídos, como banco de dados
* RDS, ElasticCache são serviços que podem usar estabilidade vertical
* É limitado em qual tamanho um hardware pode "escalar"

## Escalabilidade Horizontal

* Significa aumentar o número de instâncias / sistemas para sua aplicação
* Implica em sistemas distribuidos
* É muito importante para aplicações web / aplicações modernas
* É fácil escalar na nuvem, podendo usar o Amazon EC2

## Alta disponibilidade

* Andam de mão juntas com escalonamento horizontal
* Significa executar a aplicação ao menos em 2 data centers (== AZ)
* O objetivo é sobreviver a perda/falhas de data centers
* A alta disponibilidade pode ser passiva (por RDS Multi AZ por exemplo)
* A alta disponibilidade pode ser ativa (por escalonamento horizontal)

## Alta disponibilidade & Escalabilidade por EC2

* Escalonamento Vertical: Aumentar o número de instâncias (= scale up / down)

  * De: t2name - 0.5G de RAM, 1 CPU
  * Para: u-i12tbI.metal = 12.3TB de RAM, 448 vCPUs

* Escalonamento Horizontal: Aumentar o número de instâncias (=scale out / in)

  * Auto Scaling Group
  * Load Balancer

* Alta disponibilidade: Executar instânciar para a mesma aplicação através de multi AZ

  * Auto Scaling Group multi AZ
  * Load Balancer multi AZ

  

## Load Balancer

* São servidores que repassam o fluxo de internet para multiplos servidores (EC2 Instances) downastream.

### Porque usar load balancer?

* Dividir cargar por multiplas instâncias
* Expor um único ponto de acesso (DNS) para sua aplicação
* Tratar falhar das instâncias
* Executar health checks para suas instâncias
* Prover SSL (HTTPS) para websites
* Prover aderência com cookies
* Alta disponibilidade através de zonas
* Separar tráfefo publico do tráfego privado

### Porque usar um EC2 Load Balancer?

* Um ELB (EC2 Load Balancer) é um gerênciador de load balancer
  * AWS garante que irá funcionar
  * AWS cuida de upgrades, manutenção e alta disponibilidade
  * AWS provém somente umas poucas configurações de knobs
* Custa menos que configurar seu próprio load balancer, e será menos esforço no final das contas
* Integrado com muitos serviços da AWS

	## Healf Checks

* Health Checks são cruciais para os Load Balancers
* Eles habilitam um load balancer para conhecer se uma unstância está repassando o tráfigo e está disponível para responder as requisições
* O health check é coberto por uma porta e uma rota (/health é o comum)
* Se a resposta não é 200 (OK), então a instância não está "saudável"

## Tipos de Load Balancers na AWS

* Classic Load Balancer (v1 - old generation) - 2009
  * HTTP, HTTP, TCP
* Application Load Balancer (v2 - new generation) - 2016
  * HTTP, HTTPS, WebSocket
* Network Load Balancer (v2 - new generation) - 2017
  * TCP, TLS (securte TCP) & UDP
* É recomendado utilizar a nova / v2 generation que provém muito mais funcionalidade
* Pode-se configurar como **internal** (private) ou **external (public) ELBs

## Bom saber

 * LBs pode escalar, mas não instantaneamente, contactar AWS para um "warm-up"
 * Troubleshooting
   	* 4xx erros do cliente
      	* 5xx erros que a aplicação introduziu
   	* 503 capacidade estourada ou sem destino configurados
   	* Se um LB não puder conectar na sua aplicação, verificar o security groups
 * Monitoramento
   	* ELB pode acessar logs que serão "logados" em todos acessos de requisições (você pode debugar por requisição)
      	* CloudWatch Metrics permite agregar estatisticas (ex. connections count)

## Classic Load Balancers (V1)

* Suporte a TCP (camada 4), HTTP & HTTPS (camada 7)
* Health checks são sobre TCP ou HTTP
* Hostname fixo
  * XXX.region.elb.amazonaws.com

## Application Load Balancer (v2)

* Camada 7 - HTTP
* Tem multiplicas aplicação HTTP atráves de máquinas (target groups)
* Suporta por HTTP/2 e WebSocket
* Suporte a redirecionamentos (ex. HTTP para HTTPs)
* Tabela de rotas para diferentes target groups:
  * Baseado em path na URL (example.com/**users** & example.com/**posts**)
  * Baseado em hostname na URL (**one**.example & **other**.example.com)
  * Baseado em Query String e Headers (example.com/users?**id=1234&order=false**)

* ALB são bons para micro serviços & aplicações baseadas em containers (ex. Docker & Amazon ECS)
* Tem uma funcionalidade de mapeater portas para redicerios para uma porta dinâmica no ECS
* Em comparação, é necessário multiplos Classic Load Balancer por aplicação

### Target Groups

* Instâncias de EC2 (podem ser gerências por um ASG) - HTTP
* ECS tasks (gerênciado por ECS por ela mesma) - HTTP
* Lambda Functions  - Requisições HTTP são traduzidas por um evento JSON
* Endereço IP - devem ser IPs privados
* ALB pode rotiar para multiplos target groups
* Health checks são para o nível de target group

## Network Load Balancer (v2)

* Permitem:
  * Repassar tráfico de TCP & UDP por instância
  * Trata milhões de requisições por segundo
  * Menor latência ~100ms  (vs 400 ms por ALB)
* NLB tem um IP estático por AZ e suporta um IP Elastic (útil para whitelisting de um IP especifico)
* NLB são usado para performance extrema, tráfico de TCP ou UDP
* Não incluso no AWS free tier

## Load balancer Stickiness

* Quando existe a necessidade de um mesmo cliente sempre ser redirecionado para o mesma instância pelo load balancer
* Funciona com o Classic Load Balancer & Application Load Balancers
* O "cookie" é usado para "stickiness" com uma data de expiração para seu controle
* Caso de uso: certifique que o usuário não perca os dados da sessão
* Habilitar "stickiness" pode deixar desbalanceados as instâncias do EC2

## Cross-Zone load Balancing

* ALB:
  * Sempre ligado (não pode ser desligado)
  * Sem cobrança para dados inter AZ
* NLB:
  * Desabilitado por padrão
  * Paga por uso de dados internos AZ se habilitado
* CLB:
  * Através do console -> Ligado por padrão
  * Através do CLI / API -> Desligado por padrão
  * Sem cobrança para dados internos a AZ se ligado

## SSL/TLS

* Um certificado SSL permite transitar entre cliente e load balancer e ser encriptado em transito (in-flight encryption)
* SSL refere-se para uma camada de socket segura, parado para coneções encriptadas
* TLS certifica que são usados, mas para pessoas ainda refere-se a SSL
* Certificado públicos de SSL são usados por Autoridades de Certificado (CA)
* Comodo, Symantec, GoDaddy, GlobalSign, Digicert, Letsencrypt, outros...
* Certificado SSL tem uma data de expiração e devem ser renovados se não configurados

### Load Balancer - Certificados SSL 

* Load Balancer utiliza um certificado X.509 (SSL/TLS provém certificado)
* Pode gerênciar certificados utilizando ACM (AWS Certificate Manager)
* Como alternativa, você pode fazer upload de seu próprio certificado
* HTTPS listener:
  * Pode especificar um certificado padrão
  * Adicionar uma lista de certificados opcionais para suportar multiplos dominios
  * Clientes pode usar SNI (Server Name Indication) para especificar o hostname que eles buscam
  * Habilidade para especificar uma politica de segurança para velhas versões de SSL / TLS  (legacy clientes)

## SSL - Server Name Indication (SNI)

* SNI resolve o problema de multiplos certificados SSL dentro do servidor web (para servir múltiplos websites)
* É um "novo" protocolo, e requer um cliente para **indicar** o hostname que o servidor destino no SSL inicial de handshake
* O servidor irá procurar o certificado correto e retorna um padrão

**NOTA:**

* Somente funciona em ALB & NLB (nova geração), CloudFront
* Não funciona com o CLB (velha geração)

* CLB:
  * Suporta somente um certificado SSL
  * Pode utilizar múltiplos CLB para múltiplos hostname com múltiplos certificados SSL
* ALB:
  * Suportam múltiplos listeners com múltiplos certificados SSL
  * Utiliza SNI para funcionar
* NLB:
  * Suporta múltiplos listeners com múltiplos certificados SSL
  * Utiliza SNI para funcionar

## ELB - Connection Draining

* Nome da funcionalidade:
  * CLB: Connection Draining
  * Target Group: Deregistration Delay (para ALB e NLB)
* Tempo para completar "in-flight requests" em quanto a instância é desregistrada ou não saudável
* Stops envia novas requisições para a instância que é desregistrada
* Entre 1 e 3600 segundos, o padrão é 300 segundos
* Pode ser desabilitada (configurar valor para 0)
* Configurar um valor baixo se uma requisição é curta

## Auto Scaling Group

* Na vida real, a carga do seu website e aplicação pode ser alterada
* Na nuvem, você pode criar ou aumentar os servidores de forma rápida
* O objetivo do ASG é para:
  * Scale out (adicionar instâncias do EC2) para corresponder o aumento de carga
  * Scale in (remover instâncias do EC2) para diminuir a carga
  * Garantir um minimo e máximo número de máquina em execução
  * Automaticamente registrar uma nova instância para o load balancer
* Atributos de configuração:
  * AMI + Tipo de Instância
  * EC2 User Data
  * Volume EBS
  * Security Groups
  * SSH Key Pair
* Tamanho Min / Max / Capacidade Inicial
* Network + Informação de Subnets
* Informação de Load Balancer (LB)
* Politicas de Escalonamento

### Alarmes de Escalonamento

* Baseado no CloudWatch alarms
* Um alamerme monitora uma métrica (tal como Média de consumo de CPU)
* Métricas são computadas por todas instâncias do ASG
* Baseado no alarme:
  * Criar politicas de scale-out (aumentar o número de instâncias)
  * Criar políticas de scale-in (diminuir o número de instâncias)
* Para definir as melhores regras de escalonamento diretamente gerênciadas pelo EC2:
  * Média de uso de CPU
  * Número de requisições por instâncias de ELB
  * Média de Network In
  * Médica de Network Out
* Essas regras facilitam as configurações e podem fazer mais sentido
* Baseados em métricas customizadas (ex. número de usuários conectados)
* Enviar uma métrica customizada da aplicação no EC2 para o CloudWatch (**PutMetric API**)
* Criar um alarme no CloudWatch para reagir baixa/alta de valores
* Utilizar o CloudWatch alarm como politica de escalonamento por ASG
* Pode utilizar métricas ou em agendamentos (se você conhece os padrões de visitantes)
* Pode iniciar baseado em Launch Templates
* Para atualizar, próver uma nova configuração de lançamento / template
* Roles IAM anexas para um ASG irão mapear uma instância de EC2
* ASG são grátis, você paga por recurso lançado
* Ter instâncias abaixo no ASG significa que se elas terminarem por qualquer razão, o ASG irá automaticamente recriar novas instâncias por substituição. Segurança extra.
* ASG pode terminar instâncias que estão não saudáveis para um LB (e substituir elas)

### Políticas de Escalonamento

* Target Tracking Scaling:
  * Mais simples e mais fácil de configurar
  * Exemplo: Quero médica de ASG CPU fique por volta de 40%
* Simple / Step Scalin:
  * Quando um CloudWatch alarm é disparado (ex. CPU > 70%), adicione 2 unidades
  * Quando um CloudWatch alarm é disparado (ex. CPU <30%), remove 1 unidades
* Schedule Actions:
  * Antecipação de escalonamento baseado e padrões conhecidos de uso
  * Exemplo: aumentar a capacidade minima para 10 as 5:00 nas sextas-feiras

## Escalonamento Cooldowns

* O período de cooldown ajuda a garantir que o grupo de Auto Escalonamento não iniciar ou termina instâncias adicionais antes de um atividades anterior de escalonamento se torne efetivo
* Por padrão para o cooldown de grupo de Auto Escalonamento pode criar para especificar uma Simple Scaling Policy
* Um período scaling-specif cooldown sobrescreve o período padrão de cooldown
* Um uso comum para scaling-specifc cooldown é com uma politica de scale-in, uma politica que termina instancias baseados em um critério especifico ou métrica. Porque esta politica termina instancias, O Amazon EC2 Auto Scaling precisa de menos tempo para determinar quando terminar instâncias adicionais.
* Se o período padrão de cooldown for de 300 segundos para muito longo é possível reduzir o custo aplicando scaling-specific cooldown com período de 180 segundos para a politica de scale-in.
* Se sua aplicação de scaling up e down para múltiplas vezes de cada hora, modificar o Auto Scaling Groups cooldown timers e o CloudWatch Alarm Periodo que dispara o scale in.
