+++
title = "Démarrer avec Hugo"
description = ""
tags = [
    "go",
    "golang",
    "hugo",
    "development",
]
date = "2017-04-29"
categories = [
    "Development",
    "golang",
]
menu = "main"
+++

## Étape 1. Installer Hugo

Allez sur [hugo releases](https://github.com/spf13/hugo/releases) et téléchargez la version appropriée pour votre OS et votre architecture.

Sauvegardez-la à un endroit spécifique car nous allons l'utiliser pour la prochaine étape.

Vous trouverez plus d'instructions disponibles sur [installer hugo](/overview/installing/)

## Étape 2. Construire les Docs

Hugo a son propre exemple de site qui est aussi le site de documentation que vous lisez ici.

Suivez les étapes :

 1. Clonez le [dépôt hugo](http://github.com/spf13/hugo)
 2. Allez dans le dépôt
 3. Lancez Hugo en mode serveur et construisez les docs
 4. Ouvrez votre navigateur sur http://localhost:1313

Commandes correspondantes :

    git clone https://github.com/spf13/hugo
    cd hugo
    /path/to/where/you/installed/hugo server --source=./docs
    > 29 pages created
    > 0 tags index created
    > in 27 ms
    > Web Server is available at http://localhost:1313
    > Press ctrl+c to stop

Une fois rendu à cet endroit, suivez le reste de cette page sur votre build local.

## Étape 3. Changez le site des docs

Arrêtez le processus Hugo en pressant ctrl+c dans la fenêtre de terminal.

Maintenant, nous allons relancer Hugo, mais cette fois-ci avec Hugo en mode watch.

    /path/to/hugo/from/step/1/hugo server --source=./docs --watch
    > 29 pages created
    > 0 tags index created
    > in 27 ms
    > Web Server is available at http://localhost:1313
    > Watching for changes in /Users/spf13/Code/hugo/docs/content
    > Press ctrl+c to stop


Ouvrez votre [éditeur favori](http://vim.spf13.com) et modifiez l'une des pages de contenus sources. Que penser de modifier ce fichier pour *corriger la typo*.

Les fichiers de contenu se trouvent dans `docs/content/`. À moins qu'autre chose ne soit spécifié, les fichiers sont situés dans le même endroit relatif que l'url, dans notre cas 
`docs/content/overview/quickstart.md`.

Changez et sauvegardez ce fichier... Notez ce qui se passe dans votre terminal.

    > Change detected, rebuilding site

    > 29 pages created
    > 0 tags index created
    > in 26 ms

Rafraîchissez le navigateur et observez que l'erreur de typo est désormais corrigée.

Notez la vitesse à laquelle l'opération s'est réalisée. Essayez de rafraîchir le site avant qu'il n'ait fini de construire. Disposer d'une rétroaction presque instantanée vous permet de disposer de toute votre créativité sans avoir besoin d'attendre de long builds.

## Étape 4. Amusez-vous

Le meilleur moyen d'apprendre quelque chose c'est de s'amuser avec.
