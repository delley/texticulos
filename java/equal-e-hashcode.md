
# equals() & hashCode()

Entenda de forma efetiva e correta a definição dos métodos `equals()` e `hashCode()`.

## Introdução

Não há escapatória, cedo ou tarde, um dia todo programador `Java` vai ter que sobrescrever os métodos `equals()` e `hashCode()`, e quando isso acontecer, é bom que o programador saiba o porque de está fazendo isso.

Para quem está começando a programar em `Java`, o método `equals()` até dá uma pista do que faz ou para que serve. Já o `hashCode()`... serve para quê? Para responder a essa pergunta precisamos de um embasamento mais profundo sobre [estruturas de dados](https://pt.wikipedia.org/wiki/Estrutura_de_dados), principalmente sobre [tabelas hash](https://pt.wikipedia.org/wiki/Tabela_de_dispers%C3%A3o), também conhecidas como tabelas de espalhamento.

## Tabelas *hash*

Uma tabela *hash* é uma estrutura de dados especial, que associa chaves de pesquisa a valores, com o objetivo de a partir de uma chave simples, fazer uma busca rápida e obter o valor desejado, e são tipicamente utilizadas para implementar vetores (arrays) associativos (vetores que podem ter qualquer objeto como um índice).

O fato é que Java não suporta vetores associativos, logo, fica evidente a necessidade do uso de tabelas *hash*, e é justamente dessa necessidade que o método `hashCode()` surgiu, uma vez que, suportar a geração do *hash* diretamente na classe `Object` facilita o desenvolvimento e o uso de contentores baseados em hash (**HashMap** e/ou **HashTable**).

## Definindo igualdade



## Requisitos para implementar `equals()` e `hashCode()`

**Simetria**: Para duas referências, a e b, a.equals (b) se e somente se b.equals (a)

**Reflexividade**: para todas as referências não nulas, a.equals (a)

**Transitividade**: se a.equals (b) e b.equals (c), então a.equals (c)

**Consistência com hashCode()**: dois objetos iguais devem ter o mesmo valor hashCode (), se a.equals(b) então a.hashCode() == b.hashCode()

## Referências

* [Wikipedia - Estruturas de Dados](https://pt.wikipedia.org/wiki/Estrutura_de_dados)
* [Wikipedia - Tabelas de Espalhamento](https://pt.wikipedia.org/wiki/Tabela_de_dispers%C3%A3o)
* [IBM developerWorks - Hashing it out](https://www.ibm.com/developerworks/library/j-jtp05273/)
