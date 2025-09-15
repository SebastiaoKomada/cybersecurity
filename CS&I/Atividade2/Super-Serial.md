# [Super Serial]  
###### Resolvido por @[SebastiaoKomada]  
> Desafio Web Exploitation  

## Sobre o Desafio  
O enunciado apresenta a seguinte descrição: *“Tente recuperar a bandeira armazenada neste site `http://mercury.picoctf.net:14804/`”*.  
Junto a isso, foi disponibilizada a dica: *"A bandeira está em ../flag"*

De primeiro momento, podemos pensar que temos que achar a flag está armazenada dentro do servidor. O objetivo então é explorar o código e identificar uma vulnerabilidade que nos permitisse ler esse arquivo oculto
## Solução  

### Análise inicial do código
//vou abordar ainda sobre a pagina de login, e depois disso falar do codigo.

Ao acessar o index.phps (para acessar a versão exposta), temos o seguinte codigo php:
````php
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
````

Isso nos indica que existe mais duas páginas que podemos acessar, `cookie.php` e `authentication.php`.
Ao analisar o codigo, podemos ver que se tivermos a autorização correta, é setado no nosso codigo um cookie de login que é codificado em base64, e depois é serializado.

Acessando a pagina de `cookie.php`, temos o seguinte código php:
````php
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
````
Ao final do código, podemos ver que existe uma função, onde o sistema verifica se o existe de fato um cookie, e a partir disso ele decodifica ele e verifica as permissões de usuário.
Se nao, ele lança um erro. Aqui podemos encontrar uma brecha, pois como tudo foi setado no cookie, podemos fazer o controle dele.

### Flag encontrada  

**Flag:**  
``  
