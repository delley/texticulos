+++
title = "Como Estruturar um Projeto Go para Escalar"
date = 2026-08-05T21:23:00-03:00
draft = false
description = "Um cordel técnico sobre como estruturar projetos Go com monólito modular, pacotes orientados ao domínio, interfaces pequenas, observabilidade e evolução sustentável."
tags = ["Go", "Arquitetura de Software", "Engenharia de Software", "Sistemas Distribuídos"]
categories = ["Go", "Arquitetura de Software", "Engenharia de Software", "Go em Cordel"]
slug = "como-estruturar-projeto-go-para-escalar"
images = ["/images/estrurar-projeto-escalar.png"]
+++

{{< figure src="/images/estrurar-projeto-escalar.png" alt="Encapsulamento usando internal" class="centered" >}}

> **Cordel técnico**
>
> Quando o projeto é pequeno,  
> Tudo parece funcionar.  
> Mas basta o sistema crescer  
> Para a conta começar.  
> Sem fronteira e disciplina,  
> Todo ajuste faz quebrar.

Estruturar um projeto Go para escalar não significa criar dezenas de diretórios, aplicar todos os padrões arquiteturais disponíveis ou dividir prematuramente a aplicação em microserviços.

Uma estrutura sustentável deve favorecer:

- baixo acoplamento;
- alta coesão;
- dependências explícitas;
- testabilidade;
- observabilidade;
- evolução incremental.

Para a maioria dos sistemas, um bom ponto de partida é um **monólito modular organizado por domínio**.

Neste artigo, veremos como estruturar uma aplicação Go de modo que ela possa crescer sem transformar cada nova funcionalidade em uma travessia por um labirinto de pacotes, interfaces e dependências.

<!--more-->

## O que significa escalar um projeto Go?

> **Cordel técnico**
>
> Escalar não é somente  
> Mais instância levantar.  
> É deixar código e equipe  
> Em conjunto avançar.  
> Sem que toda nova mudança  
> Faça o sistema tombar.

Quando falamos em escala, normalmente pensamos em tráfego, quantidade de usuários ou volume de processamento.

Entretanto, uma aplicação também precisa escalar em outras dimensões:

- quantidade de funcionalidades;
- número de integrações;
- tamanho da equipe;
- volume de regras de negócio;
- quantidade de executáveis;
- complexidade operacional;
- frequência de implantação.

Um projeto Go escala bem quando permite:

1. adicionar funcionalidades sem quebrar módulos não relacionados;
2. substituir integrações externas com impacto controlado;
3. testar regras de negócio sem iniciar toda a infraestrutura;
4. executar múltiplas instâncias da aplicação;
5. identificar rapidamente onde determinado comportamento está implementado;
6. extrair módulos para serviços independentes, caso isso se torne necessário.

Portanto, escalar não é apenas uma questão de infraestrutura. É também uma questão de **organização e capacidade de evolução**.

---

## Comece com um monólito modular

> **Cordel técnico**
>
> Microserviço é bonito  
> No desenho e apresentação.  
> Mas espalhar um sistema cedo  
> Pode aumentar a confusão.  
> Primeiro firme as fronteiras,  
> Depois pense em distribuição.

Um monólito modular mantém a aplicação em uma única unidade de implantação, mas divide o código em módulos de negócio claramente definidos.

Uma estrutura inicial pode ser:

```text
my-project/
├── cmd/
│   ├── api/
│   │   └── main.go
│   ├── worker/
│   │   └── main.go
│   └── migrate/
│       └── main.go
│
├── internal/
│   ├── account/
│   ├── billing/
│   ├── order/
│   ├── platform/
│   └── app/
│
├── migrations/
├── api/
│   └── openapi.yaml
├── deployments/
├── scripts/
├── go.mod
├── go.sum
└── Makefile
```

A ideia central é organizar o código pelas capacidades do sistema, e não apenas pelos tipos técnicos utilizados.

Uma estrutura exclusivamente técnica normalmente se parece com isto:

```text
internal/
├── handlers/
├── services/
├── repositories/
├── controllers/
└── models/
```

Embora pareça organizada, ela espalha uma única funcionalidade por vários diretórios.

Para modificar o fluxo de pedidos, por exemplo, provavelmente será necessário navegar por:

```text
handlers/order.go
services/order.go
repositories/order.go
models/order.go
controllers/order.go
```

Uma organização orientada ao domínio mantém os elementos relacionados próximos:

```text
internal/
├── customer/
├── order/
├── payment/
└── inventory/
```

Dessa forma, uma alteração no domínio de pedidos permanece concentrada dentro de `internal/order`.

Essa abordagem reduz a carga cognitiva e melhora a capacidade de compreender o impacto de uma mudança.

---

## O diretório `cmd`

