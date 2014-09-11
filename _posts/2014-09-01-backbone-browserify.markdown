---
layout: post
title:  "BACKBONE E BROWSERIFY"
date:   2014-09-01 23:28:00
categories: Javascript
---

Salve salve galera!!!! Este post é o primeiro que escrevo para o <a href="https://github.com/blogmv" target="_blank">BlogMV*</a>, e a idéia é mostrar como fazer o setup do backbone junto com o browserify. Para quem não curte muito o RequireJS, ou simplesmente prefere carregar modulos Javascritp baseados em UMD, este post vai ser bem legal. E para quem só quer aprender alguma coisa nova, também vai ser legal..hehehe


Here we go!

Basicamente irei dividir essa aplicação em 2 partes.

1 Parte:

- Instalação das dependências(Backbone, Jquery, Browserify, etc.)
- Setup do projeto. 
- Fazer um get na nossa <a href="https://github.com/blogmv/backend" target="_blank">API</a>, e retornar as informações disponiveis, que neste exemplo são os artigos e os comentários dos artigos.

2 Parte:

- Fazer o post de um comentário.
- Preparar a aplicação para produção.


Obs: Todo o código deste exemplo está disponível no <a href="https://github.com/blogmv/backbone-browserify" target="_blank">github</a>.


Primeiro de tudo acho que é fundamental saber a diferença entre AMD e UMD. O foco do post não será em explicar isso detalhadamente, por isso vou fazer um overview rápido.

###RequireJS (AMD)
AMD é um padrão de definição de módulos Javascript utilizado principalmente quado estamos falando de Web (Pois é assíncrono). Sabemos que para carregar um arquivo Javascript pelo browser, devemos utilizar a tag "Script", essa tag diz ao browser para fazer uma requisição para a url especificada no atributo "src" dentro da tag "script", e baixar o arquivo. Feito isso, conseguimos então usar de fato o arquivo Javascript.

####Como eu defino um módulo AMD ?
~~~~~~
define('MeuModulo', 
    ['DependenciaA', 'DependenciaB']
       function ( depA, depB ) {
          var MeuModulo = {
             fooBar:function(){
                console.log('Foooooo');
              }
          }     
       return MeuModulo;
    });
~~~~~~



Tenha em mente que sempre que definirmos um módulo usando esta sintaxe, esse modulo será carregado através da tag script, que o <a href="http://requirejs.org/" target="_blank">RequireJs </a>"mágicamente" transformará para você. A grande vantagem de usar RequireJS, é justamente não se preocupar em organizar a ordem em que seus scripts e dependências serão carregados pelo Browser.

Ahhh, e não se esqueça que o RequireJS é só um module loader. Ele não tem nada a ver com o tipo de sintaxe.



###CommonJS (UMD)
UMD por outro lado é um padrão mais focado em carregamento de módulos Javascript server-side. Quem usa Nodejs, provavelmente está bem mais familiarizado com a sintaxe. Como exemplo disso o NodeJs implementa módulos baseado no CommonJS.

####Como eu defino um módulo UMD ?
~~~~~~
var MeuModulo = require('ModuloQualquer');
    function fooBar(){
        console.log('foooo');
    };
    module.exports = fooBar;
~~~~~~

Tenha em mente também, que esse tipo de definição de módulo, o navegador nem pintado de ouro vai entender. 
Então como faremos para usarmos definição de módulos baseado UMD no navegador?


#Browserify

 <a href="http://browserify.org/" target="_blank">Browserify</a> pode ser uma das formas de resolver esse problema. Basicamente, o browserify vai gerar uma tag "script" da sua aplicação, e ela o navegador entenderá, o legal é que essa única tag tem tudo que sua aplicação precisa. Imaginem um bundle. É bem por ai..

Nosso app de exemplo, será um blog baseado no projeto <a href="https://github.com/blogmv/baby-steps" target="_blank">baby-steps</a> (totalmente open-source), fiquem a vontade para clonar o projeto, vai ser bem mais fácil com ele clonado ;)


