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

#### **SSH - Uso**

Com a chave setada na variável, execute o playbook **```ssh.yml```**

```shell
ansible-playbook -i inventory/<SEU INVENTARIO>/inventory.ini -u root -k ssh.yml
```

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

#### **Cluster - Uso**

Na pasta do projeto, copie crie um inventário.

```shell
cd ./inventorie
cp -rf sample <SEU INVENTARIO>
```

```shell
vim ./inventories/<SEU INVENTARIO>/inventory.ini
```

```shell
[all]
master	ansible_host=1.2.3.4
worker	ansible_host=1.2.3.4
worker	ansible_host=1.2.3.4

[master]
master

[workers]
worker
```



**Saída:**
![](/docs/images/img1.jpg)


## **Inicializando o Cluster**

No servidor master inicie o cluster kubernetes.

 ```bash
 kubeadm init --apiserver-advertise-address <IP DO MASTER> --ignore-preflight-errors=all
 ```
Finalizado a instalação do cluster o kubeadm irá gerar um comando que você precisa executar nos servidores workers., execute o comando em todos os servidores workers do seu cluster.
 
 **Comando Gerado pelo Kubeadm**
 ```bash
 kubeadm join --token <TOKEN GERADO PELO KUBEADM> <IP DO MASTER>:6443 --discovery-token-ca-cert-hash sha256:<HASH GERADO PELO KUBEADM> --ignore-preflight-errors=all
 ```

Se todos os servidores do cluster estiverem configurados corretamente, no servidor master
verifique se todos os servidores workers são listados.

```bash
kubectl get nodes -o wide
```

![](/docs/images/img2.jpg)

O status dos servidores estão como **```NotReady```**, isso porque é preciso instalar um add-on/plugin de rede para que a comunicação entre os servidores fique disponível.

Existe uma lista de [add-on/plugins](https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy) disponíveis.

 ## **Plugin de Rede**

Para esse exemplo vamos usar o [weave-net](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/).

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

Aguarde até que todos os pods de rede estejam rodando.

![](/docs/images/img3.jpg)

Verifique se todos os servidores do cluster estão disponíveis.

![](/docs/images/img4.jpg)


# **Conclusão**

Esse projeto tem como objetivo fazer a instalação e configuração do cluster kubernetes com 3 servidores,
1 master e 2 workers.

![](/docs/images/img5.jpg)