> **Cordel técnico**
>
> Cada programa tem entrada,  
> Cada entrada tem função.  
> Um atende pela API,  
> Outro escuta a integração.  
> Mas o `main` não deve carregar  
> Toda a regra da aplicação.

O diretório `cmd` contém os pontos de entrada dos executáveis do projeto.

```text
cmd/
├── api/
├── worker/
├── scheduler/
├── migrate/
└── cli/
```

Cada subdiretório representa um programa diferente:

- `api`: servidor HTTP;
- `worker`: consumidor de mensagens;
- `scheduler`: execução de tarefas periódicas;
- `migrate`: gerenciamento de migrações;
- `cli`: comandos administrativos.

O arquivo `main.go` deve permanecer pequeno.

```go
package main

import (
	"context"
	"log/slog"
	"os"
	"os/signal"
	"syscall"

	"example.com/project/internal/app"
)

func main() {
	ctx, cancel := signal.NotifyContext(
		context.Background(),
		os.Interrupt,
		syscall.SIGTERM,
	)
	defer cancel()

	application, err := app.New()
	if err != nil {
		slog.Error(
			"failed to initialize application",
			"error",
			err,
		)
		os.Exit(1)
	}

	if err := application.Run(ctx); err != nil {
		slog.Error(
			"application stopped with error",
			"error",
			err,
		)
		os.Exit(1)
	}
}
```

O `main.go` deve ser responsável principalmente por:

1. carregar a configuração;
2. inicializar a aplicação;
3. iniciar a execução;
4. capturar sinais do sistema operacional;
5. coordenar o encerramento.

Evite colocar no `main.go`:

- consultas SQL;
- regras de negócio;
- configuração detalhada de rotas;
- clientes HTTP externos;
- serialização;
- manipulação de mensagens;
- lógica de autorização;
- dezenas de construções de dependências espalhadas.

O `main` deve funcionar como a porta de entrada, não como o lugar onde toda a aplicação mora.

---

## O diretório `internal`

> **Cordel técnico**
>
> Dentro de `internal`  
> O projeto faz proteção.  
> O pacote fica guardado  
> Contra externa importação.  
> A própria linguagem ajuda  
> A manter a separação.

O diretório `internal` possui comportamento especial em Go.

Pacotes armazenados dentro dele não podem ser importados livremente por módulos externos que estejam fora da árvore permitida.

Isso permite proteger detalhes internos da aplicação.

Um módulo simples pode começar assim:

```text
internal/order/
├── order.go
├── service.go
├── repository.go
├── errors.go
├── handler.go
└── postgres.go
```

Essa estrutura plana é adequada enquanto o módulo ainda possui tamanho reduzido.

Não é necessário começar imediatamente com:

```text
domain/
application/
ports/
adapters/
infrastructure/
```

Quando o módulo crescer, a divisão pode ser feita gradualmente:

```text
internal/order/
├── domain/
│   ├── order.go
│   └── errors.go
├── application/
│   ├── create_order.go
│   └── cancel_order.go
├── postgres/
│   └── repository.go
└── http/
    └── handler.go
```

A estrutura deve acompanhar a complexidade existente.

Não crie diretórios para uma complexidade que talvez nunca apareça.

---

## Organize o sistema por domínio

> **Cordel técnico**
>
> Pedido fica com pedido,  
> Pagamento em seu lugar.  
> Tudo aquilo que é do módulo  
> Deve perto se encontrar.  
> Quem procura uma mudança  
> Não precisa peregrinar.

Organizar por domínio significa agrupar o código de acordo com as capacidades de negócio.

Uma aplicação de comércio eletrônico poderia ser organizada assim:

```text
internal/
├── customer/
├── catalog/
├── inventory/
├── order/
├── payment/
├── shipping/
└── platform/
```

Cada módulo pode conter:

- entidades;
- regras de negócio;
- casos de uso;
- contratos;
- adaptadores;
- persistência;
- handlers;
- testes.

Essa estrutura segue uma ideia próxima da **Screaming Architecture**: ao observar os diretórios do projeto, deve ser possível entender o que o sistema faz.

Uma estrutura como:

```text
internal/
├── controller/
├── entity/
├── repository/
├── service/
└── util/
```

informa quais padrões técnicos foram utilizados.

Uma estrutura como:

```text
internal/
├── billing/
├── identity/
├── order/
└── inventory/
```

informa qual problema o sistema resolve.

---

## Mantenha o domínio independente da infraestrutura

> **Cordel técnico**
>
> O pedido conhece a regra,  
> Seu estado e condição.  
> Mas não sabe se é salvo  
> Em arquivo ou conexão.  
> Banco, fila e protocolo  
> Vivem noutra direção.

As regras centrais do sistema não devem depender diretamente de HTTP, PostgreSQL, Kafka ou qualquer outro mecanismo de infraestrutura.

