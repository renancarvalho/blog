---
layout: post
title:  "COMEÇANDO COM REQUIREJS E BACKBONE USANDO COFFEESCRIPT."
date:   2014-06-03 10:18:00
categories: Backbone
---

Fala galera. Esse é meu primeiro post do blog e a ideia aqui é expor um pouco das minhas experiências, dicas, ideias e soluções adotadas.

Nos últimos 2 anos tenho trabalhado bastante com Single Page Application (SPA), utilizando Backbone, RequireJs, CoffeeScript e alguns outros frameworks, e nesse post vou mostrar como botar tudo isso pra funcionar.

Obs: Este post parte da premissa que você já conhece um pouco sobre Backbone e como ele funciona, então não vou explicar a fundo, e sim focar em como fazer o setup do Require e Backbone. Prometo fazer um post mais explicativo sobre Backbone depois.


Trabalhar com aplicações client-side, com funcionalidades complexas e grandes dependências de outros módulos JavaScript que sua aplicação pode precisar, podem ser bem difíceis (exemplo clássico: um plugin qualquer feito Jquery depende da própria library do Jquery para funcionar). Conforme sua aplicação for crescendo, gerenciar essas dependências pode ser complicado. Será muito mais saudável para a aplicação e para você também organizar isso de forma mais fácil.


Para simplificar o que eu quero dizer, imagine que nossa aplicação tenha uma dependência do Jquery, Backbone e de um plugin JavaScript que você fez e adora. Logo, teremos que referenciar esses arquivos na nossa aplicação, basicamente você faria algo parecido com isso.


Estrutura da aplicação.

{% highlight html %}
root/
.....index.html
.....scripts
........../lib
...............backbone.min.js
...............underscore.min.js
...............jquery.min.js
...............seuPlugin.min.js
.....views/
.....models/
{% endhighlight %}



{% highlight html %}
<script src="scripts/lib/undesrscore.min.js"></script>
<script src="scripts/lib/backbone.min.js"></script>
<script src="scripts/lib/jquery.min.js"></script>
<script src="scripts/lib/seuPlugin.min.js"></script>
{%endhighlight%}


Legal, mas qual o problema?

- Difícil saber exatamente as dependências entre os arquivos.
- O browser carrega os arquivos de forma assíncrona. O que aconteceria se o Backbone, que tem uma forte dependência do Underscore, fosse carregado primeiro? Crash :( 
- Difícil manutenção e gerenciamento das dependências, conforme sua aplicação for crescendo.
- Carregar tudo de uma vez, às vezes sem necessidade (lazy load fica impossível).
- Não faz bem para a saúde do desenvolvedor.. hehehehe



### RequireJs pode ajudar bastante com esse tipo de problema ###

####Mas o que então é o RequireJs?####

>"RequireJS is a JavaScript file and module loader. It is optimized for in-browser use, but it can be used in other JavaScript environments, like Rhino and Node. Using a modular script loader like RequireJS will improve the speed and quality of your code."



Uma das ideias do Require é justamente gerenciar de forma mais simples as dependências que sua aplicação possui. No nosso caso, Jquery, Backbone, Underscore e seu plugin mágico.

Para fazer o download do RequireJs [clique aqui][requireUrl]. Neste post foi utilizada a versão 2.1.13.


####Refatorando nosso arquivo html.####


{% highlight html %}
<script data-main="init.js" src="scripts/lib/require.min.js"></script>
{%endhighlight%}


Agora não temos mais a necessidade de especificar todos os nossos arquivos JavaScript no arquivo html. Dessa forma, a responsabilidade passa a ser do RequireJs. Dentro do arquivo init.coffee é onde iremos configurar nossas dependências.

{% highlight coffee%}
root/
.....index.html
.....scripts
........../lib
...............backbone.min.js
...............underscore.min.js
...............jquery.min.js
...............seuPlugin.min.js
...............require.min.js     
.....init.coffee     
.....views/
.....models/
{%endhighlight%}




###Configurando o Init.Coffee###

{% highlight coffee%}
require.config    
    baseUrl: 'scripts'     
    paths:
        Jquery : 'lib/jquery.min.js'
        Backbone : 'lib/backbone.min.js'
        underscore : 'lib/underscore.min.js'
        SeuPlugin : 'lib/seuplugin.min.js'    
 	shim : 
        Backbone : 
            deps : ['underscore', 'Jquery']
            exports : 'Backnone'
        SeuPlugin : 
            deps : ['Jquery']
            exports : 'MeuPlugin'

{%endhighlight%}



A partir de agora, teremos que escrever as dependências da nossa aplicação nesse arquivo(`Init.coffee`). Talvez você se pergunte, qual a vantagem? Apenas mudou de lugar onde eu carregarei meus arquivos JavaScript. E a resposta é:


- Ficaram mais fáceis e claras suas dependências.
- Possibilidade de trabalhar com lazy load.
- As dependências entre os arquivos estão mais explicitas.
- Você não corre mais o risco de um arquivo carregar na frente do outro.
- Faz bem para a saúde do desenvolvedor :) 


O comando `shim` ajuda bastante na forma de como escrevemos nossas dependências,
nele podemos fazer com que um arquivo que tenha dependência do outro não seja carregado antes. No nosso exemplo, estamos especificando que o Backbone depende do Underscore e do Jquery, e que vou usar o Backbone na minha aplicação com o namespace 'Backbone'.       


###Backbone entrando na jogada.###



