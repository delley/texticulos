+++ 
draft = false
date = 2020-01-30T12:26:43-03:00
title = "Go Modules: Gerenciamento de Dependências"
description = "Um pequeno textículo explicando o funcionamento do Go Modules"
slug = "go-modules" 
tags = ["Go", "Modules"]
categories = ["programação"]
series = ["Go"]
images = ["/images/go-modules.png"]
+++

{{< figure src="/images/go-modules.png" alt="Go Modules" class="centered" >}}

No últimos anos houve muita turbulência em torno do gerenciamento de dependências do Go. Vimos ferramentas como [dep](https://golang.github.io/dep/), [godep](https://github.com/tools/godep), [govendor](https://github.com/kardianos/govendor) e um monte de outras entrar em cena para tentar resolver esse problema de uma vez por todas.

## O que é o Go Modules?

> É o novo sistema de gerenciamento de dependências do Go que torna explícita e fácil o gerenciamento das informações sobre versões de dependências
>> <cite>[The Go Blog - Using Go Modules][1]</cite>

Em poucas palavras, *Go Modules* é a resposta oficial para lidarmos com o **Gerenciamento de Dependências em Go**.

## GOPATH, Configuração do projeto e ativação do Go Modules

O lançamento da versão 1.13 possibilitou a criação do diretório do projeto em qualquer lugar no computador, inclusive no diretório `GOPATH`. Em versões pré-1.13 e pós-1.11, já era possível criar o diretório em qualquer lugar, porém o recomendado era criá-lo fora do diretório `GOPATH`.

Esta é uma grande mudança em relação as versões anteriores do Go (pré-1.11), onde a prática recomendada era criar o diretório dos projetos dentro de uma pasta `src` sob o diretório `GOPATH`, conforme mostrado a seguir:

```bash
$GOPATH
├── bin
├── pkg
└── src
    └── github.com
        └── <usuário github>
            └── <projeto>
```

Nessa estrutura, os diretórios possuem as seguintes funções:
- `bin`: Guardar os executáveis de nossos programas;
- `pkg`: Guardar nossas bibliotecas e bibliotecas de terceiros;
- `src`: Guardar todo o código dos nossos projetos.

De forma resumida:
- Versões pré-1.11: A recomendação é criar o diretório do projeto sob o diretório `GOPATH`;
- Versões pós-1.11 e pré-1.13: A recomendção é criar o diretório do projeto fora do `GOPATH`;
- Versão 1.13: O diretório do projeto pode ser criado em qualquer lugar no computador.

### Ativação do Go Modules

Para utilizar módulos no seu projeto, abra seu terminal e crie um novo diretório para o projeto chamado `go-modules` em qualquer lugar em seu computador0.

> **Dica**: Crie o diretório do projeto em `$HOME/code`, mas você pode escolher um local diferente, se desejar.

```bash
$ mkdir -p $HOME/code/go-modules
```

A próxima coisa que precisamos fazer é informar ao Go que queremos usar a nova funcionalidade de módulos para ajudar a gerenciar e controlar a versão de quaisquer pacotes de terceiros que o nosso projeto importe.

Para fazer isso, primeiro precisamos decidir qual deve ser o caminho do módulo para o nosso projeto.

O importante aqui é a singularidade. Para evitar possíveis conflitos de importação com os pacotes de outras pessoas ou com a _Standard Library_ (biblioteca padrão) no futuro, escolha um caminho de módulo que seja globalmente exclusivo e improvável de ser usado por qualquer outra coisa. Na comunidade Go, uma convenção comum é criar o caminho do módulo com base em uma URL que você possui.

Supondo que nossa URL seja `delley.github.io`, vamos iniciar os módulos da seguinte forma:

```bash
$ cd $HOME/code/workshop
$ go mod init delley.github.io/go-modules
go: creating new go.mod: module delley.github.io/go-modules
```

Neste ponto, o diretório do projeto já deve possuir o aquivo `go.mod` criado.

Não há muita coisa nesse arquivo e se você abri-lo em seu editor de texto, ele deve ficar assim (mas de preferência com seu próprio caminho de módulo exclusivo):

```bash
module delley.github.io/go-modules

go 1.13
```

> **Dica**: Se você estiver criando um pacote ou aplicativo que possa ser baixado e usado por outras pessoas e programas, é recomendável que o caminho do módulo seja igual ao local do qual o código pode ser baixado.
> Por exemplo, se o seu pacote estiver hospedado em `https://github.com/foo/bar`, o caminho do módulo para o projeto deverá ser `github.com/foo/bar`.

Basicamente é isso! Posteriormente, em outro textículo, veremos como esse arquivo é usado para definir os pacotes de terceiros (e suas versões).

[1]:https://blog.golang.org/using-go-modules