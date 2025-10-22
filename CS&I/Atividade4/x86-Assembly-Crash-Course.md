# [x86 Assembly Crash Course]
###### Escrito por  por @[SebastiaoKomada]

### Introdução

Opcodes sao numeros que representam as instrucões levadas para a CPU. É importante ressaltar que o operando é tratado em relação a notação 'Endianness'

#### Instruções fundamentais: mov, nop, lea e operações de deslocamento e rotação.

A instrução `mov` move um valor para um registrador, um registrador para outro registrador ou um local da memória para um registrador.
A forma como essa instrução é escrita em código funciona da seguinte maneira:

````
mov destination, source  
````

A instrução `nop` significa nenhuma operação. Uma utilização prática dessa instrução é empregá-la quando se deseja consumir espaço em bytes, por exemplo, em uma situação de buffer overflow.

````
nop  
````

Ao contrário da instrução mov, que move o valor da origem para o destino, a instrução `lea` move o valor do endereço para o destino.

````
lea destination, source  
````

Também existem as instruções de deslocamento e rotação, que são responsáveis por mover cada bit para outra posição dentro do registrador.

### Registrador de flags (EFLAGS)

Em Assembly, existem registradores sinalizadores, também conhecidos como flags. Eles são responsáveis por indicar o resultado de determinadas operações.

Temos: CF, PF, AF, ZF, SF, DE, DF E SE

Cada flag representa uma condição específica de uma operação. Dentre elas, destacam-se as mais utilizadas:

CF `Carry Flag`: utilizada quando ocorre um transbordamento de memória.

PF `Parity Flag`: utilizada quando o resultado possui um número par de bits iguais a 1.

ZF `Zero Flag`: utilizada quando o resultado é igual a zero.

SF `Sign Flag`: utilizada quando o resultado é negativo.

### Operações aritméticas

Temos instruções básicas como adição, subtração, multiplicação e divisão.
Elas são representadas da seguinte forma:

````
add destination, value  
sub destination, value  
mul value  
div value  
````

Vale ressaltar que a operação de subtração aciona a Carry Flag quando o valor do operando de destino é menor que o da origem, e ativa a Zero Flag quando o resultado é igual a zero.

Já as instruções de multiplicação e divisão trabalham com valores de 64 bits, podendo operar com 2 registradores de 32 bits ou com um registrador de 64 bits dividido em duas partes.

### Condicionais e testes

Instrução test

A instrução `test` define a Zero Flag (ZF) quando o resultado é igual a zero.
Ela é utilizada por consumir menos bytes do que uma verificação de comparação direta com o valor zero.
````
test destination, source  
````
Instruções de Salto

Como o Assembly não possui estruturas condicionais de alto nível, como o if de outras linguagens, utiliza-se a instrução `jump` para realizar desvios condicionais no fluxo do programa.

Essas instruções são conhecidas como saltos condicionais, e existem diversos tipos, entre eles:
jz, jnz, je, jne, jg, jl, jge, jle, jb, jae, jbe.

### Stack

Por fim, temos a pilha, que funciona no modo LIFO (Last In, First Out), ou seja, o último valor inserido é o primeiro a ser removido.

Existem duas instruções principais associadas à pilha: push e pop.
A instrução push insere um valor na pilha, enquanto a instrução pop remove o valor mais recente inserido.
````
push source  
pop destination 
````