##Clonando o projeto Baby-Steps
Acesse o terminal, navegue até a pasta em que deseja instalar o projeto, e então digite:

> git clone git@github.com:blogmv/baby-steps.git

Depois disso, sua estrutura de pastas, deverá ser assim.

<img src="/assets/images/folder.png"/>

Obs:Instale o NodeJs, <a href="http://lmgtfy.com/?q=Instalar+node+Js" target="_blank">não sabe como?</a>

Agora rodando o comando `bower install`, as dependências especificadas no arquivo `bower.json` serão instaladas.
Na verdade para este projeto especifico, só precisaremos do bootstrap, as outras dependências serão instaladas via NPM mais a frente.


##Instalando as dependências especificas para o backbone + browserify.

Bom, inicialmente vamos precisar instalar algumas dependências do nosso projeto, como por exemplo, backbone, underscore, jquery, etc.

#####No Command line digite

> `npm init`

Após preencher as informações, um arquivo "package.json" será criado.

Agora digite.

> npm install backbone, jquery, underscore --save-dev

Por default o underscore vem como dependência do Backbone, mas eu vou precisar usa-lô para templates e fora do uso interno do backbone, por isso é necessário especificar ele na instalação.

Este comando vai instalar as dependências especificadas, e automaticamente `--save-dev` ,adicionará os mesmos como dependências do projeto no arquivo package.json. Uma pasta chamada node_modules foi criada pelo NPM install, la dentro estão os modulos que instalamos.

##Setup do projeto

Adapte a estrutura do projeto igual a imagem abaixo:


<img src="/assets/images/folder-2.png"/>


* Index.html - Arquivo html principal da aplicação
* Js/View - Onde ficarão as views do backbone
* Js/Model - Onde ficarão os models do backbone
* Js/Collection - Onde ficarão as collections do backbone
* Node_modules - Criada automaticamente pelo Node.
* Css - Css da aplicação
* Vendor - Criado automaticamente pelo bower quando o comando "bower install" for executado. ( vale uma olhada no arquivo .bowerrc ).
* Main.js - Arquivo Javascript principal da nossa aplicação.
* Router.js - Arquivo de Rotas do Backbone

Os demais arquivos, vamos ver adiante.

### Main.Js

Dentro do Main.js é onde nossa aplicação começa, ele te a responsabilidade de instanciar o Backbone.Router, que observará as altereções feitas na url do browser e decidir qual view renderizar.

Basicamente, iremos carregar o backbone e o Router, que esse para este preciso informar qual o caminho do modulo. Como instalamos o Backbone usando o Npm, não é necessário espeficar de onde o arquivo deverá ser carregado.

~~~~~~~
var backbone = require('backbone');
var Router = require('./router');
var router = new Router(); //instanciamos o router
router.startApp(); // essa function iniciará o history do backbone.
~~~~~~~

### Router.Js

O Router tem a função de saber o que fazer quando alguma coisa mudar na url do browser. No nosso exemplo, temos uma rota default configurada que renderizará a nossa view. No topo do arquivo, novamente precisaremos especificar as dependências que esse módulo terá, Neste caso o Backbone, Jquery, algumas views, e um model.
Note que é o $(jquery) é atribuido a Backbone.$. Isso é necessário, porque o backbone depende do jquery. Se isso não for feito, nada funcionará.

~~~~~~
//dependências do modulo.
var Backbone = require('backbone');
var $ = require('jquery');
var ActiveModel = require('./model/article');
var App = require('./view/app');
var ArticleContent = require('./view/articleContent');
var ArticleComment = require('./view/articleComment');
Backbone.$ = $

