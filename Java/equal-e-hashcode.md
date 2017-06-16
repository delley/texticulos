
# equals() & hashCode()

Entenda de forma efetiva e correta a definição dos métodos `equals()` e `hashCode()`.

## Introdução

Não há escapatórias, cedo ou tarde, um dia todo programador `Java` vai ter que sobrescrever os métodos `equals()` e `hashCode()`, e quando isso acontecer, é bom que o programador saiba o porque de está fazendo isso.

Para quem está começando a programar em `Java`, o método `equals()` até dá uma pista do que faz ou para que serve. Já o `hashCode()`... serve para quê? Para responder a essa pergunta precisamos de um embasamento mais profundo sobre `estruturas de dados`, principalmente sobre [tabelas de espalhamento](https://pt.wikipedia.org/wiki/Tabela_de_dispers%C3%A3o).

## Tabelas de espalhamento



## Definindo igualdade



## Requisitos para implementar `equals()` e `hashCode()`

**Simetria**: Para duas referências, a e b, a.equals (b) se e somente se b.equals (a)
**Reflexividade**: para todas as referências não nulas, a.equals (a)
**Transitividade**: se a.equals (b) e b.equals (c), então a.equals (c)
**Consistência com hashCode()**: dois objetos iguais devem ter o mesmo valor hashCode (), se a.equals(b) então a.hashCode() == b.hashCode()
