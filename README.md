# Configurando um Ambiente Docker Swarm com Vagrant

## Visão geral

Este projeto, fornecerá e explicará o passo a passo de como configurar um ambiente Docker Swarm usando o Vagrant, utilizando três máquinas virtuais (um node gerenciador e dois nodes trabalhadores) com sistema operacional CentOS 7. Além disso, será instalado o Docker e o Docker Compose em cada máquina virtual.

<!-- ## Primeiros passos

1. Clone ou baixe este projeto para o seu computador local.
2. Instale o Vagrant e o VirtualBox, se ainda não tiver feito.
3. Navegue até o diretório do projeto que contém o arquivo `Vagrantfile` e o script `provision.sh`. -->

<!-- Este projeto fornece uma configuração Vagrant para criar um cluster Docker Swarm com três nós: um gerenciador (manager) e dois trabalhadores (workers). O arquivo `Vagrantfile` define as máquinas virtuais e o script `provision.sh` instala o Docker e o Docker Compose em cada nó. -->

<!-- ## Requisitos

Antes de começar, certifique-se de ter os seguintes softwares instalados em sua máquina:

- [Vagrant](https://www.vagrantup.com/downloads.html)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) -->

## Configurando o Ambiente

### Passo 1: Preparando o Vagrantfile

Primeiro, crie um diretório para o projeto e dentro dele crie um arquivo chamado `Vagrantfile`. Esse arquivo é responsável por definir as configurações das máquinas virtuais que serão criadas pelo Vagrant.

Aqui está o exemplo do `Vagrantfile` que utilizaremos:

```ruby
Vagrant.configure("2") do |config|
    config.vm.provision "shell", inline: "echo config docker swarm cluster"

    config.vm.define "manager" do |manager|
        manager.vm.box = "centos/7"
        manager.vm.hostname = "manager"
        manager.vm.provision "shell", path: "provision.sh"
        manager.vm.network "private_network", ip: "192.168.1.2"
        manager.vm.network "forwarded_port", guest: 80, host: 8090
    end

    config.vm.define "worker1" do |worker1|
        worker1.vm.box = "centos/7"
        worker1.vm.hostname = "worker1"
        worker1.vm.provision "shell", path: "provision.sh"
        worker1.vm.network "private_network", ip: "192.168.1.2"
    end

    config.vm.define "worker2" do |worker2|
        worker2.vm.box = "centos/7"
        worker2.vm.hostname = "worker2"
        worker2.vm.provision "shell", path: "provision.sh"
        worker2.vm.network "private_network", ip: "192.168.1.2"
    end
end
```

Neste arquivo, definimos três máquinas virtuais: uma para o node gerenciador chamada "manager" e duas para os nodes trabalhadores chamadas "worker1" e "worker2". Cada máquina terá uma instalação do CentOS 7.

### Passo 2: Criando o Script de Provisionamento

Agora, vamos criar o arquivo `provision.sh`, que será responsável por instalar o Docker e o Docker Compose nas máquinas virtuais. Crie um novo arquivo chamado `provision.sh` no mesmo diretório do `Vagrantfile` e adicione o seguinte conteúdo:

```bash

#/bin/bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

```

Este script instala o Docker e o Docker Compose na máquina virtual.

### Passo 3: Iniciando as Máquinas Virtuais

Após preparar o `Vagrantfile` e o arquivo de provisionamento, abra o terminal, navegue até o diretório do projeto e execute o seguinte comando para iniciar as máquinas virtuais:

```bash
$ vagrant up
```

O Vagrant irá ler o arquivo `Vagrantfile`, criar as máquinas virtuais e executar o script de provisionamento em cada uma delas.

### Passo 4: Configurando o Ambiente Docker Swarm

Com as máquinas virtuais iniciadas, vamos configurar o ambiente Docker Swarm. Primeiro, acesse o node manager executando o seguinte comando:

```bash
$ vagrant ssh manager
```

Dentro do node manager, inicialize o Docker Swarm:

```bash
$ docker swarm init --advertise-addr 192.168.1.2
```

Onde 192.168.1.2 é o endereço IP do node manager na rede privada.

Em seguida, copie o token gerado pelo comando anterior, pois precisaremos dele para adicionar os nodes workers ao cluster. O token tem uma aparência semelhante a isso:

```plaintext
SWMTKN-1-1asdfjwlqfmkjrlaskdjfwe124kmsd...
```

### Passo 5: Adicionando Nós Trabalhadores ao Cluster

Agora, acesse cada um dos node worker (máquinas virtuais "worker1" e "worker2") usando o seguinte comando:

```bash
$ vagrant ssh worker1
```

ou

```bash
$ vagrant ssh worker2
```

Dentro de cada node worker, adicione-o ao cluster Docker Swarm usando o token que foi copiado anteriormente:

```bash
$ docker swarm join --token <TOKEN> 192.168.1.2:2377
```

Substitua `<TOKEN>` pelo token copiado anteriormente e 192.168.1.2 pelo endereço IP do node manager.

### Passo 6: Verificando o Cluster

De volta ao node manager , verifique se todos os nodes foram adicionados ao cluster executando o seguinte comando:

```bash
$ docker node ls
```

Este comando deve exibir uma lista dos nodes gerenciados pelo Docker Swarm, incluindo o node manager e os nodes workers.