module.exports = Backbone.Router.extend({
	routes: {
	   // Rota default da aplicação.
		"":"renderHome"   
	},

	renderHome: function() {     
		activeModel = new ActiveModel(); //Model responsável que repensenta um artigo do blog.
		
		//Injeção de dependencia do el, e do model nas views.
		app = new App({el : $(".main"),model : activeModel});
		appContent = new ArticleContent({el : $(".main"),model : activeModel});
		appComment = new ArticleComment({el : $(".discussion"),model : activeModel});
	},

	startApp:function(){
        //Function que é chamada no Main.js
		Backbone.history.start();
	}
});
~~~~~~



## Model 

No backbone o model tem a responsabilidade de representar e gerenciar os dados da aplicação. Neste exemplo os models serão responsáveis por armazenar o conteúdo de um artigo, ou o titulo de um artigo, veremos melhor adiante.
Models também devem conter a lógica de validação das informações (caso seja necessário fazer isso no browser), e também de saber onde (url) ele deverá enviar ou obter informações do servidor.

Neste exemplo temos dois models, um para representar um artigo, e o outro que representa um comentário de um artigo especifico.


- Article Model

Os comentários no código são a melhor forma de explicar o que esta acontecendo.


~~~~~~
//dependências do modulo.
var Backbone= require('backbone');
var CommentCollection = require('../collection/comments');

//Um artigo pode ter alguns comentarios, então assim que inicializamos o model, uma instância da lista comentários é criada
module.exports =  Backbone.Model.extend({
    initialize: function(){
        this.comments = new CommentCollection([], {
            "article": this
        });
    },

    //function que retorna o titulo de artigo
    getTitle: function() {
        return this.get('title');
    },
    
    //function que retorna o conteúdo de artigo
    getContent: function() {
        return this.get('content');
    }
});


~~~~~~


O legal aqui é perceber que poderiamos pegar o titulo e o conteúdo do artigo diretamente da view, fazendo algo como `model.get('title')` dentro da view. O problema principal deste tipo de implementação é quebrar o conceito de SRP inserindo um comportamento do model dentro da view. Isso vale pra qualquer tipo de "comunicação" entre a view e o model.




- Comments model


Esse model não tem muito mistério, inicialmente ele contem apenas 3 functions que são usadas para obter as informações do próprio model.

~~~~~~~
//dependências do modulo.
var Backbone = require('backbone');


module.exports =  Backbone.Model.extend({
    //Pega o nome do autor do comentário.
    	getAuthorName: function() {
		return this.get('author_name');
	},

    //Pega o email do autor do comentário.
	getAuthorEmail: function() {
		return this.get('author_email');
	},

    //Pega o conteudo do comentário.
	getContent: function() {
		return this.get('content');
	}	
});

~~~~~~~



##Collections


Collections eu gosto de interpretar como um wrapper de varios módulos. É bem parecido com o conceito de `List`.


Neste exemplo temos também duas collections, uma para os artigos e um outra para os comentários.


- Article Collection

~~~~~~
//dependências do modulo.
var Backbone = require('backbone');
var ArticleModel = require('../model/article');

module.exports = Backbone.Collection.extend({
	
    //Endereço no server onde a collection irá fazer os request(get,put,post ou delete)

	url:   'http://blogmv.apiary-mock.com/api/articles',
    
    //Model que está atribuido a collection.

	model: ArticleModel
});

~~~~~~


- Comments collection

~~~~~~
//dependências do modulo.
var Backbone = require('backbone');
var CommentModel = require('../model/comment');


module.exports =  Backbone.Collection.extend({
	model: CommentModel,

    //Endereço no server onde a collection irá fazer os requests. A diferença é que nesta collection, precisamos fazer isto de forma dinamica.

	url: function(){
		return "http://blogmv.apiary-mock.com/api/articles/"+this.article.id+"/comments"

	},
	initialize: function(models, options){
		this.article = options.article
	}
});
~~~~~~


## Views

Views tem a função de representar de forma visual as informações do model da aplicação. Basicamente uma view deve ser responsável somente por iteração com usuário, não é responsabilidade de uma view saber se algum campo esta com um valor correto ou não, views devem representam o estado do model.


