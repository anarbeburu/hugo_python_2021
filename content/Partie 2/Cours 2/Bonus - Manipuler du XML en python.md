---
title: Bonus. Manipuler du XML en Python
draft: false
weight: 25
---


### XML : eXtensible Markup Language

- Format très général pour structurer des informations dans un fichier texte
- Défini et géré par le W3C (Consortium de standardisation et developpement du Web)
- (X)HTML est un cas particulier de XML
- ~historique ?... à tendance à être remplacé par JSON, YAML, bases SQL / noSQL, ...


Quelques exemple courants:

- un XML assez standard:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<data>
    <apps>
        <app name="mailman" state="working" level="5" />
        <app name="wekan" state="inprogress" level="3" />
        <app name="nextcloud" state="working" level="7" />
        <app name="wordpress" state="working" level="7" />
        <app name="plex" state="notworking" />
    </apps>
</data>
```

- du html:

```html
<html>
    <head>
        <meta charset="UTF-8">
        <link rel="stylesheet" href="style.css">
        <script src="lib.js"></script>
    </head>
    <body>
        <p class="text-bold">Un morceau de texte</p>
        <p class="text-emph">Un autre paragraphe</p>
    </body>
</html>
```

- Un documents LibreOffice

```xml
<?xml version="1.0" encoding="UTF-8"?>
<office:document-content office:version="1.2">
    [...]
    <office:body>
        <office:text>
            <text:p text:style-name="P1">
            Hello <text:span text:style-name="T1">world!</text:span>
            </text:p>
        </office:text>
    </office:body>
</office:document-content>
```

# Un peu de vocabulaire

```html
<html>
    <head>
        <meta charset="UTF-8">
        <link rel="stylesheet" href="style.css">
        <script src="lib.js"></script>
    </head>
    <body>
        <p class="text-bold">Un morceau de texte</p>
        <p class="text-emph">Un autre paragraphe</p>
    </body>
</html>
```

- **Balises** : ouvrantes et fermantes, e.g. `<p class="red">` et `</p>`
- **Attributs** : par exemple `class="red"`
- **Noeuds éléments** : caractérisés et délimités par des balises : `head`, `body`, `script`, `p`, ...
   - peut contenir d'autres noeuds (éléments, texte, ...) et donc créer un *arbre*
- **Noeuds texte** : e.g. `"Un morceau de texte"`


## XML : Approche DOM v.s. SAX

### DOM : Document Object Model

- Lecture et chargement initial de tout le document (peut être lourd pour les gros documents !)
- Puis accès à tous les noeuds de l'arbre (~AST)
- Approche classique et répandue (c.f. Javascript)

### SAX : Simple API for XML

- Lecture et analyse "au fur et à mesure"
- Pas besoin de tout charger en mémoire
- Adaptée aux gros documents

### ElementTree

- Best of both world ? (Mais moins de fonctionnalités avancées)
- Simple à utiliser comme DOM, peut être aussi rapide que SAX


## Quelques exemples de librairies

- `xml.tree.ElementTree` : ElementTree API, inclue de base dans Python
- `lxml` : Très complète, support de nombreux standard
- `BeautifulSoup` : Interface simple, conçu pour parser du HTML contenant des erreurs
- (et pleins d'autres ...)


```html
<html>
    <head>
        <meta charset="UTF-8">
        <link rel="stylesheet" href="style.css">
        <script src="lib.js"></script>
    </head>
    <body>
        <p class="text-bold">Un morceau de texte</p>
        <p class="text-emph">Un autre paragraphe</p>
    </body>
</html>
```

---

## `xml.tree.ElementTree`

### Parser / lire

```python
from xml.etree import ElementTree as ET

root = ET.parse("monfichier.html")
body = root.find("body")

print(body[0])        # --> <Element 'p' at 0x12345>
print(body[0].tag)    # --> p
print(body[0].attrib) # --> {'class': 'text-bold'}
print(body[0].text)   # --> Un morceau de texte
print(list(body[0]))  # --> []  (pas d'elements fils)

# Trouver tout les <p> dans le body
tous_les_p = body.findall("p")
```

### Construire / ecrire

```python
from xml.etree import ElementTree as ET

root = ET.parse("monfichier.html")
body = root.find("body")

# Ajout d'un nouvel element dans <body>
# <p class="text-underline" id="new">Du texte en plus</p>

nouveau_p = ET.SubElement(body, "p", clas="text-underline", id="new")

root.write("monfichier_2.xml")
```

## Parsing itératif avec `lxml.etree.iterparse`

- `ET.parse("fichier.xml")` explose la RAM pour les gros fichiers.
- Besoin d'une technique plus efficace
- `iterparse` fourni un iterateur pour parser au fur et à mesure, qui plus est seulement sur des tags specifiques

```python
from lxml import etree

iterator = etree.iterparse("fichier.xml", tag="p")

for event, element in iterator:
    # [...] traiter l'element
```

Ça consomme toujours de la RAM ... besoin d'un trick en + ... c.f. `https://stackoverflow.com/questions/12160418`

```python
from lxml import etree

def clear_elem_and_ancestors(elem):
    elem.clear()
    for ancestor in elem.xpath('ancestor-or-self::*'):
        while ancestor.getprevious() is not None:
            del ancestor.getparent()[0]

iterator = etree.iterparse("fichier.xml", tag="p")

for event, element in iterator:
    # [...] traiter l'element
    clear_elem_and_ancestors(element)
```
