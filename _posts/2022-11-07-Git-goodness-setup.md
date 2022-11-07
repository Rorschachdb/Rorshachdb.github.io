---
title:  "Git goodness"
subtitle : "Basic Setup"
tags: [computing, informatique, git, goodness, setup]
---


Après mur réflexion, je me suis dit que je ferai un petit article pour te raconter comment faire une première
configuration sympa de git.

# De quoi qu'on cause

On va causer des bases en allant à l'essentiel. Grosso modo, on va reprendre le contenu de 
[First Time Git Setup](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup) en version slim et on va voir
le strict nécessaire pour s'authentifier de manière le plus sécurisé à un serveur git distant donc en ssh (enfin 
dis-moi dans les commentaires s'il y a mieux, après tout je me fais vieux). 

# Les bases
## La configuration où ça
1. /etc/gitconfig file: Partagé par tous les utilisateurs, il faudra passer `--system` à `git config`, bien sûr il te 
faudra des droits d'administrateur pour éditer ce fichier ou utiliser `git config --system`. Je ne m'en suis jamais servi
si tu en as déjà eu l'usage, raconte-moi ça dans les commentaires, ça m'intéresse.

2. `~/.gitconfig` ou `~/.config/git/config` : Les valeurs specifiques à ton utilisateur pour tous tes projets, cette fois,
c'est l'option `--global` qu'il faudra passer à `git config`.

3. le fichier `git/config`de ton depot il est spécifique à chacun de tes projets et si tu es malin tu as deviné qu'il te
faudra cette fois utiliser l'option `--local` avec `git config`. Bien sûr, ça ne fonctionne que si tu te trouves dans un dépot 
git

## Configuration du nom
```bash
$ git config --global user.name "John Doe"
```
## Configutration de l'email
```bash
$ git config --global user.email johndoe@example.com
```
Personnellement, je recommanderais de ne jamais utiliser un mail personnel si c'est pour du code public. Utilise donc
un mail professionnel si c'est dans un cadre pro que ton code est publié et fais toi un mail dédié pour toute activité
freelance ou opensource

# Un peu plus de configuration

## Le bon editeur au bon moment

```
git config --global core.editor "nano"
```

Cf. la section concerné pour correctement configurer un certain nombre d'éditeu dans 
l'[appendice de la documentation git](https://git-scm.com/book/en/v2/Appendix-C%3A-Git-Commands-Setup-and-Config)

## Eviter les bêtise

Lorsque l'on réalise un pull plusieurs stratégies sont disponibles :

```
git pull --merge
```
Cette commande avec l'option `--merge` (défaut) réalise en cas de divergence un commit de merge entre ton commit local 
et le commit de labranche remote.

```
git pull --rebase
```
Cette commande avec l'option `--rebase` va rejouer les commits locaux comme un rebase par-dessus les commits de 
la branche distante.

De mon point de vue la seconde option est la plus propre, elle va éviter de pousser ultérieurement tout un tas de 
petites verrues de commit qui peuvent être source de confusions pour tes camarades.

Une bonne façon d'éviter de se tromper c'est de positionner une règle de configuration globale :

```
git config --global pull.rebase true
```

Par contre, cela ne traite que des commits sans prendre en compte un eventuel working space non vide et pottentiellement
en conflit. Pour ça, cf. [Mon post sur mes aliases](2022-10-30-Git-googness-Aliases)

# Un peu de SSH

D'après [wikipedia](https://fr.wikipedia.org/wiki/Secure_Shell)

> Secure Shell (SSH) est à la fois un programme informatique et un protocole de communication sécurisé. Le protocole de
> connexion impose un échange de clés de chiffrement en début de connexion. Par la suite, tous les segments TCP sont 
> authentifiés et chiffrés. Il devient donc impossible d'utiliser un sniffer pour voir ce que fait l'utilisateur.
> Le protocole SSH a été conçu avec l'objectif de remplacer les différents protocoles non chiffrés comme rlogin, 
> telnet, rcp et rsh.
> 
## Mise en place d'une pair de clef public privé

