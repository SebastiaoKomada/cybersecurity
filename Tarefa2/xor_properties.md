
# [XOR Challenge]  
###### Resolvido por @[SebastiaoKomada]  
> Desafio CryptoHack sobre XOR

## Sobre o Desafio  
Neste desafio, vamos explorar como a operação XOR funciona em criptografia. O objetivo é usar as propriedades do operador XOR para reverter uma série de operações que criptografaram uma flag. A compreensão dessas propriedades é fundamental para resolver problemas mais avançados em sistemas de criptografia, como nos cifrões de bloco.

## Propriedades do XOR  
Existem quatro propriedades principais que precisamos considerar ao trabalhar com o operador XOR:

- **Comutativa:** \( A &#8853; B = B &#8853; A \)
- **Associativa:** \( A &#8853; (B &#8853; C) = (A &#8853; B) &#8853; C \)
- **Identidade:** \( A &#8853; 0 = A \)
- **Inversa:** \( A &#8853; A = 0 \)

Essas propriedades são cruciais para entender como manipular os valores criptografados e reverter o processo. Vamos ver como isso se aplica ao nosso desafio.

### Os Dados do Desafio
Temos as seguintes chaves e criptografia:

- **KEY1**: `a6c8b6733c9b22de7bc0253266a3867df55acde8635e19c73313`
- **KEY2 ^ KEY1**: `37dcb292030faa90d07eec17e3b1c6d8daf94c35d4c9191a5e1e`
- **KEY2 ^ KEY3**: `c1545756687e7573db23aa1c3452a098b71a7fbf0fddddde5fc1`
- **FLAG ^ KEY1 ^ KEY3 ^ KEY2**: `04ee9855208a2cd59091d04767ae47963170d1660df7f56f5faf`

Você pode acessar o desafio [aqui](https://cryptohack.org/courses/intro/xor1/).

## Solução

### Analisando o XOR  
O XOR é uma operação bit a bit que resulta em `1` se os bits comparados forem diferentes e `0` se forem iguais. O XOR é útil em criptografia porque, sendo comutativo e associativo, podemos manipular os bits de forma flexível para cifrar ou decifrar dados.

### Passos para Resolver o Desafio

#### 1. Conversão de Hexadecimal para Bytes
O primeiro passo é converter as chaves e a flag de hexadecimal para bytes. Isso é necessário porque a operação XOR é feita diretamente sobre os valores binários, e o formato hexadecimal é apenas uma representação mais legível dos dados binários.

#### 2. Aplicando o XOR  
Com as chaves em formato de bytes, podemos aplicar a operação XOR entre elas. Usando as propriedades do XOR, podemos aplicar as operações na ordem correta para descobrir a flag.

#### 3. Código em Python  
Aqui está o código em Python que aplica o XOR para encontrar a flag. Vamos usar a função `xor` da biblioteca `pwntools` para realizar as operações de forma simples e eficiente.

```python
from pwn import *

key1 = bytes.fromhex("a6c8b6733c9b22de7bc0253266a3867df55acde8635e19c73313")
key1_2 = "37dcb292030faa90d07eec17e3b1c6d8daf94c35d4c9191a5e1e"  
key2_3 = "c1545756687e7573db23aa1c3452a098b71a7fbf0fddddde5fc1"  
flag_key123 = "04ee9855208a2cd59091d04767ae47963170d1660df7f56f5faf"

def xor_strings(str1, str2):
    return xor(str1, str2)

key2 = xor_strings(bytes.fromhex(key1_2), key1)  
key3 = xor_strings(bytes.fromhex(key2_3), key2)  
key1_2_3 = xor_strings(bytes.fromhex(key1_2), key3)  

flag = xor_strings(bytes.fromhex(flag_key123), key1_2_3)

print(f"crypto{{{flag.decode()}}}")
```

### Resultado  
Ao rodar o código acima, você obterá o seguinte resultado:

>`crypto{x0r_i5_ass0c1at1v3}`
