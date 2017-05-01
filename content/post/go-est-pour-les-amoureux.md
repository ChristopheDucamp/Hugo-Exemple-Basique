+++
title = "Avant-Goût Template (Hu)go"
description = "Bref aperçu sur l'utilisation des templates Go."
tags = [
    "go",
    "golang",
    "templates",
    "themes",
    "development",
]
date = "2017-03-22"
categories = [
    "Development",
    "golang",
]
menu = "main"
+++

Hugo utilise l'excellente bibliothèque de [template/html][gohtmltemplate] [go][] pour son moteur de templates. Une motorisation extrêmemement légère qui fournit une petite quantité de logique. De notre point de vue c'est juste la bonne quantité de logique pour pouvoir créer un bon site web statique. Si vous avez utilisé d'autres systèmes de templates avec différents langages ou frameworks, vous trouverez beaucoup de similarités dans les templates go.

Ce document est un bref aperçu sur l'utilisation des templates go. La [doc go][gohtmltemplate] fournit plus de détails.

## Introduction aux Templates Go

Les modèles Go fournissent un langage de modèle extrêmement simple. Il adhère à la croyance que seul le plus élémentaire de la logique appartient au template ou à la couche vue.
Une conséquence de cette simplicité est que les modèles go analysent très rapidement.
Une caractéristique unique des modèles Go est qu'ils sont conscients du contenu. Variables et contenu seront assainis selon le contexte de l'endroit où ils sont utilisés. Plus
de détails peuvent être trouvés dans [go docs][gohtmltemplate].

## Syntaxe Basique

Les templates Go lang sont des fichiers html avec l'ajout de variables et fonctions. 

**Les variables et fonctions Go sont accessibles dans {{ }}**

Accéder à une variable prédéfinie "foo" :

    {{ foo }}

**Les paramètres sont séparés en utilisant des espaces**

Appeler la fonction add avec un input de 1, 2 :

    {{ add 1 2 }}

**Les méthodes et champs sont accédés via une notation point**

Accéder à la Page Paramètre "bar"

    {{ .Params.bar }}

**Les parenthèses peuvent être utilisées pour grouper ensemble des items**

    {{ if or (isset .Params "alt") (isset .Params "caption") }} Caption {{ end }}


## Variables

Chaque template go a une struct (object) rendue disponible pour lui. Dans hugo, chaque template est passé soit par une page ou une stuct de noeud selon le type de page que vous produisez. Plus de détails disponibles sur la page 
[variables](/layout/variables).

Une variable est accédée par référence au nom de la variable.

    <title>{{ .Title }}</title>

Les variables peuvent être aussi définie et référencées.

    {{ $address := "123 Main St."}}
    {{ $address }}


## Fonctions

Le template Go est fourni avec quelques fonctions qui fournissent des fonctionnalités basiques. Le système de template go fournit aussi un mécanisme pour applications afin d'augmenter les fonctions disponibles. Les [fonctions Hugo template](/layout/functions) fournissent quelques fonctionnalités supplementaires que nous croyons utiles pour la construction de sites web. Les fonctions sont appelées par les paramètres requis séparés par des espaces. Les fonctions de template ne peuvent être ajoutées sans recompiler Hugo.

**Exemple :**

    {{ add 1 2 }}

## Includes

Au moment d'inclure un autre template vous lui passerez la data qu'il pourra accéder. Pour passer le contexte en cours, souvenez-vous d'inclure un point précédé d'un espace. La situation des templates démarrera toujours par le répertoire /layout/ dans Hugo.

**Exemple :**

    {{ template "chrome/header.html" . }}


## Logique

Les templates Go fournissent l'itération la plus basique et la logique conditionnelle.

### Itération 

Tout comme dans go, les templates go font un usage intense du classement pour itérer sur une carte, une liste ou un slice. Voici différents exemples d'utilisation du classement. 