Considere a seguinte entidade:

```go
package order

import (
	"errors"
	"time"
)

var ErrInvalidAmount = errors.New(
	"order amount must be positive",
)

type Status string

const (
	StatusPending   Status = "pending"
	StatusConfirmed Status = "confirmed"
	StatusCanceled  Status = "canceled"
)

type Order struct {
	ID         string
	CustomerID string
	Amount     int64
	Status     Status
	CreatedAt  time.Time
}

func New(
	id string,
	customerID string,
	amount int64,
	now time.Time,
) (*Order, error) {
	if amount <= 0 {
		return nil, ErrInvalidAmount
	}

	return &Order{
		ID:         id,
		CustomerID: customerID,
		Amount:     amount,
		Status:     StatusPending,
		CreatedAt:  now,
	}, nil
}
```

A entidade concentra:

- seus dados;
- suas invariantes;
- regras de criação;
- estados possíveis;
- comportamentos do domínio.

Ela não precisa conhecer:

- JSON;
- banco de dados;
- frameworks web;
- bibliotecas de mensageria;
- variáveis de ambiente;
- APIs externas.

Quanto menos o domínio conhece sobre infraestrutura, mais simples se torna:

- testá-lo;
- reutilizá-lo;
- alterá-lo;
- substituir tecnologias ao redor dele.

---

## Defina interfaces no lado consumidor

> **Cordel técnico**
>
> Contrato bom é pequeno,  
> Tem propósito e direção.  
> Nasce perto de quem usa,  
> Não de quem faz implementação.  
> Quanto menor sua promessa,  
> Mais simples a substituição.

Em Go, uma interface normalmente deve ser declarada pelo pacote que a consome.

```go
package order

import (
	"context"
	"time"
)

type Repository interface {
	Save(ctx context.Context, order *Order) error
	FindByID(ctx context.Context, id string) (*Order, error)
}

type EventPublisher interface {
	PublishOrderCreated(
		ctx context.Context,
		order *Order,
	) error
}

type IDGenerator interface {
	NewID() string
}

type Clock interface {
	Now() time.Time
}
```

O caso de uso depende dessas interfaces:

```go
type Service struct {
	repository Repository
	publisher  EventPublisher
	ids        IDGenerator
	clock      Clock
}

func NewService(
	repository Repository,
	publisher EventPublisher,
	ids IDGenerator,
	clock Clock,
) *Service {
	return &Service{
		repository: repository,
		publisher:  publisher,
		ids:        ids,
		clock:      clock,
	}
}
```

As implementações concretas podem utilizar PostgreSQL, Kafka, UUID ou o relógio do sistema.

Em testes, essas dependências podem ser substituídas por implementações controladas.

```text
Repository
├── PostgreSQL
├── memória
└── stub de teste
```

Isso não significa criar interfaces para todas as estruturas.

Interfaces são especialmente úteis nas fronteiras com:

- banco de dados;
- mensageria;
- serviços externos;
- cache;
- armazenamento;
- geração de identificadores;
- relógio;
- sistemas de pagamento.

Uma interface deve representar uma necessidade concreta do consumidor.

Não deve existir apenas para satisfazer uma regra arquitetural.

---

## Modele casos de uso explícitos

> **Cordel técnico**
>
> Cada ação tem seu nome,  
> Seu comando e intenção.  
> Criar pedido é uma coisa,  
> Cancelar tem outra razão.  
> Separar bem cada fluxo  
> Simplifica a manutenção.

Em sistemas pequenos, um único serviço pode ser suficiente.

```go
type OrderService struct {
	repository Repository
}
```

Com o crescimento, esse serviço pode concentrar operações demais:

```text
Create
Update
Confirm
Cancel
Refund
Retry
Approve
Reject
Archive
```

Nesse momento, separar os casos de uso pode tornar a aplicação mais compreensível.

```text
internal/order/
├── create.go
├── cancel.go
├── confirm.go
├── find.go
└── repository.go
```

Um caso de uso pode ser modelado assim:

```go
type CreateCommand struct {
	CustomerID string
	Amount     int64
}

type CreateHandler struct {
	repository Repository
	ids        IDGenerator
	clock      Clock
}

func NewCreateHandler(
	repository Repository,
	ids IDGenerator,
	clock Clock,
) *CreateHandler {
	return &CreateHandler{
		repository: repository,
		ids:        ids,
		clock:      clock,
	}
}

func (h *CreateHandler) Handle(
	ctx context.Context,
	cmd CreateCommand,
) (*Order, error) {
	created, err := New(
		h.ids.NewID(),
		cmd.CustomerID,
		cmd.Amount,
		h.clock.Now(),
	)
	if err != nil {
		return nil, err
	}

	if err := h.repository.Save(ctx, created); err != nil {
		return nil, fmt.Errorf(
			"save order: %w",
			err,
		)
	}

	return created, nil
}
```

