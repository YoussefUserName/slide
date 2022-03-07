# L'objectif du projet 

Nous permettre de trouver un parfum selon:

- Notre budget
- Notre goût (en fonction de nos notes de parfum préférées)
- La manière de trier nos parfums (ordre alphabétique, par recommandation des consommateurs...)

## Packages utilisés 


```
import requests
import pandas as pd
from bs4 import BeautifulSoup as bs4

```

- Requests : permet d'envoyer Python faire une requête sur une page HTTP
- Pandas : manipuler facilement les dataframe et données à analyser
- BeautifulSoup : permet le web scrapping par l'extraction des données (HTML et XML) 

---

#Procédures

Pour ce programme nous allons nous baser sur des fonctions qui vont appeler différentes fonctionnalités:

Code fonction ```saisie_nombre_de_pages_resultat()```:

```
urls = []
def saisie_nombre_de_pages_resultat():
    nb_pages = 0
    while True:
        try:
            nb_pages = int(input("Entrez le nombre de pages de résultat souhaité (minimum 1): "))
        except ValueError:
            print("Veuillez insérer un nombre valide")
            continue
```

Première commande qui va générer un input nous demandant d'entrer le nombre de pages que l'on souhaite parcourir pour notre recherche de parfum

---

# Procédures 

```
else:
    if nb_pages <= 0:
        print("Veuillez insérer un nombre supérieur à 0")
            continue
        else:
            break
    return nb_pages + 1
```
    
    
Pourquoi ```nb_pages + 1``` ?: Pour pouvoir faire du **scrapping sur toutes les pages** nous devions commencer à la page 2, car c'est là que le numéro de la page ressort, sur la page 1 le numéro ne ressort pas

---

# Procédures

Code fonction ```saisie_type_tri_resultat()```:
```
def saisie_type_tri_resultat():
    type_tri = 0
    question = "Quel ordre de tri souhaitez-vous?\n"
    question += " - Par recommandation    : Tapez 1\n"
    question += " - Par meilleures ventes : Tapez 9\n"
    question += " - Par prix croissant    : Tapez 3\n"
    question += " - Par prix décroissant  : Tapez 5\n"
    question += " - Par ordre alphabétique: Tapez 2\n"
    question += "Votre choix: "
```

---
#Procédures

```
    while True:
        try:
            type_tri = int(input(question))
        except ValueError:
            print("Veuillez insérer un nombre valide")
            continue
        else:
            if type_tri not in [1, 2, 3, 5, 9]:
                print("Veuillez insérer un nombre compris dans la liste suivante : [1, 2, 3, 5, 9]")
                continue
            else:
                break
    return type_tri
```

---
#Procédures
```
def saisie_type_parfum_resultat():
    type_parfum = 0
    question = "Quel type de parfum souhaitez-vous?\n"
    question += " - Oriental  : Tapez 0\n"
    question += " - Vert      : Tapez 1\n"
    question += " - Musqué    : Tapez 2\n"
    question += " - Héspéridé : Tapez 3\n"
    question += " - Gourmand  : Tapez 4\n"
    question += " - Fruité    : Tapez 5\n"
    question += " - Fougère   : Tapez 6\n"
    question += " - Cuiré     : Tapez 7\n"
    question += " - Chypré    : Tapez 8\n"
    question += " - Boisé     : Tapez 9\n"
    question += " - Aquatique : Tapez 10\n"
    question += " - Aldéhydé  : Tapez 11\n"
    question += " - N'importe : Tapez -1\n"
    question += "Votre choix: "

    while True:
        try:
            type_parfum = int(input(question))
        except ValueError:
            print("Veuillez insérer un nombre valide")
            continue
        else:
            if type_parfum not in [-1, 0, 1, 2, 3, 4, 5, 7, 8, 9, 10, 11]:
                print("Veuillez insérer un nombre compris entre [-1;11]")
                continue
            else:
                break
    return type_parfum
```
---
# Procédures
##Rassemblement des fonctions dans un dictionnaire 

```
def saisie_utilisateur():
    return {
        'nb_pages': saisie_nombre_de_pages_resultat(),
        'type_tri': saisie_type_tri_resultat(),
        'type_parfum': saisie_type_parfum_resultat()
    }
```

Ceci va nous permettre de les lancer toutes en même temps lors de l'exécution du programme complet

- Fonctions pour rassembler le type de produit ainsi que son prix:
```
def verification_champ_type_produit(type_produit):
    return type_produit.text if type_produit is not None else "NaN"

def verification_champ_prix(prix):
    if prix is not None:
        if "de" in prix.text:
            begin = prix.text.index("de") + 2
        else:
            begin = 0
        end = prix.text.index("/")
        prix = prix.text[begin:end].replace(",", ".").strip()
    else:
        prix = "NaN"

    return prix
```
- ```begin = prix.text.index("de") + 2``` le "+2" va permettre de dire au programme de scrapper à partir du deuxième caractère (SCREEN À METTRE)
 
--- 
# Procédures

- De même ici, pour la collection et la notation du parfum par le consommateur:

```def verification_champ_collection(collection):
    return collection.find("strong").text if collection is not None else "NaN"

def verification_champ_note(note):
    if len(note) > 10:
        begin = note.index("width") + 6
        note = note[begin:]
        end = note.index("px")
        note = note[:end]
        note = float(note) / 14
    else:
        note = "NaN"
    return note
```
- On va noter ici les ```note = float(note) / 14``` : les notes, dans le code html du site sont sur 5 étoiles,
la 5ème étoile étant sur 70. Donc **1/5 représente 14**. C'est pourquoi **on divise par 14** la note de fin **pour que dans nos résultats finaux nous ayons une note sur 5**.

- ```begin = note.index("width") + 6``` va permettre au code de comprendre qu'il doit commencer à scrapper à partir du 6ième caractère, car on ne peut pas transformer un **string** (chaîne de caractère) en **float** (nombre entier)

(SCREEN À METTRE)

# Le Web Scraping



# La principale difficulté rencontrée 

Les nombreuses mise à jour et les changements de structure du site : remplacement des items <li> par les items <div>


