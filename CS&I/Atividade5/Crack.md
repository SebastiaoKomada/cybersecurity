# [Crack]
###### Resolvido por @SebastiaoKomada  
> Desafio: Reverse Engineering

---

### Sobre o desafio  
Encontrar a senha que faz o programa imprimir `good kitty!`.

---

## Solução

### Reconhecimento inicial  
Primeiro, confirmamos que o binário é um **ELF 64-bit dinâmico (PIE)**:

```bash
└─$ file crack
crack: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=8c584d707909182cb49dab6ebe51cca2217ab1ed, for GNU/Linux 3.2.0, not stripped
```

Ao executar o programa, ele solicita uma senha; qualquer entrada incorreta retorna `bad kitty!`:

```bash
└─$ ./crack
enter the right password
AAAAAA
bad kitty!
```

---

### Análise com Radare2  
Abrimos o programa em modo de depuração e deixamos o *analyzer* fazer a varredura completa:

```bash
r2 -AA -d ./crack
```

Listando as funções detectadas:

```r2
afl
```

Entre várias funções encontradas (muitas sem nome), três chamaram atenção:
- `main`
- `sym.ppeuler_3`
- `sym.factorial`

---

### Analisando a função `main`  
Na desassemblagem de `main`, observamos três chamadas consecutivas a funções:

```
0x55ff23d882cc      e8 f8 fe ff ff     call sym.ppeuler_3
...
0x55ff23d882e4      e8 e7 fd ff ff     call sym.imp.cbrt
...
0x55ff23d882f8      e8 88 ff ff ff     call sym.factorial
```

Logo depois, há uma comparação com o valor `8`:

```
0x55ff23d884de      48 83 f8 08       cmp rax, 8    ; 8
```

Isso indica que a senha correta possui **8 bytes**, já que o programa verifica explicitamente esse tamanho.

---

### Fluxo de dados e funções auxiliares  
A função `ppeuler_3` carrega uma constante em `rax`:

```
0x55ff23d881cd      48b8c7ea89..   movabs rax, 0x8be589eac7
```

Esse valor é passado para a função de raiz cúbica (`cbrt`), que resulta em **18**.  
Em seguida, esse valor é enviado como argumento para `factorial`, e o resultado final (armazenado em `rax`) é gravado em memória:

```
0x55ff23d882fd      48 89 44 24 10    mov qword [var_10h], rax
```

---

### Depuração e leitura da memória  
Para investigar o valor final, colocamos um *breakpoint* nessa instrução e continuamos a execução:

```
[0x563d90e902ac]> db 0x563d90e902fd
[0x563d90e902ac]> dc
INFO: hit breakpoint at: 0x563d90e902fd
```

Inspecionando o conteúdo de `rax` e da pilha, notamos dados aparentemente desorganizados — isso ocorre porque o programa constrói a senha **byte a byte** em formato *little-endian* dentro de um loop.

Após o loop, o valor final já está organizado em memória. Uma instrução próxima a esse ponto escreve o qword que inicia o buffer da senha:

```
0x55ff23d8836e      48 c7 44 24 40 ..   mov qword [var_40h], 1
```

Colocando um *breakpoint* aqui e executando novamente:

```
db 0x55ff23d8836e
dc
INFO: hit breakpoint at: 0x559027f4536e
```

Agora podemos imprimir os 8 bytes do buffer usando o comando `ps` (*print string*):

```
[0x559027f4536e]> ps 8 @ rsp+0x10
```

O programa havia montado e armazenado a senha final:

```
00sGo4M0
```

Apos receber a senha final, podemos testar no programa:

```
└─$ ./crack
enter the right password
00sGo4M0
good kitty!
```
Com isso confirmamos que a senha montada pelo programa é, de fato, a senha correta: ao fornecê-la na execução o programa retorna good kitty!.

---

### Explicação 
A sequência de chamadas (`ppeuler_3` → `cbrt` → `factorial`) revela o fluxo de cálculo que leva à geração da senha.  
A função `ppeuler_3` fornece uma constante fixa, `cbrt` aplica uma transformação matemática (raiz cúbica e truncamento) e `factorial` calcula o fatorial desse resultado, cujo retorno é usado como base para gerar os bytes da senha.

O programa então **reconstrói a senha byte a byte** dentro de um loop, escrevendo os bytes em formato *little-endian*.  
Por isso, inspecionar `rax` ou a pilha *antes* da finalização do loop mostra dados aparentemente “embaralhados”.  
A leitura correta deve ser feita **logo após o loop**, onde o buffer já está organizado. 

---

## Flag encontrada  
**Flag:** `00sGo4M0`

---

### Aprendizado
Ao longo da investigação executei e aprendi diversos comandos do **radare2** que facilitaram a depuração e a extração da senha.  
Abaixo estão os principais, com suas finalidades e exemplos práticos:

- `db <endereço>` — coloca um breakpoint.  
  Ex: `db 0x563d90e902fd`

- `dc` — continua a execução até o próximo breakpoint.  
  Ex: `dc`

- `ps <n> @ <endereço>` — imprime `n` bytes como string (útil quando o conteúdo é texto).  
  Ex: `ps 8 @ rsp+0x10` → `00sGo4M0`

- `px <n> @ <endereço>` — imprime `n` bytes em hex (útil pra ver os bytes crus e a ordem).  
  Ex: `px 8 @ rsp+0x10` → `0x00 0x30 0x73 ...`

