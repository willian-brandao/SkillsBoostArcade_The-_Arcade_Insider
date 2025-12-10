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

Para realizar configurações de instâncias no google cloud, deve-se ter um projeto que integre essas máquinas e este projeto deve estar alocado em uma região. Para configurar uma região para o projeto, usa-se o comando abaixo:

```bash
  gcloud config set compute/zone us-central1-
```

Script de inicialização instala o Apache e cria uma página inicial exclusiva para cada instância.

1.1.Criar uma instância na região us-east4-b

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

1.2 Criar segunda instância na mesma região
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

Escolher uma nova zona para criar instâncias

```bash
  ggcloud config set compute/zone us-central1
```

2.1 Criar primeira instância na zona us-central1-a
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
2.2 - Criar segunda instância na zona us-central1-a

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
**Saída:**
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

1.1 Validar se as instâncias criadas estão em execução

```bash
gcloud compute instances list
```

# Configurar Serviços para balanceamento de carga

Agora que as máquinas virtuais(instancias) estão criadas, deve-se atribuir servios à elas para para realizar o balancemento de carga.

Objetivo:
1. Criar um endereço de IP externo estático global que os clientes irão usar para acessar o balanceador de carga.
2. Criar grupos de instâncias para armazenar instâncias.
3. Criar uma verificação de integridade, que consulta as instâncias para conferir se estão íntegras. O balanceador de carga só enviar tráfego a instâncias íntegras.

Confguração dos serviços:
1. Atribuir um endereço de IP externo estático IPv4 para o balanceador de carga:

```bash
gcloud compute addresses create lb-ip-cr \          # atribui o nome ao IP, nesse caso: lb-ip-cr
--ip-version=IPV4  \                                # define que o IP reservao é IPv4
--global                                            # define que o endereço IPv4 não pertence a nenhuma região em específico  

```

2. Criar um grupo de instâncias para cada uma das zonas

```bash

# configurando o grupo de instância da primeira zona 
gcloud compute instance-groups unmanaged create REGION-resources-w --zone ZONE
gcloud compute instance-groups unmanaged create us-east4-b-resources-w --zone us-east4-b

# configurando o grupo de instância da segunda zona
gcloud compute instance-groups unmanaged create REGION-resources-w --zone ZONE
gcloud compute instance-groups unmanaged create us-central1-a-resources-w --zone us-central1-a
```

3. Adicione as instâncias criadas anteriormente aos grupos de instâncias.

```bash

gcloud compute instance-groups unmanaged add-instances us-east4-b-resources-w \
--instances www-1, www-2 \
--zone us-east4-b

gcloud compute instance-groups unmanaged add-instances us-central1-a-resources-w \
--instances www-1, www-2 \
--zone us-central1-a
```

4. Criar uma verificação de integridade.

```bash
gcloud compute health-checks create http http-basic-check
```
Documentação: https://docs.cloud.google.com/sdk/gcloud?hl=pt


