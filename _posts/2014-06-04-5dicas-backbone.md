---
layout: post
title:  "5 DICAS PARA EVITAR CABELOS BRANCOS AO USAR BACKBONE"
date:   2014-06-04 15:06:00
categories: Dicas de Backbone
---

Você acha que está ganhando mais cabelos brancos depois que começou um projeto usando Backbone? Se sim, este post é para você.

A ideia aqui é mostrar algumas implementações que se você evitar, seu código certamente trará menos problemas e será mais fácil de manter.

#### 1 - Evite o máximo possível fazer requisições Ajax usando Jquery. ####

O que eu quis dizer com isso?

{%highlight coffee %}
	#Dentro do Model.
ObterCliente:->
	$.ajax '/UrlDaApi'
		type: 'GET'
		dataType: 'html'
		success:->
			#fazer alguma coisa.
		error:->
			#fazer outra coisa.	
{%endhighlight%}


Primeiro de tudo, é importante saber que o Backbone te facilita muito nesse tipo de tarefa e que realmente não é necessário todo esse código para fazer uma requisição. O Backbone trabalha com o modelo RESTfull para requisições HTTP, e isso implica:

- Para salvar dados no server, o verbo correto é o `POST`.
- Para retornar dados do server, o ver correto é o `GET`.
- Para alterar dados no server, o verbo correto é o `PUT`.
- Para deletar dados do server, o verbo correto é o `DELETE`.   

Dentro do conceito estrutural do Backbone esse tipo de tarefa deve ser de responsabilidade exclusiva do seu Model. Então para fazer requisições, o ideal é que seus métodos sejam parecidos com este.

{%highlight coffee %}
	#Dentro do Model.
url: '/UrlDaApi'
ObterCliente:->
	@fetch(
		success:->
			#fazer alguma coisa
		error:->
			#fazer outra coisa
	)	
{%endhighlight%}


Menos códigos com o mesmo resultado.

Use então:
	
- `model.save()`, para `post`.
- `model.fetch()`, para `get`.
- `model.destroy()`, para `delete`.
- `model.save()`, para `put`. (Se o Model estiver com o atributo `id` preenchido, o Backbone automaticamente entende como uma requisição de alteração, por isso `put`).


#### 2 - Sua View não pode ser dependente de callbacks. ####

É mais elegante para sua View reagir de acordo com um evento disparado pelo Model, e também mais elegante para o seu Model, não ter functions de callbacks quando não necessário.

