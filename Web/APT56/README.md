# APT56 Intro

Si on se connectait sur le site, on pouvais voir que l'on était redirigé vers une page puis quelques secondes après, retourné vers le login. Il fallait intercepté cette page de transition qui contenait le flag dans le titre.

# APT56 1/3

On est face à un formulaire de login où on nous demande de passer admin. Les quelques tentatives de SQLi ne sont pas concluantes donc pas d'injection possible, ou filtrage très poussé pour un challenge facile. Je tente donc de me connecter avec des données bullshit: *(username: test, password: test par exemple)*. Et je tente de voir ce qu'il s'est passé. Un cookie a été crée, qui semble être du base64 encodé pour des url, en le décodant, on peut voir que c'est un objet PHP sérializé (la Classe User pour être exact). Il y a 3 champs: username, password, isadmin donc il nous est possible de recréer une pseudo classe pour le déserializé : 
```php
class User {
  public $username;
  public $password;
  public $isadmin;
}
var_dump(unserialize('O:4:"User":3:{s:8:"username";s:5:"test";s:8:"password";s:5:"test";s:7:"isadmin";s:5:"false";}'));
```
Il nous reste plus qu'à changer les valeurs, à le recoder, éditer le cookie et recharger la page: on a le flag.
```php
$cookie = "Tzo0OiJVc2VyIjozOntzOjg6InVzZXJuYW1lIjtzOjU6InRlc3QiO3M6ODoicGFzc3dvcmQiO3M6NToidGVzdCI7czo3OiJpc2FkbWluIjtzOjU6ImZhbHNlIjt9";
$unserialize = unserialize(base64_decode(urldecode($cookie)));
$unserialize->isadmin = 'true'; // en string car c'est un string qui est attendu
echo "New cookie is: " . urlencode(base64_encode(serialize($unserialize))); 
```

# APT56 2/3

N'ayant plus le site en mémoire et vu qu'ils ne sont plus en ligne, je vais faire avec ce que je me rappel des payload.  

Le but de celui ci était de récupérer le contenu de ``index.php``, le flag était commenté en PHP dedans.   

La partie du site qui nous intéressait là était le finder, disponible uniquement en admin à l'adresse ``/finder.php``. Nous spécifions un chemin vers le fichier à lire et il nous était retourné une image en base64. Un simple index.php (de mémoire), nous retournais le contenu du fichier en base64, plus qu'à le décoder et à le lire.

# APT56 3/3

De là, nous devons lire le flag qui se trouve à la racine du serveur sous ``/flag.txt``.
On peut tenter de venir récupérer le contenu du ``flag.txt`` directement en essayant une LFI depuis le finder, mais on se rend vite compte qu'on a pas la permission. Il faut trouver une autre solution.  

Y'a une partie du site qu'on a pas toucher: l'upload de fichier. Après quelque test, on se rend vite compte qu'il est vulnérable aux upload de code, donc RCE. Il vous suffisait de placer un préfixe genre de GIF, enchaîner sur votre PHP et l'uploader en modifiant le content-type et le nom pour qu'il finisse par .gif. 

Le fichier est ensuite accessible sous ``uploads/<nom>.gif`` et on peut voir que le PHP est bien exécuté.

Pour faciliter la suite, j'ai utilisé un reverse sheel qui vient se connecter à mon PC pour avoir un shell intéractif : https://pentestmonkey.net/tools/web-shells/php-reverse-shell.
Après quelques investigations on remarque que seul la commande nl peutêtre exécuté sur le flag, et donc avec ``sudo -u root /usr/bin/nl /flag.txt`` on obtient le dernier flag de cette série hyper intéressante.

> La fin du challenge se transforme en PWN.
