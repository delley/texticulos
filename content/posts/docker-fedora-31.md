---
title: "Instalando e configurando o Docker CE no Fedora 31"
date: 2019-12-14
slug: docker-fedora-31
tags: ["docker", "cgroup v2"]
categories: ["ferramentas"]
series: ["Docker"]
draft: false
---

Recentemente resolvi dar uma atualizada no Sistema Operacional do meu notebook. Já fazia cerca de um ano que estava usando a versão 29 do Fedora.

No dia 29 de outubro de 2019, o Fedora 31 foi lançado e confesso que fiquei muito entusiasmado em experimentá-lo. Então, resolvi fazer uma instalação limpa e reinstalar todos os softwares que utilizo no dia-a-dia.

Um software que utilizo bastante é o [Docker](https://www.docker.com/), e é possível instalá-lo seguindo os passos fornecidos na [documentação](https://docs.docker.com/install/linux/docker-ce/fedora/). Porém, devido a alguns inconvenientes, os passos descritos aqui podem salvar o seu dia!!! :punch:

## cgroup v2

O Fedora 31 é a primeira grande distro que vem com o `cgroup v2` (também conhecido como hierarquia unificada) habilitada por padrão (mais informações [aqui](https://www.redhat.com/sysadmin/fedora-31-control-group-v2)).

Embora a adoção do `cgroup v2` seja um passo inevitável, a maioria das implementações de contêineres, incluindo **Docker**/**Moby** e **Kubernetes**, ainda não oferecem suporte e se você instalar e tentar executar o Docker, por exemplo, irá se deparar com o seguinte erro:

```bash
$ docker run hello-world
...
docker: Error response from daemon: OCI runtime create failed: container_linux.go:346: starting container process caused "process_linux.go:297: applying cgroup configuration for process caused \"open /sys/fs/cgroup/docker/cpuset.cpus.effective: no such file or directory\"": unknown.
```

## Passos para a instalação e configuração do Docker no Fedora 31

Para resolver o problema com o `cgroup v2`, é necessário adicionar alguns passos extras nas instruções fornecidas na documentação oficial.

### 1. Alterne para cgroup v1:

A primeira coisa a ser feita é alterar o cgroup para a v1.

Isso pode ser feito por meio do utilitário `grubby`:

```bash
$ sudo dnf install grubby
$ sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=0"
```

Após isso, reinicie o sistema e siga com os passos seguintes para instalar o Docker CE (caso já o tenha instalado, não é necessário seguir os próximos passos)

### 2. Adicione o repositório do Fedora Docker CE

Uma das formas mais simples de instalar o Docker CE é por meio do repositório do Fedora Docker CE. Para adicioná-lo a seu sistema, execute o seguinte comando:

```bash
$ sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
```

### 3. Instale o Docker CE

Após adicionar o repositório, siga com a instalação do Docker CE (isso também irá instalar o `docker-ce-cli` e o `containerd.io`). Para isso, execute o seguinte comando:

```bash
$ sudo dnf install docker-ce
```

### 4. Habilite e inicie o daemon do Docker:
Para habilitar o daemon e iniciá-lo durante a inicialização do sistema, execute o seguinte comando:

```bash
$ sudo systemctl enable --now docker
```

### 5. Opcional

Você também pode adicionar seu usuário ao grupo `docker` para não precisar executá-lo com privilégios de superusuário:

```bash
$ sudo usermod -aG docker $(whoami)
```
Após executar o comando, é necessário reiniciar o sistema.

---

Bom... é isso!

Espero que a dica tenha sido útil e até a próxima! :metal:

