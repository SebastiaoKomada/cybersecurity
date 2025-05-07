# [RED]
###### Solved by @[SebastiaoKomada]
> This is a CTF about Image Analysis

## About the Challenge
**RED, RED, RED, RED**
O desafio consiste em analisar o conteúdo de uma imagem e descobrir a flag oculta.

## Solution

### Introdução
Após baixar o arquivo fornecido, identificamos que se tratava de uma imagem completamente vermelha.

### Análise
O primeiro passo foi utilizar a ferramenta **Aperi Solve** para examinar a imagem em busca de possíveis mensagens ocultas.

Ao submeter a imagem no site, acessamos a seção **Zsteg**, onde encontramos uma string que aparenta estar codificada em Base64:
cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==

Observamos que a string está repetida quatro vezes.

### Execução
Utilizando o site [dcode.fr](https://www.dcode.fr/base-64-encoding), decodificamos apenas a primeira parte da string como teste e obtemos a seguinte flag:
>`[picoCTF{r3d_1s_th3_ult1m4t3_cur3_f0r_54dn355_}]`


