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
- [x] Atualizando o Sistema
- [x] Verificando se o repositório Docker existe
- [x] Habilitando o Repositório do Docker
- [x] Verificando se o repositório Kubernetes existe
- [x] Habilitando o Repositório do Kubernetes
- [x] Instalando pacotes essenciais
- [x] Pip
- [x] Configurando Serviço NTP
- [x] Habilitando os Serviços
- [x] Desabilitando os Serviços
- [x] Removendo swapfile do /etc/fstab
- [x] Desabilitando a swap
- [x] Desabilitando o SELinux
- [x] Atualizando o hostname do servidor
- [x] Configurando o arquivo hosts dos servidores
- [x] Configurando módulos do kernel
- [x] Parametros de Rede para o Kernel
- [x] Aplicando os parametros de Rede para o Kernel

#### **Cluster - Master Tasks**

- [x] 

#### **Cluster - Worker Tasks**

- [x] 

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
