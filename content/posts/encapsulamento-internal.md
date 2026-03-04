+++ 
draft = false
date = 2026-03-04
title = "Encapsulamento em nível de módulo: \"internal\" em versos de cordel!"
description = "Entendendo o diretório `internal` em Go"
slug = "encapsulamento-internal" 
tags = ["Go", "Internal", "Encapsulamento"]
categories = ["Programação", "Go"]
series = ["Go", "Go em Cordel"]
images = ["/images/encapisulamento-internal.jpg"]
+++

{{< figure src="/images/encapisulamento-internal.jpg" alt="Encapsulamento usando internal" class="centered" >}}

> **Cordel técnico**
>
> No sertão da arquitetura  
> cada módulo tem seu chão  
> pacote público é estrada  
> mas `internal` é portão.

Entre as muitas decisões elegantes do design da linguagem Go, uma delas revela muito sobre a filosofia da linguagem: **como ela protege aquilo que deve permanecer interno a um projeto**.
Diferente de muitas linguagens orientadas a objetos que utilizam modificadores como:

*   `protected`
*   `package-private`
*   `friend`

o Go resolveu o problema de encapsulamento de maneira **simples, explícita e aplicada pelo compilador**. A solução é um mecanismo chamado: **internal**

Neste artigo vamos entender:

*   o problema que ele resolve
*   como funciona
*   por que ele existe
*   como utilizá-lo corretamente em projetos Go modernos

* * *

## O problema: APIs que vazam sem controle

> **Cordel técnico**
>
> Código solto vira trilha  
> que qualquer um pode cruzar  
> detalhe vira dependência  
> difícil depois refatorar.

Em Go, qualquer pacote pode ser importado por outro módulo, desde que:
*   esteja dentro de um módulo válido
*   possua identificadores exportados (iniciando com letra maiúscula)

Por exemplo:

```go
import "github.com/exemplo/projeto/repository"
```

Se um pacote exporta símbolos públicos, **qualquer outro projeto pode depender dele**.
Isso pode parecer inofensivo, mas em projetos grandes cria um problema sério.
Trechos internos de implementação acabam se tornando **dependências externas acidentais**.
Esse tipo de vazamento gera vários problemas:

*   dificulta refatorações    
*   cria dependências implícitas
*   transforma detalhes internos em APIs públicas
*   aumenta o acoplamento entre projetos

No mundo real, isso é como uma cerca mal feita no sertão:

> quando o dono percebe, o gado já atravessou a divisa.

Foi exatamente para evitar esse cenário que surgiu o diretório **`internal`**.

* * *

## O que é o diretório `internal`

> **Cordel técnico**
>
> O Go não usa mil palavras  
> nem regra complicada não  
> criou só uma pastinha  
> que funciona como portão.

O diretório `internal` é um **mecanismo oficial da linguagem Go para controle de importação de pacotes**.

A regra é simples:

> Um pacote localizado dentro de um diretório chamado `internal` **só pode ser importado por código que esteja dentro da mesma árvore de diretórios do módulo pai**.

Essa regra:

*   **não é uma convenção**
*   **não depende de ferramentas**
*   **é aplicada diretamente pelo compilador Go**

Ou seja, se alguém tentar importar esse pacote de fora do módulo, a compilação falha.


* * *

## A regra do `internal` na prática

> **Cordel técnico**
>
> Quem mora dentro da casa  
> pode entrar no corredor  
> mas quem vem de fora encontra  
> o compilador como vigia e guardador.

Considere a seguinte estrutura:

```
meuapp/  
├── go.mod  
├── main.go  
├── service/  
│   └── user.go  
└── internal/  
    └── repository/  
        └── user_repository.go
```

O pacote: `meuapp/internal/repository`

#### Importações permitidas

Qualquer código **dentro de `meuapp/`** pode importar esse pacote:

Exemplo:

```go
import "github.com/delley/meuapp/internal/repository"
```

Arquivos que podem importar:

* `main.go`
* `service/user.go`
* qualquer outro pacote dentro do módulo

#### Importações proibidas

Agora imagine outro projeto:

```
outroapp/  
└── main.go
```

Se ele tentar importar:

```go
import "github.com/delley/meuapp/internal/repository"
```

O compilador retornará:

```
use of internal package github.com/delley/meuapp/internal/repository not allowed
```

Ou seja: **o Go bloqueia o acesso na origem**.

#### Diagrama do comportamento do `internal`

```
             ┌───────────────────────┐
             │        meuapp         │
             │                       │
             │    ┌─────────────┐    │
             │    │   service   │    │
             │    └──────┬──────┘    │
             │           │ permitido │
             │           ▼           │
             │    ┌─────────────┐    │
             │    │  internal   │    │
             │    │ repository  │    │
             │    └─────────────┘    │
             │           ▲           │
             │           │           │
             └───────────┼───────────┘
                         │
                         │ proibido
                         │
                         │
                ┌────────┴────────┐
                │    outroapp     │
                │                 │
                │    main.go      │
                └─────────────────┘

         use of internal package not allowed
```
* * *

## `internal` não substitui visibilidade por pacote

> **Cordel técnico**
>
> Símbolo pequeno esconde  
> dentro do próprio lugar  
> mas internal fecha a cerca  
> pra ninguém de fora importar.