Esse formato torna explícitos:

- a entrada;
- as dependências;
- o fluxo;
- o resultado;
- os erros possíveis.

Além disso, evita o surgimento de serviços gigantes, difíceis de entender e testar.

---

## Trate HTTP como um adaptador

> **Cordel técnico**
>
> HTTP traz a mensagem,  
> Faz a devida tradução.  
> Mas não deve decidir  
> A política da operação.  
> Regra mora no domínio;  
> Protocolo, na adaptação.

O handler HTTP deve converter uma requisição em uma chamada para a aplicação.

```go
type Handler struct {
	createOrder *order.CreateHandler
}

type createOrderRequest struct {
	CustomerID string `json:"customerId"`
	Amount     int64  `json:"amount"`
}

func (h *Handler) Create(
	w http.ResponseWriter,
	r *http.Request,
) {
	var request createOrderRequest

	if err := json.NewDecoder(r.Body).Decode(&request); err != nil {
		writeError(
			w,
			http.StatusBadRequest,
			"invalid request body",
		)
		return
	}

	created, err := h.createOrder.Handle(
		r.Context(),
		order.CreateCommand{
			CustomerID: request.CustomerID,
			Amount:     request.Amount,
		},
	)
	if err != nil {
		handleOrderError(w, err)
		return
	}

	writeJSON(w, http.StatusCreated, created)
}
```

O handler deve ser responsável por:

- leitura da requisição;
- validação estrutural;
- autenticação;
- autorização;
- conversão entre DTOs e comandos;
- tradução de erros;
- serialização da resposta.

O handler não deve concentrar regras como:

- cálculo de preço;
- validação de estoque;
- transição de estado;
- políticas de cancelamento;
- processamento de pagamento.

Essas decisões pertencem ao domínio ou aos casos de uso.

---

## Centralize a composição das dependências

> **Cordel técnico**
>
> Na raiz se monta o sistema,  
> Peça encontra conexão.  
> O contrato recebe o concreto,  
> Sem oculta resolução.  
> Dependência bem visível  
> Melhora a compreensão.

A aplicação precisa de um lugar onde as dependências concretas são criadas e conectadas.

Esse ponto é conhecido como **composition root**.

```go
func New(config Config) (*Application, error) {
	db, err := database.Open(config.Database)
	if err != nil {
		return nil, fmt.Errorf(
			"open database: %w",
			err,
		)
	}

	orderRepository := orderpostgres.NewRepository(db)
	eventPublisher := kafka.NewPublisher(config.Kafka)
	clock := systemclock.New()
	idGenerator := uuidgenerator.New()

	createOrder := order.NewCreateHandler(
		orderRepository,
		idGenerator,
		clock,
	)

	orderHandler := orderhttp.NewHandler(
		createOrder,
		eventPublisher,
	)

	router := httpserver.NewRouter()
	orderhttp.RegisterRoutes(router, orderHandler)

	return &Application{
		server: httpserver.New(
			config.HTTP,
			router,
		),
		db: db,
	}, nil
}
```

Essa composição explícita permite visualizar:

- quais implementações estão sendo utilizadas;
- quais componentes dependem de quais contratos;
- quais recursos precisam ser encerrados;
- quais dependências podem ser substituídas em testes.

Evite esconder dependências por meio de:

- variáveis globais;
- funções `init`;
- service locators;
- singletons mutáveis;
- registradores globais;
- acesso direto a configurações em qualquer pacote.

Em Go, construtores explícitos costumam ser suficientes para a maior parte dos projetos.

---

## Separe a infraestrutura compartilhada

> **Cordel técnico**
>
> Plataforma faz a estrada,  
> Traz conexão e observação.  
> O domínio traz a regra,  
> A política e decisão.  
> Quando os dois ficam misturados,  
> Toda mudança vira tensão.

Capacidades técnicas compartilhadas podem ficar em um pacote de plataforma.

```text
internal/platform/
├── database/
├── httpserver/
├── messaging/
├── observability/
├── cache/
└── resilience/
```

Esse diretório pode conter:

- configuração de banco de dados;
- criação de pool de conexões;
- servidor HTTP;
- middlewares;
- métricas;
- tracing;
- logging;
- produtores de eventos;
- consumidores;
- mecanismos de retry;
- circuit breakers.

Entretanto, `platform` não deve se tornar um diretório genérico para código sem classificação.

Evite transformá-lo em algo semelhante a:

```text
internal/platform/
├── utils/
├── helpers/
├── commons/
└── miscellaneous/
```

Pacotes com nomes genéricos normalmente escondem responsabilidades pouco definidas.

