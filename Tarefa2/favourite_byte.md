# [Favourite byte]
###### Resolvido por @[SebastiaoKomada]
> Desafio CryptoHack sobre XOR

## Sobre o Desafio
Neste desafio, o objetivo é utilizar a operação XOR com uma chave desconhecida para decifrar uma mensagem oculta. A chave utilizada no XOR é um byte único e o texto criptografado foi fornecido em formato hexadecimal. O desafio é encontrar essa chave e reverter a operação XOR para obter a flag. 

### Os Dados do Desafio
Temos o texto criptografado, que foi gerado utilizando um XOR com um byte desconhecido. O texto criptografado está representado em hexadecimal da seguinte maneira:

- **Texto Cifrado:** `73626960647f6b206821204f21254f7d694f7624662065622127234f726927756d`

O objetivo é descobrir a chave XOR e utilizar essa chave para descriptografar o texto, revelando a flag.

Você pode acessar o desafio [aqui](https://cryptohack.org/courses/intro/xorkey0/).

## Solução

### Analisando o XOR  
A operação XOR é uma operação bit a bit que resulta em `1` quando os bits são diferentes, e `0` quando são iguais. Quando usamos XOR com uma chave, podemos reverter a operação aplicando o XOR novamente com a mesma chave.

### Passos para Resolver o Desafio

#### 1. Conversão de Hexadecimal para Bytes
O primeiro passo é converter o texto criptografado de hexadecimal para bytes. Isso é necessário porque o XOR é uma operação realizada diretamente nos valores binários.

#### 2. Tentando todas as Possíveis Chaves
Sabemos que a chave utilizada no XOR é um único byte, ou seja, pode ter qualquer valor de `0` a `255` (representados como números inteiros). Precisamos tentar todas essas possibilidades até encontrar a chave correta, que ao ser aplicada ao texto cifrado, nos dê uma string que começa com a palavra "crypto".

#### 3. Código em Python  
Aqui está o código em Python que tenta todas as possíveis chaves XOR para encontrar a flag:

```python
from pwn import *

texto_cifrado = bytes.fromhex("73626960647f6b206821204f21254f7d694f7624662065622127234f726927756d")

def encontrar_chave_xor(texto_cifrado):
    ordinais_texto = [o for o in texto_cifrado]
    for chave in range(256):
        possivel_flag_ordinais = [chave ^ o for o in ordinais_texto]
        possivel_flag = "".join(chr(o) for o in possivel_flag_ordinais)
        if possivel_flag.startswith("crypto"): 
            return possivel_flag

print(encontrar_chave_xor(texto_cifrado))
```
### Explicação
Neste desafio, usamos a operação XOR de forma simples, tentando todas as possibilidades de chave (de `0` a `255`) e verificando qual delas resulta em um texto que começa com a palavra "crypto". Ao encontrar a chave correta, conseguimos descriptografar o texto e obter a flag.

### Resultado  
Ao rodar o código acima, você obterá o seguinte resultado:

>`crypto{0x10_15_my_f4v0ur173_by7e}`