Até então o nosso exemplo contém três views. Uma representa a lista de articles, outra a exibição do article em si, e a ultima representa os comentários escritos para cada article.


- App

Essa view é responsável por mostrar a lista de artigos do blog. 

<img src="/assets/images/view1.png"/>



~~~~~~
//dependências do modulo.
var Backbone = require('backbone');
var $ = require('jquery');
var _ = require('underscore');
var ArticleCollection = require('../collection/articles');

module.exports = Backbone.View.extend({  

    
    template: _.template( $('#tmp-article-list').html() ),

    initialize: function (options) {
        //recebendo os parametros que foram passados pelo router.       
 
        this.el = options.el;
        this.activeModel = options.model;

        this.collection = new ArticleCollection();
        //buscando os artigos no servidor

        this.collection.fetch();         
        
        //escutando os eventos de sync, que serão disparados quando a requisição para o server for concluida, nesse será chamada imediamtamente a função render() e setFirstModelAsActive();

        this.listenTo(this.collection, 'sync', this.render);
        this.listenTo(this.collection, 'sync', this.setFirstModelAsActive);
    },

    //Renderiza o conteúdo da collection de artigos,
    //dentro do index.html existe um foreach que mosta a lista de itens.

    render: function() {
        this.$el.find('.list-group').html(this.template({
            'collection' : this.collection
        }));
    },

    //Pega o primeiro item da Collection e seta como o item ativo.

    setFirstModelAsActive: function() {
        var index = this.collection.at(0);

        if (!index) {
            this.activeModel.clear();
            return
        }

        index = index.toJSON();
        this.activeModel.set(index);
    }
});

~~~~~~ 



- Article content

<img src="/assets/images/view2.png"/>
  
  
  
~~~~~~
//dependências do modulo.

var Backbone = require('backbone');
var _ = require('underscore');
var $ = require('jquery');

module.exports = Backbone.View.extend({
	template: _.template( $('#tmp-article-content').html() ),
	
    //recebendo os parametros que foram passados pelo router.
	initialize: function (options) {
		this.el = options.el;
		this.activeModel = options.model;

        //escutando qualquer mudança no model e re-renderizando a view.
		this.listenTo(this.activeModel, 'change', this.render);
	},

    //renderizando a view.
	render: function() {
		this.$el.find('article').html(this.template({
			'activeModel' : this.activeModel
		}));
	}
	
});
~~~~~~



- Article Comments



<img src="/assets/images/view3.png"/>

~~~~~~
//dependências do modulo.

var Backbone = require('backbone');
var _ = require('underscore');
var $ = require('jquery');

module.exports = Backbone.View.extend({


	template: _.template( $('#tmp-article-comments').html() ),

	initialize: function (options) {
		this.activeModel = options.model;
		this.el = options.el;
        //Quando o  artigo principal for setado no model, o evento change será disparado e com isso carregar os omentários do artigo em questão.
        
		this.listenTo(this.activeModel, 'change', this.fetchComments);
	},
    
    //Carrega os comentários, assim que a requisição for feita, é chamado então a funcion this.render(), que renderizará a view.
	fetchComments: function(){
		this.activeModel.comments.once('sync', this.render, this);
		this.activeModel.comments.fetch();
	},


    //Com a collection de comentários em memoria, a view é renderizada. lembrando que estamos fazendo a lógica de template dentro do arquivo index.html.
	render: function() {
		this.$el.html(this.template({
			'collection' : this.activeModel.comments
		}));
	}
});


~~~~~~


- Index.html

Esse é nosso arquivo html com uma logica simples para templates. Estamos neste exemplo utilizando o underscore como engine template.