É importante entender que `internal` **não controla visibilidade de símbolos**.
Ele controla **quem pode importar o pacote**.
Em Go existem dois níveis distintos de encapsulamento:

| Mecanismo | Escopo |
| --- | --- |
| identificadores minúsculos | dentro do pacote |
| diretório `internal` | dentro da árvore do módulo |

Eles atuam **em camadas diferentes da arquitetura**, sendo **complementares**, não concorrentes.

* * *

## `internal` em múltiplos níveis

> **Cordel técnico**
>
> Dentro da mesma fazenda  
> tem curral e tem divisão  
> cada cerca guarda um trecho  
> do domínio da aplicação.

Nada impede que você tenha vários diretórios `internal` em diferentes partes do projeto.

Exemplo:

```
meuapp/  
├── internal/  
│   └── config/  
├── service/  
│   ├── internal/  
│   │   └── validator/  
│   └── user.go
```

Isso gera duas regras diferentes.

*   Pacote `meuapp/internal/config`:  
    → acessível por qualquer pacote dentro de `meuapp`    
*   `meuapp/service/internal/validator`:  
    → acessível **apenas** por pacotes dentro de `meuapp/service`

Isso permite **isolamento por domínio**, não apenas por aplicação.

**Acesso permitido**:
```
user.go ───► validator
user.go ───► config
```

**Acesso proibido**:
```
config ─X─► validator
outro módulo ─X─► config
```
* * *

## Por que o Go introduziu o `internal`

> **Cordel técnico**
>
> Não foi moda nem capricho  
> de quem gosta de inovar  
> foi remédio pra arquitetura  
> não deixar API vazar.

O objetivo do `internal` é direto e pragmático. Ele existe para resolver três problemas reais:

1. **Evitar vazamento de API**  
   Pacotes internos deixam claro que não fazem parte da API pública.
   Isso impede dependências externas acidentais.

2. **Forçar arquitetura saudável**  
   Sem `internal`, qualquer camada poderia importar qualquer outra.
   Com `internal`, o design arquitetural fica mais protegido.

3. **Comunicar intenção**  
   Quando um desenvolvedor vê um diretório chamado: `internal`, a mensagem é clara: ***“Este código não faz parte da API pública.”***

* * *

## Uso típico em projetos reais

> **Cordel técnico**
>
> Projeto bem organizado  
> tem começo, meio e razão  
> `cmd` chama o programa  
> `internal` guarda o coração.

Em projetos bem estruturados, o `internal` costuma abrigar:

*   domínio 
*   regras de negócio
*   infraestrutura    
*   adapters
*   casos de uso

Um layout bastante comum:

```
myapp/
│
├── cmd/
│   └── api/
│       └── main.go
│
├── internal/
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── adapters/
│
└── pkg/
```
| Diretório  | Papel                |
| ---------- | -------------------- |
| `cmd`      | pontos de entrada    |
| `internal` | coração da aplicação |
| `pkg`      | código reutilizável por terceiros (quando necessário)  |


* * *

## Erros comuns ao usar `internal`

> **Cordel técnico**
>
> Se usar sem pensamento  
> vira só mais um porão  
> diretório cheio de código  
> sem nenhuma direção.

1. Colocar tudo em `internal` sem critério  
   Se um pacote **é parte da API pública**, ele não deveria estar em `internal`.

2. Usar `internal` para esconder problemas de design  
   Ele não corrige dependências ruins, ele apenas restringe quem pode importar.

3. Criar dependências cruzadas entre `internal`  
   O compilador permite, mas a arquitetura sofre.

* * *

## Quando NÃO usar `internal`

> **Cordel técnico**
>
> Se o código é estrada pública  
> pra outros poderem passar  
> não levante cerca alta  
> deixe o pacote importar.

Existem casos onde `internal` **não é apropriado**.

Por exemplo:

*   bibliotecas públicas    
*   SDKs
*   pacotes utilitários reutilizáveis
*   frameworks

Se o pacote deve ser usado por outros projetos, ele **não deve ficar em `internal`**.

* * *

## Regra prática para decidir

> **Cordel técnico**
>
> Pergunta simples resolve  
> sem precisar filosofar  
> isso é parte da API  
> ou só código do lugar?

Uma pergunta simples resolve 90% dos casos: **Esse pacote deveria ser importável por outro projeto?**

*   **Sim** → não use `internal`
*   **Não** → use `internal`
    

* * *

## Conclusão

> **Cordel técnico**
>
> O Go gosta de clareza  
> regra simples de aplicar  
> quem é da casa atravessa  
> quem é de fora não vai entrar.

O diretório `internal` é um dos exemplos mais elegantes da filosofia do Go, **simples**, **explícito**, **direto**, **sem mágica**, **sem convenções frágeis**, apenas uma regra clara aplicada pelo compilador.
    
Quando bem utilizado, ele **protege sua base de código**, **reduz acoplamento**, **facilita evolução**, **comunica intenção arquitetural**.

No fim das contas, o `internal` não é apenas um diretório.

É uma **regra de arquitetura aplicada pela própria linguagem**.

## Referências

Go Blog. _Internal Packages_. Disponível em: [https://go.dev/doc/go1.4#internalpackages](https://go.dev/doc/go1.4#internalpackages). Acesso em: 21 fev. 2026.