Implementação ruim :(

{%highlight coffee %}
	#Dentro da View.
initialize:->
	model = new ModelCliente()
	model.fetch(
			success:->
				@render()
			error:->
				@renderMensagemDeError()
		)
render:->
	#fazer alguma coisa.
renderMensagemDeError:->
	#fazer outra coisa.
{%endhighlight%}


Refatorando:

{%highlight coffee %}
	#Dentro da view
initialize:->
	model = new ModelCliente()
	model.on 'sync',@render,@
	model.on 'error',@renderMensagemDeError,@
	model.ObterCliente()
render:->
	#fazer alguma coisa.
renderMensagemDeError:->
	#fazer outra coisa.
{%endhighlight%}
{%highlight coffee %}
	#Dentro do Model
ObterCliente:->
	@fetch()
{%endhighlight%}

> Para mais informações sobre eventos disparados pelo Model, [clique aqui.][urlBackboneEvent]

Não importa quando ou como o Model fará a requisição, de qualquer forma suas functions serão chamadas.
Ficou menos complexo e mais organizado. Concorda?


#### 3 - Evite deixar suas Views "vagando por aí". ####


Provavelmente você usa alguns eventos em sua aplicação. Partindo do principio que o Backbone é um framework orientado a eventos, não vejo problema nenhum em usá-los. Desse modo, existem varias formas de fazermos binds de eventos em nossa view.

Binding de eventos em Views:
{%highlight coffee %}
	# Bind de eventos de interação com usuário.
events:->
	"click #salvar":"SalvarCliente"
	"change input": "AtualizarModel"
{%endhighlight%}

{%highlight coffee %}
	#Dentro da view.
	#Bind de eventos da view com o model.
initialize:->
	@model = new ModelCliente()
	@model.on "change", @exibirMensagem, @
	@model.on "sync", @render , @
{%endhighlight%}

Eventos não são um problema, desde que você os gerencie da forma correta.

O que eu quero dizer? Imagine que seu arquivo Router.coffee seja parecido com o abaixo:

{%highlight coffee %}
routes:
	"ViewA":"renderViewA"
	"ViewB":"renderViewB"

renderViewA:->
	viewA = new ViewA(el: $("#app"))
	viewA.render()

renderViewB:->
	viewB = new ViewB(el: $("#app"))
	viewB.render()
{%endhighlight%}

Ao 'mudar' da ViewA para a ViewB, um novo conteúdo provavelmente deverá ser renderizado, talvez visualmente você não perceba o problema, mas certamente se você fez algum bind de eventos na ViewA, e depois renderizou a ViewB, esses eventos ainda estarão 'ligados'.

Uma forma de resolver esse problema é, sempre que uma alguma View precisar ser renderizada, é importante tentar 'matar' alguma View que foi renderizada anteriormente.

{%highlight coffee %}
routes:
	"ViewA":"renderViewA"
	"ViewB":"renderViewB"

renderViewA:->
	@viewA = new ViewA(el: $("#app"))
	@renderizarView(@viewA)

renderViewB:->
	@viewB = new ViewB(el: $("#app"))
	@renderizarView(@viewB)

renderizarView:(view)->
	if @viewAtual
		@viewAtual.undelegateEvents()
		@viewAtual.remove()	
	@viewAtual = view
	@viewAtual.render()
{%endhighlight%}



#### 4 - Isole sua interface ao integrar com API's terceiras. ####

Caso seja necessário integrar com aplicações terceiras, utilize a function `parse` do Backbone para isolar sua interface de qualquer mudança server-side. Exemplo:

Imagine que as informações de um cliente venham de uma API que não seja implementada por você ou pelo seu time, e suponhamos que seu Model tenha os atributos: Nome, Idade, Sexo, Estado.

Json recebido da API:

{%highlight coffee%}
{Nome:'Renan Carvalho', Idade: 25, Sexo: 'Masculino', Estado:'Rio de Janeiro'}
{%endhighlight%}

Ao fazer o bind do Model com a View, seu template (html) deverá esperar pelos mesmos atributos do Model. Tudo provavelmente funcionará, certo? Certíssimo.

Mas o que aconteceria se a API sofresse uma pequena alteração e retornasse o Json da seguinte forma:


{%highlight coffee%}
{nmCliente:'Renan Carvalho', dtNascimento:'25/10/1988', Sexo:'Masculino', UfCliente:'RJ'}
{%endhighlight%}

Seu Model não vai conseguir fazer o bind dos atributos default com os atributos retornados pelo server, pois são diferentes (Nome != nmCliente). Então o model criará esses 'novos' atributos dentro de si, e você certamente teria que mudar seu arquivo de template (html) para que sua View continuasse funcionando.

Para resolver esse problema, não é necessário mudar os atributos default do seu Model e nem nada na View. Implemente o método `parse` no seu Model e faça com que os campos retornados pelo server sejam atualizados no seu Model de acordo com sua nomenclatura.

Implementando o `parse` no Model:

{%highlight coffee%}
defaults:
	'NomeCliente':''
    'Idade':''
    'Sexo':''
    'Estado':''
parse:(ObjetoRetornadoPeloServer)->
	@.set "NomeCliente", ObjetoRetornadoPeloServer.nmCliente
    @.set "Idade", @CalcularIdade(ObjetoRetornadoPeloServer.dtNascimento)
    @.set "Sexo", ObjetoRetornadoPeloServer.Sexo
    @.set "Estado", ObjetoRetornadoPeloServer.UfCliente
ObterCliente:->
	@fetch()
CalcularIdade:(DataDeNascimento)->
	#calcularIdade
{%endhighlight%}

> `parse` sempre será chamado quando algum dado retornar do server em requisições usando `fetch()` e `save()`. 

Útil isso hein...

#### 5 - Views não sabem e nem devem saber de nada. ####

Evite que sua View faça validações. Ela não deve ter esse tipo de comportamento.

View de cliente:
{%highlight coffee%}
salvar:->
	idade = $("#idade")
	if idade > 18
		@.model.salvarCliente()
{%endhighlight%}

Existe uma forma de simplificar essa tarefa usando um plugin bem legal que é o [Backbone Validation][validation], vale apena uma lida rápida, é bem simples.

Então para criarmos validações, basta implementarmos a function `validate` dentro do nosso Model.

Refatorando a View:
{%highlight coffee%}
#adicionar o atributo 'Idade' no model usando model.set.
salvar:->
	@model.salvarCliente()
{%endhighlight%}

Implementando validação no Model:
{%highlight coffee%}
validation:
	Idade:
		required: true
		min: 18
		msg: 'O Cliente precisa ter mais que 18 anos.'
salvarCliente:->	
	forcarAValidacaoDoModel = true
	if @isValid(forcarAValidacaoDoModel)
		@save()
{%endhighlight%}

<<<<<<< HEAD
> Ao passar True como parâmetro na function 'isValid()', Backbone irá performar todas as suas validações escritas dentro da function 'validation'.
=======
> Ao passar True como parâmetro na function 'isValid()', o Backbone performa todas as suas validações escritas dentro da function 'validation'.
>>>>>>> efa754fd3c9adcfc76a24c33038f2a24a8a40831
Esta é uma forma de garantir que seu model não faça um `post` em um estado inválido.




#### Conclusão ####


<<<<<<< HEAD
Três das cinco dicas citadas (1, 2 e 5), nada mais são do que a própria convenção usada no Backbone. Se você realmente se preocupa com a qualidade do seu código, vale a pena estudar mais afundo o funcionamento do framework e algumas convenções, o mesmo pode ajudar em qualquer outro framework. Geralmente algumas dessas implementações 'ruins' com Backbone são causadas pelo fato do desenvolvedor não conhecer o funcionamento do framework, e optar por resolver o problema usando Jquery/Javascript da forma 'tradicional', ou alguma linha de raciocínio divergente com a ideia proposta pela arquitetura MVC.   
=======
Três das cinco dicas citadas (1, 2 e 5), nada mais são do que a própria convenção usada no Backbone. Se você realmente se preocupa com a qualidade do seu código, vale a pena estudar mais afundo o funcionamento do framework e algumas convenções, o mesmo pode ajudar em qualquer outro framework. Geralmente algumas dessas implementações 'ruims' com Backbone são causadas pelo fato do desenvolvedor não conhecer o funcionamento do framework, e optar por resolver o problema usando Jquery/Javascript da forma 'tradicional', ou alguma linha de raciocínio divergente com a ideia proposta pela arquitetura MVC.   
>>>>>>> efa754fd3c9adcfc76a24c33038f2a24a8a40831

Seguindo esses passos, menos cabelos brancos...

[urlBackboneEvent]: http://backbonejs.org/#Events 
[validation]: http://thedersen.com/projects/backbone-validation