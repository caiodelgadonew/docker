# Curso Docker DCA 
**Curso 100% Gratuito!**

Material do Curso  

[![caiodelgadonew/docker](https://img.shields.io/github/stars/caiodelgadonew/docker?label=caiodelgadonew%2Fdocker&logo=github&style=for-the-badge)](https://github.com/caiodelgadonew/docker)

Redes Sociais

[![Linktr.ee](https://img.shields.io/website?down_message=caiodelgadonew&label=LINKTR.EE&logo=linktree&style=for-the-badge&up_message=caiodelgadonew&url=https%3A%2F%2Flinktr.ee%2Fcaiodelgadonew)](https://linktr.ee/caiodelgadonew)


O conteúdo será ministrado ao vivo no canal do Youtube

[![Youtube Channel Subscribers](https://img.shields.io/youtube/channel/subscribers/UCQnpN5AUd36lnMHuIl_rihA?label=YOUTUBE&logo=youtube&style=for-the-badge&logoColor=red)](https://www.youtube.com/caiodelgadonew) 

## Preparação do desktop

> **IMPORTANTE!** - Antes de começar assista o vídeo [Curso Docker Grátis - Importante](https://youtu.be/RIfSgnWkdgo)

1. Assistir o vídeo  [Vagrant 101 - Utilizando Infraestrutura como Código para Desenvolvimento e Estudo](https://youtu.be/PX6OmeIbjC4)

2. Garantir que o [Intel VMX/VT](https://www.asus.com/br/support/FAQ/1043786/) ou [AMD-V/SVM](https://www.asus.com/br/support/FAQ/1038245/) está habilitado na BIOS

3. Instalar os pacotes:
  - [Virtualbox](https://www.virtualbox.org/wiki/Downloads)
  - [Vagrant](https://www.vagrantup.com/downloads)
  - [git](https://git-scm.com/download/)

4. Clonar este repositório
```bash
git clone https://github.com/caiodelgadonew/docker-dca.git
```

5. Fazer o Download das Vagrant Boxes
```bash
$ vagrant box add --provider virtualbox ubuntu/bionic64 
$ vagrant box add --provider virtualbox centos/7 
``` 
