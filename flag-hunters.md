# [Flag Hunters]
###### Solved by @[SebastiaoKomada]
> This is a CTF about Analysis

## About the Challenge
As letras pulam dos versos para o refrão como se fosse uma chamada de sub-rotina. Há um refrão oculto que este programa não imprime por padrão. Você consegue fazê-lo ser impresso? Pode haver algo nele para você.

## Solution

### Introdução
Ao executar o comando no terminal:
nc verbal-sleep.picoctf.net 59898

Uma música começa a tocar, e logo em seguida o programa solicita uma senha. Como não sabemos a senha inicialmente, o refrão continua normalmente.

### Análise
Para entender melhor o funcionamento do desafio, analisamos o código-fonte disponível. Utilizando o comando `cat lyric-reader.py`, acessamos o conteúdo do programa.

Na linha 121, observamos um bloco condicional `if` que verifica o conteúdo inserido. Essa parte do código é responsável por controlar a transição para o refrão oculto. Com base nisso, decidimos testar a entrada `;RETURN 0` como senha, para forçar a execução do trecho escondido.

### Execução
Rodamos novamente o comando:
nc verbal-sleep.picoctf.net 59898

E inserimos a senha: ;RETURN 0

O programa reinicia a música e, em seguida, revela a flag:
>`[picoCTF{70637h3r_f0r3v3r_c659e814}]`
