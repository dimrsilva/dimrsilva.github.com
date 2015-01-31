---
layout: post
title: "Aplicando widgets em elementos após o carregamento da página"
date: 2013-07-21 22:48
comments: true
categories: [JavaScript, jQuery, Ajax, DRY]
keywords: JavaScript, jQuery, Ajax, DRY, Widgets, Plugins
description: Implementação DRY de carregamendo de widgets jQuery em elementos HTML após o carregamento da página
---

Quando estamos fazendo alguma página com o jQuery, é comum a utilização de vários widgets.
Um exemplo são os widgets disponíveis no jQuery UI.

Normalmente esses widgets são aplicados a determinados seletores:

``` javascript
$(window).load(function() {
    $('input.datepicker').datepicker();
});
```

Um problema nesses casos, é quando precisamos adicionar dinamicamente conteúdos às páginas, como no exemplo:

``` javascript
$.ajax({
    url: 'exemplo.html',
    success: function(new_content) {
        $('#content').replaceWith(new_content);
    }
});
```

Nesse caso, para carregar um possível datepicker no novo conteúdo, deveria ser chamado novamente o trecho `$('input.datepicker').datepicker();`.

Um problema de chamar explicitamente, é que o código deverá ser repetido em diversos locais, aumentando o custo de manutenção e comprometendo o conceito de [DRY](http://pt.wikipedia.org/wiki/Don't_repeat_yourself "Don't repeat yourself").

Outro problema é que para alguns widgets, se houver chamada duplicada sobre um mesmo elemento, pode causar efeitos duplicados. É importante que o jQuery reconheça apenas os novos elementos adicionados a página.

##Utilizando behaviors

Para solucionar isso, podemos criar funções de comportamentos.
Elas são responsáveis por carregar determinado comportamento em todo novo conteúdo adicionado a página.
Essa solução foi inspirada nos [behaviors](http://dojotoolkit.org/reference-guide/1.9/dojo/behavior.html "dojo.behavior") do [Dojo](http://dojotoolkit.org/ "Dojo Toolkit").

``` javascript
var Example = {}
Example.behaviors = []
Example.load = function(context) {
    for(var i in this.behaviors) {
        this.behaviors[i](context);
    }
}

```

Depois disso, para registrarmos os comportamentos, fazemos:

``` javascript
Example.behaviors.push(function(context) {
    // É importante observar a utilização da variável context
    // Ele limita a busca do jQuery para apenas o novo conteúdo que foi carregado
    $('input.datepicker', context).datepicker();
    $('a.ajaxanchor', context).click(function(event) {
        event.preventDefault();
        $.ajax({
            url: $(this).attr('href')
        });
    });
});

```

Então, para adicionar os comportamentos aos elementos da página, é só fazer a chamada ao `Example.load`:

``` javascript
//Ao iniciar a página
$(window).load(function() {
    Example.load(document); //Carrega todo o documento
});

//Após carregar novo conteúdo
$.ajax({
    url: 'exemplo.html',
    success: function(new_content) {
        $('#content').replaceWith(new_content);
        Example.load(new_content);
    }
});
```

##Conclusão

Utilizando as funções descritas no exemplo, é possível implementar comportamentos padrões em diversos elementos repetidos da página, mesmo quando forem carregados após o carregamento total da página. Além disso, a utilização do `context` para limitar a busca de elementos pelo jQuery aumenta a performance da página.

PS.: Nenhum desses códigos foram testados, espero ainda dar um upgrade nesse post.