A infraestrutura compartilhada deve fornecer capacidades técnicas, não regras de negócio.

---

## Projete a aplicação para ser stateless

> **Cordel técnico**
>
> Processo nasce e termina,  
> Pode a instância desligar.  
> O estado que é importante  
> Nela não deve morar.  
> Para outra assumir o trabalho,  
> O dado precisa ficar.

Aplicações preparadas para escalar horizontalmente devem evitar depender de estado local.

Não armazene informações críticas em:

- variáveis globais;
- mapas em memória;
- arquivos locais;
- sessões no processo;
- goroutines;
- caches locais usados como fonte de verdade.

Utilize componentes apropriados:

| Necessidade | Componente possível |
|---|---|
| Estado transacional | PostgreSQL |
| Cache e coordenação | Redis |
| Arquivos | Object storage |
| Eventos | Kafka ou NATS |
| Busca textual | OpenSearch |
| Métricas | Prometheus |
| Traces | OpenTelemetry |

A aplicação deve conseguir executar várias instâncias sem depender de qual delas processou a requisição anterior.

Isso facilita:

- balanceamento de carga;
- reinício de containers;
- autoscaling;
- rolling updates;
- recuperação de falhas.

---

## Trate idempotência explicitamente

> **Cordel técnico**
>
> A mensagem pode voltar,  
> A chamada se repetir.  
> Quem não trata duplicata  
> Vê o efeito ressurgir.  
> Idempotência é o recurso  
> Que impede o erro de insistir.

Em sistemas distribuídos, requisições e mensagens podem ser processadas mais de uma vez.

Isso pode acontecer por:

- retry do cliente;
- timeout;
- falha de rede;
- reentrega do broker;
- reinício do consumidor;
- interrupção durante o processamento.

Operações críticas devem possuir mecanismos de idempotência.

```go
type IdempotencyRepository interface {
	Reserve(
		ctx context.Context,
		key string,
	) (bool, error)

	Complete(
		ctx context.Context,
		key string,
		result []byte,
	) error
}
```

Um fluxo possível é:

```text
cliente envia Idempotency-Key
             │
             ▼
 aplicação tenta reservar a chave
             │
       ┌─────┴─────┐
       │           │
      nova      existente
       │           │
       ▼           ▼
 executa ação   retorna resultado anterior
       │
       ▼
 persiste o resultado
```

Esse mecanismo é especialmente importante em:

- pagamentos;
- criação de pedidos;
- reservas;
- transferências;
- processamento de eventos;
- integrações sujeitas a retry.

---

## Resolva concorrência no armazenamento

> **Cordel técnico**
>
> Consultar antes de inserir  
> Pode parecer proteção.  
> Mas duas instâncias consultando  
> Chegam à mesma conclusão.  
> A verdade concorrente  
> Deve estar na persistência em ação.

Considere este fluxo:

```go
if !repository.Exists(email) {
	repository.Insert(email)
}
```

Duas instâncias podem executar a verificação simultaneamente e concluir que o registro não existe.

Em seguida, ambas tentam criá-lo.

Esse problema deve ser tratado com mecanismos fornecidos pelo armazenamento:

- constraints únicas;
- transações;
- locking otimista;
- operações atômicas;
- controle de versão;
- `SELECT ... FOR UPDATE`;
- compare-and-swap.

Por exemplo, uma constraint única no banco protege o sistema mesmo quando várias instâncias executam a operação simultaneamente.

A aplicação ainda deve tratar adequadamente o erro de conflito retornado pela persistência.

---

## Use processamento assíncrono com responsabilidade

> **Cordel técnico**
>
> Nem toda tarefa precisa  
> Na chamada terminar.  
> Trabalho lento ou secundário  
> Pode a fila encaminhar.  
> Mas evento sem garantia  
> Faz o estado se quebrar.

Algumas operações podem ser realizadas de forma assíncrona:

- envio de notificações;
- geração de relatórios;
- atualização de índices;
- integração com terceiros;
- processamento de imagens;
- propagação de eventos.

Um fluxo comum é:

```text
API → banco → broker → worker
```

Entretanto, existe um problema quando a operação é persistida no banco, mas a publicação do evento falha.

```text
pedido salvo
     │
     ▼
falha ao publicar evento
     │
     ▼
estado persistido sem notificação
```

Uma solução é o padrão **Transactional Outbox**:

```text
┌───────────────────────────┐
│ Transação no banco        │
│                           │
│ 1. salva a entidade       │
│ 2. registra o evento      │
└──────────────┬────────────┘
               │
               ▼
       publicador da outbox
               │
               ▼
             broker
```

A entidade e o evento são persistidos na mesma transação.

Depois, um processo separado publica os eventos registrados na tabela de outbox.

Essa abordagem reduz o risco de inconsistência entre banco de dados e mensageria.

