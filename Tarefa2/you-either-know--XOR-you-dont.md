
# [You either know, XOR you don't]
###### Resolvido por @[SebastiaoKomada]
> Desafio CryptoHack sobre XOR

## Sobre o Desafio
Neste desafio, a tarefa é descobrir uma chave secreta que foi usada para criptografar uma flag com a operação XOR. O texto criptografado foi fornecido em formato hexadecimal e a chave secreta é desconhecida. O objetivo é reverter o processo XOR para descobrir a flag.

### Os Dados do Desafio
O texto criptografado foi gerado utilizando a operação XOR com uma chave secreta. O texto criptografado está em formato hexadecimal, conforme abaixo:

- **Texto Cifrado:** `0e0b213f26041e480b26217f27342e175d0e070a3c5b103e2526217f27342e175d0e077e263451150104`

A tarefa é descobrir a chave utilizada no XOR e utilizá-la para descriptografar o texto e revelar a flag.

Você pode acessar o desafio [aqui](https://cryptohack.org/courses/intro/xorkey1/).

## Solução

### Analisando o XOR  
A operação XOR é uma operação bit a bit, onde o resultado será `1` quando os bits forem diferentes e `0` quando forem iguais. O interessante dessa operação é que, ao aplicar o XOR duas vezes com a mesma chave, podemos reverter o processo.

### Passos para Resolver o Desafio

#### 1. Conversão de Hexadecimal para Bytes
O primeiro passo é converter o texto criptografado de hexadecimal para bytes. O XOR é uma operação feita diretamente em valores binários, por isso precisamos trabalhar com bytes.

#### 2. Tentando Reverter a Operação XOR
Sabemos que a chave usada no XOR é um valor desconhecido, mas podemos tentar encontrar essa chave ao aplicar o XOR com partes conhecidas da flag, como o prefixo `"crypto{"` e o sufixo `"}"`. Isso nos dará pistas para encontrar a chave.

#### 3. Código em Python  
Aqui está o código em Python para resolver o desafio:

```python
from pwn import *

cipher_hex = '0e0b213f26041e480b26217f27342e175d0e070a3c5b103e2526217f27342e175d0e077e263451150104'
cipher_bytes = bytes.fromhex(cipher_hex)

start_flag = bytes('crypto{', 'utf-8')
end_flag = bytes('}', 'utf-8')

first_part_xored = xor(cipher_bytes[0:7], start_flag)
last_part_xored = xor(cipher_bytes[-1:], end_flag)

print(f"XOR entre primeiros bytes e 'crypto{{' => {first_part_xored.decode()}")
print(f"XOR entre último byte e '}}' => {last_part_xored.decode()}")

key = first_part_xored.decode() + last_part_xored.decode()

if key == 'myXORkey':
    print(f"A chave correta é: {key}")
else:
    print(f"A chave obtida não é a esperada. Chave gerada: {key}")

flag = xor(bytes('myXORkey', 'utf-8'), cipher_bytes)

print(flag.decode())
```

### Explicação
Neste código, aplicamos o XOR nos primeiros bytes do texto criptografado com a string `"crypto{"` e no último byte com o caractere `"}"`. O resultado disso nos dá a chave usada para a criptografia. Quando a chave é verificada como `myXORkey`, usamos essa chave para descriptografar a mensagem completa.

### Resultado  
Ao rodar o código, o resultado obtido será a flag:

>`crypto{1f_y0u_Kn0w_En0uGH_y0u_Kn0w_1t_4ll}`
