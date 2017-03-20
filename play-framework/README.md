# Play Framework
O `Play` é um framework para desenvolvimento web. Foi inicialmente escrito em `Java`, e posteriormente, reescrito em `Scala`. Apesar de ser um framework web, o `Play` não segue a especificação `Java EE`, ou seja, não usa a API Servlet, que é comumente usada por outros frameworks web Java. Outro ponto que merece destaque, é fato no `Play` usar internamente o [Netty](https://netty.io/) como servidor HTTP.

### Notas
A partir da versão 2.4, o Play adotou o Guice para forncer Injeção de Dependências. Em decorrência disso, ocorreram algumas mudanças como as rotas são geradas. Até a versão 2.4, o padrão era usar o gerador de rotas estáticas. A partir da versão 2.5, o padrão é o gerador de rotas injetadas, ou seja, os métodos dos controles não precisam mais serem `static`.

Outra mudança siginificativa foi a troca dos controles estáticos, padrão até a versão 2.4, por injeção de dependência, padrão a partir da versão 2.5.   

Mais informações podem ser obtidas no [Play 2.4 Migration Guide](https://playframework.com/documentation/2.5.x/Migration24#dependency-injection) e no [Play 2.5 Migration Guide](https://playframework.com/documentation/2.5.x/Migration25#routes-generated-with-injectedroutesgenerator).

### Referências
[Slides da palestra sobre Play ministrada no Meetup Java ABC](http://talk.sabercompartilhar.com.br)

[Conheça o Play Framework: Rápido e Ágil no Desenvolvimento Web](http://blog.sabercompartilhar.com.br/conheca-o-play-framework-rapido-e-agil/)

[Criando um HelloWorld com Play Framework](http://blog.sabercompartilhar.com.br/primeiros-passos-com-o-play-framework-hello-world/)

[Começando um projeto no Play! Framework usando Java](http://blog.caelum.com.br/comecando-um-projeto-no-play-framework-usando-java/)

[Uma avaliação do play! framework](http://luizricardo.org/tag/play-framework/)

[Qual o futuro do Play Framework em Java ?](http://www.boaglio.com/index.php/2014/12/29/qual-o-futuro-do-play-framework-em-java/)

[Play 2.3 com Activator – o que mudou](http://www.boaglio.com/index.php/2014/11/16/play-2-3-com-activator-o-que-mudou/)

[Saindo da mesmisse com Play framework e MongoDB](http://www.boaglio.com/index.php/2014/03/12/saindo-da-mesmisse-com-play-framework-e-mongodb/)

[Facilidade do Play 2 em fazer CRUD com MySQL](http://www.boaglio.com/index.php/2014/03/25/facilidade-play-2-em-fazer-crud-com-mysql/)

[Play Framework – alta produtividade em Java](http://blog.erkobridee.com/2011/12/05/play-framework-alta-produtividade-em-java/)

Vídeos no YouTube:
[Play! Framwork - Demonstração prática detalhada](https://www.youtube.com/watch?v=prAdy2rIzTU)
