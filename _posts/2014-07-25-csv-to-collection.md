---
layout: post
title: "VOCÊ ACHA CHATO IMPORTAR ARQUIVO CSV? VOU TENTAR TE AJUDAR"
date:   2014-07-25 18:06:00
categories: Javascript
---

Ultimamente tenho feito muitas tarefas de importação de arquivos csv. Trabalho em uma corretora de investimentos, e por lá esse tipo de tarefa é bem comum.


Sempre achei chato e estranho ter que enviar um arquivo csv para o servidor (API), ainda mais quando não é necessário salvar o arquivo. Fazer isso com <a href="http://blueimp.github.io/jQuery-File-Upload/" target="_blank">Jquery</a> até da, mas particularmente acho custoso, meio bagunçado e difícil de testar.


###Um Plugin bem simples.

Criei então um plugin bem simples que facilita um pouco a vida, basicamente o arquivo csv importado é transformado em uma collection, simples assim!

Dessa forma, é mais tranquilo fazer as requisições para o servidor (API), afinal o conteúdo do arquivo agora é uma collection.

Ainda está bem básico. Existem diversas coisas a serem implementadas para que o Plugin fique mais consistente, e qualquer ajuda é sempre bem vinda.


### Como usar?

Você vai precisar de:

- Criar um input do tipo file na view.
- Definir o Plugin como dependência da view.
- Configurar o evento de change do input.
- Passar o conteúdo do input para o plugin.
- Escutar o evento de Load que sera invocado retornando a collection com o conteúdo do csv.


###Exemplo prático.

O Plugin funciona com AMD ou UMD, então basicamente o que você precisa fazer é carregá-lo de acordo com seu ambiente.

#####Require.Js:

    define([
    'Backbone',
	'libs/csvToCollection.min'   
    ], function(Backbone){
      var SimpleView = Backbone.View.extend({
    
       initialize:function(options) {
    		el = options.el;		
    		this.render();
    	},
    
    	events:
    	{
    		"change #file":"importToCollection"
    	},
    
    	importToCollection: function(e){
    		document.addEventListener("Load", function(collection) {			
    			console.log(collection.detail);
    		});
    		Backbone.csvToCollection(e);
    	},
    
    	render: function() {
    		// Criando o input
    		return $(this.el).append("<input type='file' id='file'>");		
    	}
    
      });  
      return SimpleView;  
    });

#####Common.Js (Browserify):
    
    Backbone = require('backbone')
    CsvToCollection = require('./libs/csvToCollection.js')
    
    module.exports = Backbone.View.extend({	
    
    	initialize:function(options) {
    		el = options.el;
    		this.render();
    	},
    
    	events:
    	{
    		"change #file":"importToCollection"
    	},
    
    	importToCollection: function(e){
    		document.addEventListener("Load", function(collection) {			
    			console.log(collection.detail);
    		});
    		Backbone.csvToCollection(e);
    	},
    
    	render: function() {
    		return $(this.el).append("<input type='file' id='file'>");		
    	}
    
    });
    
    
Tem funcionado bem pro meu cenário e acredito que pode ajudar bastante gente. 

    
###E agora?
Pretendo incrementar bastante o Plugin e qualquer ajuda é bem vinda.

Vou implementar algumas coisas do tipo:

- Implementar testes. (Sim eu errei muitoo em não fazer TDD ). 
- Automaticamente rodar a validação do model, vai facilitar bastante.
- Passar a instância da sua collection.


O código está disponível no meu <a href="https://github.com/renancarvalho/CsvToCollection" target="_blank">GitHub</a>.

- IE : 10+
- Chrome : 21+
- Firefox : 21+
- Opera : 12.1+ 


Espero que seja útil, Abraço!