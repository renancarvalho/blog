---
layout: post
title:  "5 Dicas para evitar cabelos brancos ao usar Backbone"
date:   2014-06-04 15:06:00
categories: Dicas de Backbone
---

Se você acha que está ganhando mais cabelos brancos depois que começou um projeto usando Backbone? Este post é para você.

A idéia aqui é mostrar alguns tipos de códigos que se você evitar em suas aplicações, seu código certamente trará menos problemas, e será mais fácil de manter.

#### 1 - Evite o máximo possível fazer requisições ajax que não sejam através do Model. ####

O que eu quis dizer com isso?

![Ajax](http://i.imgur.com/B4IVNL6.png)

Primeiro de tudo é necessário entender, que o Backbone te facilita muito nesse tipo de tarefa e que realmente não é necessário todo esse código para fazer uma requisição.O Backbone trabalha com o modelo RESTfull para requisições HTTP, e isso implica que:

- Para salvar algo no server, o verbo correto é o `POST`
- Para retornar algo do server, o ver correto é o `GET`
- Para alterar algo no server, o verbo correto é o `PUT`
- Para deletar algo do server, o verbo correto é o `DELETE`   

Dentro do conceito estrutural do Backbone, esse tipo de tarefa deve ser de responsabilidade exclusiva do seu Model. Então para fazer requisições, o ideal é que seus metodos sejam parecidos com estes.

![model.fetch()](http://i.imgur.com/m3ypX5r.png)


Muito menos linha de código com o mesmo resultado.


Use:
	
- `model.save()`, para `post`.
- `model.fetch()`, para `get`.
- `model.destroy()`, para delete.
- `model.save()`, para `put`.(Se o Model estiver com o atributo id preenchido, o Backbone automaticamente entende como uma requisição de alteração, por isso `put`.


#### 2 - Sua View não pode ser dependente de callback ####

É mais elegante para sua View, reagir de acordo com um evento disparado pelo Model. E também é mais elegante para o seu Model, não ter funções de callbacks, quando não necessário.

Ruim :(

![Render ruim](http://i.imgur.com/ozoPYIq.png)


Melhor assim.

![escutando eventos](http://i.imgur.com/Nlym7yZ.png)




> Para mais informações sobre eventos disparados pelo Model, [clique aqui.][urlBackboneEvent]

Não importa quando ou como o Model fará a requisição, de qualquer forma suas functions serão chamadas.
Ficou menos complexo e mais organizado e alguém discorda?


#### 3 - Views não controlam dados ####

As regras de domínio de sua aplicação(client-side), é de responsabilidade do Model e não da View, se você se deparou escrevendo um código em que sua View manipula algum dado, pare, respire fundo e pense.

 

{%highlight coffee%}
ola:->
{%endhighlight%}


Views devem simplesmente refletir e reagir de acordo com o que está no Model.



#### 4 - Isole sua interface ao integrar com API's terceiras ####

Se sua aplicação precisa integrar com aplicações terceiras, utilize a function `parse` do Backbone para isolar sua interface de qualquer mudança server-side.

exemplo:

Imagine que as informações de um cliente venham de uma API que não seja implementada por voçê ou pelo seu time. E suponhamos que seu Model tenha os atributos: Nome,Idade,Sexo,Estado. E chegue esse Json no seu Model.


{%highlight coffee%}

{Nome:'Renan Carvalho', Idade: 25, Sexo: 'Masculino', Estado:'Rio de Janeiro'}

{%endhighlight%}

Ao fazer o bind do Model com a View, seu template deverá esperar pelos mesmo atributos do Model, certo? certíssimo.

O que aconteceria se a APi mudasse, e retornasse o Json da seguinte forma?


{%highlight coffee%}

{nmCliente:'Renan Carvalho', dtNascimento:'25/10/1988', Sexo:'Masculino', UfCliente:'RJ'}

{%endhighlight%}

Seu Model não vai conseguir fazer o bind dos atributos default com os atributos retornados pelo server, pois são diferentes(Nome != nmCliente).E Você certamente teria que mudar seu arquivo de template para que seua View continue funcionando.

Para resolver esse problema, não é necessário mudar os atributos default do seu Model e nem nada na View, apenas implemente o método `parse` no seu Model e faça com que os campos retornados pelo server, sejam atualizados no seu Model de acordo com sua nomenclatura.



{%highlight coffee%}



{%endhighlight%}


Bom isso hein...



#### 5 - View não sabem e nem devem saber de nada. ####


Evite que sua View faça validações, ela não sabe de nada...

Código ruim:

{%highlight coffee%}



{%endhighlight%}

 
Código bom:


{%highlight coffee%}



{%endhighlight%}



#### Conclusão ####


Todas as dicas que mencionei acima (exceto a número 4), nada mais é do que a própria convenção do Backbone. Se você realmente se preocupa com a qualidade do seu código, vale apenas estudar mais afundo o funcionamento do framework e algumas convenções, isso vale pra qualquer outro framework. Geralmente alguns dessas implementações 'ruims' são porque o desenvolvedor não conhece, o funcionamento do framework e resolve o problema usando Jquery, ou alguma linha de raciocínio divergente com a ideia proposta pela arquitetura MVC, ou MV(alguma coisa).   

Seguindo esses passos, menos cabelos brancos nós temos.

[urlBackboneEvent]: http://backbonejs.org/#Events