~~~~~~
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Backbone + RequireJS</title>
        <link rel="stylesheet" href="assets/vendor/bootstrap/dist/css/bootstrap.min.css">
        <link rel="stylesheet" href="assets/css/component.css">
    </head>
    <body>
        <header class="header">
            <div class="container">
                <div class="col-md-12">
                    <h1>blogMV*</h1>
                </div>
            </div>
        </header>

        <div class="jumbotron">
            <div class="container">
                <h1>Backbone + Require</h1>
                <p>This is an example of the Single Page App built with Backbone and RequireJS.</p>
            </div>
        </div>

        <section class="container main">
            <div class="row">
                <div class="col-md-3">
                    <h2>JS News</h2>
                    <ul class="list-group">
                     <script type="text/html" id="tmp-article-list">
                            <% collection.each(function(model){ %>
                                <li>
                                    <a href="article/<%= model.id %>" class="list-group-item">
                                        <%= model.getTitle() %>
                                    </a>
                                </li>
                            <% }); %>
                        </script>
                    </ul>
                </div>

                <section class="col-md-9">
                    <article>
                        <script type="text/html" id="tmp-article-content">
                            <h2><%= activeModel.getTitle() %></h2>
                            <p><%= activeModel.getContent() %></p>
                        </script>
                    </article>

                    <h2>Discussion</h2>

                    <ul class="discussion">
                        <script type="text/html" id="tmp-article-comments">
                            <% collection.each(function(model){ %>
                                <li>
                                    <b><%= model.getAuthorName() %></b>
                                    <p><%= model.getContent() %></p>
                                </li>
                            <% }); %>
                        </script>
                    </ul>
                </section><!--col-md-9-->
            </div>
        </section><!-- container -->

        <section class="row comments">
            <div class="col-md-12">
                <h3>Write a comment</h3>

                <form>
                    <span class="col-md-6">
                        <input type="text" id="name" class="form-control input-lg" placeholder="Name">
                    </span>

                    <span class="col-md-6">
                        <input type="email" id="email" class="form-control input-lg" placeholder="Email">
                    </span>

                    <span class="col-md-12">
                        <textarea id="msg" class="form-control input-lg" placeholder="Comment"></textarea>
                        <button class="btn btn-lg btn-alert btn-block" type="submit">Post comment</button>
                    </span>
                </form>
            </div>
        </section>

        <footer id="footer">
            <p>
                Made with ❤ by a bunch of geeks from the island of magic |
                <a href="https://twitter.com/zigolis">@zigolis</a>
                <a href="https://twitter.com/fjorgemota">@fjorgemota</a>
                <a href="https://twitter.com/chiefgui">@chiefgui</a>
                <a href="https://twitter.com/rcarvalhojs">@rcarvalhojs</a>
            </p>
        </footer>

        <script type="text/javascript" src="assets/js/bundle.js"></script>
    </body>
</html>




 

~~~~~~

##Finalizando essa farra toda!

Beleza, já temos nossa aplicação legal, as view definidas, nossas collection bem legais também, mas e o Browserify?
Analisando bem nossa App, temos um arquivo que pode ser considerado o "pai" de todos, quero dizer que, é teoricamente o arquivo que começa tudo. Esse arquivo é o Mains.Js, é ele que inicia o router que posteriormente inicia toda a aplicação. Bora gera então um bundle disso tudo?

Reparem que no final do arquivo index.html, existe uma tag script apontando para um arquivo chamado bundle.js. Esse arquivo foi gerado pelo browserify, e contem exatamente *TUDO* que nossa aplicação precisa para funcionar. Esse bundle vai conter todas dependecias que for especificada dentro de cada módulo. Para gerar o bundle, use o comando

> browserify main -o bundle.js


Onde: 

- Main É o nome do módulo principal da aplicação, ou seja, onde tudo começa. 
- '-O' É a sintaxe que diz para o browserify gerar um output do modulo especificado.
- bundle.js é o nome do arquivo que você deseja que seja gerado. Este pode ser qualquer nome.



Testem ai, que se tudo estiver certo, o resultado deve ser este.


<img src="/assets/images/final.png"/>

Daqui a pouco estarei postando a parte 2 deste post.

Fiquem na paz!


































