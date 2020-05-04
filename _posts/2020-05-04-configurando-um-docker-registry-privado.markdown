---
layout: post
title:  "Como configurar um Docker Registry Privado"
date:   2020-05-04 10:55:36 +0530
categories: Docker Devops
---

Muitas pessoas usam o Dockerhub como um registro central para armazenar imagens públicas do Docker. No entanto, para controlar o acesso às suas imagens, é necessário usar um registro privado do ***Docker Registry***. 

O ***Docker Registry*** privado é um repositório para armazenamento de suas imagens usadas na criação de contêiners. Um ***Docker Registry*** privado do Docker permite que você compartilhe suas imagens, mantendo uma fonte consistente, privada e centralizada de verdade com os elementos básicos de sua arquitetura. Um ***Docker Registry*** pode ser considerado privado, se na requisição `pull` o mesmo exigir autenticação.

Este post será sobre a instação de ***Docker Registry*** privado, trata-se mais de uma anotação de um trabalho realizado.

Pré requisitos:

- Sistema operacional CentOS 7 x86_64 instalação mínima e atualizado;
- [Docker Engine](https://docs.docker.com/engine/install/centos/) ;
- [Docker Registry](https://docs.docker.com/registry/configuration/) ;


Para começarmos vamos cria a pasta `/var/docker_data/images` que armazenará nossos certificados.

```bash
# mkdir -p /var/docker_data/certs/
```

Criando a pasta onde o Docker daemon necessita ter os certificados criados.

```bash
# mkdir -p /etc/docker/certs.d/meu.servidor.com.br:5000/
```

Agora criaremos nosso certificado com validade de 1 ano.

```bash
# openssl req \
	-newkey rsa:4096 -nodes -sha256 -keyout /var/docker_data/certs/domain.key \
	-x509 -days 365 -out /var/docker_data/certs/domain.crt
```
---
```bash
Generating a RSA private key
.......................................
....................................++++
..................++++
writing new private key to '/var/docker_data/certs/domain.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:BR
State or Province Name (full name) [Some-State]:<enter>
Locality Name (eg, city) []:<enter>
Organization Name (eg, company) [Internet Widgits Pty Ltd]:<enter>
Organizational Unit Name (eg, section) []:<enter>
Common Name (e.g. server FQDN or YOUR name) []:meu.servidor.com.br
Email Address []:fulano@meu.servidor.com.br
```
---
```bash
# ls -lha /var/docker_data/certs/
total 8.0K
drwxr-xr-x 2 root docker   42 Mar 16 15:05 .
drwxr-xr-x 3 root docker   19 Mar 16 14:56 ..
-rw-r--r-- 1 root root   2.2K Mar 16 15:05 domain.crt
-rw------- 1 root root   3.2K Mar 16 15:01 domain.key
```

Para que o Docker daemon confie nesses certificado, será necessário copiar o arquivo domain.crt para `/etc/docker/certs.d/meu.servidor.com.br:5000/ca.crt` do host do Docker. Você não precisa reiniciar o Docker.

```bash
# cp /var/docker_data/certs/domain.crt /etc/docker/certs.d/meu.servidor.com.br:5000/ca.crt

# cp /var/docker_data/certs/domain.crt /etc/pki/ca-trust/source/anchors/ca.crt

# update-ca-trust

# service docker restart
```

Criando a pasta de armazenamento das imagens
```bash
# mkdir -p /var/docker_data/images
```

Buscando a imagem oficial do Docker ***Registry*** .
```bash
# docker search registry
# docker search registry
NAME                     DESCRIPTION                                     STARS      OFFICIAL            AUTOMATED
registry                 The Docker Registry 2.0 implementation for s…   2881       [OK]
distribution/registry    WARNING: NOT the registry official image!!! …   57                             [OK]
```

Subindo um container do ***Docker Registry*** com as pastas criadas mapeadas
```bash
# docker run -d -p 5000:5000 \
	-v /var/docker_data/images:/var/lib/registry \
	-v /var/docker_data/certs:/certs \
	-e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
	-e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
	--restart=always \
	--name meu-registry \
	registry
```

Verificando se o container subiu.
```bash
# docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED             STATUS          PORTS   NAMES
b9b94eb65745   registry     "/entrypoint.sh /etc…"   3 minutes ago       2 minutes ago           meu-registry
```

Podemos criar uma imagem baixando do Docker Hub ou a partir de um arquivo Dockerfile. Neste caso, criaremos nossa imagem base utilizando a oficial `CentOS` <br/>
na versão `7.6.1810` baixando diretamente do Docker Hub e armazenando no nosso ***Docker Registry*** privado. 

Criando uma TAG para a imagem.
```bash
# docker tag centos:7.6.1810 meu.servidor.com.br:5000/centos:1.0
```

Realizando o push da TAG criada, como uma imagem localmente.
```bash
# docker push meu.servidor.com.br:5000/centos:1.0
The push refers to repository [meu.servidor.com.br:5000/centos]
55a77731ed26: Pushed
71f2244bc14d: Pushed
f2cb0ecef392: Pushed
latest: digest: sha256:3936fb3946790d711a68c58be93628e43cbca72439079e16d154b5db216b58da size: 948
```

Verificando se a imagem foi armazenada.
```bash
# ls -lha /var/docker_data/images/docker/registry/v2/repositories/
total 0
drwxr-xr-x 3 root root 20 Mar 16 17:55 .
drwxr-xr-x 4 root root 39 Mar 16 17:55 ..
drwxr-xr-x 5 root root 55 Mar 16 17:55 centos
```

Acessando a url `https://meu.servidor.com.br:5000/v2/_catalog` é possível observar o catálago de imagens privado.

```json
{"repositories":["centos","httpd-private-registry","my-nginx"]}
```

Existem várias ferramentas que fornecem serviços de gerenciamento de imagens em ***Docker Registry*** privado, entre elas eu recomendo o [Portus](http://port.us.org/) . 

**Portus** é um serviço de autorização de código aberto e uma interface de usuário para o ***Docker Registry***. É um aplicativo `on-premise` que permite aos usuários administrar e proteger seus registros do Docker.


Agora que o serviço está disponibilizado, devemos configurar os `Clients` para **enxergar** o ***Docker Registry*** privado.Para isso, é necessário copiar o certificado criado no ***Docker Registry*** privado e adicionar o endereço do servidor ao `/etc/hosts` das máquinas `Clients`.

Adicionando o endereço do servidor `meu.servidor.com.br` na máquina `192.168.1.53`. 
```bash
# mkdir -p /etc/docker/certs.d/meu.servidor.com.br:5000
# echo 192.168.1.52 meu.servidor.com.br >> /etc/hosts
```

Copiando o certificado do servidor `meu.servidor.com.br` para a máquina `192.168.1.53`.
```bash
# scp -prvC /var/docker_data/certs/domain.crt root@192.168.1.53:/etc/docker/certs.d/meu.servidor.com.br:5000/

# docker pull meu.servidor.com.br:5000/my-nginx:latest
latest: Pulling from my-nginx
68ced04f60ab: Already exists
28252775b295: Pull complete
a616aa3b0bf2: Pull complete
Digest: sha256:3936fb3946790d711a68c58be93628e43cbca72439079e16d154b5db216b58da
Status: Downloaded newer image for meu.servidor.com.br:5000/my-nginx:latest
meu.servidor.com.br:5000/my-nginx:latest
```

Agora podemos testar a criação de uma `TAG` criada na máquina `192.168.1.53`, subir ela para o ***Docker Registry*** privado e baixar a mesma na máquina `192.168.1.54`.
  
```bash
# docker tag meu.servidor.com.br:5000/centos:1.0 meu.servidor.com.br:5000/httpd-private-registry:1.1

docker push meu.servidor.com.br:5000/httpd-private-registry:1.1

The push refers to repository [meu.servidor.com.br:5000/httpd-private-registry]
25a92d79dbfe: Pushed
b5432b464616: Pushed
e6699b4fc2e3: Pushed
762ba19e7ef1: Pushed
f2cb0ecef392: Mounted from my-nginx
1.1: digest: sha256:d3df077ec2ddbe0a62279c672b9c792055b96f6d22ed1e45371bcd70393730f9 size: 1367
```

Verificando se a imagem foi armazenada no `meu.servidor.com.br`.

```bash
# ls -lha /var/docker_data/images/docker/registry/v2/repositories/
total 0
drwxr-xr-x 5 root root 66 Mar 17 13:45 .
drwxr-xr-x 4 root root 39 Mar 16 17:55 ..
drwxr-xr-x 5 root root 55 Mar 16 17:55 centos
drwxr-xr-x 5 root root 55 Mar 17 13:46 httpd-private-registry
drwxr-xr-x 5 root root 55 Mar 17 11:00 my-nginx
```

Na máquina `192.168.1.54`, realizamos os mesmos procedimentos da máquina client `192.168.1.53`.
Adicionando o endereço do servidor `meu.servidor.com.br` na máquina `192.168.1.54`. 

```bash
# mkdir -p /etc/docker/certs.d/meu.servidor.com.br:5000
# echo 192.168.1.52 meu.servidor.com.br >> /etc/hosts
```

Copiando o certificado do servidor para a máquina `192.168.1.54`.
```bash
# scp -prvC /var/docker_data/certs/domain.crt root@192.168.1.54:/etc/docker/certs.d/meu.servidor.com.br:5000/
```
Baixando a imagem criada na máquina `192.168.1.53` e armazenada no servidor `meu.servidor.com.br`. 
```bash
# docker pull meu.servidor.com.br:5000/httpd-private-registry:1.1
1.1: Pulling from httpd-private-registry
68ced04f60ab: Already exists
35d35f1e0dc9: Pull complete
8a918bf0ae55: Pull complete
d7b9f2dbc195: Pull complete
d56c468bde81: Pull complete
Digest: sha256:d3df077ec2ddbe0a62279c672b9c792055b96f6d22ed1e45371bcd70393730f9
Status: Downloaded newer image for meu.servidor.com.br:5000/httpd-private-registry:1.1
meu.servidor.com.br:5000/httpd-private-registry:1.1
```

Verificando a imagem na máquina client.
```bash
# docker images
REPOSITORY                                        TAG     IMAGE ID            CREATED             SIZE

meu.servidor.com.br:5000/httpd-private-registry   1.1     c5a012f9cf45        2 weeks ago         165MB
```
 
<div style="background-color: #f5f0ff; border: 1px #e1e4e8 solid;padding: 16px;">
O Docker Registry por default trabalha com https, dessa forma foi gerado um certificados SSL para o Docker Registry privado. É recomendado a utilização de certificados válidos, neste post foi utilizado um certificado auto-assinado para mera exemplificação.<div>

<br/>
Com o procedimento e testes finalizados, temos agora um repositório de imagens Docker privado para utilização em projetos corporativos.
