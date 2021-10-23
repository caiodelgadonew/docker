# Capitulo 10 - Enterprise

Toda a parte enterprise da certificação Docker DCA está relacionado a sua plataforma chamada **MKE** ou ** Mirantis Kubernetes Engine**


## Mirantis Kubernetes Engine

O **MKE** é uma plataforma que anteriormente era chamada de **Docker Enterprise** ou **UCP**. Ela fornece uma maneira de gerenciamento de clusters e deploy de aplicações através do uso de Docker Swarm e Kubernetes.


O **MKE** é composto de um conjunto de diversas ferramentas como:
* **Universal Control Plane - UCP** - Interface de Gerenciamento Web
* **Calico** - Solução open source de redes e segurança de redes
* **Istio** - Ingress Gateway para o cluster
* **Mirantis Secure Registry** - Registry com features de segurança (anteriormente conhecido como **dtr**)

### Requisitos Mínimos

* 8GB of RAM para nodes manager
* 4GB of RAM para nodes worker
* 2 vCPUs para nodes manager
* 25GB de disco livre.

### Requisitos para Produção 

* 16GB of RAM para nodes manager
* 4 vCPUs para nodes manager
* 25~100GB de disco livre. Recomendado o uso de SSD.

> **ATENÇÃO**: Caso não tenha os requisitos mínimos para executar o **MKE** em sua máquina, não se preocupe, a prova é teórica e você precisa entender apenas o funcionamento e os conceitos.

## Instalando o MKE

Começaremos alterando nosso `Vagrantfile` para as configurações mínimas.

```bash
$ vim Vagrantfile
```

```hcl
machines = {
   "master"   => {"memory" => "8096", "cpu" => "2", "ip" => "100", "image" => "ubuntu/bionic64"},
   "node01"   => {"memory" => "4096", "cpu" => "2", "ip" => "110", "image" => "ubuntu/bionic64"},
   "node02"   => {"memory" => "4096", "cpu" => "2", "ip" => "120", "image" => "centos/7"},
   "registry" => {"memory" => "4096", "cpu" => "2", "ip" => "200", "image" => "ubuntu/bionic64"}
}
```

Recrie as máquinas através do comando `vagrant up` e faça a instalação do Docker em todas as máquinas

Agora que temos nossas máquinas disponíveis vamos instalar o MKE na máquina master, para isto iremos acessar a máquina e executar o container do `mirantis/ucp` para verificar a ultima versão disponível

```bash
$ vagrant ssh master
$ docker container run --rm -it --name ucp -v /var/run/docker.sock:/var/run/docker.sock mirantis/ucp --version
```

> Como se trata de uma solução de gerenciamento, precisamos que o `docker.sock` esteja exposto para o container.

```bash
docker run --rm -it \
        -v /var/run/docker.sock:/var/run/docker.sock \
        mirantis/ucp \
         version 3.4.6 (1b0bd58)
``` 

Agora que temos a versão atual do nosso UCP podemos executar a instalação do mesmo. Como o UCP precisa de diversos outros containers, é uma boa prática ter todas as imagens já baixadas na máquina que será o manager, podemos fazer isto através do comando:

```bash
$ docker container run --rm -it --name ucp -v /var/run/docker.sock:/var/run/docker.sock mirantis/ucp:3.4.6 images --pull missing
```

> O processo é um pouco demorado pelo tamanho e quantidade de imagens que serão baixadas, note também que estamos fixando a versão do UCP na execução, isto é altamente necessário.

Após a conclusão do processo podemos executar a instalação do UCP através do comando

```bash
$  docker container run --rm -it --name ucp -v /var/run/docker.sock:/var/run/docker.sock mirantis/ucp:3.4.6 install --host-address 10.20.20.100 --interactive
```

Vamos agora responder algumas perguntas e aguardar a instalação:
```bash
Admin Username: admin
Admin password: caiodelgadonew@youtube
Confirm Admin password: caiodelgadonew@youtube
Additional Aliases:    <ENTER>
``` 

A instalação estará completa ao exibir a mensagem:
```bash
INFO[0121] All Installation Steps Completed             
``` 

## Acessando o Dashboard

