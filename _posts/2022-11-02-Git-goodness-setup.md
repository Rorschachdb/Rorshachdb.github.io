---
title:  "Git goodness"
subtitle : "Les aliases"
date:   2022-10-30 23:03:36 +0100
tags: [computing, informatique, git, goodness]
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
# Un peu de SSH

## Mise en place d'une clef

## Avoir l'agent ssh toujours sous la main

## Enregistrer sa clef

### Sous GitHub

### Sous GitLab

### Avec un git ssh

# Et Après
Eh bien pourquoi pas relire mon merveilleux (et je pèse mes mots) article sur les Aliases Git