---

## Classifique e envolva erros

> **Cordel técnico**
>
> Erro também tem contexto,  
> Tem origem e classificação.  
> Para o cliente vai o contrato,  
> Para o log, investigação.  
> Quem devolve qualquer detalhe  
> Pode expor a aplicação.

Erros de domínio devem ser identificáveis.

```go
var (
	ErrOrderNotFound = errors.New(
		"order not found",
	)

	ErrOrderCanceled = errors.New(
		"order already canceled",
	)
)
```

Ao propagar um erro, acrescente contexto usando `%w`.

```go
return fmt.Errorf(
	"find order %q: %w",
	id,
	err,
)
```

Na borda HTTP, traduza os erros para respostas adequadas:

```go
switch {
case errors.Is(err, order.ErrOrderNotFound):
	writeError(
		w,
		http.StatusNotFound,
		"order not found",
	)

case errors.Is(err, order.ErrOrderCanceled):
	writeError(
		w,
		http.StatusConflict,
		"order already canceled",
	)

default:
	logger.ErrorContext(
		ctx,
		"request failed",
		"error",
		err,
	)

	writeError(
		w,
		http.StatusInternalServerError,
		"internal error",
	)
}
```

O cliente não deve receber detalhes internos, stack traces ou informações de infraestrutura.

Esses dados pertencem aos logs e às ferramentas de observabilidade.

---

## Centralize a configuração

> **Cordel técnico**
>
> Configuração espalhada  
> É difícil de rastrear.  
> Quando a falta só aparece  
> Após o tráfego chegar,  
> O sistema perde tempo  
> Tentando se recuperar.

Evite acessar `os.Getenv` diretamente em vários pacotes.

Prefira uma estrutura tipada:

```go
type Config struct {
	Environment   string
	HTTP          HTTPConfig
	Database      DatabaseConfig
	Observability ObservabilityConfig
}

type HTTPConfig struct {
	Address         string
	ReadTimeout     time.Duration
	WriteTimeout    time.Duration
	ShutdownTimeout time.Duration
}
```

Carregue e valide a configuração durante a inicialização:

```go
func LoadConfig() (Config, error) {
	cfg := Config{
		Environment: os.Getenv("APP_ENV"),
		HTTP: HTTPConfig{
			Address: ":8080",
		},
	}

	if cfg.Environment == "" {
		return Config{}, errors.New(
			"APP_ENV is required",
		)
	}

	return cfg, nil
}
```

A aplicação deve falhar cedo quando uma configuração obrigatória estiver ausente.

Esse comportamento é preferível a descobrir a falha somente durante o processamento de uma requisição.

---

## Implemente graceful shutdown

> **Cordel técnico**
>
> Quando chega o encerramento,  
> Não se deve abandonar  
> Requisição pela metade,  
> Nem mensagem sem tratar.  
> Encerrar com segurança  
> Também faz parte de escalar.

Em ambientes com containers e orquestradores, a aplicação recebe sinais de encerramento.

Ela deve responder a esses sinais de forma controlada.

```go
func (a *Application) Run(
	ctx context.Context,
) error {
	serverErrors := make(chan error, 1)

	go func() {
		serverErrors <- a.server.ListenAndServe()
	}()

	select {
	case err := <-serverErrors:
		return err

	case <-ctx.Done():
		shutdownCtx, cancel := context.WithTimeout(
			context.Background(),
			15*time.Second,
		)
		defer cancel()

		return a.server.Shutdown(shutdownCtx)
	}
}
```

Durante o graceful shutdown, a aplicação deve:

1. parar de aceitar novas requisições;
2. concluir requisições em andamento;
3. cancelar tarefas;
4. encerrar consumidores;
5. fechar conexões;
6. descarregar buffers;
7. finalizar dentro de um prazo.

Toda goroutine de longa duração deve possuir alguma estratégia de cancelamento.

Uma goroutine sem controle de ciclo de vida pode causar:

- vazamento de recursos;
- deadlocks;
- processamento após o encerramento;
- perda de mensagens;
- dificuldade em testes.

---

## Inclua observabilidade desde o início

> **Cordel técnico**
>
> Sistema sem observação  
> É viagem no escuro.  
> Quando a falha se apresenta,  
> O diagnóstico é inseguro.  
> Log, métrica e rastreamento  
> Fazem o caminho mais seguro.

Uma aplicação deve fornecer sinais suficientes para entender seu comportamento em produção.

O conjunto mínimo inclui:

- logs estruturados;
- métricas;
- traces distribuídos;
- correlation ID;
- health check;
- readiness check.

Um log estruturado pode ser escrito assim:

```go
logger.InfoContext(
	ctx,
	"order created",
	"order_id",
	created.ID,
	"customer_id",
	created.CustomerID,
	"amount",
	created.Amount,
)
```

