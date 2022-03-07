
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

```{r, echo = FALSE, out.width="50%"}
knitr::include_graphics("site.png", error = FALSE)
```
```{r, echo = FALSE, out.width="50%"}
knitr::include_graphics("site2.png", error = FALSE)
```

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
```begin = prix.text.index("de") + 2``` le "+2" va permettre de dire au programme de scrapper à partir du deuxième caractère 

```{r, echo = FALSE, out.width="50%"}
knitr::include_graphics("price.png", error = FALSE)
```
--- 
# Procédures

- De même ici, pour la collection et la notation du parfum par le consommateur:

```
def verification_champ_collection(collection):
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

```{r, echo = FALSE, out.width="50%"}
knitr::include_graphics("note.png", error = FALSE)
```

--- 
#Procédures
```
def creation_fichier_csv(produits_parfum_csv):
    df = pd.DataFrame.from_records([{
        "marque": "Marque",
        "collection": "Collection",
        "type de produit": "Type de produit",
        "prix pour 100 ml": "Prix pour 100 ml",
        "note": "Note"
    }])
    df.to_csv(produits_parfum_csv, mode='w+', index=False, header=False)


def ajout_parfum_csv(collection, marque, note, produits_parfum_csv, type_produit, prix):
    df = pd.DataFrame.from_records([{
        "marque": marque,
        "collection": collection,
        "type de produit": type_produit,
        "prix pour 100 ml": prix,
        "note": note
    }])
    df.to_csv(produits_parfum_csv, mode='a', index=False, header=False)
```

- Ces deux fonctions se répondent et vont nous permettre d'organiser nos données dans un dataframe pour que ce soit lisible et compréhensible

---

# Le web scrapping 

Site choisi pour la réalisation de notre projet: **Notino.fr**

```
def get_page(num_page, s, type_parfum, type_plist, type_tri):
    url = "https://www.notino.fr/parfums-femme/?f=" + \
          str(num_page) + '-' + str(type_tri) + "-55544-55545" + str(type_plist[type_parfum])

    res = s.post(url)
    if res.url not in urls:
        urls.append(url)
        soup = bs4(res.text, "lxml")
        return soup.find_all("li", {"class": "item"})

    return None
```
- Pour pouvoir 'scrapper' toutes les pages, nous rajoutons les numéros de pages qui vont être générés par nos fonctions. Ainsi que le type de tri des parfums, et enfin, leurs caractéristiques olfactives

---
# Le web scrapping

```
def traitement_parfums(parfums, produits_parfum_csv):
    for parfum in parfums:
        marque = parfum.find("span", class_="brand").text
        type_produit = verification_champ_type_produit(parfum.find("span", class_="subname"))
        prix = verification_champ_prix(parfum.find("span", class_="unit-price"))
        collection = verification_champ_collection(parfum.find("span", class_="name"))
        note = verification_champ_note(str(parfum.find("span", class_="product-rating")))
        ajout_parfum_csv(collection, marque, note, produits_parfum_csv, type_produit, prix)


def afficher_resutat(produits_parfum_csv):
    df = pd.read_csv(produits_parfum_csv)
    print(df)


def ajout_code_produits_parfum_csv(produits_parfum_csv):
    df = pd.read_csv(produits_parfum_csv)
    df.insert(0, "Code", [i for i in range(0, df.index.stop)])
    df.to_csv(produits_parfum_csv, mode='w+', index=False, header=True)
```
---
# Le web scrapping

```
def programme_principal():
    saisie = saisie_utilisateur()
    nb_pages = saisie['nb_pages']
    type_tri = saisie['type_tri']
    type_parfum = saisie['type_parfum']
    type_plist = [
        -34693, -34696, -57258, -34691, -57259,
        -34694, -57257, -34700, -34699, -34702,
        -34697, -34698, ""
    ]
    produits_parfum_csv = "produits_parfum.csv"
    creation_fichier_csv(produits_parfum_csv)
    s = requests.session()

    for num_page in range(1, nb_pages):
        parfums_page_num_page = get_page(num_page, s, type_parfum, type_plist, type_tri)
        if parfums_page_num_page is not None:
            traitement_parfums(parfums_page_num_page, produits_parfum_csv)
            
    ajout_code_produits_parfum_csv(produits_parfum_csv)
    afficher_resutat(produits_parfum_csv)
```

```{r, echo = FALSE, out.width="50%"}
knitr::include_graphics("url.png", error = FALSE)
```

---

# Lancement du programme 
```
if __name__ == '__main__':
        programme_principal()
```

- la commande ```if __name__ == '__main__'``` permet de contourner la distinction que va faire Python entre le module principal ```__main__``` et le module importé ```__name__```. Ici nous avons des fonctions créées (donc des modules importés) qui doivent fusionner entre elles, il faut alors toutes les rassembler dans le même module principal ```__main__```.
Puis nous exécutons notre programme.

---

**FIN**



