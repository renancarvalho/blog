---
layout: post
title: "GULP E BROWSERIFY, UMA ALTERNATIVA AO REQUIRE.JS"
date:   2014-08-02 19:15:00
categories: Javascript
---

Acho que uma das frases mais certas que existe no mundo de tecnologia e desenvolvimento de software: "Não existe a melhor solução". Traduzindo pro dito popular, "Não existe bala de prata."

Pode acontecer de um pattern resolver seu problema e não ser a solução para outro. Este post é para quem está um pouco cansado/insatisfeito de trabalhar com RequireJs e está buscando uma alternativa. Eis aqui uma: Gulp + Browserify.

#Gulp#
<img style="width:100px;" src="https://raw2.github.com/gulpjs/artwork/master/gulp-2x.png"/>

Gulp é um task runner (no mesmo estilo que o <a href="http://gruntjs.com/" target="_blank">Grunt</a>) que pode ajudar a automatizar a enorme quantidade de tarefas que você pode ter no seu projeto, tarefas essas do tipo:

- Compilar arquivos Coffeescript para Javascript.
- Compilar arquivos Sass para Css.
- Mover arquivos de um lugar para outro.
- Minify ou Uglify arquivos Javascript ou Css.
- Rodar testes de Javascript.
- E MUIIIIIITO mais.

Não é difícil trabalhar com Gulp, acho que a grande mágica é conhecer os poderosos plugins que já existem. Uma boa "googlada" vai te ajudar bastante, não tem mistério.

A <a href='http://gulpjs.com/' target='_blank'>Documentação oficial do Gulp</a> é bem legal, lá existem vários exemplos de como começar, instalar, configurar e etc.

###RequireJs (AMD) x CommonJS (UMD)###


Para entender melhor a contiuação do post, é interessante entender a diferença entre AMD (Asynchronous Definition Module) e UMD (Universal Definition Module).

Existem vários frameworks facilitando a tarefa de carregar módulos Javascript, e convenhamos é algo importantíssimo. Todos eles praticamente são baseados em duas categorias, AMD e UMD.

#####AMD
A grande proposta de usar AMD é coseguir definir módulos Javascript com  as dependências, e carregá-los de forma assíncrona, ou seja, carregar os módulos Javascript sob demanda.


{%highlight Javascript %}    
    define('MeuModulo', 
        ['DependenciaA', 'DependenciaB'], //dependências desse módulo que podem ser outros módulos.
        function ( depA, depB ) {
            var MeuModulo = {
                AlgumaCoisa:function(){
                    console.log('Fiz alguma coisa');
                }
            }     
            return MeuModulo;
    });     
{%endhighlight%}


O código acima é um exemplo simples de como implementar um módulo Javascritp usando <ahref="http://requirejs.org/" target="_blank">RequireJS</a>, que é um module loader baseado na idéia de AMD.

Prós em usar RequireJs:

- Otimizado para trabalhar com browser.
- Possibilidade de Lazy Load.
- Facilita o gerenciamento de arquivos/módulos Javascript.

Contras em usar RequireJS.

- Pode ser custoso/demorado configurar seu ambiente.
- <strike>Pelo fato de ser assíncrono, não usar Bundle fica complicado.</strike>


#####UMD
Já o UMD (Universal Module Definition) é um estilo de carregar módulos Javascript mais familiar para quem trabalha com ambiente NodeJS (Server side). CommonJs tem um foco maior na parte Server side, como exemplo disso o NodeJs implementa módulos baseado no CommonJS.


{%highlight Javascript %}    
    var moduloQualquer = require('ModuloQualquer');
    function FuncaoQualquer(){
        console.log('qualquer coisa');
    };
    module.exports = FuncaoQualquer;
{%endhighlight%}

Quem usa NodeJS certamente está mais familiarizado com essa sintaxe, que na minha opinião é bem mais simples. Para isto funcionar você obviamente precisa ter o NodeJS instalado.

> para instalar o NodeJS vale apena dar uma olhada no google, é bem simples. No linux é basicamente `sudo apt-get install NodeJS`.

Mas se eu preciso do NodeJS para fazer este tipo de criação de módulo funcionar e sou um dev Front End, como vou fazer? 



#Browserify#
<img style="width:300px!important;" src="http://browserify.org/images/wizard.png"/>

É dificil explicar exatamente o que é o Browserify, mas vou tentar.
Browserify é uma forma de carregar ou criar módulos Javascript usando uma sintaxe no estilo NodeJS. A principal diferença entre Browserify e RequireJs, é que o Browserify não é assíncrono, na verdade ele vai gerar um bundle com todas as suas dependências e módulos.

Mãos a obra.

Vou fazer setup de um ambiente com Backbone usando NodeJS para carregarmos nossos módulos e dependências, Browserify para gerarmos um bundle, e o Gulp pra agilizarmos tudo. \0/

Pra começar, instale o NodeJS e o Npm (Node Package Manager). Dá uma olhada no google que é fácil.

Com o NodeJS e o Npm instalados, monte uma estrutura de arquivos igual a esta:

{%highlight html %} 
Gulp+Browserify
----Js
------Main.js 
------Router.js
------ViewQualquer.js    
Index.html 

{%endhighlight%}

Index.html - Nosso arquivo html simples com a tag main para exibirmos nossa view:

{%highlight html %}    
<!doctype html>
<html>
    <head>
        	<title>Gulp e Browserify</title>
    </head>
    <body>
        	<main id="conteudo"></main>
    </body>
</html>
{%endhighlight%}



Main.js - Digamos que seria nosso Js principal. Nele carregaremos o Backbone.Router, que na verdade é um módulo e o instanciaremos chamando o método `start`:

{%highlight Javascript %}    
Router = require('./router')
router = new Router();
router.start();
{%endhighlight%}

