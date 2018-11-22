# **Configuração do ambiente Kubernetes Cluster com Ansible**

### **Descrição**

Instalação e configuração dos requisitos para configurar um cluster kubernetes
com ansible.

#### **Requisitos:**
 
 - OpenSSH
 - Ansible 2.6.+
 - Conexão livre entre os servidores.


### **./inventories/staging/hosts**

Configure o arquivo ```./inventories/staging/hosts```

```bash
vim ./inventories/staging/hosts

[cluster-k8s]
<server-master>     # Hostname ou IP do servidor master.
<server-workder-1>  # Hostname ou IP do servidor worker-1.
<server-workder-2>  # Hostname ou IP do servidor workder-2.
```

### **/etc/hosts**

Configure o arquivo ```/etc/hosts``` com IP e hostname dos servidores que compoem
o cluster.

```bash
echo "<IP>  <MASTER>" >> "/etc/hosts"
echo "<IP>  <WORKER-1>" >> "/etc/hosts"
echo "<IP>  <WORKER-2>" >> "/etc/hosts" 
```


Dentro da pasta do projeto execute a instalação.

Com o parametro ```-C``` você testa se todo o processo irá executar sem erro.
Caso não aconteça apareça nenhum erro, execute o mesmo comando removendo o parametro.

**Teste:**
```bash
ansible-playbook -i inventories/stagins/hosts -l cluster-k8s -k site.yml -C
```

**Execução:**
```bash
ansible-playbook -i inventories/stagins/hosts -l cluster-k8s -k site.yml
```

![]("./docs/images/img1.jpg")