# Provisionando Kubernetes Cluster com Ansible

## **Descrição**

Instalação e configuração dos requisitos para configurar um cluster kubernetes com ansible em sistemas operacionais like RedHat.

### **Requisitos:**

- [x] SSH
- [x] Ansible 2.9.+
- [x] Conexão livre entre os servidores.
- [x] Conexão com a Internet
- [x] Sistema Operacional like RedHat
- [x] Hardware
  - [x] vCPU: 2
  - [x] Memoria: 2G
  - [x] Disco: 65GB

### **Inventário**

Na pasta do projeto, crie um inventário.

```shell
cd ./inventorie
cp -rf sample <SEU INVENTARIO>
```

Configure o inventário que criou com os servidores que farão parte do cluster kubernetes, master e workers.

```shell
vim ./inventories/<SEU INVENTARIO>/inventory.ini
```

```shell
[all]
master  ansible_host=1.2.3.4
worker  ansible_host=1.2.3.4
worker  ansible_host=1.2.3.4

[master]
master

[workers]
worker
```

![inventory](/docs/images/inventory.png)

### **SSH**

Configure os servidores com sua chave ssh.

**Role:** **[./roles/ssh/vars/main.yml](./roles/ssh/vars/main.yml)**

#### **SSH - Tasks**

- [x] Criando grupo de serviço
  - Grupo para o usuário de serviço
- [x] Criando usuário de serviço
  - Usuário de serviço para execução dos playbooks
- [x] Adicionando Grupo de Serviços no Sudoers
  - Adicionando o usuário no sudoers
- [x] Configurando Chave SSH no usuário de serviço
  - Copiando chave SSH para o usuário de serviço

#### **SSH - Variáveis**

Dentro da role ssh configure sua chave ssh na variável **```ssh_key```** **[./roles/ssh/vars/main.yml](./roles/ssh/vars/main.yml)**

```yaml
ssh_key:
  # SSH Information: ~${USER}/.ssh/id_rsa.pub
  - ""
```

![playbook_ssh_0](/docs/images/playbook_ssh_0.png)

#### **SSH - Uso**

Com a chave setada na variável, execute o playbook **```ssh.yml```**

```shell
ansible-playbook -i inventory/<SEU INVENTARIO>/inventory.ini -u root -k ssh.yml
```

![playbook_ssh_1](/docs/images/playbook_ssh_1.png)

![playbook_ssh_2](/docs/images/playbook_ssh_2.png)

### **Cluster**

Instalação e configuração do cluster kubernetes.

**Roles:**

- [x] **[./roles/common](./roles/common)**
- [x] **[./roles/master](./roles/master)**
- [x] **[./roles/worker](./roles/worker)**

#### **Cluster - Common Tasks**

- [x] Verificando Pré Requísitos
  - Verifica os pré requisítos do sistema operacional para configurar o cluster
- [x] Atualizando o Sistema
  - Atualiza o sistema operacional
- [x] Verificando se o repositório Docker existe
  - Verifica se o repositório do Docker já existe no sistema
- [x] Habilitando o Repositório do Docker
  - Adiciona o repositório do Docker ao sistema
- [x] Verificando se o repositório Kubernetes existe
  - Verifica se o repositório do Kubernetes já existe no sistema
- [x] Habilitando o Repositório do Kubernetes
  - Adiciona o repositório do Kubernetes ao sistema
- [x] Instalando pacotes essenciais
  - Instala os pacotes essenciais ao sistema operacional e a configuração do cluster
- [x] Pip
  - Módulos para o bom funcionamento do ansible
- [x] Configurando Serviço NTP
  - Instalação e configuração do serviço NTP do sistema operacional
- [x] Habilitando os Serviços
  - Habilita os serviços essenciais para o bom funcionamento do sistema operacional e do cluster
- [x] Desabilitando os Serviços
  - Desabilita serviços não essenciais para o sistema operacional
- [x] Removendo swapfile do /etc/fstab
  - Removendo a swap do arquivo fstab
- [x] Desabilitando a swap
  - Remove e desabilita a SWAP do sistema operacional
- [x] Desabilitando o SELinux
  - Desabilita o SELinux
- [x] Atualizando o hostname do servidor
  - Atualiza o hostname dos servidores conforme configurado no inventário
- [x] Configurando o arquivo hosts dos servidores
  - Adiciona entrada dos servidores no arquivo hosts
- [x] Configurando módulos do kernel
  - Habilita módulos do kernel para o bom funcionamento do cluster kubernetes
- [x] Parametros de Rede para o Kernel
  - Adiciona parametros ao kernel para o bom funcionamento do cluster kubernetes
- [x] Aplicando os parametros de Rede para o Kernel
  - Aplica os parametros de rede para o kernel do sistema operacional

#### **Cluster - Master Tasks**

- [x] Resetando o Cluster Master Node
  - Reseta o cluster caso ele esteja iniciando previamente
- [x] Inicializando o Cluster Kubernetes
  - Inicializa o cluster com o kubeadm
- [x] Garantindo que o diretório .kube existe
  - Configura o diretório .kube no home do usuário
- [x] Criando link da configuração do kubernetes
  - Cria um link simbólico da configuração de acesso ao cluster pelo kubectl
- [x] Configurando a rede com o Weavenet
  - Configura o plugin de rede Weavenet
- [x] Token do Cluster
  - Lista o Token do cluster gerado na inicialização do cluster
- [x] CA Hash
  - Hash do arquivo da CA do cluster gerado na inicialização do cluster
