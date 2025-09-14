# [Forbidden Paths]  
###### Resolvido por @[SebastiaoKomada]  
> Desafio Web Exploitation  

## Sobre o Desafio  
O enunciado apresenta a seguinte descrição: *“Você pode pegar a bandeira? Sabemos que os arquivos do site vivem em `/usr/share/nginx/html/` e o sinalizador está em `/flag.txt`, mas o site está filtrando caminhos de arquivo absolutos. Você pode superar o filtro para ler o sinalizador?”*.  

Desde o início, fica claro que o desafio envolve uma vulnerabilidade de **Path Traversal** (ou Directory Traversal).  
O sistema revela o caminho absoluto onde os arquivos ficam armazenados, mas impõe um filtro contra esse tipo de acesso direto. Dessa forma, o objetivo é contornar essa restrição navegando por diretórios com sequências do tipo `../` até alcançar o arquivo da flag.  

## Solução  

### Primeiros testes  
Ao acessar o sistema web, encontramos uma página simples contendo um campo de entrada:  

![input-page](https://github.com/user-attachments/assets/f68749e4-8a35-4358-b47e-62917684573b)  

Testando entradas básicas, como `flag.txt`, a resposta foi imediata:  
```
File does not exist
```  

Isso já deixava evidente que a aplicação estava verificando o caminho fornecido, aceitando apenas arquivos relativos à pasta atual.  

### Analisando o caminho  
Da descrição do enunciado, sabemos que:  
- O diretório raiz da aplicação é `/usr/share/nginx/html/`.  
- A flag está armazenada em `/flag.txt`.  

Portanto, para alcançar a flag, seria necessário **voltar da pasta atual até a raiz** do sistema de arquivos e, de lá, abrir o arquivo desejado.  

Cada ocorrência de `../` permite retornar um nível de diretório. Assim, aplicando a sequência quatro vezes (`../../../../`), conseguimos retroceder até o diretório raiz.  

### Acesso ao arquivo da flag  
Com esse raciocínio, bastava enviar o seguinte valor no input:  

```
../../../../flag.txt
```  

Ao submeter esse caminho, a aplicação revelou diretamente o conteúdo do arquivo contendo a flag:  

![flag-output](https://github.com/user-attachments/assets/fd13f668-94c0-46a6-83d3-50369f52d2e0)  

### Flag encontrada  
Com isso, foi possível identificar a flag escondida:  

**Flag:**  
`picoCTF{7h3_p47h_70_5ucc355_e5fe3d4d}`  
