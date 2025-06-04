
# [XOR Starter]  
###### Resolvido por @[SebastiaoKomada]  
> Este é um desafio de CryptoHack sobre XOR

## Sobre o Desafio  
O XOR é um operador bitwise que retorna 0 quando os bits comparados são iguais, e 1 quando são diferentes. Em livros, o operador XOR é frequentemente denotado por ⊕, mas na maioria dos desafios e linguagens de programação, utiliza-se o caractere ^.  
Quando trabalhamos com números binários maiores, aplicamos o XOR bit a bit. Por exemplo: 0110 ^ 1010 = 1100. Podemos XOR números inteiros convertendo-os primeiro de decimal para binário. Para XOR em strings, cada caractere é convertido para o número que representa seu valor Unicode e, então, aplicamos o XOR.  
Neste desafio, dada a string "label", você deve aplicar o XOR a cada caractere com o valor inteiro 13. Após a operação, converta os resultados de volta para uma string e envie a flag no formato: `crypto{new_string}`.

Você pode acessar este desafio clicando [aqui](https://cryptohack.org/courses/intro/xor0/).

## Solução  

### Introdução  
O primeiro passo é entender o comportamento do operador XOR descrito no enunciado do desafio.  
Neste caso, o exercício solicita que o XOR seja aplicado a cada caractere da string "label", utilizando o número 13 como valor da operação.

### Análise  
A primeira coisa a fazer é instalar a biblioteca `pwntools`. Para isso, basta abrir o terminal e rodar os seguintes comandos:  

1. Criar um ambiente virtual isolado:
    ```bash
    python3 -m venv nome_da_venv
    ```

2. Ativar o ambiente virtual:
    ```bash
    source nome_da_venv/bin/activate
    ```

Ao rodar esses comandos, sua linha de comando ficará assim:
![image](https://github.com/user-attachments/assets/a951548d-8b18-4d67-aabc-266a47cf7761)

Em seguida, instale a biblioteca `pwntools` com:
![image](https://github.com/user-attachments/assets/a9d92bd6-1c79-409c-9634-75681c5d8033)

### Decodificação  
Agora, partimos para o código em Python. Precisamos criar uma função que receba a string "label" e o valor do XOR. Para cada caractere, utilizaremos as funções `ord` e `chr`. A função `ord` converte o caractere em seu valor Unicode, e `chr` faz a operação inversa. Depois disso, aplicamos o XOR sobre os valores inteiros resultantes.

```python
def xor_string(label, xor_value):
    nova_palavra = ""
    for letra in label:
        nova_palavra += chr(ord(letra) ^ valor_xor)
    return nova_palavra

label = "label"
valor_xor = 13  
nova_palavra = xor_string(label, valor_xor)  
print(f"crypto{{{nova_palavra}}}")
```

### Resultado  
Ao rodar o código, você obterá o seguinte resultado:

>`crypto{aloha}`
