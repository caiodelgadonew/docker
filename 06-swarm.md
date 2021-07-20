# Docker Swarm

Docker Swarm é uma ferramenta de orquestração de containers, que possibilita o usuário a gerenciar multiplos containers distribuidos em diversas máquinas hosts. Habilitando ferramentas para dimensionamento, rede, proteção e manutenção dos containers.

## Conceitos

**Cluster** 

Cluster é um conjunto de computadores que trabalham em grupo de modo a ser visto como um sistema único.

Utilizamos clusters para ganhar mais poder computacional e maior confiabilidade orquestrando um número de máquinas de "baixo custo" para obter um maior poder de processamento.


**Node** 

Utilizamos o nome Node, ou nó,  quando nos referimos a qualquer elemento computacional que faz parte de um cluster, seja ele um nó do tipo primário (master) ou um do tipo secundário (follower).

Iremos nos referir aos nossos "computadores¨  como Master/Node

Em um passado não tão distante, os termos _Master/Slave_ eram utilizados para se referir aos nós e as ações na qual eles desempenhavam. Em 2020 um forte movimento aconteceu para acabar com essa terminologia, uma vez que ela traz referências a escravidão. Não é sobre aguardar alguem ser ofendido. É sobre remover estes termos com base em coisas terríveis e desumanas.

Algumas alternativas a serem utilizadas para se referir aos nodes:


* Main / Secondary
* Conductor/ Follower
* Leader / Follower
* Host / Client
* Sender / Reciever
* Producer / Consumer
* Primary / Replica
* Primary / Secondary
* Manager / Worker

## Raft Consensus


Quando trabalhamos com sistemas distribuidos, precisamos de algum algorítmo para tratar da confiabilidade do cluster. Um dos algoritmos mais utilizados para este meio é o Raft Consensus.

O Docker Swarm implementa o algorítmo Raft Consensus para gerencia do estado global do cluster.

O nome Raft é uma palavra em inglês que quer dizer Jangada, a referênca é porque para montarmos uma jangada, necessitamos de, ao menos, três toras de madeira.

Consensus é uma palavra em ingles que quer dizer Consenso. É também um dos problemas fundamentais em sistemas distribuidos com tolerância a falhas. Involve multiplos servidores aceitando valores. 

