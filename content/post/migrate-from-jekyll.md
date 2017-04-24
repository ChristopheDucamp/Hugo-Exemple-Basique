---
date: 2014-03-10
linktitle: Migrer en provenance de Jekyll
menu:
  main:
    parent: tutorials
prev: /tutorials/mathjax
title: Migrer de Jekyll vers Hugo
weight: 10
---

## Migrer le contenu statique vers `static`
Il y a une règle dans Jekyll faisant que tout répertoire ne démarrant pas par un `_` sera copié comme tel vers l'output du `_site`. Hugo conserve tout le contenu statique sous `static`. Vous devriez par conséquent migrer tout là-bas.
Avec Jekyll, quelque chose qui ressemblait à 

    ▾ <root>/
        ▾ images/
            logo.png

devrait devenir : 

    ▾ <root>/
        ▾ static/
            ▾ images/
                logo.png

En outre, vous voudrez que tous les fichiers qui devraient résider à la racine (tel que `CNAME`) soient migrés vers  `static`.

## Créez votre fichier de configuration Hugo
Hugo sait lire votre configuration JSON, YAML ou TOML. Hugo supporte aussi les paramètres personnalisés de configuration. Regardez la [documentation de configuration Hugo](/overview/configuration/) pour les détails.

## Réglez votre répertoire de configuration de publication sur `_site`
La valeur par défaut est pour Jekyll de publier vers `_site` et pour Hugo de publier vers `public`. Si comme moi, vous avez  [`_site` mappé vers un sous-module git sur la branche `gh-pages`](http://blog.blindgaenger.net/generate_github_pages_in_a_submodule.html), vous voudrez faire l'une de ces deux alternatives : 

1. Changez votre sous-module to point to map `gh-pages` to public instead of `_site` (recommended).

        git submodule deinit _site
        git rm _site
        git submodule add -b gh-pages git@github.com:your-username/your-repo.git public

2. Ou, modifiez la configuration Hugo pour utiliser `_site` au lieu de `public`.

        {
            ..
            "publishdir": "_site",
            ..
        }

## Convertir les templates Jekyll en templates Hugo

C'est la majeure partie du travail ici. La documentation est votre amie. Vous devriez vous référer à la [Documentation de Jekyll] (http://jekyllrb.com/docs/templates/) si vous avez besoin de rafraîchir votre mémoire sur la façon dont vous avez construit votre blog et [le modèle de Hugo](/layout/templates/) pour apprendre la façon de faire d'Hugo.

Pour information, la conversion de mes templates pour [heyitsalex.net](http://heyitsalex.net/) ne m'a pris pas plus de quelques heures.

## Convertir les plugins Jekyll en shortcodes Hugo
Jekyll a des [plugins](http://jekyllrb.com/docs/plugins/) ; Hugo a des [shortcodes](/doc/shortcodes/). It's fairly trivial to do a port.

### Implémentation
As an example, I was using a custom [`image_tag`](https://github.com/alexandre-normand/alexandre-normand/blob/74bb12036a71334fdb7dba84e073382fc06908ec/_plugins/image_tag.rb) plugin to generate figures with caption when running Jekyll. As I read about shortcodes, I found Hugo had a nice built-in shortcode that does exactly the same thing.

Le plugin de Jekyll :

    module Jekyll
      class ImageTag < Liquid::Tag
        @url = nil
        @caption = nil
        @class = nil
        @link = nil
        // Patterns
        IMAGE_URL_WITH_CLASS_AND_CAPTION =
        IMAGE_URL_WITH_CLASS_AND_CAPTION_AND_LINK = /(\w+)(\s+)((https?:\/\/|\/)(\S+))(\s+)"(.*?)"(\s+)->((https?:\/\/|\/)(\S+))(\s*)/i
        IMAGE_URL_WITH_CAPTION = /((https?:\/\/|\/)(\S+))(\s+)"(.*?)"/i
        IMAGE_URL_WITH_CLASS = /(\w+)(\s+)((https?:\/\/|\/)(\S+))/i
        IMAGE_URL = /((https?:\/\/|\/)(\S+))/i
        def initialize(tag_name, markup, tokens)
          super
          if markup =~ IMAGE_URL_WITH_CLASS_AND_CAPTION_AND_LINK
            @class   = $1
            @url     = $3
            @caption = $7
            @link = $9
          elsif markup =~ IMAGE_URL_WITH_CLASS_AND_CAPTION
            @class   = $1
            @url     = $3
            @caption = $7
          elsif markup =~ IMAGE_URL_WITH_CAPTION
            @url     = $1
            @caption = $5
          elsif markup =~ IMAGE_URL_WITH_CLASS
            @class = $1
            @url   = $3
          elsif markup =~ IMAGE_URL
            @url = $1
          end
        end
        def render(context)
          if @class
            source = "<figure class='#{@class}'>"
          else
            source = "<figure>"
          end
          if @link
            source += "<a href=\"#{@link}\">"
          end
          source += "<img src=\"#{@url}\">"
          if @link
            source += "</a>"
          end
          source += "<figcaption>#{@caption}</figcaption>" if @caption
          source += "</figure>"
          source
        end
      end
    end
    Liquid::Template.register_tag('image', Jekyll::ImageTag)

is written as this Hugo shortcode:

    <!-- image -->
    <figure {{ with .Get "class" }}class="{{.}}"{{ end }}>
        {{ with .Get "link"}}<a href="{{.}}">{{ end }}
            <img src="{{ .Get "src" }}" {{ if or (.Get "alt") (.Get "caption") }}alt="{{ with .Get "alt"}}{{.}}{{else}}{{ .Get "caption" }}{{ end }}"{{ end }} />
        {{ if .Get "link"}}</a>{{ end }}
        {{ if or (or (.Get "title") (.Get "caption")) (.Get "attr")}}
        <figcaption>{{ if isset .Params "title" }}
            {{ .Get "title" }}{{ end }}
            {{ if or (.Get "caption") (.Get "attr")}}<p>
            {{ .Get "caption" }}
            {{ with .Get "attrlink"}}<a href="{{.}}"> {{ end }}
                {{ .Get "attr" }}
            {{ if .Get "attrlink"}}</a> {{ end }}
            </p> {{ end }}
        </figcaption>
        {{ end }}
    </figure>
    <!-- image -->

### Usage
I simply changed:

    {% image full http://farm5.staticflickr.com/4136/4829260124_57712e570a_o_d.jpg "One of my favorite touristy-type photos. I secretly waited for the good light while we were "having fun" and took this. Only regret: a stupid pole in the top-left corner of the frame I had to clumsily get rid of at post-processing." ->http://www.flickr.com/photos/alexnormand/4829260124/in/set-72157624547713078/ %}

to this (this example uses a slightly extended version named `fig`, different than the built-in `figure`):

    {{%/* fig class="full" src="http://farm5.staticflickr.com/4136/4829260124_57712e570a_o_d.jpg" title="One of my favorite touristy-type photos. I secretly waited for the good light while we were having fun and took this. Only regret: a stupid pole in the top-left corner of the frame I had to clumsily get rid of at post-processing." link="http://www.flickr.com/photos/alexnormand/4829260124/in/set-72157624547713078/" */%}}

As a bonus, the shortcode named parameters are, arguably, more readable.

## Touches Finales
### Réparer le contenu
Depending on the amount of customization that was done with each post with Jekyll, this step will require more or less effort. There are no hard and fast rules here except that `hugo server --watch` is your friend. Test your changes and fix errors as needed.

### Nettoyage
You'll want to remove the Jekyll configuration at this point. If you have anything else that isn't used, delete it.

## Un exemple pratique dans une diff
[Hey, it's Alex](http://heyitsalex.net/) was migrated in less than a _father-with-kids day_ from Jekyll to Hugo. You can see all the changes (and screw-ups) by looking at this [diff](https://github.com/alexandre-normand/alexandre-normand/compare/869d69435bd2665c3fbf5b5c78d4c22759d7613a...b7f6605b1265e83b4b81495423294208cc74d610).