**Exemple 1 : Utiliser le Contexte**

    {{ range array }} 
        {{ . }}
    {{ end }}

**Exemple 2 : Déclarer une valeur de nom de variable**

    {{range $element := array}} 
        {{ $element }} 
    {{ end }}

**Exemple 3 : Déclarer une clé et une valeur de nom de variable**

    {{range $index, $element := array}}
        {{ $index }} 
        {{ $element }} 
    {{ end }}

### Conditionnels 

If, else, with, or, & and fournissent le framework pour gérer la logique conditionnelle dans les Templates Go. Comme range, chaque déclaration est fermée par `end`.


Les Templates Go traitent les valeurs suivantes comme false : 

* false
* 0 
* toute array, slice, map, ou chaîne de longueur zéro

**Exemple 1 : If**

    {{ if isset .Params "title" }}<h4>{{ index .Params "title" }}</h4>{{ end }}

**Exemple 2 : If -> Else** 

    {{ if isset .Params "alt" }} 
        {{ index .Params "alt" }}
    {{else}}
        {{ index .Params "caption" }}
    {{ end }}

**Exemple 3 : And & Or**

    {{ if and (or (isset .Params "title") (isset .Params "caption")) (isset .Params "attr")}}

**Exemple 4 : With**

Un moyen alternatif d'écrire "if" et de référencer ensuite la même valeur est d'utiliser à la place "with". With relie le contexte `.` dans sa portée, et ignore le bloc si la variable est absente.

Le premier exemple du dessus pourrait être ainsi simplifié : 

    {{ with .Params.title }}<h4>{{ . }}</h4>{{ end }}

**Exrmple 5 : If -> Else If** 

    {{ if isset .Params "alt" }} 
        {{ index .Params "alt" }}
    {{ else if isset .Params "caption" }}
        {{ index .Params "caption" }}
    {{ end }}

## Pipes

L'un des composants les plus puissants de go templates est la capacité à empiler des actions l'une après l'autre. Cela se fait en utilisant des pipes. Emprunt à Unix, le concept est simple, la production de chaque pipeline devient l'input du pipe suivant.

En raison de la syntaxe très simple des templates go, le pipe est essentiel pour être en mesure de chaîner les appels de fonction. Une limitation des pipes est qu'ils ne peuvent travailler qu'avec une seule valeur et que cette valeur devient le dernier paramètre du pipeline suivant.

Quelques exemples simples devraient aider à transmettre comment utiliser le pipe.

**Exemple 1 :**

    {{ if eq 1 1 }} Same {{ end }}

est le même que 

    {{ eq 1 1 | if }} Same {{ end }}

Cela peut sembler étrange de placer le if à la fin, mais cela fournit une bonne illustration sur la façon d'utiliser les pipes.

**Exemple 2 :**

    {{ index .Params "disqus_url" | html }}

Accède au paramtère de page appelé "disqus_url" et échappe le  HTML.

**Exemple 3 :**

    {{ if or (or (isset .Params "title") (isset .Params "caption")) (isset .Params "attr")}}
    Trucs Ici
    {{ end }}

Pourrait être ré-écrit sous 

    {{  isset .Params "caption" | or isset .Params "title" | or isset .Params "attr" | if }}
    Trucs Ici
    {{ end }}


## Contexte (aka. the dot)

Le concept le plus facilement négligé à comprendre au sujet des modèles d'aller est que {{ . }} fait toujours référence au contexte actuel. Dans le niveau supérieur de votre modèle, ce sera le jeu de données mis à sa disposition. À l'intérieur d'une itération, il aura la valeur de l'élément courant. Lorsque l'intérieur d'une boucle du contexte a changé. . ne se référera plus aux données disponibles sur toute la page. Si vous avez besoin d'accéder à cela à partir de la boucle, vous voudrez probablement le définir en une variable au lieu de dépendre du contexte.

