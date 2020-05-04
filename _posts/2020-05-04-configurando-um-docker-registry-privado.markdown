---
layout: post
title:  "Como configurar um Docker Registry Privado"
date:   2020-05-04 10:55:36 +0530
categories: Docker Devops
---

Muitas pessoas usam o Dockerhub como um registro central para armazenar imagens públicas do Docker. No entanto, para controlar o acesso às suas imagens, é necessário usar um registro privado do ***Docker Registry***. O ***Docker Registry*** privado é um repositório para armazenamento de suas imagens usadas na criação de contêiners. Um ***Docker Registry*** privado do Docker permite que você compartilhe suas imagens, mantendo uma fonte consistente, privada e centralizada de verdade com os elementos básicos de sua arquitetura. Um ***Docker Registry*** pode ser considerado privado se na requisição `pull`, o mesmo exigir autenticação.

Este post será sobre a instação de ***Docker Registry*** privado, trata-se mais de uma anotação de um trabalho realizado.

Pré requisitos:

- Sistema operacional CentOS 7 x86_64 instalação mínima e atualizado;
- Docker Engine [Docker Engine](https://docs.docker.com/engine/install/centos/) ;
- Docker Registry  [Docker Registry](https://docs.docker.com/registry/configuration/) ;