Para acessar o dashboard vamos abrir o browser no endereço https://master.docker-dca.example e conectar com o usuário e senha que criamos 

![Login](resources/10login.png)

Após o login podemos enviar uma licença, requisitar uma licença de teste ou pular. Vamos clicar em `Skip For Now`

![Licença](resources/10license.png)

Será exibido o dashboard do MKE, e agora precisamos entender o funcionamento do mesmo.

![Dashboard](resources/10dashboard.png)

No dashboard temos um menu a esquerda onde temos as opções:

* **admin** - Configurações de perfil, administrador, sobre e support bundle
* **Dashboard** - Tela inicial com informações do Swarm, Kubernetes e recursos.
* **Access Control** - Controle de acesso para Organizações e Times, Usuários, Regras e Concessões
* **Shared Resources** - Collections, Stacks, Containers, Imagens, Nós
* **Kubernetes** - Todos os recursos e configurações do Kubernetes
* **Swarm** - Todos os recursos e configurações do swarm.


## Adicionando Nós

O primeiro passo é adicionar os nós restantes ao nosso cluster, o dashboard já nos entrega o comando completo para execução, basta clicar em **Add Nodes** na parte inferior do dashboard

![Add Nodes](resources/10addnodes.png)

Será exibida uma tela com todas as informações necessárias para adicionar um nó ao nosso cluster, deixaremos as opções de **node type** `Linux` e **node role** `Worker` vamos copiar o comando e executa-lo em todos os outros nós.

```bash
$ vagrant ssh node01
$ docker swarm join --token SWMTKN-1-40xlaju97ux404y2z8p5tixyy5q5ag0064uo367og5i98nzuwt-7kvbotg83csbjskcxzbfsjgnp 10.20.20.100:2377
$ exit

$ vagrant ssh node02
$ docker swarm join --token SWMTKN-1-40xlaju97ux404y2z8p5tixyy5q5ag0064uo367og5i98nzuwt-7kvbotg83csbjskcxzbfsjgnp 10.20.20.100:2377
$ exit

$ vagrant ssh registry
$ docker swarm join --token SWMTKN-1-40xlaju97ux404y2z8p5tixyy5q5ag0064uo367og5i98nzuwt-7kvbotg83csbjskcxzbfsjgnp 10.20.20.100:2377
$ exit
```

Voltando para a página do **UCP** vamos em **Shared Resources** e em seguida em **Nodes**

![Nodes](resources/10nodes.png)

Vamos aguardar o processo até que os nodes fiquem em estado de `Healthy`

![Healthy Nodes](resources/10healthynodes.png)

Voltando ao dashboard conseguimos visualizar as métricas dos nodes managers e workers.

![Node Metrics](resources/10metrics.png)

## Mirantis Secure Registry

Agora que temos os nós adicionados em nosso cluster, podemos definir o nó que será nosso **DTR** (Mirantis Secure Registry era anteriormente chamado de **DTR** Docker Trusted Registry)

Para adicionarmos o registry vamos no menu **Admin**>**Admin Settings**>**Mirantis Secure Registry**

E vamos selecionar o `UCP NODE` `registry.docker-dca.example` e marcar `Use a PEM-encoded TLS CA certificate for MKE` após isto podemos copiar o comando para `Unix Shell` e executar na máquina `registry`

![Install Registry](resources/10installdtr.png)


```bash
$ vagrant ssh registry
$ docker run -it --rm docker/dtr install --ucp-node registry.docker-dca.example --ucp-username admin --ucp-url https://master.docker-dca.example --ucp-ca  "-----BEGIN CERTIFICATE-----
MIIBfjCCASSgAwIBAgIUaRHfl3/fvCS+PhFfq/gQiEpw/8QwCgYIKoZIzj0EAwIw
HTEbMBkGA1UEAxMSVUNQIENsaWVudCBSb290IENBMB4XDTIxMTAyMzE1NDEwMFoX
DTI2MTAyMjE1NDEwMFowHTEbMBkGA1UEAxMSVUNQIENsaWVudCBSb290IENBMFkw
EwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEa4NQLljC7h3eWFdLxljHpCvXBvoEgpvk
DNdTzpnFYrpUHgdLbCCRLZCnemoWDTbAUnj1D0rN7ZnalxF8FLe5cqNCMEAwDgYD
VR0PAQH/BAQDAgEGMA8GA1UdEwEB/wQFMAMBAf8wHQYDVR0OBBYEFKC1J6c292rz
j16c85hbCTt8klgFMAoGCCqGSM49BAMCA0gAMEUCIQD4VcngZ/ZHR98nXTX2u5Ef
1P0J6zKxElbGuQhHkuToDwIgSrX4eftF4SX1byWbpwMi7Q5mpNdOUW686FweqxAq
xdw=
-----END CERTIFICATE-----
"

ucp-password: caiodelgadonew@youtube
```