**Exemple :**

      {{ $title := .Site.Title }}
      {{ range .Params.tags }}
        <li> <a href="{{ $baseurl }}/tags/{{ . | urlize }}">{{ . }}</a> - {{ $title }} </li>
      {{ end }}

Remarquez qu'une fois que nous avons entré la boucle la valeur de {{ . }} a changé. Nous avons défini une variable à l'extérieur de la boucle, nous avons donc accès à elle à partir de l'intérieur
la boucle.

# Paramètres Hugo

Hugo fournit l'option de passer les valeur vers le langage de template à travers la configutration du site (pour les valeurs globales du site), ou à travers les méta-données de chaque pièces spécifique de contenu. Vous pouvez définir toutes les valeurs de n'importe quel type (supportées par votre format de configuration/front matter) et les utiliser comme vous voudrez dans vos templates.
## Utiliser les Paramètres de Contenu (page)

Dans chaque morceau de contenu vous pouvez fournir des variables à utiliser par les templates. Ceci se produit dans le [front matter](/content/front-matter). 

Un exemple de cela est utilisé dans ce site de documentation. La plupart des pages tirent partie An example of this is used in d'avoir une table des matières fournie. Parfois la table des matières ne fait simplement pas beaucoup de sens. Nous avons défini une variable dans notre front matter de quelques pages pour la désactiver de l'affichage.

Voici l'exemple de front matter :

```
---
title: "Permalinks"
date: "2013-11-18"
aliases:
  - "/doc/permalinks/"
groups: ["extras"]
groups_weight: 30
notoc: true
---
```

Voici la code correspondant à l'intérieur du template : 

      {{ if not .Params.notoc }}
        <div id="toc" class="well col-md-4 col-sm-6">
        {{ .TableOfContents }}
        </div>
      {{ end }}



## Utiliser les Paramètres de Site (config)

Dans votre fichier de configuration au niveau le plus haut (par ex. `config.yaml`) vous pouvez définir les paramètres du site, qui sont les valeurs qui seront disponibles pour vous dans chrome.

Par exemple, vous pourriez déclarer : 

```yaml
params:
  CopyrightHTML: "Copyright &#xA9; 2013 John Doe. All Rights Reserved."
  TwitterUser: "spf13"
  SidebarRecentLimit: 5
```

Dans un layout de footer, vous pourriez alors déclarer un  `<footer>` qui est uniquement fourni si le paramètre  `CopyrightHTML` est fourni, et s'il est donné, vous le déclareriez comme étant HTML-safe, de manière à ce que l'entité HTML ne soit pas échappée. Ceci vous permettrait de mettre à jour aisément votre fichier de configuration au niveau le plus haut à chaque premier janvier, au lieu de chasser dans vos templates.

```
{{if .Site.Params.CopyrightHTML}}<footer>
<div class="text-center">{{.Site.Params.CopyrightHTML | safeHtml}}</div>
</footer>{{end}}
```

Une autre façon d'écrire le "if", puis de référencer la même valeur est d'utiliser "with" à la place. Avec le nouveau contexte  `.` dans sa portée, il ignore le bloc si la variable est absente :

```
{{with .Site.Params.TwitterUser}}<span class="twitter">
<a href="https://twitter.com/{{.}}" rel="author">
<img src="/images/twitter.png" width="48" height="48" title="Twitter: {{.}}"
 alt="Twitter"></a>
</span>{{end}}
```

Enfin, si vous souhaitez extraire des "constantes magiques" de vos mises en page, vous pouvez le faire, comme dans cet exemple :

```
<nav class="recent">
  <h1>Posts Récents</h1>
  <ul>{{range first .Site.Params.SidebarRecentLimit .Site.Recent}}
    <li><a href="{{.RelPermalink}}">{{.Title}}</a></li>
  {{end}}</ul>
</nav>
```


[go]: http://golang.org/
[gohtmltemplate]:http://golang.org/pkg/html/template