Esse formato é mais útil que concatenar valores em uma mensagem:

```go
log.Printf(
	"Pedido %s criado para cliente %s",
	id,
	customerID,
)
```

A estrutura permite:

- filtrar por atributos;
- correlacionar requisições;
- construir dashboards;
- investigar incidentes;
- criar alertas.

Também é necessário evitar métricas com alta cardinalidade.

Não utilize identificadores únicos como labels:

```text
order_id
customer_id
email
request_id
URL completa
```

Labels devem possuir um conjunto relativamente limitado de valores.

---

## Estruture os testes por finalidade

> **Cordel técnico**
>
> Teste rápido orienta,  
> Teste integrado dá garantia.  
> Cada tipo tem seu espaço,  
> Seu custo e serventia.  
> Não dependa só do topo  
> Para provar a engenharia.

Uma estrutura escalável deve permitir testar regras de negócio sem iniciar toda a aplicação.

### Testes unitários

```go
func TestCreateOrder(t *testing.T) {
	repository := &repositoryStub{}

	clock := fixedClock{
		value: time.Date(
			2026,
			8,
			5,
			12,
			0,
			0,
			0,
			time.UTC,
		),
	}

	ids := fixedIDGenerator{
		value: "order-123",
	}

	handler := NewCreateHandler(
		repository,
		ids,
		clock,
	)

	created, err := handler.Handle(
		context.Background(),
		CreateCommand{
			CustomerID: "customer-1",
			Amount:     10_000,
		},
	)
	if err != nil {
		t.Fatalf(
			"unexpected error: %v",
			err,
		)
	}

	if created.ID != "order-123" {
		t.Errorf(
			"expected order-123, got %s",
			created.ID,
		)
	}
}
```

### Testes de integração

Validam implementações reais de infraestrutura:

```text
internal/order/postgres/
├── repository.go
└── repository_test.go
```

Esses testes podem verificar:

- consultas SQL;
- constraints;
- transações;
- serialização;
- migrations;
- integração com brokers;
- comportamento de clientes externos.

### Testes de contrato

Validam se consumidores e provedores respeitam contratos esperados.

São especialmente úteis em:

- APIs HTTP;
- eventos;
- protobuf;
- OpenAPI;
- integrações entre serviços.

### Testes ponta a ponta

Validam os fluxos críticos do sistema completo.

Devem ser usados com moderação, pois costumam ser:

- lentos;
- frágeis;
- mais difíceis de diagnosticar;
- dependentes de infraestrutura.

Uma estratégia equilibrada costuma possuir:

```text
muitos testes unitários
alguns testes de integração
poucos testes ponta a ponta
```

---

## Não extraia microserviços cedo demais

> **Cordel técnico**
>
> Antes da fronteira em rede,  
> Faça a fronteira no código.  
> Se o domínio está isolado,  
> Seu futuro fica lógico.  
> Distribuir sem disciplina  
> Multiplica o diagnóstico.

Microserviços adicionam complexidades que não aparecem nos diagramas mais simples:

- comunicação remota;
- autenticação entre serviços;
- retries;
- consistência eventual;
- descoberta de serviços;
- circuit breakers;
- tracing distribuído;
- versionamento de contratos;
- observabilidade;
- implantação;
- filas;
- coordenação operacional.

Um módulo deve ser extraído quando houver uma razão concreta:

- necessidade de escala independente;
- ownership por uma equipe diferente;
- ciclo de implantação próprio;
- requisitos específicos de disponibilidade;
- isolamento de falhas;
- tecnologia especializada;
- armazenamento independente;
- fronteira de domínio madura.

Um módulo interno:

```text
internal/billing/
```

pode futuramente se tornar:

```text
services/billing/
```

Essa extração será mais segura se o módulo já possuir:

- contratos explícitos;
- domínio isolado;
- dependências controladas;
- persistência delimitada;
- testes próprios;
- comunicação bem definida.

A fronteira lógica deve existir antes da fronteira de rede.

---

## Evite arquitetura cerimonial

> **Cordel técnico**
>
> Se precisa de oito tipos  
> Para um dado atravessar,  
> Talvez a arquitetura  
> Tenha vindo atrapalhar.  
> Abstração é ferramenta,  
> Não troféu para mostrar.

Arquitetura excessivamente cerimonial pode produzir estruturas como:

```text
CreateOrderController
CreateOrderUseCase
CreateOrderInteractor
CreateOrderInputPort
CreateOrderOutputPort
CreateOrderPresenter
CreateOrderGateway
CreateOrderRepositoryAdapter
```

Para casos simples, isso aumenta:

- indireção;
- quantidade de arquivos;
- tempo de navegação;
- carga cognitiva;
- dificuldade de manutenção.

Uma alternativa mais idiomática em Go pode ser:

