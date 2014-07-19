---
layout: post
title: "GERENCIE OS ESTADOS QUE SUA APLICAÇÃO PODE TER"
date:   2014-06-24 21:06:00
categories: Dicas de Backbone
---

Gerenciar como sua aplicação deve se comportar pode ser uma tarefa fácil ou difícil. Implementar bem suas exceções de negócio é um ponto interessante que qualquer desenvolvedor deve se preocupar.

Atualmente trabalho em um projeto interessante com algum as regras de negócio bem particulares, utilizando Backbone na parte client e o conceito de RESTful na comunicação com o servidor. Basicamente a tela tem um `input` simples, em que o usuário logado na aplicação pode fazer uma busca especifica por CPF ou código do cliente. A aplicação submete a informação digitada para o servidor, e lá é verificado se o usuário que esta logado tem privilégios para fazer aquele tipo de busca, se sim devem ser exibidas as informações daquele CPF ou código especifico.

Entendendo melhor:

- Caso o usuário não tenha privilégios para fazer o tipo de busca, uma mensagem deve ser exibida informando que o usuário não tem permissão suficiente para ver aquele cliente.
- Caso o usuário tenha privilégios, mas o CPF/Cliente não existe na base, uma mensagem deverá exibida informando que não foi encontrado nenhum cliente.
- Caso o usuário tenha privilégios e o CPF/Cliente exista, as informações do cliente serão exibidas.

Realmente, neste cenário, é importante mostrar ao usuário o que ocorreu durante a pesquisa, não era válido exibir uma mensagem genérica para ambos os tipos de exceções.  

####Facilitando a  vida com REST####

Trabalhar com REST facilitou na implementação desta funcionalidade. O primeiro passo foi alinhar os `status code` que o servidor poderia retornar, uma rápida pesquisa na internet foi o suficiente para entender melhor o que poderia ser feito com os `status code` disponíveis.

Os `status code` definidos foram:

- Quando o usuário tem privilégio e o CPF/Cliente existe na base, o `status code 200 (OK) ` deve ser retornado junto com as informações do cliente.
- Quando o usuário não tem privilégios para o tipo de operação, o `status code 401 (Unauthorized)` deve ser retornado.
- Quando o cliente CPF/Cliente não existe na base, o `status code 204 (No Content)` deve ser retornado   
 
Utilizar o `status code` correto é a realmente importante, pois é ele que dirá a sua aplicação client-side o que aconteceu no servidor, sem contar que seu código fica bem mais organizado e padronizado.

Explicando rapidamente os `status code` utilizados nesse exemplo.

- `status code OK` (200). Tudo ocorreu como esperado no servidor e alguma informação foi retornada. 
- `status code No Content` (204). O servidor recebeu a requisição, fez todo o processamento codificado na aplicação, tudo deu certo, mas não retornou nenhum dado. 
- `status code Unauthorized` (401). O recurso necessita de algum tipo de autenticação para ser acessado.

> Unauthorized e Forbiden são bem parecidos, a diferença é que o Forbiden não leva em consideração qualquer tipo de autenticação, enquanto para o Unauthorized a autenticação do usuário deve ser considerada.

> Para requisições do tipo post, é interessante utilizar o `status code` created (201) ao invés de OK (200).

#### Backbone e REST ####

Definido nossos `status code`, vamos analisar como codificá-los usando Backbone. No Model, quando fizermos a requisição para o servidor, seu código provavelmente será parecido com este:

{%highlight coffee %}
	#Dentro do Model.
BuscarCliente:->
  @resposta = @fetch(
			    success:()=>
	   			  @trigger "renderizaUsuarioNaoEncontrado" if @requisicao.status is 204
	              @trigger "renderizarInformacoes" if @requisicao.status is 200
				error:()=>
	   			  @trigger "usuarioSemPermissao" if @requisicao.status is 401
)	
{%endhighlight%}

Nossa regra de negócio não permite termos uma mensagem genérica, logo é necessário ter um tratamento especifico para cada `status code` retornado pelo servidor. Como não faz sentido e é completamente errado renderizar algum tipo de mensagem dentro do Model, disparamos então uma trigger indicando o que aconteceu com a requisição para a View, e ela que se vire. Lembrando que quando o Backbone realiza o `fetch()`, um objeto com a requisição é retornado. Isso também poderia ser feito com os parâmetros recebidos nos callbacks, fica a critério do desenvolvedor. 


{%highlight coffee %}
	#Dentro da View.
	escutarEventos:->
		@model.on 'renderizaUsuarioNaoEncontrado',@exibirMsgUsuarioNaoEncontrado,@
		@model.on 'renderizarInformacoes',@renderizarInformacoesDoUsuario,@
		@model.on 'usuarioSemPermissao',@exibirMsgUsuarioSemPermissao,@
	eibirMsgUsuarioNaoEncontrado:->
		#exibir alguma mensagem para o usuário.
	renderizarInformacoesDoUsuario:->
		#renderizar as informações do usuário.
	exibirMsgUsuarioSemPermissao:->
		#exibir alguma mensagem para o usuário.
	
{%endhighlight%} 

Pronto, desta forma conseguimos ter um comportamento específico para cada reação informada pelo servidor e isso para o usuário é importante.

O código acima foi a minha primeira implementação e funcionou bem, mas confesso que achei estranho ter que disparar 3 triggers no Model e escutá-los na View. Também me deu um pouco mais de trabalho em meus testes com Jasmine, então decidi refatorar:

{%highlight coffee %}
	#Dentro do Model.
BuscarCliente:->
  @resposta = @fetch()
			    success:()=>
				  @trigger("sucesso",@resposta.status)  
				error:()=>
	   			  @trigger("erro",@resposta.status)
)	
{%endhighlight%}


{%highlight coffee %}
	#Dentro da View.
	escutarEventos:->
		@model.on 'sucesso',@renderizarTipoFluxoDeSucesso, @
		@model.on 'erro',@renderizarFluxoDeErro, @
	renderizarTipoFluxoDeSucesso:(statusCode)->
		@renderizarInformacoesDoUsuario() if statusCode is 200
		@exibirMsgUsuarioNaoEncontrado() if statusCode is 204
	renderizarFluxoDeErro:(statusCode)->
		@exibirMsgUsuarioSemPermissao() if statusCode is 401
	exibirMsgUsuarioNaoEncontrado:->
		#exibir alguma mensagem para o usuário.
	renderizarInformacoesDoUsuario:->
		#renderizar as informações do usuário.
	exibirMsgUsuarioSemPermissao:->
		#exibir alguma mensagem para o usuário.

{%endhighlight%}


Particularmente não gosto de gerenciar triggers, tento sempre usá-las o mínimo possível. No código acima, utilizo somente 2 triggers independente de quantos `status code` diferentes a aplicação possa ter, e passo para a View a responsabilidade de saber qual conteúdo renderizar com o `status code` retornado pelo servidor. Concordo que o código cresceu um pouco dentro da View, mas em contrapartida se algum tipo de mensagem nova for necessária na aplicação a partir de um `staus code`, não será necessário modificar o Model.

Essa implementação me agrada, pois acho que o código fica mais claro e simples. O mais importante de tudo é que deixamos o usuário saber exatamente o que está acontecendo na aplicação, o que é necessário.

É isso galera. Concordam com a implementação? Sim? Não? Comentem :)





 
