# Constitution du corpus
Dans  cette partie, le programme utilisé pour scraper wikipédia et les résultats de ce scrapping afin de constituer le corpus.

## L'application de scrapping de wikipédia
Pour constituer notre corpus de d'articles wikipédia traitant du covid-19 dans différentes langues nous avons fini par créer notre propre application de scrapping dont voici le programme :

``` python
import csv
import random
import requests
import matplotlib.pyplot as plt
import numpy as np
import matplotlib.dates as mdates
import json
from datetime import datetime
from tkinter import *
from tkinter import messagebox

# Définition du tableau qui récupère les données du scrapping wikipédia
tab_1 =[]

# Importer un fichier csv avec le code ISO 639-1 des langues
file = open('.file\\languages_code.csv', encoding='utf-8')
filecsv = csv.reader(file)

# création d'un tableau pour y mettre les langues et les codes ISO correspondants et d'un second tableau dans lequel avoir une couleur aléatoire pour chaque langue
langues = []
langues_1 = []

# extranction des langues et des codes ISO dans 1 variable avec une couleur aléatoire et ajout de cette variable dans le tableau langues
for line in filecsv:
    color = '#'+''.join([random.choice('0123456789ABCDEF') for j in range(6)])
    langue = (line[1], line[0], color)
    langues.append(langue)
#suppression de la première entrée qui a pour code alpha 2 et en langue anglais pour éviter le doublon avec anglais
del(langues[0])
for langue in langues:
    lang = langue[0]
    langues_1.append(lang)

# Définition de la fonction permettant de choisir une langue dans la liste
def chooselg(event):
    select = liste_langues.get()
    print('vous avez selectionné :', select)
    for lg in langues:
        if select == lg[0]:
            print(f'le code ISO pour {select} est {lg[1]}')

# Définition de la fonction de validation qui va lancer le scrapping wikipédia si un mot a été saisie et si une langue a été sélectionnée
def valid():
    global api
    recherche = champ.get()
    lgRecherche = liste_langues.get()
    if recherche == '' and lgRecherche == '':
        print('Vous devez entrer un mot et selectionner une langue')
        messagebox.showinfo('ERREUR', 'veuillez entrer un mot et selectionner une langue')
    elif lgRecherche == '':
        print('Vous devez selectionner une langue')
        messagebox.showinfo('ERREUR', 'veuillez selectionner une langue')
    elif recherche == '':
        print('Vous devez renseigner un mot')
        messagebox.showinfo('ERREUR', 'veuillez entrer un mot')
    else :
        query = requests.Session()

        parametre_1 = {
            'action': 'query',
            'format': 'json',
            'list': 'search',
            'srlimit': 1000,
            'utf-8': 1,
            'srsearch': recherche
        }
        for lg in langues:
            if lgRecherche == lg[0]:
                api = f'https://{lg[1]}.wikipedia.org/w/api.php'
        requete_1 = query.get(url=api, params=parametre_1)
        resultat = requete_1.json()
        for articles in resultat['query']['search']:
            if recherche.capitalize() in articles['title'] or recherche in articles['title'] or recherche.upper() in \
                    articles['title']:
                titre = articles['title']
                parametre_2 = {
                    "action": "query",
                    "format": "json",
                    "utf-8": 1,
                    "prop": "revisions",
                    "rvlimit": 1,
                    "rvprop": "timestamp",
                    "rvdir": "newer",
                    "titles": titre

                }
                requete_2 = query.get(url=api, params=parametre_2)
                resultat = requete_2.json()
                for keys, values in resultat['query']['pages'].items():
                    information = (values['pageid'], values['title'], values['revisions'][0]['timestamp'], lgRecherche)
                    tab.insert('', 'end', iid=information[0], values=(information[1], information[2], information[3]))

                for lg in langues:
                    if lgRecherche == lg[0]:
                        color_langue = lg[2]
                        informations = [information[1], information[2], information[3], color_langue]
                        tab_1.append(informations)
    print(tab_1)

# Définition de la fonction afficher qui récupère les données du tableau et affiche dans une fenêtre une timeline
def afficher():
    dates = []
    titres = []
    colors = []

    for articles in tab_1:
        dates.append(articles[1].split("T")[0])
        infos= f'{articles[0]} \n {articles[2]} \n {articles[1].split("T")[0]}'
        colors.append(articles[3])
        titres.append(infos)
    dates = [datetime.strptime(d, '%Y-%m-%d') for d in dates]

    niveaux = np.tile([-10, 10, -9, 9, -8, 8, -7, 7, -6, 6, -5, 5,-4, 4, -3, 3, -2, 2, -1, 1],
                     int(np.ceil(len(dates) / 20)))[:len(dates)]

    fig, ax = plt.subplots(figsize=(8.8, 4), constrained_layout=False)
    ax.set(title="Matplotlib release dates")

    # création des lignes verticales rouges pour positionner les annotations selon les dates
    ax.vlines(dates, 0, niveaux, color="tab:red")
    ax.plot(dates, np.zeros_like(dates), "-o",
            color="k", markerfacecolor="w")

    # annotations des lignes verticales
    for d, n, t, c in zip(dates, niveaux, titres, colors):
        ax.annotate(t, xy=(d, n),
                    xytext=(0, np.sign(n) * 0), textcoords="offset points",
                    horizontalalignment="right",
                    verticalalignment="bottom" if n > 0 else "top",
                    fontsize='8',
                    color = '#F9FAFA',
                    bbox=dict(boxstyle='round', facecolor=c)
        )

    # format de l'axe des abcisses avec un interval d'un mois
    ax.xaxis.set_major_locator(mdates.MonthLocator(interval=1))
    ax.xaxis.set_major_formatter(mdates.DateFormatter("%b %Y"))
    plt.setp(ax.get_xticklabels(), rotation=30, ha="right")

    # l'axe des ordonnées et les contours sont retirés
    ax.yaxis.set_visible(False)
    for spine in ["left", "top", "right"]:
        ax.spines[spine].set_visible(False)

    ax.margins(y=0.1)
    plt.show()

# Définition de la fonction de téléchargement au format json en cliquant sur le bouton
def export_json():
    f = open('donnees_wikipedia.json', 'w', encoding='utf-8')
    json.dump(tab_1, f)
    f.close()

def export_csv():
    fichier = open('donnes_wikipedia.csv', 'w', encoding='utf-8')
    for ligne in tab_1:
        str_temp = ','.join(ligne)
        fichier.write(str_temp + '\n')
    fichier.close()

# Définition de la fonction qui permet de supprimer les entrées dans le tableau de l'interface graphique et du de la liste
def clear():
    for entrees in tab.get_children():
        tab.delete(entrees)
    del tab_1[:]

#Définition d'une fonction qui va fermer l'application après avoir demander à l'utilisateur
def quit():
    reponse = messagebox.askyesno('Quitter ?')
    if reponse == 1:
        fenetre.destroy()

fenetre = Tk ()
fenetre.title('Projet Wiki-viz')
width= int(fenetre.winfo_screenwidth())
height = int(fenetre.winfo_screenheight())
dimensions = f'{width}x{height}'
fenetre.geometry(dimensions)
fenetre.minsize(1080,720)


# Définition de l'en-tête avec un titre, un sous-titre et les participants au projet
header = LabelFrame(fenetre, padx=5, pady=5, background='#BBE6FC').pack(fill=X)
titre = Label(header, text='Projet Wiki-viz', font=('Helvetica', 30), background='#BBE6FC').pack(fill=X)
sous_titre = Label(header, text='Projet de fin d\'étude M2 DEFI', font=('Helvetica', 20), background='#BBE6FC').pack(fill=X)
auteurs = Label(header, text='Nolwenn Baudry, Aly Konate, Julien Matteï ,Elvira Mingedi, Salio Toure', font=('Helvetica', 10), background='#BBE6FC').pack(fill=X)

# Définition du champs de recherche et de son label
champ_label = Label(fenetre, text='Mot recherché : ', font='Helvetica')
champ_label.pack()
champ = Entry(fenetre, width=53)
champ.pack()

# Import du module ttk de tkinter pour pouvoir faire une liste et un tableau
from tkinter import ttk
from tkinter.ttk import *

# Définition de la liste des langues et de son label
liste_label = Label(fenetre, text='Langues :', font='Helvetica')
liste_label.pack()
liste_langues = ttk.Combobox(fenetre, width=50, values=sorted(langues_1))
liste_langues.current()
liste_langues.bind('<<ComboboxSelected>>')
liste_langues.pack()

# Définition d'un bouton pour valider le mot recherché et la langue sélectionnée
validation = Button(fenetre, text='Valider', width= 30, command=valid).pack(pady=3)

# Définition du tableau à afficher avec le nom des colonnes
tab = Treeview(fenetre, columns =('Titre', 'Date_de_creation', 'langue'))
tab.column('Titre', width= 500)
tab.heading('Titre', text='Titre')
tab.heading('Date_de_creation', text='Date de création')
tab.heading('langue', text='Langue')
tab['show']='headings'
tab.pack(pady=10)

# Définition d'un bouton pour afiicher pour afficher la timeline
afficher = Button(fenetre, text='Afficher timeline', width= 30, command=afficher).pack(pady=3)

# Définition d'un bouton pour de téléchargement pour télécharger les données au format json
dl_json = Button(fenetre, text='Telecharger les données en json', width= 30, command=export_json).pack(pady=3)

# Définition d'un bouton pour de téléchargement pour télécharger les données au format csv
dl_csv = Button(fenetre, text='Telecharger les données en csv', width= 30, command=export_csv).pack(pady=3)

#Définition d'un bouton pour tout supprimer dans la liste et le tableau de l'interface
suppression = Button(fenetre, text='Supprimer les données', width=30, command=clear).pack(pady=3)

quitter = Button(fenetre, text='Fermer l\'application',width=50, command=quit).pack(pady=3, ipady=3)
fenetre.mainloop()
```
## Résultat du scrapping

