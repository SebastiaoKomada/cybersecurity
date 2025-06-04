# [interencdec]
###### Solved by @[SebastiaoKomada]
> This is a CTF about Encryption

## About the Challenge
O desafio consiste em analisar o conteúdo de um arquivo codificado e descobrir a flag oculta.

## Solution

### Introdução
Após baixar o arquivo e verificar seu conteúdo, encontramos o seguinte código codificado:
YidkM0JxZGtwQlRYdHFhR3g2YUhsZmF6TnFlVGwzWVROclgyZzBOMm8yYXpZNWZRPT0nCg==

### Análise
O primeiro passo foi utilizar a ferramenta [dcode.fr](https://www.dcode.fr/en) para identificar o tipo de codificação. Descobrimos que se tratava de uma string codificada em Base64. Decodificando, obtemos:
b'd3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrX2g0N2o2azY5fQ=='

Note que o prefixo `b'...'` indica que o conteúdo está no formato de bytes. Removendo o prefixo `b` e as aspas simples, realizamos uma nova decodificação em Base64:
wpjvJAM{jhlzhy_k3jy9wa3k_h47j6k69}

A string resultante se assemelha ao formato de uma flag. Consultamos novamente o dcode para identificar o tipo de cifra utilizada, e verificamos que se trata de uma cifra ROT.

### Execução
Ao aplicar a decodificação da cifra ROT na string obtida, chegamos à flag final:
>`[picoCTF{caesar_d3cr9pt3d_a47c6d69}]`

