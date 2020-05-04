---
layout: post
title:  "Como configurar um Docker Registry Privado"
date:   2020-05-04 10:55:36 +0530
categories: Docker Devops
---

Muitas pessoas usam o Dockerhub como um registro central para armazenar imagens públicas do Docker. No entanto, para controlar o acesso às suas imagens, é necessário usar um registro privado do ***Docker Registry***. O ***Docker Registry*** privado é um repositório para armazenamento de suas imagens usadas na criação de contêiners. Um ***Docker Registry*** privado do Docker permite que você compartilhe suas imagens, mantendo uma fonte consistente, privada e centralizada de verdade com os elementos básicos de sua arquitetura. Um ***Docker Registry*** pode ser considerado privado, se na requisição `pull` o mesmo exigir autenticação.

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
NAME                                 DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
registry                             The Docker Registry 2.0 implementation for s…   2881                [OK]
distribution/registry                WARNING: NOT the registry official image!!! …   57                                      [OK]
```

Subindo um container do ***Docker Registry*** com as pastas criadas mapeadas
```bash
# docker run -d -p 5000:5000 \
	-v /var/docker_data/images:/var/lib/registry \
	-v /var/docker_data/certs:/certs \
	-e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
	-e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
	--restart on-failure \
	--name meu-registry \
	registry
```
 
