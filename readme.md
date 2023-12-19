# Implantação do logs-stack(FileBeat, Logstash, New Relic)

## Processo

O serviço `logs-stack` é um conjunto de serviços na qual foi desenvolvido para realizar a coleta de logs, sendo eles:
    
- FileBeat: É um serviço utilizado para fazer a coleta de logs de um determinado arquivo ou container. Em nosso projeto o mesmo está sendo utilizado para realizar a coleta de logs dos containers\stacks que estão em execução dentro do nosso cluster. 

- Logstash: É o serviço utilizado para fazer receber os logs coletador através do filebeat e também fazer os filtros dos logs da forma que achar mais adequada.

- New Relic: É uma ferramenta de observabilidade completa, mas que para nosso caso vamos utilizar apenas a ferramenta de logs até o momento, para poder visualizar os logs que são recebidos através do Logstash.

```

FileBeat -> Logstash -> New Relic

```

## Requisitos

Requisitos necessários para realizar o deploy do serviço `logs-stack`:

- Conta criada no New Relic, cada conta você irá possuir uma `license_key` que será utilizada para a integração entre o Logstash e o New Relic através dos plugins utilizados. A `license_key` deverá ser inserida no arquivo `logstash.yml`.


## Create Config

Para a utilização dos serviços é necessário parametrizar as configurações a serem utilizadas.

1- Filebeat

Como as configurações já estão realizadas realizadas basta criar uma config dentro do nosso cluster a partir do nosso arquivo `fielebeat.yml` que está presente neste diretorio para que o serviço utilize essa configuração durante execução, segue o comando:

```
docker config create filebeat ./filebeat.yml

```

2- Logstash

As principais configurações do Logstash também estão realizadas e presente nos arquivos `logstash.yml`(configuração do host de recebimento de logs) e `logstash.conf`(pipeline a ser excutada do logstash).

Como formas de identificar qual e o cliente e o ambiente de execução do serviços `logs-stack` foi colocado duas variáveis no arquivo `docker-compose.yml` para serem preenchidas, sendo elas `ClienteID` e `EnvironmentDeploy` podendo serem preenchidas conforme o conveniente.

Para que o Logstash faça a integração com o New Relic é necessário a inserção da `license_key` gerada pelo New Relic dentro do arquivo `logstash.conf` no campo `output`, após a chave ser inserida salve o arquivo e execute o comando para criar a configuração dentro do cluster swarm:

```
docker config create logstash-conf ./logstash.yml

docker config create logstash-pipeline ./logstash.conf

```

3- New Relic

O New Relic faz a comunicação com o Logstash através de um plugin específico chamado `logstash-output-newrelic` na qual é instalado de forma automatizada dentro do container do Logstash no momento de sua execução, ou seja não é necessário nenhuma ação referente ao New Relic.

Conforme dito anteriormente é necessário apenas a inserção da `license_key` do New Relic dentro do arquivo `logstash.conf` no campo `output`.

## Deploy

Após a criação das configurações mencionadas basta realizar o deploy da stack a partir deste diretório com o seguinte comando:

```
docker stack deploy -c docker-compose.yml logs-stack

```
