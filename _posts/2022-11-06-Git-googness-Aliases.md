---
title:  "Git goodness"
subtitle : "Les aliases"
tags: [computing, informatique, git, goodness, setup]
---


Bienvenue sur ce premier article qui va inaugurer une série sur la mise en place de mon environement de development

# Mes sources
Avant de rentrer dans le vif du sujet, voici les liens qui me permettent de retrouver les alias qui m'intéressent 
lorsque je réinstalle un environement Git. Enfin ça, c'était jusqu'à ce que je centralise tout ici pour toi.

* [10 git aliases for a faster and productive git workflow](https://snyk.io/blog/10-git-aliases-for-faster-and-productive-git-workflow/)
* [Les aliases Git de l'extrême](https://www.synbioz.com/blog/tech/les-aliases-git-de-l-extreme)

Et n'oublis pas si tu ne le connais pas le site ultime pour tout savoir de git : [Learn Git Branching](https://learngitbranching.js.org/)

# Prérequis

Je ne te ferai pas l'insulte de te rappeler le minimum de configuration. Bon en fait si, n'oubli pas de configurer ton 
nom d'utilisateur et ton mail, ils sont indispensables. Si tu publies ton code sur un repo distant, il te faudra 
t'authentifier auprès de celui-ci et donc configurer le mode d'authentification adéquat.

# Les alias

Des alias, j'en utilise pas mal, et je ne vais pas te faire mon top 10 (ou 8 ou 5 ou 137). Je vais plutôt te les donner
par thématique. Si je mets un '*' dans le titre de la section, c'est que l'alias prend un paramètre.

Mais avant tout ...

## Ajouter un alias

Le plus simple pour moi, c'est l'édition directe de l'un des trois fichiers de configuration de git dans la section 
alias. Moi ça m'évite de me mélanger entre les caractères de déclaration de chaîne ou les échappements.
Elle doit ressembler à ceci :

```
[alias]
        # ALIAS.f : Fetch from a repository and prune any remote-tracking
        f   = fetch -p -t
        # ALIAS.st : Shortcut for status
        st = status
        # ALIAS.co : Shortcut for checkout
        co = checkout
```

Les alias peuvent aussi être directement ajouté depuis la ligne de commande comme toute propriété de configuration de
git :

```bash
$ git config --global alias.st 'status -sb'
```
## Lister les alias

Bon comme je te l'ai dit, ça va peut-être faire beaucoup (et puis j'en ajoute souvent quand je me rends compte qu'une
commande devient répétitive) alors commençons par les lister.

```
[alias]
    # ALIAS.alias : list all aliases
    alias = ! git config --get-regexp ^alias\\. | sed -e s/^alias\\.// -e s/\\ /\\ =\\ /
```

Et le résultat :

```bash
$ git alias
>f = fetch -p -t
>st = status
>co = checkout
>cob = checkout -b
>del = branch -D
>delmb = !git branch --merged | grep -v '*' | xargs -n 1 git branch -d
>aliasc = ! git config --get-regexp ^alias\. | sed -e s/^alias\.// -e s/\ /\ =\ /
>alias = ! cat ~/.gitconfig|grep '^[[:blank:]]*. ALIAS.'|sed s/^.*ALIAS\.//
>br = branch --format='%(HEAD) %(color:yellow)%(refname:short)%(color:reset) - %(contents:subject) %(color:green)(%(committerdate:relative)) [%(authorname)]' --sort=-committerdate
>ci = !f() { git commit -m "$1"; }; f
>cia = !f() { git commit -am "$1"; }; git add -A && f
>ciap = !f() { git add -A ; git commit -am "$1"; git done; }; f
>save = !f() { git commit -am "Save point at $(date --utc +'%Y%m%d-%H:%M:%S') ; PLEASE REBASE"; }; git add -A && f
>undo = reset HEAD~1 --mixed
>clear = !git reset --hard
>lg = !git log --pretty=format:"%C(magenta)%h%Creset -%C(red)%d%Creset %s %C(dim green)(%cr) [%an]" --abbrev-commit -30
>done = !git push -u origin HEAD
>rank = shortlog -sn --no-merges
>upd = !f() { branch=$1; if [ -z $branch ]] ; then branch=`git symbolic-ref HEAD | cut -d "/" -f 3-`; else git checkout $branch; fi; git pull --rebase origin $branch; }; f
>upds = !git stash && git upd && git stash pop
```
On remarque tout de suite le marqueur spécifique **`!`** , il indique que l'on va faire du bash et non pas mixer 
des commandes git. Dans un premier temps (`git config --get-regexp ^alias\\.`) on va lister les alias, mais avant 
l'affichage définitif, on va supprimer le prefix "alias." (`sed -e s/^alias\\.// -e s/\\ /\\ =\\ /`). Malheureusement 
ça indique la correspondance entre alias et commande sans en donner le descriptif.

Du coup je m'impose de commenter proprement mes alias et je fais de variantes de la commande : 

```
[alias]
    # ALIAS.aliasC : list all aliases
    aliasC = ! git config --get-regexp ^alias\\. | sed -e s/^alias\\.// -e s/\\ /\\ =\\ /
    # ALIAS.alias : list all aliases
    alias = "! cat ~/.gitconfig|grep '^[[:blank:]]*. ALIAS.'|sed s/^.*ALIAS\\.//"
```

Je renomme ma commande alias en aliasC (pour code) et j'ajoute la commande alias qui va cette fois travailler sur les
commentaires de mon fichier `.gitconfig` personnel. Dans un premier temps `cat ~/.gitconfig` pour faire une sortie, 
puis grep '^[[:blank:]]*. ALIAS.' pour ne travailler que sur les lignes de commentaire concernées et enfin 
`sed s/^.*ALIAS\\.//` pour n'afficher que la partie interessante du commentaire.

```bash
$ git alias
>f : Fetch from a repository and prune any remote-tracking
>st : Shortcut for status
>co : Shortcut for checkout
>cob : Shortcut for checkout creating branch
>del : delete branch even if not merged
>delMb delete all local branches already merged
>aliasC : list all aliases code
>alias : list all aliases
>br : pretty list branches
>ci : Commit with message
>cia Commit with message and addition of current changes
>ciap : Commit with message and addition of current changes and push to origin remote
>save : Create a save commit with current modifications creating default message
>undo : Back to previous commit with changes left in working area
>clear : Back to previous commit with changes removed
>lg : Pretty log tree
>done : push to origine setting upstream
>rank : rank committers
>upd : pull but replace standard pull with checkout rebase
>updS : stash then update then stash pop
```

## Des raccourcis

### Pour afficher ...

#### Le statut

```
[alias]
    # ALIAS.st : Shortcut for status
    st = status
```
Je dois vraiment le commenter celui-là ? C'est un simple shortcut il y en aura d'autre plus loin commit = ci, checkout 
= co. Parce que bon hein un... un bon codeur se doit d'être une feignasse selon moi.

#### Les branches
```
[alias]
    # ALIAS.br : pretty list branches
    br = branch --format='%(HEAD) %(color:yellow)%(refname:short)%(color:reset) - %(contents:subject) %(color:green)(%(committerdate:relative)) [%(authorname)]' --sort=-committerdate
```

De belles branches colorées et triées à partir du commit le plus récent. Explication du formatage de gauche à gauche aà droite
* `%(HEAD) ` c'est la colonne d'affichage (par une étoile) du HEAD actuel
* `%(color:yellow)` le champs suivant s'affiche en jaune
* `%(refname:short)` le nom de la reference (ici la branche)
* `%(color:reset)` on repasse à la couleur par défaut
* `%(contents:subject)` le commentaire du commit le plus récent
* `%(color:green)` on passe au vert
* `(%(committerdate:relative))` affichage de la date relative entre parenthèse (il y a 2 jours, une semaine, ...)
* `[%(authorname)]` affichage de l'auteur du commit entre crochet

#### Un zolie log comme dans ton ihm

```
[alias]
    # ALIAS.lg : Pretty compact logs
    lg = log --graph --decorate --pretty=format:'%C(yellow)%h %Cred%cr %Cblue(%an)%C(cyan)%d%Creset %s' --abbrev-commit
    # ALIAS.lgg : Pretty compact log graph
    lgg = lg --all
```

De beaux commits colorés pour le premier et pour le deuxieme les branches hors HEAD en plus.

Explication du formatage de gauche à gauche à droite : 
* %C(yellow)%h on passe au jaune pour afficher le hash du commit
* %Cred%cr au rouge pour afficher la date relative du commit
* %Cblue(%an) au bleu pour l'autheur
* %C(cyan)%d au cyan pour les references vers le commit s'il y en a 
* %Creset on repasse à la couleur par défaut
* %s on affiche le sujet du commit (message)

#### Les meilleurs commiters

```
# ALIAS.rank : rank committers
        rank = shortlog -sn --no-merges
```

Pour se la friser et voir qui bosse le plus où qui ne bosse pas :-(

### Pour Manipuler les branches ...

#### Raccourci pour checkout *
```
[alias]
    # ALIAS.co : Shortcut for checkout
    co = checkout
```

#### Raccourci de création de branche avec passage à la branche *
```
[alias]
    # ALIAS.cob : Shortcut for checkout creating branch
    cob = checkout -b
```
#### Raccourci de suivi d'une branche distante *

```
# ALIAS.track : track a remote branch
track = checkout --track             
```
Prend en paramètre une branche distante, l'utilise comme upstream pour créer une nouvelle branche locale de même nom 
si elle n'existe pas et checkout dessus.

#### Supprimer une branche *
```
[alias]
    # ALIAS.del : delete branch even if not merged
    del = branch -D
```

Attention cependant cette commande va supprimer ta branche sans vérifier que les commits qui la composent seront
toujours référencés après l'opération. SI ça te fait peur tu peux toujours remplacer `-D` par `-d`

#### Suppression des branches mergés par rapport à la branche local
```
[alias]
    # ALIAS.delMb delete all local branches already merged regarding current branch
    delMb = "!git branch --merged | grep -v '*' | xargs -n 1 git branch -d"
```

L'explication :
* `git branch --merged` va nous permettre de lister les branches mergées avec la branche courante (dont l'historique est 
entierement commun a la branche courante) y compris celle-ci qui est indiqué par un `*`
* `grep -v '*'` va nous permettre d'enlever de la liste les résultats contenant `*`
* `xargs -n 1 git branch -d` applique la commande git branch -d de suppression de branch mergé à chaque ligne du 
résultat précédent, c'est-à-dire la liste des branches mergées avec la branche courante à l'exclusion de la branche courante 
elle-même.

### Pour bien commiter

#### Commiter avec un message *

```
# ALIAS.ci : Commit with message
ci = "!f() { git commit -m \"$1\"; }; f"
```
Declare une fonction pour réaliser le commit puis l'appel avec les paramètres de l'alias en paramètres.

#### Ajouter toutes les modifications courantes et commiter avec un message *

```
# ALIAS.cia Commit with message and addition of current changes
cia = "!f() { git commit -m \"$1\"; }; git add -A && f"
```

Comme la précédente, mais avant l'appel de la fonction, on va ajouter toutes les modifications courante au stage (-A).

#### Finir le boulot

```
# ALIAS.done : push to origine setting upstream
done = !git push -u origin HEAD
```
Un git push qui permet de pousser vers l'origine sans spécifier la branche.

#### Faire un commit de sauvegarde intermédiaire

```
# ALIAS.save : Create a save commit with current modifications creating default message
save="!f() { git commit -am \"Save point at $(date --utc +'%Y%m%d-%H:%M:%S') ; PLEASE REBASE\"; }; git add -A && f"
```

On ajoute tout et on sauvegarde sous la forme d'un commit avec un message prédéfini :

`Save point at 20221105-16:30:27 ; PLEASE REBASE`

Comme te l'indique le message ce genre de commit ne devrait jamais atterrir dans les branches que tu pousses et je les
nettoie avec un rebase interactif.

#### Ajouter toutes les modifications courantes et commiter avec un message et pousser vers l'origine *

```
# ALIAS.ciap : Commit with message and addition of current changes and push to origin remote
ciap = "!f() { git add -A ; git commit -m \"$1\"; git done; }; f"
```

Une combinaison des commandes précédentes lorsque je veux pousser mon commit immédiatement.

### Pour gérer son historique ...

#### Annuler le dernier commit en récupérant les modifications dans le working directory

```
[alias]
    # ALIAS.undo : Back to previous commit with changes left in working area
    undo = reset HEAD^ --mixed
```

#### Annuler le dernier commit de manière définitive

```
[alias]
    # ALIAS.clear : Back to previous commit with changes removed
    clear = reset HEAD^ --hard
```