Router.js - Nosso módulo que herda da classe Router do Backbone. Ele tem como dependência uma view ('viewQualquer'):

{%highlight Javascript %}    
Backbone = require('backbone')
ViewQualquer = require('./viewQualquer')
$ = require('jquery')
Backbone.$ = $
module.exports = Backbone.Router.extend({
	routes: {
		"":"home"   
	},
	start:function(){
		Backbone.history.start();
	},
	home: function() {  	
		view = new ViewQualquer({el:$("#conteudo")});
	}
});
{%endhighlight%}

viewQualquer.js - Uma view simples que renderizará um h1 com o texto "Gulp + Browserify !" na tela:

{%highlight Javascript %}  
Backbone = require('backbone')
module.exports = Backbone.View.extend({	
	initialize:function(options) {
		el = options.el;
		this.render();
	},
	render: function() {
		return $(this.el).append("<h1>Gulp + Browserify !</h1>");		
	}
});
{%endhighlight%}


Repare que router.js e viewQualquer.js têm como dependência o Backbone, por isso o `require('backbone'), require('jquery')`, e assim com qualquer dependência.

Para que nossa aplicação consiga carregar as dependêcias externas como Backbone e Jquery, precisamos instalá-las usando o Npm.

`npm install -g backbone jquery`

> `-g` significa que estamos instalando o módulo de forma global.
> Para adicionar os módulos em um arquivo package.json, é só adicionar `--save-dev`.

Já que estamos brincando com o Npm, vamos instalar logo o Browserify.

`npm install -g browserify`

###Agora a mágica acontece###

Com tudo instalado, vamos então usar de fato o Browserify. Acesse o terminal, vá até o diretório do projeto e execute: `browserify main.js -o bundle.js`.

Perceba que um arquivo com o nome `bundle.js` foi criado no seu projeto, e é exatamente isso que o Browserify se compromete em fazer. Ele gerou um bundle com todas as dependências do projeto, tudo em um único arquivo js.
Agora é só adicionar o bundle como referência no index.html e rodar a aplicação. Você deverá ver escrito "Gulp + Browserify !".

Legal né? Mas temos ainda um problema chato, qualquer mudança que fizermos em algum arquivo Javascript, teremos que rodar o comando `Browserify main.js -o bundle.js` novamente para atualizar o bundle. E isso não é legal!

###Lembra do Gulp?####

Eu disse que o Gulp ajuda em tarefas chatas de nossa aplicação, e essa pode ser uma em que ele pode ajudar. Para isso vamos criar dentro do diretório Js um arquivo chamado gulpfile.js.

gulpfile.js - Arquivo onde ficarão nossas tasks configuradas. A idéia é fazer o Gulp executar tarefas automaticamente através deste arquivo. Vou comentar o código pra tentar simplificar:


{%highlight Javascript %}  
//carregar as dependências
var gulp = require('gulp');//Gulp
var browserify = require('gulp-browserify'); // Gulp-Browserify um plugin do browserify com gulp
var rename = require('gulp-rename'); // Gulp-rename para renomear nome de arquivos


//Crio uma task chamada Browserify, esta task pegará o arquivo main.js e fará o bundle a partir dele.
// depois uso o rename para renomear o arquivo de bundle que foi gerado. Neste caso, uso o próprio nome bunlde, e por fim o destino desse novo arquivo.
gulp.task("browserify", function() {
   gulp.src('main.js')     
        .pipe(browserify())
        .pipe(rename('bundle.js'))
        .pipe(gulp.dest('bundle/'))
});

{%endhighlight%}
>Para que as dependências funcionem, você precisará usar 
>novamente o Npm. `npm install -g gulp-browserify gulp-rename`

Agora aquele trabalho que você tinha após cada alteração em um arquivo js de regerar o bundle acabou! O Gulp faz isso automaticamente com essa task criada. Para isto, basta rodar no command line `gulp <task>`, no nosso caso `gulp browserify`.

Mesmo assim ainda não está legal. De qualquer, forma temos que rodar o comando do Gulp toda vez que algo mudar, o ideal é que o Gulp ficasse observando cada mudança e rodasse o comando sozinho, concorda?
Existe um plugin para isso, Gulp-Watch. 

> npm install -g gulp-watch

Voltando ao gulpfile.js.


{%highlight Javascript %} 
var gulp = require('gulp');
	var browserify = require('gulp-browserify');
	var rename = require('gulp-rename');
	var watch = require('gulp-watch');//Adicionando o gulp-watch
	  
	gulp.task("browserify", function() {
	   gulp.src('main.js')     
	        .pipe(browserify())
	        .pipe(rename('bundle.js'))
	        .pipe(gulp.dest('bundle/'))
	});
//Cria uma task chamada watch que ficará observando a mudança dentro da pasta js em qualquer arquivo .js e chamará a task browserify
	gulp.task("watch", function() {    
	    gulp.watch("**/*.js", ["browserify"]);
	});
{%endhighlight%}

Pronto. Agora qualquer mudança que fizermos dentro da pasta Js, o Gulp perceberá isso e rodará a task 'browserify' que irá gerar o bundle automaticamente. 


Tá com preguiça de testar tudo? Botei no <a href="https://github.com/renancarvalho/Gulp-Browserify-Backbone" target="_blank">Github</a> :)

Para funcionar é só dar um clone no projeto e digitar `npm install`.

Conclusão:

Nem sempre existe melhor solução. Usando Browserify você tem apenas uma requisição para o server e usando Uglify esse arquivo fica bem menor, melhorando mais ainda. Por outro lado é impossivel fazer Lazy Load. Acho que é mais facil inicialmente usar NodeJS e Browserify. Mas como disse antes, é śo uma alternativa.

Valeu galera um abraço!









    




