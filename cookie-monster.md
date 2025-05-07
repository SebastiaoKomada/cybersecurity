# [Cookie Monster Secret Recipe]
###### Solved by @[SebastiaoKomada]
> This is a CTF about Cookies

## About the Challenge
Cookie Monster escondeu sua receita secreta de cookies em algum lugar do seu site. Como um aspirante a detetive de cookies, sua missão é descobrir esse delicioso segredo. Será que você consegue enganar o Cookie Monster e encontrar a receita escondida?

## Solution

### Introdução
Ao acessar o link, nos deparamos com dois campos de entrada: `username` e `admin`. Inserindo valores como `admin` para ambos os campos, a tela exibe a seguinte mensagem:

Access Denied
Cookie Monster says: 'Me no need password. Me just need cookies!'
Hint: Have you checked your cookies lately?

### Análise
A mensagem sugere que a autenticação não depende de senha, mas sim dos cookies. Inspecionando os cookies armazenados no navegador, encontramos a seguinte chave: secret_recipe: cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sMHZlc19jMDBraWVzX0U2MzRERkJCfQ%3D%3D

### Decodificação
O valor do cookie esta codificado em Base64. Utilizando a ferramenta [dcode.fr](https://www.dcode.fr/base-64-encoding), decodificamos o valor e obtivemos a flag:
>`[picoCTF{c00k1e_m0nster_l0ves_c00kies_E634DFBB]`

