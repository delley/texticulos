+++ 
draft = false
date = 2019-12-14
title = "Introdução ao Go"
slug = "introducao-ao-go" 
tags = ["Go"]
categories = ["programação"]
series = ["Go"]
images = ["/images/introducao-ao-go.png"]
+++

{{< figure src="/images/introducao-ao-go.png" alt="Intodução ao Go" class="centered" >}}

O que é Go? Como o próprio site define: 

> _Go é uma linguagem de programação de código aberto que facilita a criação de software simples, confiável e eficiente._
>> <cite>[golang.org][1]</cite>

O projeto da linguagem Go foi iniciado em 2007 pelos engenheiros da Google Rob Pike, Ken Thompson e Robert Griesemer. A linguagem foi apresentada pela primeira vez em 2009 e teve a versão 1.0 lançada em 2012. 

Go tem uma certa semelhança com C, principalmente no que diz respeito a alcançar o máximo com o mínimo de recursos. Porém, não é uma versão atualizada do C. Na verdade, o Go adapta boas ideia de várias linguagens, sempre descartando funcionalidades que trazem complexidade e código não confiável. 

É uma linguagem moderna que utiliza recursos para concorrência eficientes. Aborda a abstração de dado e a programação orientada a objetos de forma muito flexível, além de ter gerenciamento de memória automático, também conhecido como _garbage collection_ (coletor de lixo)

Além de ser uma liguagem fortemente tipada, o que ajuda muito no desenvolvimento, Go também é compilada, o que faz com que seja bastante performática.

Pelo fato de ser uma linguagem compilada, o compilador é uma peça importante da linguagem, porém, em Go, em vez disso ser uma fraqueza, é na verdade um ponto forte, pois o compilador é muito rápido, faz diversas checagens antes de gerar o executável final e tudo isso torna os programas desenvolvidos em Go confiáveis e robustos.

Geralmente, programas escritos em Go são multiplataforma, ou seja, podem ser executados em diversas plataformas, desde um simples Raspberry PI até servidores Linux, Windows ou MAC.

Desde sua criação, diversas empresas tem usado a linguagem Go nos mais diversos domínios, desde a área gráfica, aplicações móveis e até em aprendizagem automática (_machine learning_). Isso mostra que, apesar de ser bem apropriada para a criação de infraestruturas como servidores em rede, ferramentas e sistemas para programadores, Go é realmente uma liguagem de propósito geral.

É possivel encontrar uma lista com empresas do mundo inteiro no seguinte link: <https://github.com/golang/go/wiki/GoUsers>

[1]:https://golang.org/