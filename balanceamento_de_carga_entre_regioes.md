# Criar um balanceamento de carga entre regiões do Google Cloud


Objetivo: Criar um balanceador de carga HTTP(S) que encaminha o tráfego para instâncias em duas regiões diferentes. 
Neste laboratório, você vai criar quatro instâncias do Compute Engine, duas em cada uma das duas regiões diferentes. 
Em seguida, vai configurar o restante do sistema para que as conexões de entrada sejam enviadas à instância adequada.

<p align="center">
  <img width="750" height="450" alt="image" src="https://github.com/user-attachments/assets/d9bebc72-3638-496a-b3b4-ec2a4159d35b"  />
</p>


Para realizar determinadas configurações deve-se selecionar o id do projeto que deseja construir um projeto. O seguinte comando realiza a ação. 

*gcloud config set project [id-projeto]*

Ex:
```bash
gcloud config set project student-04-b39de5d2fb9b
```

## Listar  o nome da conta ativa no cloud da google

Informações sobre o projeto e usuário que está ativo no momento. Segue abaixo os comandos:

Listar conta ativa.

Ex:
``
gcloud auth list
``

saída:

<img width="553" height="169" alt="image" src="https://github.com/user-attachments/assets/5dd4f48f-bdee-414f-8ddc-92b8f1bf6cb5" /><br>

<br/>

Listar o id do projeto.

``
gcloud config list project
``

saída:

<img width="586" height="109" alt="image" src="https://github.com/user-attachments/assets/5c028eaf-fa50-49b4-a70f-c7f5d587ceda" />


# 1.Configurar Instâncias 

Instâncias são máquinas virtuais criadas dentro da nuvem, essas máquinas podem ser criadas em diversos portos espalhados pelo mundo, esses pontos são conhecidos como REGIÕES. Essas regiões possuem um padrão de código que indica em qual ponto do planeta ela está localizada. Ex: **us-east4-b**.

Para realizar configurações de instâncias no google cloud, deve-se ter um projeto que integre essas máquinas. Para selecionar o projeto que irá receber a estrutura desejada realiza-se a configuração do projeto.

``
  gcloud config set compute/zone us-central1-
``

Script de inicialização instala o Apache e cria uma página inicial exclusiva para cada instância.

1.1.Criar duas instâncias em cada região:

```bash
gcloud compute instances create www-1 \
    --image-family debian-11 \
    --image-project debian-cloud \
    --machine-type e2-micro \
    --zone us-east4-b \
    --tags http-tag \
    --metadata startup-script="#! /bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo '<!doctype html><html><body><h1>www-1</h1></body></html>' | tee /var/www/html/index.html
"
```
**Saída:**
```bash

Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-90a340b1bba1/zones/us-east4-b/instances/www-1].
NAME: www-1
ZONE: us-east4-b
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.150.0.2
EXTERNAL_IP: 34.186.117.81
STATUS: RUNNING
```

### 1.1 Criar segunda instância
```bash
gcloud compute instances create www-2 \
    --image-family debian-11 \
    --image-project debian-cloud \
    --machine-type e2-micro \
    --zone us-east4-b \
    --tags http-tag \
    --metadata startup-script="#! /bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo '<!doctype html><html><body><h1>www-2</h1></body></html>' | tee /var/www/html/index.html
"
```
**Saída:**
```bash
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-90a340b1bba1/zones/us-east4-b/instances/www-2].
NAME: www-2
ZONE: us-east4-b
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.150.0.3
EXTERNAL_IP: 34.48.169.6
STATUS: RUNNING
```

## 2. Criar duas instâncias em outra região

### 2.0 - Configurar uma zona para realizar a configuração
``
  ggcloud config set compute/zone us-central1
``
### 2.1 - Criar primeira instância na zona us-central1-a
```bash
gcloud compute instances create www-3 \
    --image-family debian-11 \
    --image-project debian-cloud \
    --machine-type e2-micro \
    --zone us-central1-a \
    --tags http-tag \
    --metadata startup-script="#! /bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo '<!doctype html><html><body><h1>www-3</h1></body></html>' | tee /var/www/html/index.html
"
```

**Saída:**
```bash
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-90a340b1bba1/zones/us-central1-a/instances/www-3].
NAME: www-3
ZONE: us-central1-a
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.4
EXTERNAL_IP: 34.61.212.226
STATUS: RUNNING
```
### 2.2 - Criar segunda instância na zona us-central1-a

```bash
gcloud compute instances create www-4 \
    --image-family debian-11 \
    --image-project debian-cloud \
    --machine-type e2-micro \
    --zone us-central1-a \
    --tags http-tag \
    --metadata startup-script="#! /bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo '<!doctype html><html><body><h1>www-4</h1></body></html>' | tee /var/www/html/index.html
"
```
**Saída:***
```bash
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-90a340b1bba1/zones/us-central1-a/instances/www-4].
NAME: www-4
ZONE: us-central1-a
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.5
EXTERNAL_IP: 34.67.121.4
STATUS: RUNNING
```

## Configurar uma regra de firewall para permitir tráfego externo chegue às instâncias criadas na nuvem. 

A regra criada que permite que o tráfego na porta designada alcance as instâncias que tenham a tag, a tag tem o nome de "http-tag"

1. Criar a regra de firewall
```bash
gcloud compute firewall-rules create www-firewall \
--target-tags http-tag --allow tcp:80
```
1.1 Validar as instâncias 

```bash
gcloud compute instances list
```



Documentação: https://docs.cloud.google.com/sdk/gcloud?hl=pt