```text
order/
├── create.go
├── repository.go
└── http.go
```

Go favorece:

- composição;
- interfaces pequenas;
- pacotes coesos;
- dependências explícitas;
- nomes diretos;
- pouca indireção.

Camadas devem ser adicionadas quando solucionam um problema real.

Não devem existir apenas porque um diagrama arquitetural diz que deveriam existir.

---

## Evolução progressiva da estrutura

> **Cordel técnico**
>
> Projeto cresce por etapas,  
> Não por grande revolução.  
> Cada pasta deve nascer  
> Para resolver uma tensão.  
> Estrutura sustentável  
> Acompanha a aplicação.

### Projeto pequeno

```text
cmd/
└── api/
    └── main.go

internal/
├── order/
├── customer/
└── platform/
```

Nesse estágio, pacotes planos costumam ser suficientes.

### Projeto intermediário

```text
internal/order/
├── domain/
├── application/
├── http/
└── postgres/
```

A separação aparece conforme aumentam:

- regras;
- dependências;
- casos de uso;
- adaptadores;
- arquivos;
- integrantes da equipe.

### Projeto grande

```text
internal/
├── order/
├── billing/
├── inventory/
├── identity/
└── platform/
```

Cada módulo pode possuir internamente:

```text
billing/
├── domain/
├── application/
├── http/
├── events/
├── postgres/
└── tests/
```

O importante não é copiar uma estrutura fixa.

O importante é preservar:

- coesão;
- limites;
- direção das dependências;
- clareza;
- testabilidade.

---

## Estrutura recomendada

> **Cordel técnico**
>
> `cmd` conduz a entrada,  
> `internal` dá proteção.  
> O domínio guarda a regra,  
> Plataforma, a conexão.  
> Com contrato bem pequeno,  
> Cresce firme a aplicação.

Uma estrutura equilibrada pode ser:

```text
my-project/
├── cmd/
│   ├── api/
│   │   └── main.go
│   ├── worker/
│   │   └── main.go
│   └── migrate/
│       └── main.go
│
├── internal/
│   ├── app/
│   │   ├── application.go
│   │   ├── config.go
│   │   └── dependencies.go
│   │
│   ├── order/
│   │   ├── domain/
│   │   ├── application/
│   │   ├── http/
│   │   └── postgres/
│   │
│   ├── customer/
│   ├── billing/
│   │
│   └── platform/
│       ├── database/
│       ├── messaging/
│       ├── observability/
│       └── httpserver/
│
├── migrations/
├── api/
│   └── openapi.yaml
├── deployments/
├── scripts/
├── go.mod
├── go.sum
└── Makefile
```

Essa estrutura combina:

```text
monólito modular
+ pacotes orientados ao domínio
+ código privado em internal
+ executáveis em cmd
+ composição explícita
+ interfaces pequenas
+ aplicação stateless
+ observabilidade
+ testes nas fronteiras
```

---

## Perguntas para avaliar a arquitetura

Uma boa estrutura deve permitir responder rapidamente:

1. Onde está a regra de negócio?
2. Qual módulo é responsável por esta funcionalidade?
3. Quem depende de quem?
4. Como substituo uma integração externa?
5. Como testo esta regra sem iniciar toda a aplicação?
6. Como o sistema encerra suas operações?
7. Como identifico o que aconteceu em produção?
8. Como duplicidades são tratadas?
9. Como conflitos concorrentes são evitados?
10. Como este módulo poderia ser extraído no futuro?

Se cada resposta exige percorrer dezenas de diretórios, a estrutura provavelmente não está comunicando bem suas responsabilidades.

---

## Conclusão

> **Cordel técnico**
>
> Projeto Go bem estruturado  
> Não depende de invenção.  
> Depende de boas fronteiras,  
> Clareza e composição.  
> Do domínio no seu centro  
> E explícita dependência em ação.

Não existe uma única estrutura correta para todos os projetos Go.

Uma aplicação pequena não precisa da mesma organização de um sistema com dezenas de módulos, múltiplos executáveis e integrações distribuídas.

Por isso, a estrutura deve evoluir conforme surgem necessidades reais.

Comece com um monólito modular.

Organize o código pelas capacidades do negócio.

Mantenha o domínio independente da infraestrutura.

Use interfaces pequenas no lado consumidor.

Centralize a composição das dependências.

Projete a aplicação para ser stateless.

Adicione idempotência, graceful shutdown e observabilidade desde cedo.

Extraia microserviços somente quando houver razões técnicas e organizacionais concretas.

> **Cordel técnico**
>
> Pois escalar não é somente  
> Mais máquina para rodar.  
> É permitir que código e equipe  
> Possam juntos avançar.  
> Sem transformar cada mudança  
> Num motivo para tudo quebrar.