|Titre                                                                                                                            |Dates            |Langues  |
|--------------------------------------------------------------------------------------------------------------------------------------|--------------------|-----------|
|Covid-19                                                                                                                              |2020-02-14T00:00:00Z|Norvégien  |
|Covid-19-vaksiner                                                                                                                     |2020-08-10T00:00:00Z|Norvégien  |
|Covid-19-testing                                                                                                                      |2020-07-19T00:00:00Z|Norvégien  |
|COVID-19                                                                                                                              |2020-02-12T00:00:00Z|Danois     |
|Senfølger af COVID-19                                                                                                                 |2020-11-08T00:00:00Z|Danois     |
|Smitteoverførsel af COVID-19                                                                                                          |2021-04-16T00:00:00Z|Danois     |
|COVID-19                                                                                                                              |2020-02-12T00:00:00Z|Néerlandais|
|COVID-19-vaccin                                                                                                                       |2020-04-30T00:00:00Z|Néerlandais|
|COVID-19-diagnostiek                                                                                                                  |2020-03-29T00:00:00Z|Néerlandais|
|COVID-19                                                                                                                              |2020-02-05T00:00:00Z|Anglais    |
|COVID-19 vaccine                                                                                                                      |2020-03-08T00:00:00Z|Anglais    |
|COVID-19 pandemic                                                                                                                     |2020-01-05T00:00:00Z|Anglais    |
|COVID-19 lockdowns                                                                                                                    |2020-03-24T00:00:00Z|Anglais    |
|Pandémie de Covid-19                                                                                                                  |2020-01-19T00:00:00Z|Français   |
|Pandémie de Covid-19 en France                                                                                                        |2020-02-29T00:00:00Z|Français   |
|Vaccin contre la Covid-19                                                                                                             |2020-03-22T00:00:00Z|Français   |
|COVID-19                                                                                                                              |2020-01-14T00:00:00Z|Allemand   |
|COVID-19-Pandemie                                                                                                                     |2020-01-25T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Deutschland                                                                                                      |2020-02-26T00:00:00Z|Allemand   |
|COVID-19                                                                                                                              |2020-02-11T00:00:00Z|Italien    |
|Pandemia di COVID-19                                                                                                                  |2020-01-22T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Italia                                                                                                        |2020-02-24T00:00:00Z|Italien    |
|COVID-19                                                                                                                              |2020-03-02T00:00:00Z|Letton     |
|COVID-19 pandēmija                                                                                                                    |2020-01-24T00:00:00Z|Letton     |
|COVID-19 pandēmija Latvijā                                                                                                            |2020-03-02T00:00:00Z|Letton     |
|COVID-19                                                                                                                              |2020-03-10T00:00:00Z|Lituanien  |
|COVID-19 pandemija                                                                                                                    |2020-01-30T00:00:00Z|Lituanien  |
|COVID-19 greitasis antigeno testas                                                                                                    |2021-05-24T00:00:00Z|Lituanien  |
|COVID-19                                                                                                                              |2020-02-11T00:00:00Z|Portugais  |
|Pandemia de COVID-19                                                                                                                  |2020-01-20T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Brasil                                                                                                        |2020-02-28T00:00:00Z|Portugais  |
|CPI da COVID-19                                                                                                                       |2021-04-14T00:00:00Z|Portugais  |
|COVID-19                                                                                                                              |2020-02-14T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Elveția                                                                                                       |2020-03-21T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Transnistria                                                                                                  |2020-04-17T00:00:00Z|Roumain    |
|COVID-19                                                                                                                              |2020-02-11T00:00:00Z|Espagnol   |
|Vacuna contra la COVID-19                                                                                                             |2020-04-14T00:00:00Z|Espagnol   |
|Pandemia de COVID-19                                                                                                                  |2020-01-19T00:00:00Z|Espagnol   |
|Covid-19                                                                                                                              |2020-02-11T00:00:00Z|Suédois    |
|Covid-19-vaccin                                                                                                                       |2020-05-17T00:00:00Z|Suédois    |
|COVID-19 Vaccine Moderna                                                                                                              |2020-12-17T00:00:00Z|Suédois    |
|COVID-19                                                                                                                              |2020-02-22T00:00:00Z|Turque     |
|Mauritius'ta COVID-19 pandemisi                                                                                                       |2020-03-20T00:00:00Z|Turque     |
|COVID-19 pandemisi                                                                                                                    |2020-01-23T00:00:00Z|Turque     |



<- [Accueil](/readme.md) ------ [Les outils de la dataviz](/Les_outils_dataviz.md) ->
