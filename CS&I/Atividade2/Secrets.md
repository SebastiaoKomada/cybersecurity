# [Secrets]  
###### Resolvido por @[SebastiaoKomada]  
> Desafio Web Exploitation  

## Sobre o Desafio  
O enunciado apresenta a seguinte descrição: *“Temos várias páginas escondidas. Você consegue encontrar aquela com a bandeira?”*.  
Junto a isso, foi disponibilizada a dica: *“folders folders folders”*.  

Logo na primeira leitura, fica evidente que o desafio se concentra na exploração de **diretórios ocultos** em um servidor web.
A repetição da palavra *folders* já sugere que não seria suficiente encontrar apenas um diretório, mas sim percorrer uma cadeia de pastas até finalmente chegar ao destino onde a flag estava escondida.  

Assim, a estratégia mais natural é utilizar uma ferramenta de enumeração de diretórios para revelar o que não está exposto de maneira explícita.  

## Solução  

### Enumeração inicial  
Para iniciar a exploração, utilizamos o **Gobuster**, uma ferramenta muito eficiente para varrer diretórios e arquivos de um servidor web. Foi escolhido a wordlist `common.txt` para essa atividade. 

O comando utilizado foi:  

```bash
gobuster dir -u http://saturn.picoctf.net:57004/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -x .php,.html,.txt -t 50
```  

O resultado já apontou para um diretório chamado **`secret`**.  
> Nota: o número da porta varia a cada vez que o serviço é reiniciado, portanto pode ser diferente em outras execuções.  

### Primeira pasta encontrada: `/secret/`  
Dentro do caminho `/secret/index.html`, havia apenas um texto simples acompanhado de uma imagem. Apesar de não conter nenhuma informação útil aparente, essa página confirmava que a exploração estava na direção certa, já que dificilmente seria listada de forma pública sem motivo.  

![secret-index](https://github.com/user-attachments/assets/90b7943e-e79d-4355-90c7-d9421c7f2b7d)  

Diante disso, a mesma técnica de varredura foi aplicada novamente, agora dentro do diretório `secret`, para verificar a existência de subpastas adicionais.  

### Segunda pasta: `/hidden/`  
A nova enumeração revelou o diretório **`hidden`**, acessível em `/secret/hidden/index.html`.  
Ao visitá-lo, foi exibido um formulário de login:  

![hidden-login](https://github.com/user-attachments/assets/1fbe3be9-7270-4ea3-8c5b-1a9e056bb27c)  

Essa tela sugeria que o desafio poderia envolver autenticação ou manipulação de parâmetros. Para confirmar, foram feitos testes simples enviando credenciais comuns, como `admin:admin`, ao mesmo tempo em que as requisições eram inspecionadas através do Burp Suite.  

### Identificando o parâmetro oculto  
Durante a análise no Burp Suite, além dos parâmetros de `username` e `password`, havia também um parâmetro extra denominado **`db`**.  

```
GET /secret/hidden/index.html?username=admin&password=admin&db=superhidden%2Fxdfgwd.html HTTP/1.1
```  

O valor desse campo parecia indicar o caminho para outro recurso, porém com parte codificada em URL (`%2F` corresponde a `/`).  
Decodificando o valor, era possível perceber a referência a uma nova pasta. 

### Terceira pasta: `/superhidden/`  
Acessando manualmente o endereço `/secret/hidden/superhidden/index.html`, foi exibida a seguinte mensagem:  

```
Finally. You found me. But can you see me
```  

![superhidden-msg](https://github.com/user-attachments/assets/f2e8bd78-66ca-4915-a991-e32d59a8ad32)  

O texto deixava claro que a flag estava ali, mas não de forma visível diretamente na tela.  

### Flag encontrada  
Com uma inspeção mais cuidadosa do conteúdo da resposta HTTP — novamente utilizando o Burp Suite — foi possível localizar a flag escondida no código-fonte da página, dentro de uma tag `<h3>`:  

```html
<h3 class="flag">picoCTF{succ3ss_@h3n1c@10n_51b260fe}</h3>
```  

**Flag:**  
`picoCTF{succ3ss_@h3n1c@10n_51b260fe}`  