A instalação será concluida quando a mensagem a seguir for exibida:

```bash
INFO[0156] Installation is complete
```

Acesse o **DTR** através do endereço https://registry.docker-dca.example/ e efetue o login com o usuário e senha de administrador

![DTR Login](resources/10dtrlogin.png)

Após o login clique em `Skip for now`

## Criando Usuários e Organizações no DTR

Na tela inicial do **DTR** clique em `Users`>`New user`

![DTR New User](resources/10dtrnewuser.png)

Coloque o nome do usuário de `analista` com a senha `caiodelgadonew@youtube` e clique em Save

![DTR New User](resources/10dtranalista.png)


clique em `Organizations`>`New organization`

![DTR New Organization](resources/10dtrorg.png)

Preencha o nome como `docker-dca` e clique em Save

![DTR New Organization](resources/10dtrsaveorg.png)

Agora que temos nosso usuário e nossa organização, podemos adicionar o usuário a organização indo em `Organizations`>`docker-dca` e em seguida clicar em `Add user`

![DTR Organization](resources/10dtruserorg.png)

Preencha os dados e adicione o usuário a organização, vamos alterar seu nivel de permissionamento para `Org Owner`

![DTR Organization](resources/10dtruseronorg.png)


## Criando Repositórios no DTR

Clique em `Repositories e em seguida em `New Repository`

![DTR New Repository](resources/10dtrnewrepo.png)


Vamos criar o repositório `nginx` como público e clicar em `Save`

![DTR Nginx](resources/10dtrnginx.png)

Após a criação do repositório podemos clicar em `View Details` para visualizar os detalhes do repositório 

![DTR Nginx](resources/10dtrdetails.png)

Na tela a seguir teremos diversas informações do repositório, mirrors e como efetuar o pull de uma nova imagem

![DTR Nginx](resources/10dtrnginxdetails.png)

## Enviando imagens para o DTR

O processo de envio de imagens para o DTR é o mesmo que utilizamos com qualquer outro registry.

Como não temos uma licença do Docker Enterprise não podemos enviar imagens para o mesmo. Uma mensagem de erro será exibida, porém iremos passar por todos os passos para o envio da imagem

Em uma das máquinas do cluster iremos adicionar o parametro `insecure-registries`,  efetuar o pull de uma imagem do nginx, efetuar o login no registry, alterar a tag da imagem e enviar para o DTR

```bash
$ vagrant ssh master
$ echo '{ "insecure-registries" : ["registry.docker-dca.example"] }' | sudo tee /etc/docker/daemon.json ; sudo systemctl restart docker
$ docker image pull nginx
$ docker image tag nginx registry.docker-dca.example/docker-dca/nginx
$ docker login registry.docker-dca.example -u analista -p caiodelgadonew@youtube
$ docker push registry.docker-dca.example/docker-dca/nginx
``` 

A mensagem de erro será exibida porque não temos uma licença válida do DTR, porém o cenário esperado seria a imagem disponível no DTR

```bash
Using default tag: latest
The push refers to repository [registry.docker-dca.example/docker-dca/nginx]
9959a332cf6e: Preparing 
f7e00b807643: Preparing 
f8e880dfc4ef: Preparing 
788e89a4d186: Preparing 
43f4e41372e4: Preparing 
e81bff2725db: Waiting 
error parsing HTTP 402 response body: invalid character 'D' looking for beginning of value: "DTR doesn't have a license\n"
```