L'authentification ssh repose sur l'utilisation d'un pair de clef, l'une public que l'on publie là auprès d'organisme pour 
authentifier les connexions ssh vers ceux-ci et l'autre privée conservée par l'utilisateur (au sens large). La clef 
privée est protégée par une phrase authentication, on parle de phrase plutôt que de mot de passe pour souligner
l'importance de la [longueur dans la solidité d'un mot de passe](https://www.passwordmonster.com/).

On va donc créer une pair de clef pour utiliser dans nos échanges avec un remote git. Pour cela je me réfère toujours
à la 
[documentation de git](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)
qui a le bon goût de se mettre à jour en fonction des dernières avancées en matière de sécurité.

## Avoir l'agent ssh toujours sous la main
Comme l'indique la documentation de git pointée dans le pâragraphe précédent, une bonne manière d'éviter de ressaisir 
constamment notre passphrase est de démarrer l'agent ssh, un conteneur qui va garder en mémoire la clef privée "déchiffrée".

La manière la plus simple de démarrer l'agent ssh qui contiendra vos clefs ssh pour toutes les connexions à venir est 
de lancer `eval $(ssh-agent)` puis `ssh-add [votre clef privée]`. Cependant, démarrer l'agent ssh à chaque lancement de 
terminal... tu vas vite trouver ça très pénible. Voici un script à ajouter dans un fichier `.profile` ou `.bash_profile` :

```bash
SSH_ENV="$HOME/.ssh/environment"

function start_agent {
     echo "Initialising new SSH agent..."
     /usr/bin/ssh-agent | sed 's/^echo/#echo/' > "${SSH_ENV}"
     echo succeeded
     chmod 600 "${SSH_ENV}"
     . "${SSH_ENV}" > /dev/null
     /usr/bin/ssh-add;
}

# Source SSH settings, if applicable

if [ -f "${SSH_ENV}" ]; then
     . "${SSH_ENV}" > /dev/null
     #ps ${SSH_AGENT_PID} doesn't work under cywgin
     ps -ef | grep ${SSH_AGENT_PID} | grep ssh-agent$ > /dev/null || {
         start_agent;
     }
else
     start_agent;
fi
```
Ce code n'est pas de moi, le principe et de stocker dans le fichier '~/.ssh/environment' les données de l'agent ssh :
* socket
* numero de PID

Si cette portion de scrip est ré exécuté cela permet de sourcer le fichier environement pour récupérer ces informations
(`. "${SSH_ENV}" > /dev/null`) et si le fichier n'existe pas ou qu'aucun process de même PID n'est disponible de
redémarrer l'agent.

Par ailleurs la configuration ssh suivante (fichier ~/.ssh/config`) permet d'ajouter des clefs au démarrage de l'agent :
``` 
IdentityFile ~/.ssh/clef1
IdentityFile ~/.ssh/clef2
```

À noter les droits de ce fichier doivent être `rw-------`. 

## Enregistrer sa clef public

Une fois la paire créé il faut l'enregistrer

### Sous GitHub
L'intégralité de la procédure se trouvant [ici](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account). 
Je ne vais rien rajouter d'autant que la page est mise à jour régulièrement

### Sous GitLab
C'est la même chose pour gitlab et ça se trouve [là](https://docs.gitlab.com/ee/user/ssh.html#add-an-ssh-key-to-your-gitlab-account).

# Pour finir, la branche courante dans le prompt

Pour se faire on va éditer la variable d'environement PS1 qui permet de définir l'intitulé du prompt (l'invite de commande en bon françois).

Plus d'info sur le fonctionnement de PS1 :

* [http://www.linuxnix.com/2013/04/linuxunix-shell-ps1-prompt-explained-in-detail.html]
* [http://blog.twistedcode.org/2008/03/customizing-your-bash-prompt.html]

Pour modififier le contenu de PS1, on va éditer le fichier `.bashrc` en ajoutant le code suivant : 

```bash
parse_git_branch() {
     git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(\1)/'
}
export PS1="\u@\h \[\e[32m\]\w \[\e[91m\]\$(parse_git_branch)\[\e[00m\]$ "
```



# Et Après
Eh bien pourquoi pas relire mon merveilleux (et je pèse mes mots) [article sur les Aliases Git]((2022-10-30-Git-googness-Aliases))