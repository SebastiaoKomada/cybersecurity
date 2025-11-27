# [CSRF]

###### por @SebastiaoKomada

> Estudo sobre Cross-Site Request Forgery

---

## O que é CSRF?

CSRF (**Cross-Site Request Forgery**) é uma falha de segurança onde o navegador de um usuário autenticado acaba realizando uma ação que **ele não tinha intenção de fazer**.  
Isso acontece porque o navegador, por padrão, **envia cookies automaticamente** sempre que acessa o domínio de um site.

Ou seja: se o usuário já estiver logado, o invasor consegue fazer uma requisição “como se fosse ele”.

---

## Como um ataque CSRF costuma acontecer

Apesar de existirem variações, o fluxo geral é sempre parecido:

1. **Preparação**  
   O atacante estuda como o site envia requisições e monta um link, formulário ou script que imita essas chamadas.

2. **A vítima interage sem perceber**  
   Basta abrir uma página maliciosa, clicar em algum link ou simplesmente carregar um recurso escondido.  
   Como o usuário está logado, o navegador manda junto os cookies da sessão.

3. **Falta de validação no servidor**  
   Se a aplicação não usa token CSRF, SameSite, verificação de origem ou algo similar, ela não tem como saber se a requisição veio de fato do usuário.

---

## Riscos mais comuns

- **Realizar ações importantes sem querer**  
  Troca de senha, alteração de email, exclusão de conta, transferências etc.

- **Explorar a confiança do navegador**  
  O servidor acredita que é o usuário quem está agindo.

- **Ataques “transparentes”**  
  Nada aparece na tela, portanto a vítima nem percebe.

---

## CSRF tradicional

É o tipo mais conhecido: quando o atacante usa requisições que mudam o estado da aplicação (POST, DELETE, PATCH…).  
A vítima acaba executando essas ações sem perceber, porque o navegador faz a requisição legítima.

---

## CSRF em requisições AJAX

Mesmo em chamadas assíncronas, o ataque ainda funciona se:

- o navegador enviar cookies automaticamente  
- o servidor aceitar `credentials: true`  
- não houver validação de origem  
- não existir token CSRF

Nesse caso, o atacante esconde um script que dispara requisições em segundo plano(assincrono).

---

## CSRF com Flash (legado)

Em sistemas muito antigos, arquivos `.swf` conseguiam enviar requisições com cookies.  
Apesar de o Flash estar morto, ainda há aplicações velhas vulneráveis.

---

## CSRF usando imagens, links e outros recursos

O atacante pode usar algo simples assim:

```html
<img src="http://victim.com/action?transfer=1000&to=attacker" width="0" height="0">
```

Se a vítima abrir a página, o navegador tenta carregar a imagem e **envia a requisição automaticamente**.

---

## Tokens CSRF e Double Submit

O token CSRF é um valor:

- único  
- aleatório  
- ligado à sessão  
- impossível de adivinhar

Ele deve ir na requisição (corpo ou header), e o servidor confere se está correto.

**Double Submit Cookie:**  
O servidor envia um cookie com o token, e o cliente devolve esse mesmo valor em um campo extra. O servidor compara os dois.

---

## SameSite Cookies

Configuração do navegador para limitar o envio automático de cookies em contextos cross-site.

- **Lax:** proteção moderada; cookies ainda vão em navegações GET típicas.  
- **Strict:** o cookie só vai quando a navegação vem do mesmo site.  
- **None:** cookies enviados sempre; exige `Secure` (HTTPS).

---

## SOP, CORS e possíveis brechas

A **Same-Origin Policy (SOP)** impede ler respostas, mas **não impede enviar requisições** então ela não evita CSRF.

No CORS, a configuração abaixo é um erro grave:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

Essas duas diretivas juntas abrem portas para ataques.

---

## Validação via Referer / Origin

Alguns sistemas se apoiam apenas nesses headers para validar a origem.  
O problema é que eles podem falhar por vários motivos:

- o navegador pode omitir o Referer  
- redirecionamentos podem removê-lo  
- políticas de privacidade podem esconder essa informação  
- navegadores antigos não enviam o Origin corretamente

Por isso, esse tipo de validação **não deve ser a única defesa**.