Sou um fã de carteirinha do Backbone. Quando realmente percebi as vantagens que ele me traria, comecei a estudá-lo bastante, e hoje certamente é um dos frameworks que mais gosto de trabalhar, mas confesso que não foi amor à primeira vista.

'Aceito' alguns desenvolvedores dizerem que Backbone é difícil, que não veem muita vantagem em usá-lo e etc, mas garanto que uma vez que você de fato entender o que ele se propõe a fazer e como ele faz, será realmente muito bom.


####Mas o que é BackboneJs?####



Basicamente, Backbone é um framework que tem como principal foco organizar seu código JavaScript, utilizando conceitos da arquitetura MVC (model, view, controller). Isso não significa que ele seja um framework MVC. Existe uma comunidade grande de desenvolvedores usando Backbone, e é muito difícil você passar por alguma dificuldade que ninguém já tenha passado. Na internet existem vários blogs super legais relacionados a Backbone, e a documentação oficial é bem legal também. [Esse é o site oficial][backboneUrl] , e obviamente para fazer download é nesse link que você tem que ir. Para esse post eu usei a ultima versão (1.1.2).

Existe uma discussão sobre qual tipo de framework o Backbone se encaixaria, existem desenvolvedores que o consideram um framework MVC, outros MV*, outros MVVM e por aí vai. Sinceramente, eu não me preocupo muito em qual classificação ele se encaixa. O que vale a pena é o estudo, e tentar entender como ele pode te ajudar a desenvolver uma aplicação melhor, melhorando as práticas possíveis relacionadas a MVC, e isso tem me gerado um retorno muito positivo.

Como o foco deste post não é entrar a fundo em Backbone e nem em RequireJs, não vou explicar muito a estrutura de funcionamento do Backbone, mas novamente prometo fazer um post mais explicativo mais para frente.





 Voltando para nossa aplicação, já temos configurado o RequireJs, que faz um papel fantástico, agora vamos configurar como o Backbone entra nessa jogada. Primeiro de tudo, você precisa fazer o download do Backbone, lembrando que o Backbone tem uma forte dependência do Underscore, logo o download do Underscore também é necessário pelo [site oficial do Underscore][underscoreUrl]. Feito isso salve os arquivos na pasta 'scripts/lib' (fique a vontade para configurar onde seus arquivos ficarão, só não esqueça de mudar o caminho no RequireJs).

Se você salvou os arquivos em 'scripts/lib', seu arquivo init.coffee continuará funcionando normalmente. Falando em init.coffee, vamos dar uma olhada dentro dele.

 
Já havíamos previamente especificado o caminho do Backbone, mas não tínhamos o arquivo físico na pasta. Se você tivesse rodado a aplicação, certamente um "404 not found" apareceria no seu console. Como agora temos o arquivo, provavelmente um "200 OK" aparecerá :)



####Configurando o Backbone junto com RequireJs####



É necessário criarmos um arquivo chamado router.coffee no root da nossa aplicação, pois é nele que configuraremos as rotas (caminhos digitados na Url) que nossa aplicação usará.

{% highlight coffee%}
root/
.....index.html
.....scripts
........../lib
...............backbone.min.js
...............underscore.min.js
...............jquery.min.js
...............yourplugin.min.js
...............require.min.js     
.....init.coffee     
.....router.coffee
.....views/
.....models/


{%endhighlight%}


Precisamos agora fazer com que o nosso arquivo 'router.coffee' seja carregado como um módulo pelo RequireJs. Como nosso BaseUrl é o caminho '/scripts' e nosso arquivo esta em '/scripts', é só usar o comando "require ['router']" que se baseará no BaseUrl como caminho para buscar o arquivo especificado. Neste exemplo 'script/router.coffee'.

{% highlight coffee%}
require.config    
    baseUrl: 'scripts'    
    paths:
        Jquery : 'lib/jquery.min.js'
        Backbone : 'lib/backbone.min.js'
        underscore : 'lib/underscore.min.js'
        SeuPlugin : 'lib/seuplugin.min.js'
     shim : 
        Backbone : 
            deps : ['underscore']
            exports : 'Backnone'
        SeuPlugin : 
            deps : ['Jquery']
            exports : 'MeuPlugin'
    require ["Router"], (Router) ->
        router = new Router()
        Backbone.history.start()
{%endhighlight%}


Obs: Cuidado com a identação utilizada pelo CoffeeScript, um tab a mais pode literalmente estragar tudo.








{%highlight coffee%}
define [
    "Backbone"
    "views/HomeView"
], (Backbone, HomeView) ->
	class AppRouter extends Backbone.Router
		routes:->
            "home":"renderizarHome" 
        renderHome:->
            #Fazer alguma coisa.

{%endhighlight%}



No código acima, configuramos uma rota simples para nossa aplicação. Sempre que a Url do browser mudar, o Backbone router verificará se existe alguma rota configurada para a Url digitada. Caso exista, a function mapeada para a rota será invocada.
Neste caso, se o usuário digitar 'localhost:10000/#home' o Backbone invocará a function 'renderizarHome' que renderizará a view com as informações da Home se existir alguma view criada naquele diretório com aquele nome.

É isso, agora o Backbone é carregado pelo RequiereJs, e o router esta 'ligado', observando qualquer mudança na url.

Espero ter ajudado galera... O código está disponivivel no [GitHub][githubUrl].



[githubUrl]: http://github.com/renancarvalho
[SobreMim]:    http://localhost:4000/about.html
[requireUrl]: http://requirejs.org/docs/download.html
[backboneUrl]: http://www.backbonejs.org
[underscoreUrl]: http://underscorejs.org