O site [The Secret Lives of Data](http://thesecretlivesofdata.com/raft/) possúi um ótimo guia sobre como ocorre o raft consensus.

Temos também uma página no github dedicado ao [raft](https://raft.github.io/)

Quando temos um sistema composto por um único nó, podemos dizer que este nó armazena um valor único, temos também o cliente que envia este valor para o servidor, neste ponto o cliente e o servidor entram em um "consensus", um valor é simples com um único nó.

Quando temos diversos servidores, como podemos chegar em um consenso ? esse é o problema de sistemas distribuidos.

Como chegamos em um concenso em multiplos nodes? Utilizando algoritmos como o Raft Consensus.

Pensando de uma maneira mais macro, o raft define os nós em três estados básicos:

* Follower
* Candidate
* Leader

Todos os nós começam em um estado de seguidor, se os seguidores não encontrarem um líder, eles se tornam candidatos.

O candidato requisita votos de outros nós e os nós respondem com o voto, o candidato vira um lider se ele tiver os votos de uma maioria dos nodes. Esse processo é chamado de Eleição do Líder


O problema básico evitado ao se utilizar o raft consensus é chamado de `split-brain` ou cérebro dividido, e isto acontece da seguinte maneira:

> Estamos falando apenas de nodes do tipo Master, ou primário.

O Raft Consensus tolera até `(N-1)/2` falhas, e precisa de um quórum de `(N/2)+1` para ser funcional.
> Tratamos apenas numeros inteiros.

Imagine que temos 2 nodes. Caso um dos nodes perca a comunicação (rede), como ele irá saber se: 
* Ele está sem conectividade? 
* O outro nó está sem conectividade? 

Esta ação é chamada de `split-brain`, e o sistema se torna inoperante.

Vamos para o mesmo cenário com 3 nós, onde um node perde a comunicação (rede). O que ocorre neste momento é:
1. Os nodes começam a se perguntar (rede) se os outros nós estão operantes.
2. Os 2 nodes que estão operantes se comunicam e verificam que são a maioria `(N/2)+1` ou seja `(3/2)+1 = 2` e continuam se comunicando.
3. O nó que está inoperante verifica que ele não é a maioria, e para de responder.

Qual seria o problema que poderiamos encontrar caso o nó que ficou sem conectividade continue a operar? Teriamos duas versões das aplicações rodando, uma em cada lado dos servidores, com conteudos diferentes.

| Nós | Tolerância |
| --- | ---------- |
|  1  |      0     |
|  3  |      1     |
|  5  |      2     |
|  7  |      3     |
|  9  |      4     |

Não é comum termos mais de 5 nodes, uma vez que a comunicação entre os nós precisa ser feita de maneira rápida e eficiente, e ao aumentar o número de nodes fazemos com que o volume de replicação de dados aumente, tornando comum a utilização de `3 ou 5` nodes

Mais informações na [Documentação Oficial](https://docs.docker.com/engine/swarm/raft/)

## Criando o Cluster

Trabalharemos com as máquinas `master` `node-01` e `node-02`

Ligue as máquinas
```bash
$ vagrant up master node01 node02
``` 

Acesse a máquina master
```bash
$ vagrant ssh master
```

Para criar o cluster do swarm, precisamos apenas de que o docker esteja instalado na máquina, podemos em seguida executar o comando para verificar seu endereço IP e iniciar o cluster swarm

```bash
$ ip -c -br a show enp0s8
$ docker swarm init --advertise-addr 10.20.20.100
```

Será exibida uma mensagem informando que o swarm foi iniciado e o node é um manager, bem como o token para adicionar mais nós ao swarm.

```bash
Swarm initialized: current node (sqtbdppo34ez4i4wsngbdeply) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4n7u9o6cmhoizpx57umbp1g8nkh5zetuimx06k20nme64syy0t-1x0yu5ogxdlu2b0fs7fcnsc91 10.20.20.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

Podemos ver novamente o token para adicionar novos nós ao cluster através do comando `docker swarm join-token <manager/worker>`
```bash
$ docker swarm join-token manager
$ docker swarm join-token worker
```

## Adicionando nós ao cluster.


Na máquina master, copie o token de `worker`

```bash
$ docker swarm join-token worker
```

Em um novo terminal, acesse a máquina `node01` e execute o comando copiado anteriormente
```bash
$ vagrant ssh node01
$ docker swarm join --token SWMTKN-1-4n7u9o6cmhoizpx57umbp1g8nkh5zetuimx06k20nme64syy0t-1x0yu5ogxdlu2b0fs7fcnsc91 10.20.20.100:2377
```

Repita o passo para a máquina `node02`
```bash
$ vagrant ssh node02
$ docker swarm join --token SWMTKN-1-4n7u9o6cmhoizpx57umbp1g8nkh5zetuimx06k20nme64syy0t-1x0yu5ogxdlu2b0fs7fcnsc91 10.20.20.100:2377
```

De volta a máquina master, vamos verificar se os nós foram adicionados corretamente.
```bash
$ docker node ls
```

Verifique que o node que estamos atualmente conectados é informado com um `*` e podemos ver também quais nodes são `manager` ou `lider`

```bash
ID                            HOSTNAME                    STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
sqtbdppo34ez4i4wsngbdeply *   master                      Ready     Active         Leader           20.10.7
o122d88a3dwxdkw6nyopgmqu7     node01                      Ready     Active                          20.10.7
tj57th3ri0zouv1f8ui90zv0n     node02.docker-dca.example   Ready     Active                          20.10.7
```

> os comandos `docker node` só podem ser executados em nodes do tipo `manager`

## Promovendo um node a Manager ou rebaixando a worker

Podemos promover um node a manager ou rebaixa-lo através do comando `docker node promote` e `docker node demote`

Na máquina master, vamos promover o node01 a manager.
```bash
$ docker node ls
$ docker node promote node01
$ docker node ls
``` 

Note que seu `Manager Status` se tornou `Reachable`, ou seja, ele está disponível, caso a máquina esteja sem comunicação, ela ficará no estado de `Unreachable`

```bash
ID                            HOSTNAME                    STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
sqtbdppo34ez4i4wsngbdeply *   master                      Ready     Active         Leader           20.10.7
o122d88a3dwxdkw6nyopgmqu7     node01                      Ready     Active         Reachable        20.10.7
tj57th3ri0zouv1f8ui90zv0n     node02.docker-dca.example   Ready     Active                          20.10.7
```

Caso a máquina fique offline veremos o nosso cluster entrar em estado de `split-brain` exibindo a seguinte mensagem ao executar o comando `docker node ls`
```bash
Error response from daemon: rpc error: code = Unknown desc = The swarm does not have a leader. It's possible that too few managers are online. Make sure more than half of the managers are online.
```

Vamos agora rebaixar nosso node a worker.
```bash
$ docker node ls
$ docker node demote node01
$ docker node ls
``` 

> Trabalharemos com apenas um manager pois como falamos na etapa do raft consensus, ter dois managers é pior do que ter apenas um.


## Services e Tasks

Quando falamos de Swarm, precisamos entender dois recursos importantes, os serviços e as tasks

**Services** -> É a definição de um estado desejado.
**Tasks** -> É criada a partir de um serviço que da origem aos containers a serem executados nos nodes.

Quando utilizamos os serviços descrevemos o estado desejado, como por exemplo:

![services diagram](img/06/services-diagram.png)

Com isto o swarm manager irá criar as tasks e dividi-las entre os nodes disponíveis, de forma a atender, se possível, o estado desejado.

https://docs.docker.com/engine/swarm/how-swarm-mode-works/swarm-task-states/

https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/

https://docs.docker.com/engine/reference/commandline/service/