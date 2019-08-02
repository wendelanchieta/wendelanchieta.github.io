---
layout: post
title:  "Como montar um compartilhamento Windows no Linux usando a ferramenta CIFS"
date:   2019-08-02 21:03:36 +0530
categories: Linux cifs
---
CIFS (Common Internet File System) esta ferramenta é parte da cifs-utils suite. 
O CIFS VFS é um sistema de arquivos virtual para Linux para permitir acesso a servidores e dispositivos de armazenamento 
compatíveis com a especificação SNIA CIFS versão 1.0 ou posterior. Geralmente invocado indiretamente pelo comando mount,  
quando utiliza-se a opção ``-t cifs``.

Presente apenas no sistema operacional Linux, o kernel deste deve está preparado para suportar o cifs filesystem. O CIFS protocol é o sucessor do SMB protocol, sendo suportado pela maioria dos servidores Windows e muitos outros servidores comerciais e dispositivos de armazenamento em rede, bem como o popular servidor Open Source Samba.

Vamos a um exemplo prático de como utilizar a ferramenta.

O primeiramente é necessário a instalação do pacote cifs-utils.

# yum install cifs-utils

O próximo passo é criar o diretório local no linux, que será utilizado como ponto de montagem na maquina, neste exemplo usaremos /var/diretorioCompartilhado/. 

# mkdir /var/diretorioCompartilhado/ 

No arquivo /etc/fstab será adicionada uma linha com o comando abaixo:

# vi /etc/fstab

//192.168.1.1/diretorioCompartilhado /var/diretorioCompartilhado cifs _netdev,rsize=16384,wsize=16384,username=kettle,password=kettle@123,gid=10,uid=10,file_mode=0777 0 0

# chown -R kettle:root /var/diretorioCompartilhado/

//192.168.1.1/diretorioCompartilhado - É o IP e a pasta que está compartilhada na máquina Windows;

/var/diretorioCompartilhado - Ponto de montagem local no servidor Linux;

username=user,password=senha@123 - Especifica o nome de usuário e senha para se conectar ao servidor. Esse usuário não precisa ser exclusivamente o administrador do Windows. Pode se criar um usuário que tenha acesso a pasta;

