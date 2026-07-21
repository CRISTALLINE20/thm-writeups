# Notes PortSwigger - SQL injection

## Les types d'injection SQL
- Recuperation de donnees cachees : modifier une requete SQL pour obtenir des resultats supplementaires.
- Detournement de la logique applicative : modifier une requete pour interferer avec la logique de l'application.
- Attaques UNION : recuperer des donnees provenant d'autres tables de la base.
- Injection SQL aveugle (blind) : les resultats de la requete ne sont pas renvoyes dans les reponses, on deduit l'information autrement.

## Lab 1 - Retrieval of hidden data
Injection dans le parametre category de l'URL :
'+OR+1=1--

Affiche tous les produits, y compris les non publies.

Pourquoi ca marche : le ' ferme la chaine, OR 1=1 rend la condition toujours vraie, et -- commente le reste de la requete (dont le filtre released = 1). La base renvoie toutes les lignes.

## Lab 2 - Login bypass
Injection dans le champ username :
- username : administrator'--
- password : peu importe (vide ou x)

Pourquoi ca marche : le ' ferme la chaine du username, le -- commente le reste de la requete dont la verification du mot de passe. Le serveur ne controle jamais le password et connecte l'utilisateur administrator.

A noter : la requete contient un token csrf. En passant par Burp Repeater j'ai garde ce token tel quel. Le champ username seul aurait aussi suffi car le navigateur gere le token automatiquement.

### La requete envoyee
<img width="602" height="610" alt="image" src="https://github.com/user-attachments/assets/32b61ff8-3aec-4d6a-b429-b0c39ff196a2" />


### La reponse 
302 Found + Location: /my-account?id=administrator + cookie de session admin.
<img width="609" height="614" alt="image" src="https://github.com/user-attachments/assets/ed9defc3-a323-4677-ae85-e4180b6635f5" />


## Ce que j'ai appris
- A quoi servent les deux elements cles d'une injection SQL : le ' (apostrophe) qui ferme la chaine de caracteres, et le -- qui commente le reste de la requete pour l'ignorer.
- La difference entre une requete GET et une requete POST : dans le lab 1, l'injection passait par l'URL (GET), donc modifiable directement dans la barre d'adresse. Dans le lab login bypass, les identifiants partaient en POST (verifie via l'onglet Network), donc pas visibles dans l'URL.
- Comment modifier une requete dans Burp : intercepter le POST via Proxy, l'envoyer au Repeater (clic droit > Send to Repeater), modifier la valeur du parametre username dans le corps de la requete, puis Send. Un code 302 en reponse indique que le bypass a fonctionne.


## Difficultes rencontrees
- La prise en main de Burp a ete difficile au debut. J'ai mis du temps a comprendre l'interface et le role de chaque onglet.
- L'interception (Intercept) bloque la navigation quand elle reste activee : la page charge a l'infini tant qu'on n'a pas valide ou desactive l'interception. J'ai compris qu'il faut la repasser sur off apres avoir envoye la requete au Repeater.