- [x] Adicionando token do cluster kubernetes aos Dummy Hosts
  - Armazenando as informações de token e hash em variáveis para adicionar os nodes workers ao cluster

#### **Cluster - Worker Tasks**

- [x] Resetando o Cluster Worker Node
  - Reseta o cluster caso ele esteja iniciando previamente
- [x] Adicionando Workers ao Cluster
  - Adiciona os nodes workers ao cluster kubernetes com kubeadm

#### **Cluster - Variáveis Common**

| Variável | Descrição |
| :--- | :--- |
| update_system | Habilita atualização do sistema |
| path_docker_repo | Caminho para o repositório Docker |
| path_kubernetes_repo | Caminho para o repositório Kubernetes |
| packages.to_install | Pacotes para serem instalados |
| packages_pip.to_install | Pacotes Pip para serem instalados |
| services.to_enabled | Serviços para serem habilitados |
| services.to_disabled | Serviços para serem desabilitados |
| ntp_servers | Servidores NTP's |

```yaml
---
update_system: true

path_docker_repo: "/etc/yum.repos.d/docker.repo"
path_kubernetes_repo: "/etc/yum.repos.d/kubernetes.repo"

packages:
  to_install:
    - python
    - python-pip
    - epel-release
    - yum-utils
    - device-mapper-persistent-data
    - lvm2
    - docker-ce
    - ntp
    - bash-completion
    - libseccomp
    - kubelet
    - kubectl
    - kubeadm

packages_pip:
  to_install:
    - docker

services:
  to_enabled:
    - ntpd
    - vmtoolsd
    - docker
    - kubelet
  to_disabled:
    - firewalld

ntp_servers:
  - "server 0.br.pool.ntp.org"
  - "server 1.br.pool.ntp.org"
  - "server 2.br.pool.ntp.org"
  - "server 3.br.pool.ntp.org"
```

#### **Cluster - Variáveis Master**

| Variável | Descrição |
| :--- | :--- |
| pod_network_cidr | CIDR para a rede do cluster kubernetes |
| k8s_master_node_ip | IP do node master do cluster |
| k8s_api_secure_port | Porta da API do cluster kubernetes |
| weavenet_network | Habilita a instalação do plugin de rede Weavenet |
| weavenet_repository | URL do repositório do plugin de rede Weavenet |

```yaml
pod_network_cidr: "192.168.0.0/16"

k8s_master_node_ip: "192.168.7.133"
k8s_api_secure_port: 6443

weavenet_network: true
weavenet_repository: "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

#### **Cluster - Variáveis Worker**

| Variável | Descrição |
| :--- | :--- |
| k8s_master_node_ip | IP do node master do cluster |
| k8s_api_secure_port | Porta da API do cluster kubernetes |

```yaml
k8s_master_node_ip: "192.168.7.133"
k8s_api_secure_port: 6443
```

#### **Cluster - Uso**

Com o ssh e o inventário confirado execute

```shell
ansible-playbook -i inventory/<SEU INVENTARIO>/inventory.ini -u ansible -b cluster.yml
```

![playbook_cluster](/docs/images/playbook_cluster_0.png)

![playbook_cluster](/docs/images/playbook_cluster_1.png)

### **Metric Server**

Instalação e configuração do servidor de métricas para o cluster kubernetes

**Role:** [./roles/metric_server](./roles/metric_server)

#### **Metric Server - Tasks**

- [x] Diretórios para os arquivos YAML para deploy
  - Diretórios para armazenar os arquivos YAML de instalação do servidor de métricas
- [x] Copiando os arquivos YAML para o diretório criado
  - Copia os arquivos YAML's para o diretório criado para deploy do servidor de métricas
- [x] Deploy Metric Server
  - Instala e configura o servidor de métricas

#### **Metric Server - Variáveis**

| Variável | Descrição |
| :--- | :--- |
| N/A | N/A |

#### **Metric Server - Uso**

Para instalar e configurar o servidor de métricas execute

```shell
ansible-playbook -i inventory/<SEU INVENTARIO>/inventory.ini -u ansible -b metric-server.yml
```

![metric_server_0](/docs/images/metric_server_0.png)

### **Helm**

Instalação e configuração do gerenciados de aplicativos Helm

**Role:** [./roles/helm](./roles/helm)

#### **Helm - Tasks**

- [x] Verificando se o Helm já foi instalado
  - Verifica se o Helm já está instalado no servidor
- [x] Baixando o Helm
  - Baixa o script de instalação do Helm
- [x] Instalando Helm
  - Instala o Helm

#### **Helm - Variáveis**

| Variável | Descrição |
| :--- | :--- |
| helm_url | URL do repositório do Helm |
| helm_url_repo | URL para os repositórios de aplicativos do Helm |

```yaml
---
helm_url: "https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3"
helm_url_repo: https://kubernetes-charts.storage.googleapis.com
```

#### **Helm - Prometheus Operator***

| Variável | Descrição |
| :--- | :--- |
| prometheus_operator | Parametros para instalação do Prometheus Operator |

```yaml
prometheus_operator: "--set prometheusOperator.createCustomResource=false prometheus-operator stable/prometheus-operator"
```

#### **Helm - Uso**

Para instalar e configurar o servidor de métricas execute

```shell
ansible-playbook -i inventory/<SEU INVENTARIO>/inventory.ini -u ansible -b helm.yml
```

![helm_0](/docs/images/helm_0.png)
