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


|Column 1                                                                                                                              |Column 2            |Column 3   |
|--------------------------------------------------------------------------------------------------------------------------------------|--------------------|-----------|
|Covid-19                                                                                                                              |2020-02-14T00:00:00Z|Norvégien  |
|Covid-19-vaksiner                                                                                                                     |2020-08-10T00:00:00Z|Norvégien  |
|Covid-19-testing                                                                                                                      |2020-07-19T00:00:00Z|Norvégien  |
|Pfizer–BioNTechs COVID-19-vaksine                                                                                                     |2020-09-07T00:00:00Z|Norvégien  |
|Modernas COVID-19-vaksine                                                                                                             |2020-12-15T00:00:00Z|Norvégien  |
|Oxford–AstraZenecas COVID-19-vaksine                                                                                                  |2020-09-10T00:00:00Z|Norvégien  |
|Sinovacs COVID-19-vaksine (BBIBP-CorV)                                                                                                |2021-04-01T00:00:00Z|Norvégien  |
|Johnson & Johnsons COVID-19-vaksine                                                                                                   |2021-04-03T00:00:00Z|Norvégien  |
|Sputnik V COVID-19-vaksine                                                                                                            |2020-08-11T00:00:00Z|Norvégien  |
|CureVacs COVID-19-vaksine                                                                                                             |2020-09-10T00:00:00Z|Norvégien  |
|Novavax’ COVID-19-vaksine                                                                                                             |2021-04-01T00:00:00Z|Norvégien  |
|Senfølger av Covid-19                                                                                                                 |2021-03-14T00:00:00Z|Norvégien  |
|Sanofi–GSKs COVID-19-vaksine                                                                                                          |2021-04-01T00:00:00Z|Norvégien  |
|Covid-19 og graviditet                                                                                                                |2020-12-28T00:00:00Z|Norvégien  |
|Tiltak under covid-19-pandemien                                                                                                       |2020-03-11T00:00:00Z|Norvégien  |
|Holden-utvalget (covid-19)                                                                                                            |2021-04-06T00:00:00Z|Norvégien  |
|Smitteoverførsel av covid-19                                                                                                          |2021-05-09T00:00:00Z|Norvégien  |
|Regjeringens covid-19-utvalg                                                                                                          |2020-07-08T00:00:00Z|Norvégien  |
|Sputnik Light COVID-19-vaksine                                                                                                        |2021-05-14T00:00:00Z|Norvégien  |
|Imperial College COVID-19 Response Team                                                                                               |2020-07-28T00:00:00Z|Norvégien  |
|Covid-19-utbruddet tilknyttet Det hvite hus                                                                                           |2020-10-06T00:00:00Z|Norvégien  |
|Beat Covid-19 Ice Hockey Tournament                                                                                                   |2021-05-19T00:00:00Z|Norvégien  |
|Lag om särskilda begränsningar för att förhindra spridning av sjukdomen covid-19                                                      |2021-02-10T00:00:00Z|Norvégien  |
|Covid-19                                                                                                                              |2020-03-21T00:00:00Z|Norvégien  |
|COVID-19                                                                                                                              |2020-02-12T00:00:00Z|Danois     |
|Senfølger af COVID-19                                                                                                                 |2020-11-08T00:00:00Z|Danois     |
|Smitteoverførsel af COVID-19                                                                                                          |2021-04-16T00:00:00Z|Danois     |
|COVID-19 og graviditet                                                                                                                |2020-10-11T00:00:00Z|Danois     |
|Oxford–AstraZeneca COVID-19 vaccine                                                                                                   |2021-05-09T00:00:00Z|Danois     |
|COVID-19-vaccine                                                                                                                      |2020-09-27T00:00:00Z|Danois     |
|COVID-19-testning                                                                                                                     |2020-05-04T00:00:00Z|Danois     |
|COVID-19-pandemiens indvirkning på børn                                                                                               |2021-04-21T00:00:00Z|Danois     |
|COVID-19                                                                                                                              |2020-02-12T00:00:00Z|Néerlandais|
|COVID-19-vaccin                                                                                                                       |2020-04-30T00:00:00Z|Néerlandais|
|COVID-19-diagnostiek                                                                                                                  |2020-03-29T00:00:00Z|Néerlandais|
|Moderna-COVID-19-vaccin                                                                                                               |2021-01-09T00:00:00Z|Néerlandais|
|Pfizer–BioNTech COVID-19 vaccin                                                                                                       |2020-12-05T00:00:00Z|Néerlandais|
|Lijst van personen overleden aan de gevolgen van COVID-19                                                                             |2020-03-30T00:00:00Z|Néerlandais|
|COVID-19-vaccinatiebeleid in België                                                                                                   |2020-11-17T00:00:00Z|Néerlandais|
|COVID-19 snelle antigeentests                                                                                                         |2021-05-30T00:00:00Z|Néerlandais|
|COVID-19-app                                                                                                                          |2020-04-18T00:00:00Z|Néerlandais|
|COVID-19 Crisis Management Team                                                                                                       |2020-06-17T00:00:00Z|Néerlandais|
|COVID-19 Brigade                                                                                                                      |2021-03-13T00:00:00Z|Néerlandais|
|COVID-19-uitbraak in het Witte Huis                                                                                                   |2020-10-04T00:00:00Z|Néerlandais|
|Beleidsregel tegemoetkoming ondernemers getroffen sectoren COVID-19                                                                   |2020-04-28T00:00:00Z|Néerlandais|
|COVID-19                                                                                                                              |2020-02-05T00:00:00Z|Anglais    |
|COVID-19 vaccine                                                                                                                      |2020-03-08T00:00:00Z|Anglais    |
|COVID-19 pandemic                                                                                                                     |2020-01-05T00:00:00Z|Anglais    |
|COVID-19 lockdowns                                                                                                                    |2020-03-24T00:00:00Z|Anglais    |
|COVID-19 pandemic in the United States                                                                                                |2020-02-17T00:00:00Z|Anglais    |
|COVID-19 testing                                                                                                                      |2020-02-25T00:00:00Z|Anglais    |
|Oxford–AstraZeneca COVID-19 vaccine                                                                                                   |2020-07-19T00:00:00Z|Anglais    |
|Pfizer–BioNTech COVID-19 vaccine                                                                                                      |2020-11-09T00:00:00Z|Anglais    |
|Moderna COVID-19 vaccine                                                                                                              |2020-03-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in the United Kingdom                                                                                               |2020-02-26T00:00:00Z|Anglais    |
|COVID-19 pandemic in Japan                                                                                                            |2020-02-21T00:00:00Z|Anglais    |
|COVID-19 pandemic in Ontario                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Australia                                                                                                        |2020-02-26T00:00:00Z|Anglais    |
|List of deaths due to COVID-19                                                                                                        |2020-03-19T00:00:00Z|Anglais    |
|COVID-19 pandemic in Vietnam                                                                                                          |2020-02-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in Maharashtra                                                                                                      |2020-03-16T00:00:00Z|Anglais    |
|COVID-19 misinformation                                                                                                               |2020-02-08T00:00:00Z|Anglais    |
|COVID-19 pandemic in Indonesia                                                                                                        |2020-03-02T00:00:00Z|Anglais    |
|COVID-19 pandemic in Canada                                                                                                           |2020-02-26T00:00:00Z|Anglais    |
|COVID-19 pandemic in Brazil                                                                                                           |2020-02-27T00:00:00Z|Anglais    |
|COVID-19 pandemic in Europe                                                                                                           |2020-02-22T00:00:00Z|Anglais    |
|Symptoms of COVID-19                                                                                                                  |2020-04-29T00:00:00Z|Anglais    |
|Deployment of COVID-19 vaccines                                                                                                       |2021-01-18T00:00:00Z|Anglais    |
|Investigations into the origin of COVID-19                                                                                            |2021-01-15T00:00:00Z|Anglais    |
|COVID-19 pandemic in India                                                                                                            |2020-03-02T00:00:00Z|Anglais    |
|Johnson & Johnson COVID-19 vaccine                                                                                                    |2021-01-15T00:00:00Z|Anglais    |
|COVID-19 pandemic in France                                                                                                           |2020-02-26T00:00:00Z|Anglais    |
|COVID-19 pandemic in Turkey                                                                                                           |2020-03-10T00:00:00Z|Anglais    |
|Novavax COVID-19 vaccine                                                                                                              |2020-12-20T00:00:00Z|Anglais    |
|COVID-19 pandemic by country and territory                                                                                            |2020-01-26T00:00:00Z|Anglais    |
|COVID-19 pandemic in Florida                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 recession                                                                                                                    |2020-03-24T00:00:00Z|Anglais    |
|COVID-19 pandemic in Asia                                                                                                             |2020-02-22T00:00:00Z|Anglais    |
|COVID-19 pandemic in Taiwan                                                                                                           |2020-02-22T00:00:00Z|Anglais    |
|List of COVID-19 vaccine authorizations                                                                                               |2021-03-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in Assam                                                                                                            |2020-03-23T00:00:00Z|Anglais    |
|COVID-19 pandemic in Mexico                                                                                                           |2020-03-02T00:00:00Z|Anglais    |
|COVID-19 pandemic in Germany                                                                                                          |2020-02-26T00:00:00Z|Anglais    |
|COVID-19 pandemic in Colorado                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Minnesota                                                                                                        |2020-03-15T00:00:00Z|Anglais    |
|COVID-19 pandemic in West Bengal                                                                                                      |2020-03-22T00:00:00Z|Anglais    |
|COVID-19 vaccination in the Philippines                                                                                               |2021-01-06T00:00:00Z|Anglais    |
|COVID-19 pandemic in Romania                                                                                                          |2020-03-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in Kerala                                                                                                           |2020-03-09T00:00:00Z|Anglais    |
|COVID-19 pandemic in Texas                                                                                                            |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Gujarat                                                                                                          |2020-03-22T00:00:00Z|Anglais    |
|COVID-19 pandemic in Michigan                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Oregon                                                                                                           |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Hungary                                                                                                          |2020-03-04T00:00:00Z|Anglais    |
|COVID-19 pandemic in Punjab, India                                                                                                    |2020-03-21T00:00:00Z|Anglais    |
|COVID-19 pandemic in Malaysia                                                                                                         |2020-02-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in Uttar Pradesh                                                                                                    |2020-03-18T00:00:00Z|Anglais    |
|COVID-19 vaccination in India                                                                                                         |2021-01-16T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Philippines                                                                                                  |2020-02-14T00:00:00Z|Anglais    |
|COVID-19 pandemic in Singapore                                                                                                        |2020-02-14T00:00:00Z|Anglais    |
|COVID-19 pandemic in Illinois                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Alberta                                                                                                          |2020-03-18T00:00:00Z|Anglais    |
|COVID-19 pandemic in Karnataka                                                                                                        |2020-03-14T00:00:00Z|Anglais    |
|COVID-19 pandemic in Tamil Nadu                                                                                                       |2020-03-30T00:00:00Z|Anglais    |
|Transmission of COVID-19                                                                                                              |2020-07-20T00:00:00Z|Anglais    |
|COVID-19 lockdown in India                                                                                                            |2020-03-25T00:00:00Z|Anglais    |
|COVID-19 pandemic in Bulgaria                                                                                                         |2020-03-08T00:00:00Z|Anglais    |
|COVID-19 pandemic in Sweden                                                                                                           |2020-02-28T00:00:00Z|Anglais    |
|COVID-19 pandemic in Rajasthan                                                                                                        |2020-04-14T00:00:00Z|Anglais    |
|COVID-19 pandemic cases                                                                                                               |2020-02-06T00:00:00Z|Anglais    |
|COVID-19 pandemic in Italy                                                                                                            |2020-02-22T00:00:00Z|Anglais    |
|COVID-19 pandemic in Poland                                                                                                           |2020-02-29T00:00:00Z|Anglais    |
|Treatment and management of COVID-19                                                                                                  |2020-06-01T00:00:00Z|Anglais    |
|COVID-19 pandemic in Russia                                                                                                           |2020-03-02T00:00:00Z|Anglais    |
|COVID-19 pandemic in Virginia                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Arizona                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Bangladesh                                                                                                       |2020-03-11T00:00:00Z|Anglais    |
|COVID-19 pandemic in Switzerland                                                                                                      |2020-02-29T00:00:00Z|Anglais    |
|COVID-19 pandemic in Thailand                                                                                                         |2020-02-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in South Africa                                                                                                     |2020-03-06T00:00:00Z|Anglais    |
|COVID-19 pandemic in Oceania                                                                                                          |2020-03-05T00:00:00Z|Anglais    |
|COVID-19 pandemic in Missouri                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|Statistics of the COVID-19 pandemic in India                                                                                          |2020-06-11T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Netherlands                                                                                                  |2020-02-29T00:00:00Z|Anglais    |
|COVID-19 pandemic in Pakistan                                                                                                         |2020-03-04T00:00:00Z|Anglais    |
|COVID-19 pandemic in Ohio                                                                                                             |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Greece                                                                                                           |2020-02-26T00:00:00Z|Anglais    |
|COVID-19 pandemic in Africa                                                                                                           |2020-03-05T00:00:00Z|Anglais    |
|COVID-19 pandemic in Pennsylvania                                                                                                     |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Fiji                                                                                                             |2020-03-15T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic                                                                                                     |2020-02-08T00:00:00Z|Anglais    |
|COVID-19 pandemic in Wyoming                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 drug development                                                                                                             |2020-03-21T00:00:00Z|Anglais    |
|COVID-19 deaths                                                                                                                       |2020-04-23T00:00:00Z|Anglais    |
|COVID-19 pandemic in Nevada                                                                                                           |2020-03-15T00:00:00Z|Anglais    |
|COVID-19 pandemic in Peru                                                                                                             |2020-03-11T00:00:00Z|Anglais    |
|COVID-19 pandemic in Belgium                                                                                                          |2020-03-02T00:00:00Z|Anglais    |
|COVID-19 pandemic in Morocco                                                                                                          |2020-03-06T00:00:00Z|Anglais    |
|COVID-19 pandemic in Austria                                                                                                          |2020-02-29T00:00:00Z|Anglais    |
|COVID-19 pandemic in Spain                                                                                                            |2020-02-26T00:00:00Z|Anglais    |
|COVID-19 pandemic in Portugal                                                                                                         |2020-03-02T00:00:00Z|Anglais    |
|COVID-19 vaccination in the United Kingdom                                                                                            |2020-12-16T00:00:00Z|Anglais    |
|COVID-19 pandemic in Quebec                                                                                                           |2020-03-16T00:00:00Z|Anglais    |
|COVID-19 pandemic in Iran                                                                                                             |2020-02-21T00:00:00Z|Anglais    |
|COVID-19 pandemic in Connecticut                                                                                                      |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Belarus                                                                                                          |2020-03-03T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in 2019                                                                                             |2020-04-21T00:00:00Z|Anglais    |
|COVID-19 pandemic in Chile                                                                                                            |2020-03-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in British Columbia                                                                                                 |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Maryland                                                                                                         |2020-03-12T00:00:00Z|Anglais    |
|COVID-19 pandemic in Nepal                                                                                                            |2020-03-11T00:00:00Z|Anglais    |
|COVID-19 pandemic in Algeria                                                                                                          |2020-03-05T00:00:00Z|Anglais    |
|COVID-19 pandemic in Northern Ireland                                                                                                 |2020-03-21T00:00:00Z|Anglais    |
|COVID-19 pandemic in Odisha                                                                                                           |2020-04-11T00:00:00Z|Anglais    |
|COVID-19 apps                                                                                                                         |2020-04-01T00:00:00Z|Anglais    |
|COVID-19 pandemic in Antarctica                                                                                                       |2020-04-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in Cambodia                                                                                                         |2020-03-10T00:00:00Z|Anglais    |
|COVID-19 pandemic in Finland                                                                                                          |2020-03-01T00:00:00Z|Anglais    |
|COVID-19 pandemic in Afghanistan                                                                                                      |2020-03-10T00:00:00Z|Anglais    |
|COVID-19 pandemic in Washington (state)                                                                                               |2020-03-06T00:00:00Z|Anglais    |
|COVID-19 pandemic in New Zealand                                                                                                      |2020-03-04T00:00:00Z|Anglais    |
|COVID-19 pandemic in Colombia                                                                                                         |2020-03-06T00:00:00Z|Anglais    |
|Sanofi–GSK COVID-19 vaccine                                                                                                           |2021-03-28T00:00:00Z|Anglais    |
|Statistics of the COVID-19 pandemic in Indonesia                                                                                      |2021-02-24T00:00:00Z|Anglais    |
|COVID-19 pandemic in Scotland                                                                                                         |2020-03-16T00:00:00Z|Anglais    |
|COVID-19 pandemic death rates by country                                                                                              |2020-07-18T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in Australia                                                                                        |2021-01-11T00:00:00Z|Anglais    |
|COVID-19 rapid antigen test                                                                                                           |2021-05-19T00:00:00Z|Anglais    |
|COVID-19 CPI                                                                                                                          |2021-05-05T00:00:00Z|Anglais    |
|COVID-19 pandemic in Nigeria                                                                                                          |2020-03-10T00:00:00Z|Anglais    |
|COVID-19 pandemic in North Korea                                                                                                      |2020-03-13T00:00:00Z|Anglais    |
|Sputnik V COVID-19 vaccine                                                                                                            |2020-08-10T00:00:00Z|Anglais    |
|COVID-19 pandemic in Norway                                                                                                           |2020-03-01T00:00:00Z|Anglais    |
|Statistics of the COVID-19 pandemic in Australia                                                                                      |2020-08-16T00:00:00Z|Anglais    |
|COVID-19 pandemic in North America                                                                                                    |2020-02-27T00:00:00Z|Anglais    |
|COVID-19 pandemic in Egypt                                                                                                            |2020-03-04T00:00:00Z|Anglais    |
|COVID-19 pandemic in California                                                                                                       |2020-03-06T00:00:00Z|Anglais    |
|Statistics of the COVID-19 pandemic in Brazil                                                                                         |2020-04-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Republic of Ireland                                                                                          |2020-03-02T00:00:00Z|Anglais    |
|COVID-19 pandemic in Argentina                                                                                                        |2020-03-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in Serbia                                                                                                           |2020-03-06T00:00:00Z|Anglais    |
|COVID-19 pandemic in Myanmar                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 drug repurposing research                                                                                                    |2020-03-19T00:00:00Z|Anglais    |
|Impact of the COVID-19 pandemic on education                                                                                          |2020-03-15T00:00:00Z|Anglais    |
|COVID-19 pandemic in mainland China                                                                                                   |2020-02-02T00:00:00Z|Anglais    |
|Statistics of the COVID-19 pandemic in Peru                                                                                           |2021-04-08T00:00:00Z|Anglais    |
|COVID-19 pandemic in Madhya Pradesh                                                                                                   |2020-03-22T00:00:00Z|Anglais    |
|COVID-19 pandemic deaths                                                                                                              |2020-03-25T00:00:00Z|Anglais    |
|Impact of the COVID-19 pandemic on the environment                                                                                    |2020-04-09T00:00:00Z|Anglais    |
|COVID-19 pandemic in Delhi                                                                                                            |2020-03-19T00:00:00Z|Anglais    |
|COVID-19 pandemic in Massachusetts                                                                                                    |2020-03-11T00:00:00Z|Anglais    |
|History of COVID-19 vaccine development                                                                                               |2021-01-18T00:00:00Z|Anglais    |
|COVID-19 pandemic in Alabama                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Utah                                                                                                             |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in North Dakota                                                                                                     |2020-03-15T00:00:00Z|Anglais    |
|COVID-19 pandemic in Louisiana                                                                                                        |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Denmark                                                                                                          |2020-03-02T00:00:00Z|Anglais    |
|White House COVID-19 outbreak                                                                                                         |2020-10-03T00:00:00Z|Anglais    |
|COVID-19 vaccination in the United States                                                                                             |2021-03-06T00:00:00Z|Anglais    |
|COVID-19 pandemic in Mauritius                                                                                                        |2020-03-13T00:00:00Z|Anglais    |
|Impact of the COVID-19 pandemic                                                                                                       |2020-04-06T00:00:00Z|Anglais    |
|COVID-19 pandemic in Ecuador                                                                                                          |2020-02-29T00:00:00Z|Anglais    |
|COVID-19 pandemic in Calabarzon                                                                                                       |2020-04-11T00:00:00Z|Anglais    |
|COVID-19 pandemic in South America                                                                                                    |2020-03-11T00:00:00Z|Anglais    |
|COVID-19 vaccination in Malaysia                                                                                                      |2021-02-26T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in Singapore                                                                                        |2020-05-12T00:00:00Z|Anglais    |
|COVID-19 pandemic in England                                                                                                          |2020-03-27T00:00:00Z|Anglais    |
|Philippine government response to the COVID-19 pandemic                                                                               |2020-05-09T00:00:00Z|Anglais    |
|COVID-19 pandemic in the United Arab Emirates                                                                                         |2020-02-24T00:00:00Z|Anglais    |
|COVID-19 pandemic in New York City                                                                                                    |2020-03-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in Tanzania                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Maine                                                                                                            |2020-03-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in Washington, D.C.                                                                                                 |2020-03-12T00:00:00Z|Anglais    |
|COVID-19 pandemic in Aruba                                                                                                            |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Delaware                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Oklahoma                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Arkansas                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Montana                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Vermont                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Kansas                                                                                                           |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in South Korea                                                                                                      |2020-02-20T00:00:00Z|Anglais    |
|List of unproven methods against COVID-19                                                                                             |2020-04-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Czech Republic                                                                                               |2020-03-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in Haryana                                                                                                          |2020-03-23T00:00:00Z|Anglais    |
|COVID-19 pandemic in Saskatchewan                                                                                                     |2020-03-23T00:00:00Z|Anglais    |
|COVID-19 pandemic in Wisconsin                                                                                                        |2020-03-15T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in Ontario                                                                                          |2020-12-07T00:00:00Z|Anglais    |
|COVID-19 pandemic in Hong Kong                                                                                                        |2020-02-22T00:00:00Z|Anglais    |
|COVID-19 vaccination in Romania                                                                                                       |2021-02-01T00:00:00Z|Anglais    |
|COVID-19 pandemic in Slovakia                                                                                                         |2020-03-06T00:00:00Z|Anglais    |
|COVID-19 pandemic in Seychelles                                                                                                       |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Bihar                                                                                                            |2020-04-22T00:00:00Z|Anglais    |
|COVID-19 pandemic in Iowa                                                                                                             |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 party                                                                                                                        |2020-03-25T00:00:00Z|Anglais    |
|COVID-19 pandemic in Metro Manila                                                                                                     |2020-03-12T00:00:00Z|Anglais    |
|COVID-19 vaccination in Vietnam                                                                                                       |2021-04-01T00:00:00Z|Anglais    |
|COVID-19 pandemic in Mongolia                                                                                                         |2020-03-10T00:00:00Z|Anglais    |
|CureVac COVID-19 vaccine                                                                                                              |2021-02-17T00:00:00Z|Anglais    |
|Social impact of the COVID-19 pandemic                                                                                                |2020-02-05T00:00:00Z|Anglais    |
|COVID-19 vaccination in Italy                                                                                                         |2021-01-10T00:00:00Z|Anglais    |
|COVID-19 pandemic in Alaska                                                                                                           |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Tennessee                                                                                                        |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Iceland                                                                                                          |2020-03-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in Mississippi                                                                                                      |2020-03-15T00:00:00Z|Anglais    |
|COVID-19 pandemic in New York (state)                                                                                                 |2020-03-06T00:00:00Z|Anglais    |
|COVID-19 vaccination in Australia                                                                                                     |2021-01-26T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Maldives                                                                                                     |2020-03-11T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in the United Kingdom                                                                               |2020-09-25T00:00:00Z|Anglais    |
|COVID-19 pandemic in Puerto Rico                                                                                                      |2020-03-13T00:00:00Z|Anglais    |
|National responses to the COVID-19 pandemic                                                                                           |2020-03-10T00:00:00Z|Anglais    |
|COVID-19 pandemic in Sri Lanka                                                                                                        |2020-03-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in Manitoba                                                                                                         |2020-03-16T00:00:00Z|Anglais    |
|COVID-19 pandemic in North Carolina                                                                                                   |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Belize                                                                                                           |2020-03-14T00:00:00Z|Anglais    |
|Economic impact of the COVID-19 pandemic in India                                                                                     |2020-03-23T00:00:00Z|Anglais    |
|COVID-19 pandemic in South Dakota                                                                                                     |2020-03-13T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in May 2021                                                                                         |2021-05-01T00:00:00Z|Anglais    |
|COVID-19 pandemic in Israel                                                                                                           |2020-03-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in Suriname                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Rhode Island                                                                                                     |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Idaho                                                                                                            |2020-03-14T00:00:00Z|Anglais    |
|COVID-19 pandemic in Kentucky                                                                                                         |2020-03-15T00:00:00Z|Anglais    |
|COVID-19 pandemic in Albania                                                                                                          |2020-03-09T00:00:00Z|Anglais    |
|COVID-19 pandemic in Saint Helena, Ascension and Tristan da Cunha                                                                     |2021-03-19T00:00:00Z|Anglais    |
|COVID-19 pandemic in Croatia                                                                                                          |2020-03-01T00:00:00Z|Anglais    |
|COVID-19 pandemic in Indiana                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Hawaii                                                                                                           |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in South Sudan                                                                                                      |2020-03-13T00:00:00Z|Anglais    |
|Walvax COVID-19 vaccine                                                                                                               |2021-04-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in Kuwait                                                                                                           |2020-02-29T00:00:00Z|Anglais    |
|White House COVID-19 Response Team                                                                                                    |2021-02-09T00:00:00Z|Anglais    |
|COVID-19 vaccination in Bangladesh                                                                                                    |2021-04-30T00:00:00Z|Anglais    |
|COVID-19 pandemic in Nova Scotia                                                                                                      |2020-03-16T00:00:00Z|Anglais    |
|COVID-19 pandemic in Toronto                                                                                                          |2020-05-10T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in the Philippines                                                                                  |2020-03-27T00:00:00Z|Anglais    |
|COVID-19 pandemic in Nagaland                                                                                                         |2020-05-14T00:00:00Z|Anglais    |
|COVID-19 pandemic in Saudi Arabia                                                                                                     |2020-03-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Falkland Islands                                                                                             |2020-03-29T00:00:00Z|Anglais    |
|COVID-19 pandemic in Goa                                                                                                              |2020-03-26T00:00:00Z|Anglais    |
|COVID-19 vaccination in Canada                                                                                                        |2021-02-16T00:00:00Z|Anglais    |
|Minhai COVID-19 vaccine                                                                                                               |2021-04-03T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in the United States                                                                                |2021-01-24T00:00:00Z|Anglais    |
|Impact of the COVID-19 pandemic on politics                                                                                           |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Newfoundland and Labrador                                                                                        |2020-03-13T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in Canada                                                                                           |2020-05-26T00:00:00Z|Anglais    |
|COVID-19 vaccination in South Korea                                                                                                   |2021-04-24T00:00:00Z|Anglais    |
|COVID-19 vaccination in Sweden                                                                                                        |2021-01-06T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in June 2021                                                                                        |2020-09-25T00:00:00Z|Anglais    |
|COVID-19 pandemic in New Jersey                                                                                                       |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Namibia                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Nebraska                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 in pregnancy                                                                                                                 |2020-03-19T00:00:00Z|Anglais    |
|COVID-19 pandemic in New Mexico                                                                                                       |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Madagascar                                                                                                       |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Liechtenstein                                                                                                    |2020-03-04T00:00:00Z|Anglais    |
|COVID-19 pandemic in South Carolina                                                                                                   |2020-03-13T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in Wales                                                                                            |2021-01-02T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in the United Kingdom (January–June 2020)                                                           |2020-04-16T00:00:00Z|Anglais    |
|COVID-19 pandemic in Iraq                                                                                                             |2020-03-03T00:00:00Z|Anglais    |
|COVID-19 vaccination in Bulgaria                                                                                                      |2020-12-29T00:00:00Z|Anglais    |
|COVID-19 pandemic in Turkmenistan                                                                                                     |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Republic of Artsakh                                                                                          |2020-04-08T00:00:00Z|Anglais    |
|COVID-19 vaccination in Indonesia                                                                                                     |2021-04-24T00:00:00Z|Anglais    |
|COVID-19 pandemic in Jharkhand                                                                                                        |2020-04-21T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in June 2020                                                                                        |2020-04-29T00:00:00Z|Anglais    |
|COVID-19 vaccination in Colombia                                                                                                      |2021-03-08T00:00:00Z|Anglais    |
|COVID-19 pandemic in Guam                                                                                                             |2020-03-15T00:00:00Z|Anglais    |
|COVID-19 pandemic in West Virginia                                                                                                    |2020-03-17T00:00:00Z|Anglais    |
|Statistics of the COVID-19 pandemic in the United States                                                                              |2020-11-24T00:00:00Z|Anglais    |
|Economic impact of the COVID-19 pandemic                                                                                              |2020-05-11T00:00:00Z|Anglais    |
|COVID-19 pandemic in Curaçao                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Tajikistan                                                                                                       |2020-04-04T00:00:00Z|Anglais    |
|Use and development of software for COVID-19 pandemic mitigation                                                                      |2021-04-08T00:00:00Z|Anglais    |
|COVID-19 pandemic in Bahrain                                                                                                          |2020-03-02T00:00:00Z|Anglais    |
|COVID-19 pandemic in Gibraltar                                                                                                        |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Georgia                                                                                                          |2020-03-14T00:00:00Z|Anglais    |
|COVID-19 pandemic in New Hampshire                                                                                                    |2020-03-13T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in the United Kingdom (2021)                                                                        |2021-01-01T00:00:00Z|Anglais    |
|COVID-19 pandemic in the State of Palestine                                                                                           |2020-03-06T00:00:00Z|Anglais    |
|COVID-19 pandemic in Trinidad and Tobago                                                                                              |2020-03-12T00:00:00Z|Anglais    |
|COVID-19 pandemic in Palau                                                                                                            |2020-03-13T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in Massachusetts                                                                                    |2020-08-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in Bonaire                                                                                                          |2020-04-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in Ghana                                                                                                            |2020-03-13T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in January 2020                                                                                     |2020-01-23T00:00:00Z|Anglais    |
|COVID-19 pandemic in Mauritania                                                                                                       |2020-03-13T00:00:00Z|Anglais    |
|Economic impact of the COVID-19 pandemic in the United States                                                                         |2020-05-27T00:00:00Z|Anglais    |
|COVID-19 pandemic in Wales                                                                                                            |2020-03-25T00:00:00Z|Anglais    |
|COVID-19 pandemic in Sarawak                                                                                                          |2020-03-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in Botswana                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Oman                                                                                                             |2020-03-10T00:00:00Z|Anglais    |
|COVID-19 pandemic in Eritrea                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Dominica                                                                                                         |2020-03-17T00:00:00Z|Anglais    |
|Protests over responses to the COVID-19 pandemic                                                                                      |2020-04-16T00:00:00Z|Anglais    |
|COVID-19 misinformation by China                                                                                                      |2021-01-23T00:00:00Z|Anglais    |
|COVID-19 pandemic in Honduras                                                                                                         |2020-03-12T00:00:00Z|Anglais    |
|COVID-19 pandemic in Kenya                                                                                                            |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Uganda                                                                                                           |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Laos                                                                                                             |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Jordan                                                                                                           |2020-03-12T00:00:00Z|Anglais    |
|U.S. state and local government responses to the COVID-19 pandemic                                                                    |2020-03-16T00:00:00Z|Anglais    |
|COVID-19 vaccination in Israel                                                                                                        |2021-01-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Kosovo                                                                                                           |2020-03-13T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in March 2020                                                                                       |2020-02-26T00:00:00Z|Anglais    |
|COVID-19 pandemic in Zimbabwe                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Malta                                                                                                            |2020-03-08T00:00:00Z|Anglais    |
|COVID-19 vaccination in the Republic of Ireland                                                                                       |2021-02-12T00:00:00Z|Anglais    |
|COVID-19 pandemic in Guyana                                                                                                           |2020-03-12T00:00:00Z|Anglais    |
|Food security during the COVID-19 pandemic                                                                                            |2020-06-20T00:00:00Z|Anglais    |
|COVID-19 pandemic in Lithuania                                                                                                        |2020-03-04T00:00:00Z|Anglais    |
|COVID-19 pandemic in Saba                                                                                                             |2020-04-12T00:00:00Z|Anglais    |
|COVID-19 pandemic in Eswatini                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Melilla                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Ukraine                                                                                                          |2020-03-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in Jamaica                                                                                                          |2020-03-12T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in Italy                                                                                            |2021-01-26T00:00:00Z|Anglais    |
|COVID-19 pandemic in Mozambique                                                                                                       |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic on Diamond Princess                                                                                                 |2020-04-09T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in India (January–May 2020)                                                                         |2020-03-21T00:00:00Z|Anglais    |
|COVID-19 pandemic in Kyrgyzstan                                                                                                       |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Panama                                                                                                           |2020-03-11T00:00:00Z|Anglais    |
|COVID-19 pandemic in Bhutan                                                                                                           |2020-03-12T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in April 2020                                                                                       |2020-03-03T00:00:00Z|Anglais    |
|COVID-19 vaccination in Quebec                                                                                                        |2021-04-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in Latvia                                                                                                           |2020-03-04T00:00:00Z|Anglais    |
|COVID-19 pandemic in Lebanon                                                                                                          |2020-03-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in Cuba                                                                                                             |2020-03-12T00:00:00Z|Anglais    |
|COVID-19 pandemic in Sint Maarten                                                                                                     |2020-03-16T00:00:00Z|Anglais    |
|COVID-19 pandemic in Chhattisgarh                                                                                                     |2020-05-05T00:00:00Z|Anglais    |
|COVID-19 pandemic in Zambia                                                                                                           |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Guernsey                                                                                                         |2020-03-09T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in Vietnam                                                                                          |2021-01-05T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Faroe Islands                                                                                                |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in London                                                                                                           |2020-03-18T00:00:00Z|Anglais    |
|COVID-19 vaccination in South Africa                                                                                                  |2021-03-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in East Timor                                                                                                       |2020-03-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in Ethiopia                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Samoa                                                                                                            |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Bicol Region                                                                                                 |2020-05-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in Cameroon                                                                                                         |2020-03-06T00:00:00Z|Anglais    |
|COVID-19 pandemic in Ivory Coast                                                                                                      |2020-03-12T00:00:00Z|Anglais    |
|COVID-19 pandemic in Bolivia                                                                                                          |2020-03-11T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in February 2020                                                                                    |2020-02-08T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in November 2020                                                                                    |2020-08-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in Jersey                                                                                                           |2020-03-11T00:00:00Z|Anglais    |
|COVID-19 pandemic in Luxembourg                                                                                                       |2020-03-04T00:00:00Z|Anglais    |
|COVID-19 pandemic in Syria                                                                                                            |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Liberia                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Uruguay                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Anguilla                                                                                                         |2020-03-27T00:00:00Z|Anglais    |
|COVID-19 pandemic in Macau                                                                                                            |2020-02-22T00:00:00Z|Anglais    |
|COVID-19 pandemic in Moldova                                                                                                          |2020-03-08T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in Japan                                                                                            |2020-07-19T00:00:00Z|Anglais    |
|COVID-19 pandemic in Montserrat                                                                                                       |2020-03-18T00:00:00Z|Anglais    |
|COVID-19 pandemic in Slovenia                                                                                                         |2020-03-06T00:00:00Z|Anglais    |
|COVID-19 pandemic in Monaco                                                                                                           |2020-03-04T00:00:00Z|Anglais    |
|COVID-19 pandemic in Qatar                                                                                                            |2020-03-10T00:00:00Z|Anglais    |
|COVID-19 pandemic in Northern Mindanao                                                                                                |2020-04-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Dominican Republic                                                                                           |2020-03-01T00:00:00Z|Anglais    |
|World Health Organization's response to the COVID-19 pandemic                                                                         |2020-04-09T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Cayman Islands                                                                                               |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 vaccination in Brazil                                                                                                        |2021-05-26T00:00:00Z|Anglais    |
|COVID-19 vaccination in Singapore                                                                                                     |2021-05-12T00:00:00Z|Anglais    |
|COVID-19 pandemic in Malawi                                                                                                           |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Zamboanga Peninsula                                                                                              |2020-05-07T00:00:00Z|Anglais    |
|COVID-19 pandemic in Rwanda                                                                                                           |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Gabon                                                                                                            |2020-03-13T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in January 2021                                                                                     |2020-09-25T00:00:00Z|Anglais    |
|Face masks during the COVID-19 pandemic                                                                                               |2020-04-11T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Bahamas                                                                                                      |2020-03-15T00:00:00Z|Anglais    |
|Impact of the COVID-19 pandemic on religion                                                                                           |2020-03-16T00:00:00Z|Anglais    |
|COVID-19 pandemic cases in May 2021                                                                                                   |2021-05-02T00:00:00Z|Anglais    |
|COVID-19 lockdown in China                                                                                                            |2020-01-23T00:00:00Z|Anglais    |
|COVID-19 pandemic in Senegal                                                                                                          |2020-03-12T00:00:00Z|Anglais    |
|COVID-19 pandemic deaths in May 2021                                                                                                  |2021-05-02T00:00:00Z|Anglais    |
|COVID-19 pandemic in Sint Eustatius                                                                                                   |2020-04-02T00:00:00Z|Anglais    |
|COVID-19 pandemic in Burundi                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|NHS COVID-19                                                                                                                          |2020-05-06T00:00:00Z|Anglais    |
|Shortages related to the COVID-19 pandemic                                                                                            |2020-03-23T00:00:00Z|Anglais    |
|COVID-19 pandemic in Paraguay                                                                                                         |2020-03-11T00:00:00Z|Anglais    |
|COVID-19 pandemic in Montreal                                                                                                         |2020-04-22T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in India (2021)                                                                                     |2021-05-03T00:00:00Z|Anglais    |
|Impact of the COVID-19 pandemic on social media                                                                                       |2020-03-22T00:00:00Z|Anglais    |
|COVID-19 pandemic in Djibouti                                                                                                         |2020-03-13T00:00:00Z|Anglais    |
|Impact of the COVID-19 pandemic on animals                                                                                            |2020-10-02T00:00:00Z|Anglais    |
|COVID-19 pandemic in Benin                                                                                                            |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Georgia (U.S. state)                                                                                             |2020-03-13T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in the United States (2020)                                                                         |2020-03-18T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in July 2020                                                                                        |2020-05-06T00:00:00Z|Anglais    |
|COVID-19 pandemic in Sabah                                                                                                            |2020-03-16T00:00:00Z|Anglais    |
|COVID-19 pandemic in North Rhine-Westphalia                                                                                           |2020-03-15T00:00:00Z|Anglais    |
|COVID-19 pandemic in Cape Verde                                                                                                       |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in New York                                                                                                         |2020-03-17T00:00:00Z|Anglais    |
|COVID-19 vaccination in France                                                                                                        |2021-01-30T00:00:00Z|Anglais    |
|COVID-19 pandemic in San Marino                                                                                                       |2020-03-03T00:00:00Z|Anglais    |
|Arcturus COVID-19 vaccine                                                                                                             |2021-01-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in Chad                                                                                                             |2020-03-13T00:00:00Z|Anglais    |
|Parliamentary Under-Secretary of State for COVID-19 Vaccine Deployment                                                                |2020-12-12T00:00:00Z|Anglais    |
|COVID-19 pandemic in Nicaragua                                                                                                        |2020-03-15T00:00:00Z|Anglais    |
|COVID-19 pandemic in Kiribati                                                                                                         |2020-03-15T00:00:00Z|Anglais    |
|COVID-19 pandemic in Cyprus                                                                                                           |2020-03-09T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in England (2021)                                                                                   |2021-01-01T00:00:00Z|Anglais    |
|COVID-19 vaccination in Mexico                                                                                                        |2021-04-02T00:00:00Z|Anglais    |
|COVID-19 pandemic in Western Visayas                                                                                                  |2020-04-16T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Comoros                                                                                                      |2020-04-01T00:00:00Z|Anglais    |
|COVID-19 pandemic in Montenegro                                                                                                       |2020-03-11T00:00:00Z|Anglais    |
|COVID-19 pandemic in Georgia (country)                                                                                                |2020-03-05T00:00:00Z|Anglais    |
|COVID-19 pandemic in Brunei                                                                                                           |2020-03-09T00:00:00Z|Anglais    |
|COVID-19 community quarantines in the Philippines                                                                                     |2020-04-07T00:00:00Z|Anglais    |
|COVID-19 pandemic in Togo                                                                                                             |2020-03-08T00:00:00Z|Anglais    |
|COVID-19 pandemic in Transnistria                                                                                                     |2020-03-25T00:00:00Z|Anglais    |
|COVID-19 pandemic in Bermuda                                                                                                          |2020-03-19T00:00:00Z|Anglais    |
|COVID-19 pandemic in Andorra                                                                                                          |2020-03-02T00:00:00Z|Anglais    |
|Covivac (Vietnam COVID-19 vaccine)                                                                                                    |2021-04-01T00:00:00Z|Anglais    |
|COVID-19 pandemic in Mali                                                                                                             |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Venezuela                                                                                                        |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Azerbaijan                                                                                                       |2020-03-09T00:00:00Z|Anglais    |
|COVID-19 pandemic in Abkhazia                                                                                                         |2020-04-04T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Davao Region                                                                                                 |2020-04-11T00:00:00Z|Anglais    |
|COVID-19 pandemic in Greenland                                                                                                        |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Gambia                                                                                                       |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 surveillance                                                                                                                 |2020-03-31T00:00:00Z|Anglais    |
|COVID-19 pandemic in Congo                                                                                                            |2020-09-18T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in Indonesia                                                                                        |2020-04-12T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in December 2020                                                                                    |2020-08-03T00:00:00Z|Anglais    |
|Vetenskapsforum COVID-19                                                                                                              |2020-11-05T00:00:00Z|Anglais    |
|COVID-19 vaccination in Japan                                                                                                         |2021-05-25T00:00:00Z|Anglais    |
|COVID-19 pandemic in Somalia                                                                                                          |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Sudan                                                                                                            |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in popular culture                                                                                                  |2020-12-18T00:00:00Z|Anglais    |
|British government response to the COVID-19 pandemic                                                                                  |2020-04-06T00:00:00Z|Anglais    |
|COVID-19 vaccination in New Zealand                                                                                                   |2021-03-11T00:00:00Z|Anglais    |
|COVID-19 pandemic in Guinea                                                                                                           |2020-03-13T00:00:00Z|Anglais    |
|U.S. federal government response to the COVID-19 pandemic                                                                             |2020-07-07T00:00:00Z|Anglais    |
|COVID-19 vaccination in Peru                                                                                                          |2021-02-23T00:00:00Z|Anglais    |
|COVID-19 pandemic in Yemen                                                                                                            |2020-03-14T00:00:00Z|Anglais    |
|COVID-19 pandemic in Libya                                                                                                            |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 vaccination in Greece                                                                                                        |2021-04-11T00:00:00Z|Anglais    |
|COVID-19 pandemic in Vanuatu                                                                                                          |2020-03-17T00:00:00Z|Anglais    |
|COVID-19 pandemic in Kazakhstan                                                                                                       |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Armenia                                                                                                          |2020-03-09T00:00:00Z|Anglais    |
|COVID-19 pandemic in Saint Lucia                                                                                                      |2020-03-14T00:00:00Z|Anglais    |
|COVID-19 pandemic in Northern Cyprus                                                                                                  |2020-03-14T00:00:00Z|Anglais    |
|COVID-19 pandemic in Costa Rica                                                                                                       |2020-03-10T00:00:00Z|Anglais    |
|COVID-19 pandemic in Central Visayas                                                                                                  |2020-04-16T00:00:00Z|Anglais    |
|COVID-19 pandemic in Angola                                                                                                           |2020-03-13T00:00:00Z|Anglais    |
|CoviVac (Russia COVID-19 vaccine)                                                                                                     |2021-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Mimaropa                                                                                                         |2020-05-07T00:00:00Z|Anglais    |
|COVID-19 pandemic in South Asia                                                                                                       |2020-03-25T00:00:00Z|Anglais    |
|Mental health during the COVID-19 pandemic                                                                                            |2020-03-28T00:00:00Z|Anglais    |
|COVID-19 pandemic on cruise ships                                                                                                     |2020-02-18T00:00:00Z|Anglais    |
|COVID-19 pandemic in Vatican City                                                                                                     |2020-03-06T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in August 2020                                                                                      |2020-07-31T00:00:00Z|Anglais    |
|COVID-19 pandemic in Andhra Pradesh                                                                                                   |2020-03-22T00:00:00Z|Anglais    |
|COVID-19 pandemic in Barbados                                                                                                         |2020-03-17T00:00:00Z|Anglais    |
|Face masks during the COVID-19 pandemic in the United States                                                                          |2020-06-06T00:00:00Z|Anglais    |
|COVID-19 clusters in Australia                                                                                                        |2021-03-20T00:00:00Z|Anglais    |
|COVID-19 pandemic in Prince Edward Island                                                                                             |2020-03-16T00:00:00Z|Anglais    |
|Xenophobia and racism related to the COVID-19 pandemic                                                                                |2020-02-02T00:00:00Z|Anglais    |
|COVID-19 vaccination in Germany                                                                                                       |2021-05-10T00:00:00Z|Anglais    |
|COVID-19 misinformation by governments                                                                                                |2020-12-30T00:00:00Z|Anglais    |
|COVID-19 vaccination in Albania                                                                                                       |2021-02-08T00:00:00Z|Anglais    |
|COVID-19 pandemic in Philadelphia                                                                                                     |2020-04-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Uzbekistan                                                                                                       |2020-03-13T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in Scotland                                                                                         |2021-01-02T00:00:00Z|Anglais    |
|COVID-19 pandemic in the Isle of Man                                                                                                  |2020-03-20T00:00:00Z|Anglais    |
|COVID-19 pandemic in Uttarakhand                                                                                                      |2020-05-05T00:00:00Z|Anglais    |
|COVID-19 pandemic in North Macedonia                                                                                                  |2020-03-04T00:00:00Z|Anglais    |
|COVID-19 pandemic in Grenada                                                                                                          |2020-03-17T00:00:00Z|Anglais    |
|COVID-19 misinformation by the United States                                                                                          |2020-12-30T00:00:00Z|Anglais    |
|COVID-19 pandemic in Estonia                                                                                                          |2020-03-04T00:00:00Z|Anglais    |
|Statistics of the COVID-19 pandemic in Germany                                                                                        |2020-05-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Hubei                                                                                                            |2020-03-09T00:00:00Z|Anglais    |
|COVID-19 pandemic in Papua New Guinea                                                                                                 |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Niger                                                                                                            |2020-03-13T00:00:00Z|Anglais    |
|COVID-19 pandemic in Wallis and Futuna                                                                                                |2020-05-05T00:00:00Z|Anglais    |
|United Nations response to the COVID-19 pandemic                                                                                      |2020-08-02T00:00:00Z|Anglais    |
|Statistics of the COVID-19 pandemic in Thailand                                                                                       |2020-12-08T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in February 2021                                                                                    |2020-08-03T00:00:00Z|Anglais    |
|COVID-19 pandemic in Telangana                                                                                                        |2020-05-05T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in the United Kingdom (July–December 2020)                                                          |2020-09-25T00:00:00Z|Anglais    |
|COVID-19 and cancer                                                                                                                   |2020-05-13T00:00:00Z|Anglais    |
|List of COVID-19 simulation models                                                                                                    |2021-02-21T00:00:00Z|Anglais    |
|Development of COVID-19 tests                                                                                                         |2021-03-25T00:00:00Z|Anglais    |
|COVID-19 datasets                                                                                                                     |2020-07-12T00:00:00Z|Anglais    |
|COVID-19 pandemic in Lesotho                                                                                                          |2020-05-04T00:00:00Z|Anglais    |
|Chloroquine and hydroxychloroquine during the COVID-19 pandemic                                                                       |2021-02-28T00:00:00Z|Anglais    |
|COVID-19 vaccination in Russia                                                                                                        |2021-03-02T00:00:00Z|Anglais    |
|COVID-19 pandemic in Lakshadweep                                                                                                      |2020-05-14T00:00:00Z|Anglais    |
|Impact of the COVID-19 pandemic on children                                                                                           |2020-06-05T00:00:00Z|Anglais    |
|COVID-19 pandemic in Sierra Leone                                                                                                     |2020-03-13T00:00:00Z|Anglais    |
|European Union response to the COVID-19 pandemic                                                                                      |2020-04-07T00:00:00Z|Anglais    |
|Timeline of the COVID-19 pandemic in April 2021                                                                                       |2021-04-01T00:00:00Z|Anglais    |
|Pandémie de Covid-19                                                                                                                  |2020-01-19T00:00:00Z|Français   |
|Pandémie de Covid-19 en France                                                                                                        |2020-02-29T00:00:00Z|Français   |
|Vaccin contre la Covid-19                                                                                                             |2020-03-22T00:00:00Z|Français   |
|Pandémie de Covid-19 par pays et territoire                                                                                           |2020-03-16T00:00:00Z|Français   |
|Conséquences de la pandémie de Covid-19                                                                                               |2020-03-15T00:00:00Z|Français   |
|Pandémie de Covid-19 en Europe                                                                                                        |2020-03-18T00:00:00Z|Français   |
|Liste de personnalités mortes de la Covid-19                                                                                          |2020-03-24T00:00:00Z|Français   |
|Pandémie de Covid-19 en Belgique                                                                                                      |2020-03-05T00:00:00Z|Français   |
|Chronologie de la pandémie de Covid-19                                                                                                |2020-02-25T00:00:00Z|Français   |
|Covid-19 chez l'enfant                                                                                                                |2020-04-23T00:00:00Z|Français   |
|Statistiques sur la pandémie de Covid-19 en France                                                                                    |2020-08-15T00:00:00Z|Français   |
|Confinements liés à la pandémie de Covid-19 en France                                                                                 |2020-03-16T00:00:00Z|Français   |
|Désinformation sur la pandémie de Covid-19                                                                                            |2020-03-26T00:00:00Z|Français   |
|Application concernant la Covid-19                                                                                                    |2020-04-22T00:00:00Z|Français   |
|Pandémie de Covid-19 aux États-Unis                                                                                                   |2020-03-06T00:00:00Z|Français   |
|Vaccin d'AstraZeneca-Oxford contre la Covid-19                                                                                        |2020-12-30T00:00:00Z|Français   |
|Pandémie de Covid-19 en Algérie                                                                                                       |2020-03-10T00:00:00Z|Français   |
|Pandémie de Covid-19 en Haïti                                                                                                         |2020-03-30T00:00:00Z|Français   |
|Pandémie de Covid-19 au Sénégal                                                                                                       |2020-03-19T00:00:00Z|Français   |
|Chronologie de la pandémie de Covid-19 en France                                                                                      |2020-03-24T00:00:00Z|Français   |
|Enquêtes sur l'origine de la Covid-19                                                                                                 |2021-02-05T00:00:00Z|Français   |
|Pandémie de Covid-19 en Suisse                                                                                                        |2020-03-01T00:00:00Z|Français   |
|Pandémie de Covid-19 au Québec                                                                                                        |2020-03-20T00:00:00Z|Français   |
|Développement et recherche de médicaments contre la Covid-19                                                                          |2020-03-24T00:00:00Z|Français   |
|Pandémie de Covid-19 en Afrique                                                                                                       |2020-03-16T00:00:00Z|Français   |
|Paliers d'alerte de la Covid-19 au Québec                                                                                             |2020-09-25T00:00:00Z|Français   |
|Pandémie de Covid-19 au Cameroun                                                                                                      |2019-05-08T00:00:00Z|Français   |
|Pandémie de Covid-19 en Tunisie                                                                                                       |2020-03-15T00:00:00Z|Français   |
|Pandémie de Covid-19 au Maroc                                                                                                         |2020-03-14T00:00:00Z|Français   |
|Pandémie de Covid-19 en Inde                                                                                                          |2020-03-19T00:00:00Z|Français   |
|Chronologie de la pandémie de Covid-19 au Québec                                                                                      |2020-07-15T00:00:00Z|Français   |
|Pandémie de Covid-19 en Italie                                                                                                        |2020-02-27T00:00:00Z|Français   |
|Pandémie de Covid-19 au Togo                                                                                                          |2020-04-09T00:00:00Z|Français   |
|Pandémie de Covid-19 à Madagascar                                                                                                     |2020-04-02T00:00:00Z|Français   |
|Pandémie de Covid-19 à Monaco                                                                                                         |2020-04-02T00:00:00Z|Français   |
|Pandémie de Covid-19 en Israël                                                                                                        |2020-04-10T00:00:00Z|Français   |
|Conseil scientifique Covid-19                                                                                                         |2020-03-18T00:00:00Z|Français   |
|Pandémie de Covid-19 au Mexique                                                                                                       |2020-03-27T00:00:00Z|Français   |
|Pandémie de Covid-19 au Pérou                                                                                                         |2020-04-23T00:00:00Z|Français   |
|Pandémie de Covid-19 au Brésil                                                                                                        |2020-03-25T00:00:00Z|Français   |
|Pandémie de Covid-19 au Canada                                                                                                        |2020-03-11T00:00:00Z|Français   |
|Pandémie de Covid-19 en Chine                                                                                                         |2020-03-09T00:00:00Z|Français   |
|Controverse scientifique sur l'hydroxychloroquine dans la lutte contre la Covid-19                                                    |2020-06-28T00:00:00Z|Français   |
|Liste de vaccins contre la Covid-19 autorisés                                                                                         |2021-03-18T00:00:00Z|Français   |
|Pandémie de Covid-19 en république démocratique du Congo                                                                              |2020-03-22T00:00:00Z|Français   |
|Pandémie de Covid-19 en Espagne                                                                                                       |2020-03-05T00:00:00Z|Français   |
|Pandémie de Covid-19 au Royaume-Uni                                                                                                   |2020-03-07T00:00:00Z|Français   |
|Covid-19 pendant la grossesse                                                                                                         |2020-04-05T00:00:00Z|Français   |
|Pandémie de Covid-19 en Turquie                                                                                                       |2020-03-23T00:00:00Z|Français   |
|Pandémie de Covid-19 au Mali                                                                                                          |2020-03-27T00:00:00Z|Français   |
|Pandémie de Covid-19 à Taïwan                                                                                                         |2020-04-01T00:00:00Z|Français   |
|Politique vaccinale contre la Covid-19                                                                                                |2021-04-24T00:00:00Z|Français   |
|Pandémie de Covid-19 au Burkina Faso                                                                                                  |2020-03-14T00:00:00Z|Français   |
|Pandémie de Covid-19 à La Réunion                                                                                                     |2020-03-21T00:00:00Z|Français   |
|Pandémie de Covid-19 en Allemagne                                                                                                     |2020-03-01T00:00:00Z|Français   |
|Transmission de la Covid-19                                                                                                           |2020-11-12T00:00:00Z|Français   |
|Pandémie de Covid-19 au Bénin                                                                                                         |2020-03-20T00:00:00Z|Français   |
|Pandémie de Covid-19 en Guinée                                                                                                        |2020-03-16T00:00:00Z|Français   |
|Pandémie de Covid-19 en Côte d'Ivoire                                                                                                 |2020-03-26T00:00:00Z|Français   |
|Pandémie de Covid-19 en Suède                                                                                                         |2020-03-21T00:00:00Z|Français   |
|Traitement et gestion de la Covid-19                                                                                                  |2021-03-02T00:00:00Z|Français   |
|Pandémie de Covid-19 en Russie                                                                                                        |2020-03-18T00:00:00Z|Français   |
|Pandémie de Covid-19 à Mayotte                                                                                                        |2020-04-10T00:00:00Z|Français   |
|Pandémie de Covid-19 au Japon                                                                                                         |2020-03-12T00:00:00Z|Français   |
|Modélisation mathématique de l'épidémie Covid-19                                                                                      |2021-04-14T00:00:00Z|Français   |
|Conséquences de la pandémie de Covid-19 sur l'économie du Québec                                                                      |2020-09-23T00:00:00Z|Français   |
|Pandémie de Covid-19 à Cuba                                                                                                           |2020-03-18T00:00:00Z|Français   |
|Pandémie de Covid-19 au Viêt Nam                                                                                                      |2020-04-01T00:00:00Z|Français   |
|Champignon noir (complication liée à la COVID-19)                                                                                     |2021-05-29T00:00:00Z|Français   |
|Pandémie de Covid-19 aux Comores                                                                                                      |2020-05-03T00:00:00Z|Français   |
|Pandémie de Covid-19 à Laval                                                                                                          |2020-09-27T00:00:00Z|Français   |
|Pandémie de Covid-19 au Tchad                                                                                                         |2020-04-05T00:00:00Z|Français   |
|Campagne de vaccination contre la Covid-19 en France                                                                                  |2021-01-09T00:00:00Z|Français   |
|Pandémie de Covid-19 en Colombie                                                                                                      |2020-03-27T00:00:00Z|Français   |
|Pandémie de Covid-19 en Amérique                                                                                                      |2020-03-24T00:00:00Z|Français   |
|Pandémie de Covid-19 au Qatar                                                                                                         |2020-05-07T00:00:00Z|Français   |
|Campagne de vaccination contre la Covid-19 au Québec                                                                                  |2021-03-24T00:00:00Z|Français   |
|Pandémie de Covid-19 au Luxembourg                                                                                                    |2020-03-10T00:00:00Z|Français   |
|Pandémie de Covid-19 en Tchéquie                                                                                                      |2020-03-14T00:00:00Z|Français   |
|Port du masque pendant la pandémie de Covid-19                                                                                        |2020-05-03T00:00:00Z|Français   |
|Pandémie de Covid-19 au Portugal                                                                                                      |2020-03-20T00:00:00Z|Français   |
|Crise économique liée à la pandémie de Covid-19                                                                                       |2020-04-21T00:00:00Z|Français   |
|Pandémie de Covid-19 en Estrie                                                                                                        |2020-09-27T00:00:00Z|Français   |
|Pandémie de Covid-19 à Montréal                                                                                                       |2020-09-25T00:00:00Z|Français   |
|Mouvements d'opposition au port du masque et aux mesures de confinement ou de restrictions des libertés durant la pandémie de Covid-19|2020-08-13T00:00:00Z|Français   |
|Pandémie de Covid-19 en Australie                                                                                                     |2020-03-01T00:00:00Z|Français   |
|Pandémie de Covid-19 en Grèce                                                                                                         |2020-03-31T00:00:00Z|Français   |
|Pandémie de Covid-19 au Salvador                                                                                                      |2021-05-25T00:00:00Z|Français   |
|Pandémie de Covid-19 en Normandie                                                                                                     |2020-05-23T00:00:00Z|Français   |
|Pandémie de Covid-19 en Norvège                                                                                                       |2020-03-13T00:00:00Z|Français   |
|Pandémie de Covid-19 en Asie                                                                                                          |2020-03-24T00:00:00Z|Français   |
|Pandémie de Covid-19 en Éthiopie                                                                                                      |2021-03-24T00:00:00Z|Français   |
|Pandémie de Covid-19 aux Seychelles                                                                                                   |2020-05-07T00:00:00Z|Français   |
|Pandémie de Covid-19 au Rwanda                                                                                                        |2020-04-14T00:00:00Z|Français   |
|Confinement lié à la pandémie de Covid-19 en Italie                                                                                   |2020-03-10T00:00:00Z|Français   |
|Pandémie de Covid-19 en Corée du Sud                                                                                                  |2020-02-25T00:00:00Z|Français   |
|Impact de la pandémie de Covid-19 sur les droits de l'homme                                                                           |2020-06-03T00:00:00Z|Français   |
|Impact environnemental de la pandémie de Covid-19                                                                                     |2020-05-17T00:00:00Z|Français   |
|Pandémie de Covid-19 en Tanzanie                                                                                                      |2021-05-26T00:00:00Z|Français   |
|Rebonds de la pandémie de Covid-19 en Europe                                                                                          |2020-08-16T00:00:00Z|Français   |
|Pandémie de Covid-19 au Burundi                                                                                                       |2020-04-19T00:00:00Z|Français   |
|Pandémie de Covid-19 au Botswana                                                                                                      |2020-07-16T00:00:00Z|Français   |
|Pandémie de Covid-19 en Hongrie                                                                                                       |2020-04-17T00:00:00Z|Français   |
|Pandémie de Covid-19 en Islande                                                                                                       |2020-04-09T00:00:00Z|Français   |
|Pandémie de Covid-19 en Azerbaïdjan                                                                                                   |2020-03-27T00:00:00Z|Français   |
|Pandémie de Covid-19 en Croatie                                                                                                       |2020-04-21T00:00:00Z|Français   |
|Pandémie de Covid-19 au Niger                                                                                                         |2020-04-03T00:00:00Z|Français   |
|Pandémie de Covid-19 à Singapour                                                                                                      |2020-05-18T00:00:00Z|Français   |
|Pandémie de Covid-19 en Andorre                                                                                                       |2020-03-23T00:00:00Z|Français   |
|Pandémie de Covid-19 au Lesotho                                                                                                       |2020-06-21T00:00:00Z|Français   |
|Pandémie de Covid-19 aux Philippines                                                                                                  |2021-01-09T00:00:00Z|Français   |
|Pandémie de Covid-19 en Océanie                                                                                                       |2020-03-20T00:00:00Z|Français   |
|Pandémie de Covid-19 à Saint-Christophe-et-Niévès                                                                                     |2020-05-07T00:00:00Z|Français   |
|Pandémie de Covid-19 en Corée du Nord                                                                                                 |2020-03-11T00:00:00Z|Français   |
|Pandémie de Covid-19 aux Salomon                                                                                                      |2020-10-03T00:00:00Z|Français   |
|Pandémie de Covid-19 en Géorgie                                                                                                       |2020-03-29T00:00:00Z|Français   |
|Pandémie de Covid-19 en Argentine                                                                                                     |2020-03-29T00:00:00Z|Français   |
|Pandémie de Covid-19 en Indonésie                                                                                                     |2020-04-18T00:00:00Z|Français   |
|Pandémie de Covid-19 au Vatican                                                                                                       |2020-04-09T00:00:00Z|Français   |
|Pandémie de Covid-19 en Angola                                                                                                        |2020-03-28T00:00:00Z|Français   |
|Pandémie de Covid-19 aux Fidji                                                                                                        |2020-03-21T00:00:00Z|Français   |
|Pandémie de Covid-19 en Thaïlande                                                                                                     |2020-03-30T00:00:00Z|Français   |
|Pandémie de Covid-19 au Vanuatu                                                                                                       |2020-11-11T00:00:00Z|Français   |
|Pandémie de Covid-19 aux Pays-Bas                                                                                                     |2020-03-14T00:00:00Z|Français   |
|Pandémie de Covid-19 au Saguenay–Lac-Saint-Jean                                                                                       |2020-11-28T00:00:00Z|Français   |
|Pandémie de Covid-19 en Iran                                                                                                          |2020-02-21T00:00:00Z|Français   |
|Pandémie de Covid-19 en Pologne                                                                                                       |2020-03-23T00:00:00Z|Français   |
|Conséquences de la pandémie de Covid-19 sur l'éducation au Québec                                                                     |2020-10-02T00:00:00Z|Français   |
|Pandémie de Covid-19 en Afrique du Sud                                                                                                |2020-03-19T00:00:00Z|Français   |
|Pandémie de Covid-19 en Érythrée                                                                                                      |2020-07-17T00:00:00Z|Français   |
|Pandémie de Covid-19 au Cambodge                                                                                                      |2021-04-15T00:00:00Z|Français   |
|Pandémie de Covid-19 en Irak                                                                                                          |2020-06-21T00:00:00Z|Français   |
|Programme de vaccination contre la Covid-19 du Royaume-Uni                                                                            |2021-01-05T00:00:00Z|Français   |
|Polémiques sanitaires liées à la Covid-19 en France                                                                                   |2020-12-19T00:00:00Z|Français   |
|Pandémie de Covid-19 au Danemark                                                                                                      |2020-03-19T00:00:00Z|Français   |
|Pandémie de Covid-19 à Sao Tomé-et-Principe                                                                                           |2020-05-07T00:00:00Z|Français   |
|Pandémie de Covid-19 au Chili                                                                                                         |2020-03-27T00:00:00Z|Français   |
|Pandémie de Covid-19 dans la Capitale-Nationale                                                                                       |2020-09-27T00:00:00Z|Français   |
|Pandémie de Covid-19 en Équateur                                                                                                      |2020-03-27T00:00:00Z|Français   |
|Conséquences de la pandémie de Covid-19 sur les religions                                                                             |2020-06-09T00:00:00Z|Français   |
|Pandémie de Covid-19 en Bolivie                                                                                                       |2020-05-13T00:00:00Z|Français   |
|Pandémie de Covid-19 en République centrafricaine                                                                                     |2020-07-16T00:00:00Z|Français   |
|Pandémie de Covid-19 en Polynésie française                                                                                           |2020-03-27T00:00:00Z|Français   |
|Confinements liés à la pandémie de Covid-19 au Québec                                                                                 |2021-01-16T00:00:00Z|Français   |
|Pandémie de Covid-19 en Irlande                                                                                                       |2020-04-02T00:00:00Z|Français   |
|Pandémie de Covid-19 au Yémen                                                                                                         |2020-05-07T00:00:00Z|Français   |
|Pandémie de Covid-19 au Cap-Vert                                                                                                      |2020-05-06T00:00:00Z|Français   |
|Pandémie de Covid-19 au Mozambique                                                                                                    |2021-05-14T00:00:00Z|Français   |
|Musées des Offices au temps de la pandémie de Covid-19                                                                                |2021-03-20T00:00:00Z|Français   |
|Pandémie de Covid-19 aux Palaos                                                                                                       |2020-06-06T00:00:00Z|Français   |
|Pandémie de Covid-19 dans les Visayas centrales                                                                                       |2021-01-08T00:00:00Z|Français   |
|Pandémie de Covid-19 en Slovaquie                                                                                                     |2020-04-17T00:00:00Z|Français   |
|Pandémie de Covid-19 en République dominicaine                                                                                        |2020-04-30T00:00:00Z|Français   |
|Pandémie de Covid-19 en Guyane                                                                                                        |2020-04-15T00:00:00Z|Français   |
|Pénuries liées à la pandémie de Covid-19                                                                                              |2020-04-26T00:00:00Z|Français   |
|Pandémie de Covid-19 en Nouvelle-Calédonie                                                                                            |2020-04-15T00:00:00Z|Français   |
|Pandémie de Covid-19 en Ukraine                                                                                                       |2020-03-27T00:00:00Z|Français   |
|Stigmatisation sociale associée à la Covid-19                                                                                         |2020-08-16T00:00:00Z|Français   |
|Pseudo-traitements contre la Covid-19                                                                                                 |2020-12-27T00:00:00Z|Français   |
|Pandémie de Covid-19 en Martinique                                                                                                    |2020-04-15T00:00:00Z|Français   |
|Pandémie de Covid-19 au Monténégro                                                                                                    |2020-05-07T00:00:00Z|Français   |
|Pandémie de Covid-19 en Guadeloupe                                                                                                    |2020-04-15T00:00:00Z|Français   |
|Applaudissements aux fenêtres pendant la pandémie de Covid-19                                                                         |2020-06-18T00:00:00Z|Français   |
|Pandémie de Covid-19 en Roumanie                                                                                                      |2020-03-21T00:00:00Z|Français   |
|Pandémie de Covid-19 à Trinité-et-Tobago                                                                                              |2020-05-07T00:00:00Z|Français   |
|Campagne de vaccination contre la Covid-19 en Belgique                                                                                |2021-03-06T00:00:00Z|Français   |
|Pandémie de Covid-19 en Égypte                                                                                                        |2020-03-23T00:00:00Z|Français   |
|Sécurité et respect de la vie privée des applications Covid-19                                                                        |2020-10-14T00:00:00Z|Français   |
|Pandémie de Covid-19 en Autriche                                                                                                      |2020-03-22T00:00:00Z|Français   |
|Xénophobie et racisme liés à la pandémie de Covid-19                                                                                  |2020-02-26T00:00:00Z|Français   |
|Conséquences de la pandémie de Covid-19 sur les personnes âgées                                                                       |2020-05-29T00:00:00Z|Français   |
|Pandémie de Covid-19 en Estonie                                                                                                       |2020-04-20T00:00:00Z|Français   |
|Pandémie de Covid-19 au Gabon                                                                                                         |2021-03-24T00:00:00Z|Français   |
|Pandémie de Covid-19 au Zimbabwe                                                                                                      |2020-04-11T00:00:00Z|Français   |
|Pandémie de Covid-19 en Biélorussie                                                                                                   |2020-03-29T00:00:00Z|Français   |
|Pandémie de Covid-19 en Ouganda                                                                                                       |2020-03-28T00:00:00Z|Français   |
|Pandémie de Covid-19 au Bangladesh                                                                                                    |2021-05-14T00:00:00Z|Français   |
|Pandémie de Covid-19 en Nouvelle-Zélande                                                                                              |2020-03-18T00:00:00Z|Français   |
|Pandémie de Covid-19 en Papouasie-Nouvelle-Guinée                                                                                     |2020-03-21T00:00:00Z|Français   |
|Pandémie de Covid-19 dans l'art et la culture                                                                                         |2020-09-19T00:00:00Z|Français   |
|Pandémie de Covid-19 à Saint-Marin                                                                                                    |2020-04-14T00:00:00Z|Français   |
|Pandémie de Covid-19 en Malaisie                                                                                                      |2020-04-05T00:00:00Z|Français   |
|Impact de la pandémie de Covid-19 sur la télévision aux États-Unis                                                                    |2021-05-07T00:00:00Z|Français   |
|Pandémie de Covid-19 au Timor oriental                                                                                                |2020-07-17T00:00:00Z|Français   |
|Pandémie de Covid-19 en Arabie saoudite                                                                                               |2020-05-25T00:00:00Z|Français   |
|Pandémie de Covid-19 aux États fédérés de Micronésie                                                                                  |2021-01-12T00:00:00Z|Français   |
|Pandémie de Covid-19 en Finlande                                                                                                      |2020-03-25T00:00:00Z|Français   |
|Confinement lié à la pandémie de Covid-19 en Espagne                                                                                  |2020-03-15T00:00:00Z|Français   |
|Inégalités hommes-femmes face à la pandémie de Covid-19                                                                               |2020-04-07T00:00:00Z|Français   |
|Pandémie de Covid-19 aux Maldives                                                                                                     |2020-05-07T00:00:00Z|Français   |
|Pandémie de Covid-19 en Eswatini                                                                                                      |2020-05-07T00:00:00Z|Français   |
|Impact de la pandémie de Covid-19 sur le cinéma                                                                                       |2021-01-19T00:00:00Z|Français   |
|Pandémie de Covid-19 au Groenland                                                                                                     |2020-04-17T00:00:00Z|Français   |
|Pandémie de Covid-19 en Guinée équatoriale                                                                                            |2021-05-16T00:00:00Z|Français   |
|Pandémie de Covid-19 au Kenya                                                                                                         |2020-03-23T00:00:00Z|Français   |
|Pandémie de Covid-19 à Saint-Pierre-et-Miquelon                                                                                       |2020-04-15T00:00:00Z|Français   |
|Pandémie de Covid-19 en Montérégie                                                                                                    |2021-03-27T00:00:00Z|Français   |
|Pandémie de Covid-19 aux Samoa                                                                                                        |2020-11-19T00:00:00Z|Français   |
|COVID-19                                                                                                                              |2020-01-14T00:00:00Z|Allemand   |
|COVID-19-Pandemie                                                                                                                     |2020-01-25T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Deutschland                                                                                                      |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Österreich                                                                                                       |2020-03-01T00:00:00Z|Allemand   |
|Liste von Todesopfern der COVID-19-Pandemie                                                                                           |2020-03-27T00:00:00Z|Allemand   |
|COVID-19-Impfung in Deutschland                                                                                                       |2020-12-04T00:00:00Z|Allemand   |
|Falschinformationen zur COVID-19-Pandemie                                                                                             |2020-04-08T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Deutschland/Statistik                                                                                            |2020-06-15T00:00:00Z|Allemand   |
|COVID-19-Pandemie/Statistik                                                                                                           |2020-03-18T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Frankreich                                                                                                       |2020-03-04T00:00:00Z|Allemand   |
|Priorisierung der COVID-19-Impfmaßnahmen                                                                                              |2020-11-09T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Israel                                                                                                           |2020-03-18T00:00:00Z|Allemand   |
|COVID-19-Pandemie in den Vereinigten Staaten                                                                                          |2020-03-08T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Schweden                                                                                                         |2020-03-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Schweiz                                                                                                      |2020-02-29T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Indien                                                                                                           |2020-03-19T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Vereinigten Königreich                                                                                           |2020-03-15T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Italien                                                                                                          |2020-02-23T00:00:00Z|Allemand   |
|COVID-19-App                                                                                                                          |2020-04-02T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Volksrepublik China                                                                                          |2020-01-25T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Belgien                                                                                                          |2020-03-16T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Brasilien                                                                                                        |2020-03-20T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Bayern                                                                                                           |2020-03-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Königreich der Niederlande                                                                                       |2020-03-14T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Europa                                                                                                           |2021-04-29T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Russland                                                                                                         |2020-03-29T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Sachsen                                                                                                          |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Schutzmaßnahmen-Ausnahmenverordnung                                                                                          |2021-05-05T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Niedersachsen                                                                                                    |2020-03-29T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Hessen                                                                                                           |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Venezuela                                                                                                        |2020-03-31T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Vietnam                                                                                                          |2020-03-14T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Tschechien                                                                                                       |2020-03-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Nordkorea                                                                                                        |2020-03-28T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Hamburg                                                                                                          |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Japan                                                                                                            |2020-03-14T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Namibia                                                                                                          |2020-04-22T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Kroatien                                                                                                         |2020-03-23T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Türkei                                                                                                       |2020-03-27T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Ischgl                                                                                                           |2004-01-23T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Dänemark                                                                                                         |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Kosovo                                                                                                           |2020-04-01T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Australien                                                                                                       |2020-03-29T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Nordrhein-Westfalen                                                                                              |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Taiwan                                                                                                           |2020-01-25T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Peru                                                                                                             |2020-04-08T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Neuseeland                                                                                                       |2020-03-29T00:00:00Z|Allemand   |
|Proteste gegen Schutzmaßnahmen wegen der COVID-19-Pandemie in Deutschland                                                             |2020-03-31T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Griechenland                                                                                                     |2020-03-24T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Berlin                                                                                                           |2020-03-24T00:00:00Z|Allemand   |
|COVID-19-Insolvenzaussetzungsgesetz                                                                                                   |2020-03-29T00:00:00Z|Allemand   |
|COVID-19 Case-Cluster-Study                                                                                                           |2020-04-11T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Kanada                                                                                                           |2020-03-20T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Chile                                                                                                            |2020-01-25T00:00:00Z|Allemand   |
|Chronik der COVID-19-Pandemie in den Vereinigten Staaten                                                                              |2020-03-08T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Norwegen                                                                                                         |2020-03-12T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Portugal                                                                                                         |2020-03-20T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Kambodscha                                                                                                       |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Deutschland/Testung                                                                                              |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Afrika                                                                                                           |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Spanien                                                                                                          |2020-03-12T00:00:00Z|Allemand   |
|Proteste in Deutschland während der COVID-19-Pandemie                                                                                 |2020-03-31T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Tansania                                                                                                         |2020-04-07T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Südkorea                                                                                                         |2020-02-27T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Mecklenburg-Vorpommern                                                                                           |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Irland                                                                                                           |2020-03-31T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Finnland                                                                                                         |2020-03-14T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Ägypten                                                                                                          |2020-03-22T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Serbien                                                                                                          |2020-04-01T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Bahrain                                                                                                          |2020-04-08T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Baden-Württemberg                                                                                                |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Weißrussland                                                                                                     |2020-04-01T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Polen                                                                                                            |2020-03-14T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Thailand                                                                                                         |2020-04-05T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Deutschland/Chronik der Ausbreitung                                                                              |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Deutschland/Statistik/2020                                                                                       |2021-04-04T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Albanien                                                                                                         |2020-03-29T00:00:00Z|Allemand   |
|COVID-19-Pandemie auf den Seychellen                                                                                                  |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Rheinland-Pfalz                                                                                                  |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Kreis Heinsberg                                                                                                  |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Studie                                                                                                                       |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Iran                                                                                                             |2020-03-06T00:00:00Z|Allemand   |
|Sozioökonomische Auswirkungen der COVID-19-Pandemie                                                                                   |2020-01-25T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Singapur                                                                                                         |2020-03-30T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Luxemburg                                                                                                        |2020-03-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Mexiko                                                                                                           |2020-03-30T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Kuba                                                                                                             |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Malaysia                                                                                                         |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Freien Hansestadt Bremen                                                                                     |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Südafrika                                                                                                        |2020-03-24T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Ukraine                                                                                                      |2020-03-03T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Uruguay                                                                                                          |2020-04-06T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Ungarn                                                                                                           |2020-03-30T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Madagaskar                                                                                                       |2020-04-07T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Dominikanischen Republik                                                                                     |2020-04-06T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Argentinien                                                                                                      |2020-03-30T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Nordmazedonien                                                                                                   |2020-04-01T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Bosnien und Herzegowina                                                                                          |2020-04-01T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Indonesien                                                                                                       |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Bolivien                                                                                                         |2020-04-08T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Rumänien                                                                                                         |2020-03-31T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Bulgarien                                                                                                        |2020-04-01T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Mauritius                                                                                                        |2020-04-11T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Nepal                                                                                                            |2020-04-08T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Slowenien                                                                                                        |2020-04-01T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Zypern                                                                                                           |2020-03-31T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Uganda                                                                                                           |2020-04-10T00:00:00Z|Allemand   |
|Liste der infolge der COVID-19-Pandemie erlassenen deutschen Gesetze und Verordnungen                                                 |2020-03-24T00:00:00Z|Allemand   |
|COVID-19-Pandemie auf den Philippinen                                                                                                 |2020-04-05T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Slowakei                                                                                                     |2020-04-01T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Afghanistan                                                                                                      |2020-03-30T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Katar                                                                                                            |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Antarktis                                                                                                    |2020-12-22T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Schleswig-Holstein                                                                                               |2020-02-26T00:00:00Z|Allemand   |
|Umweltauswirkungen der COVID-19-Pandemie                                                                                              |2020-04-19T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Sachsen-Anhalt                                                                                                   |2020-02-26T00:00:00Z|Allemand   |
|Chronik der COVID-19-Pandemie in den Vereinigten Staaten 2020                                                                         |2021-02-20T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Brandenburg                                                                                                      |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Ecuador                                                                                                          |2020-04-09T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Kolumbien                                                                                                        |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Montenegro                                                                                                       |2020-04-01T00:00:00Z|Allemand   |
|Sanofi–GSK COVID-19-Impfstoff                                                                                                         |2021-05-18T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Bhutan                                                                                                           |2020-04-10T00:00:00Z|Allemand   |
|Juristische Beurteilung der Maßnahmen gegen die COVID-19-Pandemie in Deutschland                                                      |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Nigeria                                                                                                          |2020-04-11T00:00:00Z|Allemand   |
|Schulpräsenzpflicht in der COVID-19-Pandemie                                                                                          |2021-02-20T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Saarland                                                                                                         |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie auf Schiffen                                                                                                        |2020-01-25T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Turkmenistan                                                                                                     |2020-04-08T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Aserbaidschan                                                                                                    |2020-04-06T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Island                                                                                                           |2020-03-03T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Republik Arzach                                                                                              |2020-05-20T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Marokko                                                                                                          |2020-03-06T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Libanon                                                                                                          |2020-04-06T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Thüringen                                                                                                        |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Osttimor                                                                                                         |2020-03-12T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Jemen                                                                                                            |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Fidschi                                                                                                          |2020-03-15T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Saudi-Arabien                                                                                                    |2020-04-08T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Liechtenstein                                                                                                    |2020-02-29T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Georgien                                                                                                         |2020-04-08T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Andorra                                                                                                          |2020-03-23T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Botswana                                                                                                         |2020-03-13T00:00:00Z|Allemand   |
|Gesetz zur Abmilderung der Folgen der COVID-19-Pandemie im Zivil-, Insolvenz- und Strafverfahrensrecht                                |2020-03-29T00:00:00Z|Allemand   |
|Störungen des Lernprozesses in Bildungseinrichtungen während der COVID-19-Pandemie in Deutschland                                     |2021-06-03T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Zentralafrikanischen Republik                                                                                |2020-04-06T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Mongolei                                                                                                     |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie/Statistik/2HJ2020                                                                                                   |2020-09-03T00:00:00Z|Allemand   |
|COVID-19-Pandemie in den Vereinigten Arabischen Emiraten                                                                              |2020-04-06T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Gibraltar                                                                                                        |2020-03-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Paraguay                                                                                                         |2020-04-12T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Kenia                                                                                                            |2020-04-07T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Vatikanstadt                                                                                                 |2020-03-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie auf Réunion                                                                                                         |2020-05-24T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Jordanien                                                                                                        |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Armenien                                                                                                         |2020-04-07T00:00:00Z|Allemand   |
|COVID-19-Pandemie in San Marino                                                                                                       |2020-03-14T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Monaco                                                                                                           |2020-03-04T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Litauen                                                                                                          |2020-04-01T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Malta                                                                                                            |2020-04-01T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Äthiopien                                                                                                        |2020-04-01T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Niger                                                                                                            |2020-04-07T00:00:00Z|Allemand   |
|Absagen und Einschränkungen von Veranstaltungen in Deutschland aufgrund der COVID-19-Pandemie                                         |2020-03-16T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Irak                                                                                                             |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Deutschland/Stellungnahmen der Leopoldina                                                                        |2021-03-07T00:00:00Z|Allemand   |
|Access to COVID-19 Tools (ACT) Accelerator                                                                                            |2020-07-19T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Landkreis Tirschenreuth                                                                                          |2020-04-05T00:00:00Z|Allemand   |
|COVID-19-Pandemie auf den Malediven                                                                                                   |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Türkischen Republik Nordzypern                                                                               |2020-05-21T00:00:00Z|Allemand   |
|COVID-19-Pandemie in den palästinensischen Autonomiegebieten                                                                          |2020-03-28T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Kamerun                                                                                                          |2020-03-06T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Palau                                                                                                            |2020-04-15T00:00:00Z|Allemand   |
|Software zur Bekämpfung der COVID-19-Pandemie                                                                                         |2021-04-10T00:00:00Z|Allemand   |
|Folgen der COVID-19-Pandemie für das Bildungs- und Erziehungssystem in Deutschland                                                    |2020-02-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Tunesien                                                                                                         |2020-03-30T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Honduras                                                                                                         |2020-05-16T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Algerien                                                                                                         |2020-04-07T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Pakistan                                                                                                         |2020-04-06T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Kap Verde                                                                                                        |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie auf St. Helena, Ascension und Tristan da Cunha                                                                      |2020-05-27T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Grönland                                                                                                         |2020-03-26T00:00:00Z|Allemand   |
|Fernsehansprache von Angela Merkel anlässlich der COVID-19-Pandemie                                                                   |2020-03-30T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Syrien                                                                                                           |2020-04-08T00:00:00Z|Allemand   |
|COVID-19-Pandemie in New York City                                                                                                    |2020-03-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Demokratischen Republik Kongo                                                                                |2020-04-17T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Benin                                                                                                            |2020-04-15T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Main-Tauber-Kreis                                                                                                |2020-04-01T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Somalia                                                                                                          |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Estland                                                                                                          |2020-03-04T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Transnistrien                                                                                                    |2020-05-18T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Sri Lanka                                                                                                        |2020-04-07T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Guyana                                                                                                           |2020-04-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Südossetien                                                                                                      |2020-05-20T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Gambia                                                                                                           |2020-04-16T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Jamaika                                                                                                          |2020-03-12T00:00:00Z|Allemand   |
|Amtshilfe der Bundeswehr aus Anlass der COVID-19-Pandemie                                                                             |2020-03-28T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Panama                                                                                                           |2020-04-27T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Eswatini                                                                                                         |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Kuwait                                                                                                           |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Nicaragua                                                                                                        |2020-05-15T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Bangladesch                                                                                                      |2020-04-07T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Angola                                                                                                           |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Mali                                                                                                             |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Oman                                                                                                             |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Barbados                                                                                                         |2020-04-21T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Guatemala                                                                                                        |2020-04-14T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Ghana                                                                                                            |2020-04-16T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Mosambik                                                                                                         |2020-04-20T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Lettland                                                                                                         |2020-04-01T00:00:00Z|Allemand   |
|Internationaler Verkehr in der COVID-19-Pandemie                                                                                      |2020-01-25T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Vanuatu                                                                                                          |2020-03-05T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Elfenbeinküste                                                                                               |2020-04-06T00:00:00Z|Allemand   |
|COVID-19-Pandemie auf den Komoren                                                                                                     |2020-03-05T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Myanmar                                                                                                          |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Nauru                                                                                                            |2020-04-14T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Suriname                                                                                                         |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Costa Rica                                                                                                       |2020-05-15T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Kasachstan                                                                                                       |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie auf den Färöern                                                                                                     |2020-03-26T00:00:00Z|Allemand   |
|COVID-19-Pandemie auf den Salomonen                                                                                                   |2020-04-19T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Eritrea                                                                                                          |2020-04-06T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Liberia                                                                                                          |2020-05-12T00:00:00Z|Allemand   |
|COVID-19-Pandemie in der Republik Moldau                                                                                              |2020-04-01T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Samoa                                                                                                            |2020-04-20T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Togo                                                                                                             |2020-03-08T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Ruanda                                                                                                           |2020-04-11T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Guinea                                                                                                           |2020-05-06T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Sudan                                                                                                            |2020-04-15T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Libyen                                                                                                           |2020-05-15T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Abchasien                                                                                                        |2020-05-19T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Kiribati                                                                                                         |2020-04-16T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Südsudan                                                                                                         |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Äquatorialguinea                                                                                                 |2020-04-06T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Brunei                                                                                                           |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Senegal                                                                                                          |2020-04-02T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Sambia                                                                                                           |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Haiti                                                                                                            |2020-05-17T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Burundi                                                                                                          |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Trinidad und Tobago                                                                                              |2020-04-07T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Laos                                                                                                             |2020-04-10T00:00:00Z|Allemand   |
|COVID-19-Pandemie auf Französisch-Polynesien                                                                                          |2020-05-23T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Gabun                                                                                                            |2020-04-15T00:00:00Z|Allemand   |
|COVID-19-Rückholprogramm der deutschen Bundesregierung                                                                                |2020-03-20T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Simbabwe                                                                                                         |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Papua-Neuguinea                                                                                                  |2020-03-31T00:00:00Z|Allemand   |
|COVID-19-Pandemie in El Salvador                                                                                                      |2020-05-04T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Mauretanien                                                                                                      |2020-04-05T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Dominica                                                                                                         |2020-04-24T00:00:00Z|Allemand   |
|COVID-19-Pandemie im Tschad                                                                                                           |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Sierra Leone                                                                                                     |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Dschibuti                                                                                                        |2020-03-13T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Belize                                                                                                           |2020-03-14T00:00:00Z|Allemand   |
|COVID-19-Pandemie in Grenada                                                                                                          |2020-04-27T00:00:00Z|Allemand   |
|Auswirkungen der COVID-19-Pandemie auf die Gesundheitsversorgung                                                                      |2020-05-18T00:00:00Z|Allemand   |
|COVID-19                                                                                                                              |2020-02-11T00:00:00Z|Italien    |
|Pandemia di COVID-19                                                                                                                  |2020-01-22T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Italia                                                                                                        |2020-02-24T00:00:00Z|Italien    |
|Pandemia di COVID-19 nel mondo                                                                                                        |2020-02-27T00:00:00Z|Italien    |
|Vaccino anti COVID-19                                                                                                                 |2020-08-12T00:00:00Z|Italien    |
|Vaccino anti COVID-19 Pfizer-BioNTech                                                                                                 |2020-11-22T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Francia                                                                                                       |2020-02-28T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Europa                                                                                                        |2020-02-26T00:00:00Z|Italien    |
|Vaccino anti COVID-19 AstraZeneca                                                                                                     |2020-10-15T00:00:00Z|Italien    |
|Gestione della pandemia di COVID-19 in Italia                                                                                         |2020-11-19T00:00:00Z|Italien    |
|Pandemia di COVID-19 negli Stati Uniti d'America                                                                                      |2020-02-25T00:00:00Z|Italien    |
|Vaccino anti COVID-19 Moderna                                                                                                         |2020-12-28T00:00:00Z|Italien    |
|Vaccino anti COVID-19 Johnson & Johnson                                                                                               |2021-02-02T00:00:00Z|Italien    |
|Misure di confinamento nel mondo dovute alla pandemia di COVID-19                                                                     |2020-06-26T00:00:00Z|Italien    |
|Cronologia della pandemia di COVID-19                                                                                                 |2020-02-25T00:00:00Z|Italien    |
|Statistiche della pandemia di COVID-19 in Italia                                                                                      |2021-03-19T00:00:00Z|Italien    |
|Pandemia di COVID-19 in India                                                                                                         |2020-05-14T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Asia                                                                                                          |2020-04-20T00:00:00Z|Italien    |
|Test diagnostici per la COVID-19                                                                                                      |2020-03-27T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Cina                                                                                                          |2020-03-23T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Spagna                                                                                                        |2020-02-27T00:00:00Z|Italien    |
|Vaccino anti COVID-19 Novavax                                                                                                         |2021-02-02T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Australia                                                                                                     |2020-05-28T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Thailandia                                                                                                    |2020-07-15T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Brasile                                                                                                       |2020-05-10T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Germania                                                                                                      |2020-02-28T00:00:00Z|Italien    |
|Pandemia di COVID-19 nel Regno Unito                                                                                                  |2020-03-22T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Malta                                                                                                          |2020-10-01T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Argentina                                                                                                     |2020-06-07T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Inghilterra                                                                                                   |2020-09-27T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Russia                                                                                                        |2020-03-18T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Giappone                                                                                                      |2020-02-27T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Albania                                                                                                       |2020-06-05T00:00:00Z|Italien    |
|COVID-19 in gravidanza                                                                                                                |2020-04-05T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Perù                                                                                                          |2020-05-22T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Marocco                                                                                                       |2020-05-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Portogallo                                                                                                    |2020-06-20T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Iran                                                                                                          |2020-03-03T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Israele                                                                                                       |2020-07-06T00:00:00Z|Italien    |
|Pandemia di COVID-19 in America                                                                                                       |2020-04-19T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Bielorussia                                                                                                   |2020-06-13T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Turchia                                                                                                       |2020-05-11T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Svezia                                                                                                        |2020-05-25T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Romania                                                                                                       |2020-06-29T00:00:00Z|Italien    |
|Pandemia di COVID-19 a San Marino                                                                                                     |2020-04-15T00:00:00Z|Italien    |
|Pandemia di COVID-19 sulla Diamond Princess                                                                                           |2020-05-26T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Iraq                                                                                                          |2020-07-22T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Tunisia                                                                                                       |2020-04-13T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Uruguay                                                                                                       |2020-06-12T00:00:00Z|Italien    |
|Pandemia di COVID-19 nella Città del Vaticano                                                                                         |2020-04-07T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Messico                                                                                                       |2020-06-07T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Namibia                                                                                                       |2020-06-12T00:00:00Z|Italien    |
|Pandemia di COVID-19 nella Repubblica Dominicana                                                                                      |2020-06-13T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Africa                                                                                                        |2020-04-19T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Irlanda                                                                                                       |2020-09-30T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Austria                                                                                                       |2020-06-06T00:00:00Z|Italien    |
|Pandemia di COVID-19 nelle Filippine                                                                                                  |2020-04-14T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Cile                                                                                                          |2020-06-01T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Finlandia                                                                                                     |2020-03-06T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Egitto                                                                                                        |2020-05-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Pakistan                                                                                                      |2020-06-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 nel Liechtenstein                                                                                                |2020-05-24T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Mauritius                                                                                                      |2020-06-12T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Senegal                                                                                                       |2020-04-17T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Georgia                                                                                                       |2020-12-28T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Lituania                                                                                                      |2020-06-30T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Bolivia                                                                                                       |2020-06-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Tanzania                                                                                                      |2020-06-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Taiwan                                                                                                         |2020-03-25T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Svizzera                                                                                                      |2020-03-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Azerbaigian                                                                                                   |2020-12-27T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Colombia                                                                                                      |2020-06-07T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Armenia                                                                                                       |2020-05-23T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Cipro                                                                                                          |2020-07-04T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Polonia                                                                                                       |2021-01-20T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Gibilterra                                                                                                     |2020-12-20T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Ungheria                                                                                                      |2020-09-30T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Eritrea                                                                                                       |2020-06-04T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Corea del Sud                                                                                                 |2020-02-24T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Libia                                                                                                         |2020-06-12T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Moldavia                                                                                                      |2021-01-22T00:00:00Z|Italien    |
|Pandemia di COVID-19 sulle navi da crociera                                                                                           |2020-05-12T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Ecuador                                                                                                       |2020-06-07T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Grecia                                                                                                        |2020-08-11T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Slovenia                                                                                                      |2020-03-26T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Oceania                                                                                                       |2020-04-21T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Madagascar                                                                                                    |2020-06-04T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Zimbabwe                                                                                                      |2020-06-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 a New York                                                                                                       |2020-04-11T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Bangladesh                                                                                                    |2020-06-16T00:00:00Z|Italien    |
|Reazioni internazionali alla pandemia di COVID-19 in Italia                                                                           |2020-11-17T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Croazia                                                                                                       |2020-07-23T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Belgio                                                                                                        |2020-03-29T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Nigeria                                                                                                       |2020-06-12T00:00:00Z|Italien    |
|Commissione vaticana COVID-19                                                                                                         |2021-01-06T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Bulgaria                                                                                                      |2020-06-26T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Norvegia                                                                                                      |2020-10-02T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Venezuela                                                                                                     |2020-06-07T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Macao                                                                                                          |2020-03-27T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Scozia                                                                                                        |2020-09-30T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Florida                                                                                                       |2020-12-28T00:00:00Z|Italien    |
|Pandemia di COVID-19 in California                                                                                                    |2021-01-01T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Danimarca                                                                                                     |2020-10-01T00:00:00Z|Italien    |
|Restrizioni agli spostamenti correlate alla pandemia di COVID-19                                                                      |2020-03-31T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Botswana                                                                                                      |2020-06-02T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Cuba                                                                                                           |2020-06-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Kenya                                                                                                         |2020-06-12T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Giamaica                                                                                                      |2020-06-10T00:00:00Z|Italien    |
|Test rapido antigene COVID-19                                                                                                         |2021-05-29T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Costa d'Avorio                                                                                                |2020-06-03T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Lesotho                                                                                                       |2020-06-05T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Mozambico                                                                                                     |2020-05-30T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Hong Kong                                                                                                      |2020-04-14T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Indonesia                                                                                                     |2020-08-02T00:00:00Z|Italien    |
|Pandemia di COVID-19 nelle Comore                                                                                                     |2020-05-21T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Sudafrica                                                                                                     |2020-05-07T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Singapore                                                                                                      |2020-12-19T00:00:00Z|Italien    |
|Pandemia di COVID-19 nei Paesi Bassi                                                                                                  |2020-03-24T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Togo                                                                                                          |2020-06-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Islanda                                                                                                       |2020-03-27T00:00:00Z|Italien    |
|Pandemia di COVID-19 nel Principato di Monaco                                                                                         |2020-06-26T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Serbia                                                                                                        |2020-10-02T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Guernsey                                                                                                       |2020-12-19T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Kosovo                                                                                                        |2021-01-20T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Burkina Faso                                                                                                  |2020-06-02T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Estonia                                                                                                       |2020-05-25T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Algeria                                                                                                       |2020-06-02T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Alaska                                                                                                        |2020-12-31T00:00:00Z|Italien    |
|Misure di confinamento in Spagna dovute alla pandemia di COVID-19                                                                     |2020-05-31T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Ghana                                                                                                         |2020-06-10T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Texas                                                                                                         |2020-10-05T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Etiopia                                                                                                       |2020-06-10T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Uganda                                                                                                        |2020-06-11T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Capo Verde                                                                                                     |2020-06-03T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Afghanistan                                                                                                   |2020-06-11T00:00:00Z|Italien    |
|Pandemia di COVID-19 nell'Isola di Man                                                                                                |2020-12-17T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Massachusetts                                                                                                 |2021-01-03T00:00:00Z|Italien    |
|Pandemia di COVID-19 nella Repubblica Democratica del Congo                                                                           |2020-06-11T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Galles                                                                                                        |2020-09-25T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Ruanda                                                                                                        |2020-06-05T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Irlanda del Nord                                                                                              |2020-09-30T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Panama                                                                                                         |2020-06-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Guinea                                                                                                        |2020-05-24T00:00:00Z|Italien    |
|Pandemia di COVID-19 ad Akrotiri e Dhekelia                                                                                           |2020-12-26T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Mali                                                                                                          |2020-06-11T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Lettonia                                                                                                      |2020-06-30T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Benin                                                                                                         |2020-05-31T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Angola                                                                                                        |2020-05-23T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Washington (stato)                                                                                             |2021-01-06T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Sierra Leone                                                                                                  |2020-06-10T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Paraguay                                                                                                      |2020-06-12T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Lussemburgo                                                                                                   |2020-06-29T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Macedonia del Nord                                                                                            |2020-07-17T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Liberia                                                                                                       |2020-06-05T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Ciad                                                                                                          |2020-05-24T00:00:00Z|Italien    |
|COVID-19 e medicina tradizionale cinese                                                                                               |2020-12-30T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Niger                                                                                                         |2020-06-05T00:00:00Z|Italien    |
|Pandemia di COVID-19 ad Andorra                                                                                                       |2020-03-28T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Brunei                                                                                                        |2020-06-20T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Gabon                                                                                                         |2020-06-04T00:00:00Z|Italien    |
|Pandemia di COVID-19 in eSwatini                                                                                                      |2020-06-04T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Kirghizistan                                                                                                  |2020-04-13T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Burundi                                                                                                       |2020-06-01T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Mauritania                                                                                                    |2020-06-10T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Sudan                                                                                                         |2020-06-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Porto Rico                                                                                                     |2020-06-10T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Mississippi                                                                                                   |2021-01-05T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Guinea-Bissau                                                                                                 |2020-06-04T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Washington                                                                                                     |2020-12-30T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Nicaragua                                                                                                     |2020-06-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Gambia                                                                                                        |2020-06-05T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Jersey                                                                                                         |2020-12-18T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Belize                                                                                                        |2020-06-15T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Malawi                                                                                                        |2020-04-13T00:00:00Z|Italien    |
|Pandemia di COVID-19 nelle Seychelles                                                                                                 |2020-06-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 nella Repubblica del Congo                                                                                       |2020-06-10T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Zambia                                                                                                        |2020-06-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Arabia Saudita                                                                                                |2020-05-24T00:00:00Z|Italien    |
|Pandemia di COVID-19 a Bonaire                                                                                                        |2020-07-08T00:00:00Z|Italien    |
|Pandemia di COVID-19 in Sudan del Sud                                                                                                 |2020-06-11T00:00:00Z|Italien    |
|COVID-19                                                                                                                              |2020-03-02T00:00:00Z|Letton     |
|COVID-19 pandēmija                                                                                                                    |2020-01-24T00:00:00Z|Letton     |
|COVID-19 pandēmija Latvijā                                                                                                            |2020-03-02T00:00:00Z|Letton     |
|COVID-19 pandēmija Krievijā                                                                                                           |2020-04-07T00:00:00Z|Letton     |
|COVID-19 pandēmija Baltkrievijā                                                                                                       |2020-04-08T00:00:00Z|Letton     |
|COVID-19 diagnosticēšana                                                                                                              |2020-03-02T00:00:00Z|Letton     |
|COVID-19 pandēmija Igaunijā                                                                                                           |2020-04-07T00:00:00Z|Letton     |
|COVID-19 pandēmija Lietuvā                                                                                                            |2020-04-07T00:00:00Z|Letton     |
|COVID-19 vakcīna                                                                                                                      |2021-05-25T00:00:00Z|Letton     |
|COVID-19 pandēmijas statistika Latvijā                                                                                                |2021-04-14T00:00:00Z|Letton     |
|COVID-19 pandēmija Ķīnā                                                                                                               |2020-03-22T00:00:00Z|Letton     |
|Mēmes saistībā ar Covid-19 pandēmijas laiku Latvijā                                                                                   |2021-05-04T00:00:00Z|Letton     |
|COVID-19                                                                                                                              |2020-03-10T00:00:00Z|Lituanien  |
|COVID-19 pandemija                                                                                                                    |2020-01-30T00:00:00Z|Lituanien  |
|COVID-19 greitasis antigeno testas                                                                                                    |2021-05-24T00:00:00Z|Lituanien  |
|COVID-19 vakcina                                                                                                                      |2020-05-25T00:00:00Z|Lituanien  |
|Moderna COVID-19 vakcina                                                                                                              |2021-04-24T00:00:00Z|Lituanien  |
|COVID-19 pandemija Lietuvoje                                                                                                          |2020-03-18T00:00:00Z|Lituanien  |
|COVID-19                                                                                                                              |2020-02-11T00:00:00Z|Portugais  |
|Pandemia de COVID-19                                                                                                                  |2020-01-20T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Brasil                                                                                                        |2020-02-28T00:00:00Z|Portugais  |
|CPI da COVID-19                                                                                                                       |2021-04-14T00:00:00Z|Portugais  |
|Vacina contra a COVID-19                                                                                                              |2020-03-15T00:00:00Z|Portugais  |
|Pandemia de COVID-19 por país                                                                                                         |2020-01-29T00:00:00Z|Portugais  |
|Lista de mortes por COVID-19                                                                                                          |2020-03-21T00:00:00Z|Portugais  |
|Impactos da pandemia de COVID-19                                                                                                      |2020-03-11T00:00:00Z|Portugais  |
|Diagnóstico de COVID-19                                                                                                               |2020-03-14T00:00:00Z|Portugais  |
|Lista de mortes por COVID-19 no Brasil                                                                                                |2020-08-30T00:00:00Z|Portugais  |
|Negacionismo da COVID-19                                                                                                              |2020-04-25T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Portugal                                                                                                      |2020-02-28T00:00:00Z|Portugais  |
|Vacinação contra a COVID-19 no Brasil                                                                                                 |2021-01-18T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Angola                                                                                                        |2020-03-22T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19                                                                                                    |2020-02-04T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na América                                                                                                       |2020-03-07T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Europa                                                                                                        |2020-03-03T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Moçambique                                                                                                    |2020-03-23T00:00:00Z|Portugais  |
|Impactos da pandemia de COVID-19 na educação                                                                                          |2020-04-12T00:00:00Z|Portugais  |
|Estatísticas da pandemia de COVID-19 no Brasil                                                                                        |2020-03-25T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no estado de São Paulo                                                                                           |2020-04-08T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Índia                                                                                                         |2020-03-21T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Itália                                                                                                        |2020-02-29T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Espanha                                                                                                       |2020-03-02T00:00:00Z|Portugais  |
|Impactos da pandemia de COVID-19 nos Jogos Olímpicos de Verão de 2020                                                                 |2020-04-04T00:00:00Z|Portugais  |
|Pandemia de COVID-19 nos Estados Unidos                                                                                               |2020-02-26T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na França                                                                                                        |2020-03-04T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19 em 2019                                                                                            |2020-09-01T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Pará                                                                                                          |2020-05-04T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Amazonas                                                                                                      |2020-04-30T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19 no Brasil                                                                                          |2020-05-09T00:00:00Z|Portugais  |
|Evacuações relacionadas à pandemia de COVID-19                                                                                        |2020-03-14T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Ásia                                                                                                          |2020-02-27T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Oceania                                                                                                       |2020-03-08T00:00:00Z|Portugais  |
|Desinformação na pandemia de COVID-19                                                                                                 |2020-03-27T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no estado do Rio de Janeiro                                                                                      |2020-04-10T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Minas Gerais                                                                                                  |2020-04-12T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na África                                                                                                        |2020-03-02T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Reino Unido                                                                                                   |2020-03-20T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Vietnã                                                                                                        |2021-03-30T00:00:00Z|Portugais  |
|COVID-19 na gravidez                                                                                                                  |2020-04-11T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Irã                                                                                                           |2020-02-29T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Maranhão                                                                                                      |2020-04-30T00:00:00Z|Portugais  |
|Impactos da pandemia de COVID-19 nos jogos eletrônicos                                                                                |2020-05-05T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Suécia                                                                                                        |2020-03-01T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Japão                                                                                                         |2020-03-13T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19 em março de 2020                                                                                   |2020-03-20T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Bahia                                                                                                         |2020-04-17T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Canadá                                                                                                        |2020-02-26T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na China continental                                                                                             |2020-02-29T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Bélgica                                                                                                       |2020-04-12T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Antártida                                                                                                     |2020-04-20T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Chéquia                                                                                                       |2020-03-20T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Paraná                                                                                                        |2020-04-17T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Alemanha                                                                                                      |2020-03-04T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Argentina                                                                                                     |2020-03-15T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Catar                                                                                                         |2020-03-19T00:00:00Z|Portugais  |
|Impactos da pandemia de COVID-19 na televisão brasileira                                                                              |2020-04-21T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Suíça                                                                                                         |2020-03-05T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19 na Argentina                                                                                       |2020-11-29T00:00:00Z|Portugais  |
|Impactos da pandemia de COVID-19 no futebol                                                                                           |2020-03-30T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em navios de cruzeiro                                                                                            |2020-02-23T00:00:00Z|Portugais  |
|Impactos da pandemia de COVID-19 na política                                                                                          |2020-04-30T00:00:00Z|Portugais  |
|Impactos da pandemia de COVID-19 no desporto                                                                                          |2020-03-27T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no México                                                                                                        |2020-03-24T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19 nos Estados Unidos                                                                                 |2020-04-13T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Israel                                                                                                        |2020-04-12T00:00:00Z|Portugais  |
|Impactos da pandemia de COVID-19 na religião                                                                                          |2020-03-27T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19 em fevereiro de 2020                                                                               |2020-03-20T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Ceará                                                                                                         |2020-05-02T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Pernambuco                                                                                                    |2020-05-02T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Rússia                                                                                                        |2020-03-24T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19 na China continental                                                                               |2020-04-14T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Armênia                                                                                                       |2020-03-17T00:00:00Z|Portugais  |
|Impactos da pandemia de COVID-19 na televisão                                                                                         |2020-03-27T00:00:00Z|Portugais  |
|Impactos da pandemia de COVID-19 na aviação                                                                                           |2020-03-27T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Coreia do Norte                                                                                               |2020-03-17T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Timor-Leste                                                                                                   |2020-03-25T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19 no Japão                                                                                           |2020-05-02T00:00:00Z|Portugais  |
|Impactos da pandemia de COVID-19 na Igreja Católica                                                                                   |2020-08-13T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Rio Grande do Sul                                                                                             |2020-05-04T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Bielorrússia                                                                                                  |2020-03-18T00:00:00Z|Portugais  |
|Recessão causada pela pandemia de COVID-19                                                                                            |2021-03-27T00:00:00Z|Portugais  |
|Impactos da pandemia de COVID-19 no cinema                                                                                            |2020-04-20T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19 em janeiro de 2020                                                                                 |2020-03-19T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Egito                                                                                                         |2020-03-05T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Cuba                                                                                                          |2020-03-17T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Uruguai                                                                                                       |2020-03-26T00:00:00Z|Portugais  |
|Desenvolvimento e pesquisa de medicamentos contra a COVID-19                                                                          |2020-08-05T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Mato Grosso do Sul                                                                                            |2020-05-03T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Piauí                                                                                                         |2020-05-02T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19 em maio de 2020                                                                                    |2020-04-30T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Rio Grande do Norte                                                                                           |2020-05-03T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Tocantins                                                                                                     |2020-05-04T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Paraíba                                                                                                       |2020-05-02T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Alagoas                                                                                                       |2020-05-01T00:00:00Z|Portugais  |
|Pandemia de COVID-19 nos Açores                                                                                                       |2020-04-08T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Madeira                                                                                                       |2020-04-08T00:00:00Z|Portugais  |
|Casos de xenofobia e racismo relacionados à pandemia de COVID-19                                                                      |2020-03-13T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Benim                                                                                                         |2020-03-22T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Burquina Fasso                                                                                                |2020-03-20T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na África do Sul                                                                                                 |2020-03-20T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Espírito Santo                                                                                                |2020-05-03T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Polônia                                                                                                       |2020-11-04T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Gana                                                                                                          |2020-03-19T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Roménia                                                                                                       |2021-01-30T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Hungria                                                                                                       |2020-04-09T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Macau                                                                                                         |2020-04-06T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Acre                                                                                                          |2020-04-30T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Goiás                                                                                                         |2020-05-03T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Coreia do Sul                                                                                                 |2020-02-29T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Arábia Saudita                                                                                                |2020-04-21T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Noruega                                                                                                       |2020-03-27T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Grécia                                                                                                        |2020-03-23T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Roraima                                                                                                       |2020-05-04T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Botswana                                                                                                      |2020-04-03T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Quênia                                                                                                        |2020-03-22T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Austrália                                                                                                     |2020-03-12T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Butão                                                                                                         |2020-03-21T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Rondônia                                                                                                      |2020-05-04T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Maurício                                                                                                      |2020-03-23T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Mato Grosso                                                                                                   |2020-05-03T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Marrocos                                                                                                      |2020-03-21T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Londres                                                                                                       |2020-04-29T00:00:00Z|Portugais  |
|Aplicativos COVID-19                                                                                                                  |2020-05-26T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19 em abril de 2020                                                                                   |2020-04-02T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Suriname                                                                                                      |2020-03-19T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Aruba                                                                                                         |2020-04-10T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Amapá                                                                                                         |2020-05-01T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Sergipe                                                                                                       |2020-05-04T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Albânia                                                                                                       |2020-03-17T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Inglaterra                                                                                                    |2020-04-12T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Líbia                                                                                                         |2020-04-08T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Croácia                                                                                                       |2020-03-19T00:00:00Z|Portugais  |
|Protestos contra respostas à pandemia de COVID-19                                                                                     |2020-05-03T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19 em Angola                                                                                          |2020-04-01T00:00:00Z|Portugais  |
|Comissão Vaticana COVID-19                                                                                                            |2021-01-14T00:00:00Z|Portugais  |
|Testes ao COVID-19                                                                                                                    |2020-03-09T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Paraguai                                                                                                      |2020-03-17T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Finlândia                                                                                                     |2020-03-06T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Palestina                                                                                                     |2020-03-23T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Namíbia                                                                                                       |2020-03-21T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Nicarágua                                                                                                     |2020-03-24T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Iêmen                                                                                                         |2020-05-07T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19 no Reino Unido                                                                                     |2020-05-01T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Argélia                                                                                                       |2020-03-07T00:00:00Z|Portugais  |
|Cronologia da pandemia de COVID-19 na Argélia                                                                                         |2020-04-04T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Sérvia                                                                                                        |2020-03-16T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Irlanda                                                                                                       |2020-03-16T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Afeganistão                                                                                                   |2020-04-19T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Hong Kong                                                                                                     |2020-05-05T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Turquia                                                                                                       |2020-03-24T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Áustria                                                                                                       |2020-03-03T00:00:00Z|Portugais  |
|Pandemia de COVID-19 nas Canárias                                                                                                     |2020-03-22T00:00:00Z|Portugais  |
|NHS COVID-19                                                                                                                          |2020-07-19T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Escócia                                                                                                       |2020-04-21T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Gabão                                                                                                         |2020-03-22T00:00:00Z|Portugais  |
|Pandemia de COVID-19 nas Comores                                                                                                      |2020-05-12T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Seicheles                                                                                                     |2020-03-23T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Malásia                                                                                                       |2020-03-20T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Etiópia                                                                                                       |2020-03-18T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Colômbia                                                                                                      |2020-03-23T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Guiana                                                                                                        |2020-03-18T00:00:00Z|Portugais  |
|Pandemia de COVID-19 no Malawi                                                                                                        |2020-04-08T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Venezuela                                                                                                     |2020-03-18T00:00:00Z|Portugais  |
|Pandemia de COVID-19 em Madagascar                                                                                                    |2020-03-23T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na Bolívia                                                                                                       |2020-03-19T00:00:00Z|Portugais  |
|Pandemia de COVID-19 na cidade de Nova York                                                                                           |2021-02-06T00:00:00Z|Portugais  |
|Pandemia de COVID-19 nas Bermudas                                                                                                     |2020-04-22T00:00:00Z|Portugais  |
|COVID-19                                                                                                                              |2020-02-14T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Elveția                                                                                                       |2020-03-21T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Transnistria                                                                                                  |2020-04-17T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Franța                                                                                                        |2020-03-11T00:00:00Z|Roumain    |
|Pandemia de COVID-19                                                                                                                  |2020-01-22T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Germania                                                                                                      |2020-03-14T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Rusia                                                                                                         |2020-03-21T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Bulgaria                                                                                                      |2020-03-14T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Austria                                                                                                       |2020-03-17T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Statele Unite ale Americii                                                                                    |2020-03-14T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Ungaria                                                                                                       |2020-03-17T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Grecia                                                                                                        |2020-03-21T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Iran                                                                                                          |2020-03-13T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Belgia                                                                                                        |2021-02-27T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Italia                                                                                                        |2020-03-13T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Serbia                                                                                                        |2020-03-22T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în România                                                                                                       |2020-03-10T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Turcia                                                                                                        |2020-03-21T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Republica Moldova                                                                                             |2020-03-12T00:00:00Z|Roumain    |
|Vaccin anti-COVID-19                                                                                                                  |2020-03-14T00:00:00Z|Roumain    |
|Cronologia pandemiei de COVID-19 din 2019-2020                                                                                        |2020-04-01T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Danemarca                                                                                                     |2020-03-17T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Spania                                                                                                        |2020-03-14T00:00:00Z|Roumain    |
|Diagnosticare COVID-19                                                                                                                |2020-03-17T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în China                                                                                                         |2020-03-13T00:00:00Z|Roumain    |
|Medicamente împotriva COVID-19                                                                                                        |2020-03-14T00:00:00Z|Roumain    |
|Pandemia de COVID-19 în Etiopia                                                                                                       |2020-12-15T00:00:00Z|Roumain    |
|Teste rapide de antigeni COVID-19                                                                                                     |2021-05-29T00:00:00Z|Roumain    |
|Pandemia de coronaviroză (COVID-19) în Europa                                                                                         |2020-03-24T00:00:00Z|Roumain    |
|Decese provocate de pandemia de coronaviroză (COVID-19) în România                                                                    |2020-05-20T00:00:00Z|Roumain    |
|Dezinformări privind pandemia de coronaviroză (COVID-19)                                                                              |2020-03-23T00:00:00Z|Roumain    |
|Evoluția COVID-19 în România                                                                                                          |2020-09-02T00:00:00Z|Roumain    |
|Cronologia pandemiei de COVID-19 în noiembrie 2019 - ianuarie 2020                                                                    |2020-04-01T00:00:00Z|Roumain    |
|COVID-19                                                                                                                              |2020-02-11T00:00:00Z|Espagnol   |
|Vacuna contra la COVID-19                                                                                                             |2020-04-14T00:00:00Z|Espagnol   |
|Pandemia de COVID-19                                                                                                                  |2020-01-19T00:00:00Z|Espagnol   |
|Vacuna de Oxford-AstraZeneca para la COVID-19                                                                                         |2020-12-08T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en México                                                                                                        |2020-02-29T00:00:00Z|Espagnol   |
|COVID-19 en el embarazo                                                                                                               |2020-03-23T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Hidalgo                                                                                                       |2020-05-25T00:00:00Z|Espagnol   |
|Vacuna de Pfizer-BioNTech para la COVID-19                                                                                            |2020-11-11T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en México                                                                                               |2021-01-07T00:00:00Z|Espagnol   |
|Vacuna de Johnson & Johnson para la COVID-19                                                                                          |2021-01-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en América                                                                                                       |2020-03-09T00:00:00Z|Espagnol   |
|Pruebas de COVID-19                                                                                                                   |2020-03-25T00:00:00Z|Espagnol   |
|Vacunación contra el COVID-19 en Uruguay                                                                                              |2021-03-05T00:00:00Z|Espagnol   |
|Negacionismo de la COVID-19                                                                                                           |2020-06-15T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Argentina                                                                                            |2020-12-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Bolivia                                                                                                       |2020-03-11T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Guyana                                                                                                        |2020-03-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en la Polinesia Francesa                                                                                         |2021-05-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Veracruz                                                                                                      |2020-05-20T00:00:00Z|Espagnol   |
|Vacuna de Moderna para la COVID-19                                                                                                    |2020-12-27T00:00:00Z|Espagnol   |
|Comando de Operaciones COVID-19                                                                                                       |2020-08-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Islas Malvinas                                                                                                |2020-04-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Venezuela                                                                                                     |2020-03-13T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Chile                                                                                                         |2020-03-09T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Ecuador                                                                                                       |2020-03-09T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Canadá                                                                                               |2021-02-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Cuba                                                                                                          |2020-03-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Perú                                                                                                          |2020-03-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Panamá                                                                                                        |2020-03-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Nicaragua                                                                                                     |2020-03-22T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Europa                                                                                                        |2020-03-13T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en el Reino Unido                                                                                                |2020-03-19T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Uruguay                                                                                                       |2020-03-13T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Surinam                                                                                                       |2020-03-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en la India                                                                                                      |2020-03-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Guatemala                                                                                                     |2020-03-14T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Isla de Pascua                                                                                                |2020-04-09T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Guayana Francesa                                                                                              |2020-03-19T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Rumania                                                                                              |2021-02-05T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Australia                                                                                            |2021-02-22T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Brasil                                                                                                        |2020-03-08T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Estados Unidos                                                                                       |2021-03-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en República Centroafricana                                                                                      |2020-06-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Montenegro                                                                                                    |2021-05-15T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Michoacán                                                                                                     |2020-06-21T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Honduras                                                                                                      |2020-03-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en San Bartolomé                                                                                                 |2020-12-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 por país y territorio                                                                                            |2020-04-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en República Checa                                                                                               |2020-06-26T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Islas Marianas del Norte                                                                                      |2020-05-11T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Guinea-Bisáu                                                                                                  |2020-05-05T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Costa Rica                                                                                                    |2020-03-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Chihuahua                                                                                                     |2020-06-29T00:00:00Z|Espagnol   |
|Impacto de la pandemia de COVID-19 por sexo                                                                                           |2020-04-09T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en República Popular China                                                                                       |2020-03-18T00:00:00Z|Espagnol   |
|Salud mental durante la pandemia de COVID-19                                                                                          |2020-06-25T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Nuevo León                                                                                                    |2020-06-13T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en República de China                                                                                            |2020-04-15T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Coahuila de Zaragoza                                                                                          |2020-06-14T00:00:00Z|Espagnol   |
|Impacto de la pandemia de COVID-19 en la ciencia y la tecnología                                                                      |2020-07-06T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Beni (Bolivia)                                                                                                |2020-06-19T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en República Democrática del Congo                                                                               |2020-06-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Sonora                                                                                                        |2020-04-13T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Estado de México                                                                                              |2020-05-20T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Tacna                                                                                                |2021-05-05T00:00:00Z|Espagnol   |
|Decoding COVID-19                                                                                                                     |2020-06-07T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Cajamarca                                                                                            |2021-05-04T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Ayacucho                                                                                             |2021-05-05T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Cuzco                                                                                                |2021-05-04T00:00:00Z|Espagnol   |
|Confinamiento por la pandemia de COVID-19                                                                                             |2020-03-18T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Colombia                                                                                                      |2020-03-11T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en España                                                                                                        |2020-03-09T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en los Estados Unidos                                                                                            |2020-03-07T00:00:00Z|Espagnol   |
|Impacto socioeconómico de la pandemia de COVID-19                                                                                     |2020-03-12T00:00:00Z|Espagnol   |
|Impacto de la pandemia de COVID-19 en la industria de la música                                                                       |2020-07-18T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Bosnia y Herzegovina                                                                                          |2020-09-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Argentina                                                                                                     |2020-03-09T00:00:00Z|Espagnol   |
|Protestas contra el confinamiento por la pandemia de COVID-19                                                                         |2020-04-21T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Guatemala                                                                                            |2021-04-26T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Perú                                                                                                 |2021-01-27T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Honduras                                                                                             |2021-04-17T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Bolivia                                                                                              |2021-03-31T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Venezuela                                                                                            |2021-04-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en África                                                                                                        |2020-05-31T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en El Salvador                                                                                          |2021-04-01T00:00:00Z|Espagnol   |
|Impacto en la aviación de la pandemia de COVID-19                                                                                     |2020-05-11T00:00:00Z|Espagnol   |
|Transmisión de COVID-19                                                                                                               |2020-09-02T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Oceanía                                                                                                       |2020-05-10T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Granada                                                                                                       |2020-04-16T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en El Salvador                                                                                                   |2020-03-19T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Colombia                                                                                             |2021-02-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Indonesia                                                                                                     |2020-06-29T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Chile                                                                                                |2021-01-07T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Asia                                                                                                          |2020-03-16T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Israel                                                                                                        |2020-03-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Albania                                                                                                       |2020-04-07T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Paraguay                                                                                             |2021-02-08T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Fiyi                                                                                                          |2020-09-05T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Guayana Esequiba                                                                                              |2020-04-30T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Sudáfrica                                                                                                     |2020-06-25T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Ecuador                                                                                              |2021-01-25T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Angola                                                                                                        |2020-06-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Belice                                                                                                        |2020-05-01T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Moquegua                                                                                             |2021-05-05T00:00:00Z|Espagnol   |
|Comisión vaticana COVID-19                                                                                                            |2020-11-11T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Zambia                                                                                                        |2020-08-17T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Portugal                                                                                                      |2020-03-14T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Sahara Occidental                                                                                             |2020-04-28T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Lambayeque                                                                                           |2021-05-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Filipinas                                                                                                     |2020-11-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Francia                                                                                                       |2020-03-18T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Antártida                                                                                                     |2020-04-28T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Armenia                                                                                                       |2020-06-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Nueva Zelanda                                                                                                 |2020-04-30T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Islandia                                                                                                      |2020-06-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Baréin                                                                                                        |2020-08-17T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Montserrat                                                                                                    |2020-12-11T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Dominica                                                                                                      |2020-05-06T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Costa Rica                                                                                           |2021-04-18T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Guam                                                                                                          |2020-07-08T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Pakistán                                                                                                      |2020-06-27T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Hidalgo                                                                                              |2021-05-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Serbia                                                                                                        |2020-06-26T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Benín                                                                                                         |2020-09-15T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Camboya                                                                                                       |2020-09-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Bután                                                                                                         |2020-06-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Uzbekistán                                                                                                    |2020-05-15T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Afganistán                                                                                                    |2020-06-26T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Chad                                                                                                          |2020-06-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Tanzania                                                                                                      |2020-12-13T00:00:00Z|Espagnol   |
|Recesión por la pandemia de COVID-19                                                                                                  |2020-10-02T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Nigeria                                                                                                       |2020-06-25T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Botsuana                                                                                                      |2020-06-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Vanuatu                                                                                                       |2020-11-11T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Marruecos                                                                                                     |2020-03-26T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Gabón                                                                                                         |2020-11-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Rumania                                                                                                       |2020-08-18T00:00:00Z|Espagnol   |
|Confinamiento por la pandemia de COVID-19 en Perú                                                                                     |2020-03-17T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Kazajistán                                                                                                    |2020-07-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Brunéi                                                                                                        |2020-06-28T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Catar                                                                                                         |2020-07-21T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Islas Salomón                                                                                                 |2020-10-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Líbano                                                                                                        |2020-08-09T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Guinea Ecuatorial                                                                                             |2020-04-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Massachusetts                                                                                                 |2020-06-05T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en América Central                                                                                               |2020-06-22T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Níger                                                                                                         |2020-06-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Cuzco                                                                                                         |2020-07-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Irlanda                                                                                                       |2020-09-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Anguila                                                                                                       |2020-08-18T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Amazonas (Perú)                                                                                      |2021-05-06T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Lesoto                                                                                                        |2020-06-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Singapur                                                                                                      |2020-06-28T00:00:00Z|Espagnol   |
|Medidas sanitarias por la pandemia de COVID-19 en Argentina                                                                           |2020-03-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Polonia                                                                                                       |2020-04-30T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Arizona                                                                                                       |2020-07-06T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Pernambuco                                                                                                    |2020-05-02T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Malí                                                                                                          |2020-12-31T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Bahamas                                                                                                       |2020-08-17T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Jamaica                                                                                                       |2020-09-10T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Crimea                                                                                                        |2020-04-30T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Andorra                                                                                                       |2020-03-24T00:00:00Z|Espagnol   |
|Vacuna de Novavax contra la COVID-19                                                                                                  |2021-03-07T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Iowa                                                                                                          |2020-06-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Hungría                                                                                                       |2020-09-03T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Letonia                                                                                                       |2020-03-25T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Turkmenistán                                                                                                  |2020-07-21T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Finlandia                                                                                                     |2020-03-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Canadá                                                                                                        |2020-03-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Irán                                                                                                          |2020-03-16T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Kuwait                                                                                                        |2020-10-01T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Italia                                                                                               |2021-01-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Chipre                                                                                                        |2020-04-28T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Haití                                                                                                         |2020-05-15T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Hawái                                                                                                         |2020-06-15T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Antigua y Barbuda                                                                                             |2020-07-12T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en India                                                                                                |2021-01-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Gibraltar                                                                                                     |2020-04-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Dinamarca                                                                                                     |2020-06-21T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Vermont                                                                                                       |2020-05-08T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Túnez                                                                                                         |2020-06-25T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Oklahoma                                                                                                      |2020-05-15T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Samoa Americana                                                                                               |2020-11-10T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Samoa                                                                                                         |2020-11-19T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Sierra Leona                                                                                                  |2020-06-29T00:00:00Z|Espagnol   |
|Confinamiento por la pandemia de COVID-19 en México                                                                                   |2020-03-26T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Huancavelica                                                                                                  |2020-08-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Mongolia                                                                                                      |2020-03-14T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en República Dominicana                                                                                          |2020-03-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Bonaire                                                                                                       |2021-01-02T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Israel                                                                                               |2021-01-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Burkina Faso                                                                                                  |2020-06-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Alaska                                                                                                        |2020-06-28T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Sebastopol                                                                                                    |2020-04-30T00:00:00Z|Espagnol   |
|Brote de COVID-19 en la Casa Blanca                                                                                                   |2020-10-03T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Transnistria                                                                                                  |2020-12-05T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Australia                                                                                                     |2020-03-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Misuri                                                                                                        |2020-05-30T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Washington                                                                                                    |2020-05-05T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Texas                                                                                                         |2020-05-11T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Aruba                                                                                                         |2020-08-26T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Alemania                                                                                                      |2020-03-13T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Japón                                                                                                         |2020-03-14T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Saba                                                                                                          |2020-12-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Laos                                                                                                          |2020-06-28T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Malasia                                                                                                       |2020-07-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Mauritania                                                                                                    |2020-04-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Rusia                                                                                                         |2020-04-30T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en América del Norte                                                                                             |2021-01-05T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Wallis y Futuna                                                                                               |2020-10-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Georgia                                                                                                       |2020-05-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Corea del Norte                                                                                               |2020-03-26T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Austria                                                                                                       |2020-03-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Kansas                                                                                                        |2020-06-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Emiratos Árabes Unidos                                                                                        |2020-06-25T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Kosovo                                                                                                        |2020-12-17T00:00:00Z|Espagnol   |
|Confinamiento por la pandemia de COVID-19 en Chile                                                                                    |2020-06-03T00:00:00Z|Espagnol   |
|Confinamiento por la pandemia de COVID-19 en Colombia                                                                                 |2020-04-25T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Islas Marshall                                                                                                |2020-10-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Ucayali                                                                                                       |2020-08-17T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Tlaxcala                                                                                                      |2020-08-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en San Cristóbal y Nieves                                                                                        |2020-11-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Florida                                                                                                       |2020-06-19T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Italia                                                                                                        |2020-03-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Campeche                                                                                                      |2020-08-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Bermudas                                                                                                      |2020-09-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Trinidad y Tobago                                                                                             |2020-09-10T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Paraguay                                                                                                      |2020-03-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Virginia                                                                                                      |2020-05-06T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Moquegua                                                                                                      |2020-07-29T00:00:00Z|Espagnol   |
|Desinformación sobre la pandemia de COVID-19                                                                                          |2020-04-07T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Santa Lucía                                                                                                   |2020-12-18T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Lima                                                                                                          |2020-08-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Cabo Verde                                                                                                    |2020-07-11T00:00:00Z|Espagnol   |
|Confinamiento por la pandemia de COVID-19 en Ecuador                                                                                  |2020-03-21T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Grecia                                                                                                        |2020-09-06T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Paraíba                                                                                                       |2020-05-02T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Madre de Dios                                                                                        |2021-05-06T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Bielorrusia                                                                                                   |2020-04-25T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Moldavia                                                                                                      |2020-11-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Tacna                                                                                                         |2020-07-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Ica                                                                                                           |2020-07-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Ciudad del Vaticano                                                                                           |2020-03-22T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Tocantins                                                                                                     |2020-05-05T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Áncash                                                                                                        |2020-07-26T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Luisiana                                                                                                      |2020-06-11T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Colorado                                                                                                      |2020-06-22T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Utah                                                                                                          |2020-05-11T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Bélgica                                                                                                       |2020-04-22T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Nuevo México                                                                                                  |2020-05-21T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Nuevo Hampshire                                                                                               |2020-05-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Wyoming                                                                                                       |2020-05-02T00:00:00Z|Espagnol   |
|Controles de los riesgos laborales en respecto del COVID-19                                                                           |2020-08-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Ciudad de México                                                                                              |2020-07-19T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Montana                                                                                                       |2020-05-30T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Arabia Saudita                                                                                                |2020-03-23T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Egipto                                                                                                        |2020-06-05T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Ohio                                                                                                          |2020-05-15T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Delaware                                                                                                      |2020-06-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Palaos                                                                                                        |2021-06-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Comoras                                                                                                       |2020-10-05T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Tailandia                                                                                                     |2020-06-28T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Tabasco                                                                                                       |2020-08-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Timor Oriental                                                                                                |2020-12-11T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Kiribati                                                                                                      |2021-05-23T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Costa de Marfil                                                                                               |2020-12-03T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Suecia                                                                                                        |2020-03-15T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Estados Federados de Micronesia                                                                               |2021-01-11T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Goiás                                                                                                         |2020-05-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Sri Lanka                                                                                                     |2020-10-31T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Guanajuato                                                                                                    |2020-08-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Nebraska                                                                                                      |2020-05-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Lambayeque                                                                                                    |2020-07-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Noruega                                                                                                       |2020-03-09T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Seychelles                                                                                                    |2020-11-25T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Estonia                                                                                                       |2020-06-17T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Querétaro                                                                                                     |2020-08-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Puerto Rico                                                                                                   |2020-04-13T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Aguascalientes                                                                                                |2020-06-30T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Croacia                                                                                                       |2020-03-23T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Países Bajos                                                                                                  |2020-03-19T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Indiana                                                                                                       |2020-06-13T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en España                                                                                               |2021-01-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Jordania                                                                                                      |2021-04-09T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Tumbes                                                                                                        |2020-08-02T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Illinois                                                                                                      |2020-06-14T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Guerrero                                                                                                      |2020-08-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Islas Caimán                                                                                                  |2020-10-28T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Vietnam                                                                                                       |2020-04-17T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Loreto                                                                                                        |2020-07-25T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Colima                                                                                                        |2020-08-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Nepal                                                                                                         |2020-06-25T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Reino Unido                                                                                          |2021-01-09T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Tamaulipas                                                                                                    |2020-08-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Papúa Nueva Guinea                                                                                            |2020-06-30T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en California                                                                                                    |2020-06-30T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Durango                                                                                                       |2020-06-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Puebla                                                                                                        |2020-08-03T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Tennessee                                                                                                     |2020-05-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Bahía                                                                                                         |2020-05-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Virginia Occidental                                                                                           |2020-05-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en la ciudad de Nueva York                                                                                       |2020-09-06T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Nueva Caledonia                                                                                               |2020-05-17T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Mendoza                                                                                                       |2020-10-18T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Andhra Pradesh                                                                                                |2021-05-15T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Quebec                                                                                                        |2020-05-25T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Nayarit                                                                                                       |2020-06-30T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Arequipa                                                                                                      |2020-07-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Alabama                                                                                                       |2020-06-28T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Morelos                                                                                                       |2020-08-03T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Huánuco                                                                                                       |2020-07-28T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Bangladés                                                                                                     |2020-07-21T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Yucatán                                                                                                       |2020-08-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Suiza                                                                                                         |2020-03-19T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Cajamarca                                                                                                     |2020-07-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en São Paulo                                                                                                     |2020-05-02T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Nueva York                                                                                                    |2020-05-18T00:00:00Z|Espagnol   |
|Confinamiento por la pandemia de COVID-19 en India                                                                                    |2020-06-10T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Luxemburgo                                                                                                    |2020-03-15T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Córdoba (Argentina)                                                                                           |2020-10-18T00:00:00Z|Espagnol   |
|Mucormicosis asociada al COVID-19                                                                                                     |2021-05-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Islas Vírgenes Británicas                                                                                     |2020-11-28T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Chiapas                                                                                                       |2020-08-03T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Apurímac                                                                                                      |2020-08-18T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Amazonas (Perú)                                                                                               |2020-07-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Hong Kong                                                                                                     |2020-06-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Amazonas (Brasil)                                                                                             |2020-05-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Oregón                                                                                                        |2020-05-14T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Pensilvania                                                                                                   |2020-05-13T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Carolina del Norte                                                                                            |2020-05-16T00:00:00Z|Espagnol   |
|Prueba rápida de antígeno COVID-19                                                                                                    |2021-05-30T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Sinaloa                                                                                                       |2020-08-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Irak                                                                                                          |2020-07-27T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Zacatecas                                                                                                     |2020-06-30T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Asturias                                                                                                      |2020-07-12T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Filipinas                                                                                            |2021-02-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Quintana Roo                                                                                                  |2020-08-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Espírito Santo                                                                                                |2020-05-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Liechtenstein                                                                                                 |2020-03-15T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Puno                                                                                                          |2020-08-19T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en La Libertad                                                                                                   |2020-07-29T00:00:00Z|Espagnol   |
|Confinamiento por la pandemia de COVID-19 en Bolivia                                                                                  |2020-03-15T00:00:00Z|Espagnol   |
|Impacto en la religión por la pandemia de COVID-19                                                                                    |2020-03-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Eslovenia                                                                                                     |2021-02-22T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Oaxaca                                                                                                        |2020-08-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Jalisco                                                                                                       |2020-06-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Ucrania                                                                                                       |2020-03-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Baja California                                                                                               |2020-07-02T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en República Turca del Norte de Chipre                                                                           |2020-04-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Groenlandia                                                                                                   |2020-04-09T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Tucumán (Argentina)                                                                                           |2021-01-26T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Barbados                                                                                                      |2020-09-10T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Georgia (Estados Unidos)                                                                                      |2020-06-16T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Wisconsin                                                                                                     |2020-05-03T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Acrotiri y Dhekelia                                                                                           |2020-04-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Distrito Federal (Brasil)                                                                                     |2020-05-02T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Curazao                                                                                                       |2020-08-17T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Alagoas                                                                                                       |2020-05-02T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Argelia                                                                                                       |2020-09-02T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Piura                                                                                                         |2020-07-22T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Diamond Princess                                                                                              |2020-12-08T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Piauí                                                                                                         |2020-05-02T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Sergipe                                                                                                       |2020-05-05T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Yemen                                                                                                         |2020-04-25T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Rhode Island                                                                                                  |2020-05-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Ceará                                                                                                         |2020-05-02T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Azerbaiyán                                                                                                    |2020-03-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Paraná                                                                                                        |2020-05-02T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Kentucky                                                                                                      |2020-06-11T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Madre de Dios                                                                                                 |2020-07-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en San Martín                                                                                                    |2020-07-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Nueva Jersey                                                                                                  |2020-05-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Mato Grosso                                                                                                   |2020-05-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Río de Janeiro                                                                                                |2020-05-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Mato Grosso del Sur                                                                                           |2020-05-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Minesota                                                                                                      |2020-06-04T00:00:00Z|Espagnol   |
|Confinamiento por la pandemia de COVID-19 en Filipinas                                                                                |2021-02-19T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Río Grande del Sur                                                                                            |2020-05-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Entre Ríos                                                                                                    |2020-10-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Islas Vírgenes de los Estados Unidos                                                                          |2020-05-09T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Carolina del Sur                                                                                              |2020-05-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Lituania                                                                                                      |2021-05-14T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Lima Metropolitana                                                                                            |2020-07-21T00:00:00Z|Espagnol   |
|Obelisco en homenaje a a los médicos fallecidos por el COVID-19                                                                       |2020-07-28T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Acre                                                                                                          |2020-05-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Roraima                                                                                                       |2020-05-05T00:00:00Z|Espagnol   |
|Monumento en recuerdo de las víctimas de la pandemia del Covid-19 (Madrid)                                                            |2020-06-21T00:00:00Z|Espagnol   |
|Confinamiento por la pandemia de COVID-19 en Paraguay                                                                                 |2020-05-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Misisipi                                                                                                      |2020-05-31T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Idaho                                                                                                         |2020-06-15T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Maryland                                                                                                      |2020-06-09T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Maine                                                                                                         |2020-06-09T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Washington D. C.                                                                                              |2020-05-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Junín                                                                                                         |2020-08-10T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Pará                                                                                                          |2020-05-04T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Ayacucho                                                                                                      |2020-07-29T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Dakota del Norte                                                                                              |2020-05-16T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Rondonia                                                                                                      |2020-05-05T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Baja California Sur                                                                                           |2020-07-02T00:00:00Z|Espagnol   |
|Denuncia contra Larreta y Quirós por privatizar la vacunación contra la COVID-19                                                      |2021-02-23T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Dakota del Sur                                                                                                |2020-05-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Turquía                                                                                                       |2020-04-10T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Nevada                                                                                                        |2020-05-24T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en San Luis Potosí                                                                                               |2020-08-04T00:00:00Z|Espagnol   |
|Impacto en el funcionamiento de internet por la pandemia de COVID-19                                                                  |2020-03-17T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Arkansas                                                                                                      |2020-07-05T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Isla de Man                                                                                                   |2020-12-21T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Santa Fe (Argentina)                                                                                          |2020-10-28T00:00:00Z|Espagnol   |
|Impacto en la educación por la pandemia de COVID-19                                                                                   |2020-05-03T00:00:00Z|Espagnol   |
|Estadísticas de la pandemia COVID-19 en Argentina                                                                                     |2021-04-08T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Connecticut                                                                                                   |2020-06-21T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Cochabamba (Bolivia)                                                                                          |2021-01-28T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Salta                                                                                                         |2020-10-20T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Bulgaria                                                                                                      |2020-06-21T00:00:00Z|Espagnol   |
|Impacto de la pandemia de COVID-19 en el transporte público                                                                           |2020-07-08T00:00:00Z|Espagnol   |
|Confinamiento por la pandemia de COVID-19 en Estados Unidos                                                                           |2021-02-05T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Pasco                                                                                                         |2020-08-09T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Corea del Sur                                                                                                 |2020-03-14T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Santa Helena, Ascensión y Tristan da Cunha                                                                    |2021-05-19T00:00:00Z|Espagnol   |
|Impacto en el teatro por la pandemia de COVID-19                                                                                      |2020-04-11T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en San Marino                                                                                                    |2020-03-22T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Míchigan                                                                                                      |2020-06-04T00:00:00Z|Espagnol   |
|Escándalos de vacunación irregular contra la COVID-19                                                                                 |2021-03-03T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Neuquén                                                                                                       |2020-10-22T00:00:00Z|Espagnol   |
|Impacto de la pandemia de COVID-19 en otros problemas de salud                                                                        |2021-04-02T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en La Pampa                                                                                                      |2020-10-20T00:00:00Z|Espagnol   |
|Impacto de la pandemia de COVID-19 en los hospitales                                                                                  |2021-04-02T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Río Grande del Norte                                                                                          |2020-05-04T00:00:00Z|Espagnol   |
|Vacunación contra la COVID-19 en Huancavelica                                                                                         |2021-05-06T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Base Naval de la Bahía de Guantánamo                                                                          |2020-05-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Tayikistán                                                                                                    |2020-05-07T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en República Popular de Donetsk                                                                                  |2020-04-28T00:00:00Z|Espagnol   |
|Confinamiento por la pandemia de COVID-19 en España                                                                                   |2020-03-15T00:00:00Z|Espagnol   |
|Sinofobia y sentimiento antiasiático por la pandemia de COVID-19                                                                      |2020-02-12T00:00:00Z|Espagnol   |
|Impacto de la pandemia de COVID-19 en la alimentación                                                                                 |2020-05-11T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Amapá                                                                                                         |2020-05-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Santa Cruz (Argentina)                                                                                        |2020-10-20T00:00:00Z|Espagnol   |
|Impacto medioambiental de la pandemia de COVID-19                                                                                     |2020-04-18T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Minas Gerais                                                                                                  |2020-05-01T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en la Ciudad Autónoma de Buenos Aires                                                                            |2020-06-12T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Macao                                                                                                         |2020-12-10T00:00:00Z|Espagnol   |
|Protocolo centralizado de rastreo COVID-19                                                                                            |2021-05-19T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Jujuy                                                                                                         |2020-10-20T00:00:00Z|Espagnol   |
|Impacto en el cine por la pandemia de COVID-19                                                                                        |2020-03-21T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en República Popular de Lugansk                                                                                  |2020-04-28T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Buenos Aires (provincia)                                                                                      |2020-05-13T00:00:00Z|Espagnol   |
|Pandemia de COVID-19 en Palestina                                                                                                     |2021-04-09T00:00:00Z|Espagnol   |
|Covid-19                                                                                                                              |2020-02-11T00:00:00Z|Suédois    |
|Covid-19-vaccin                                                                                                                       |2020-05-17T00:00:00Z|Suédois    |
|COVID-19 Vaccine Moderna                                                                                                              |2020-12-17T00:00:00Z|Suédois    |
|Covid-19-vaccinering i Sverige                                                                                                        |2021-05-19T00:00:00Z|Suédois    |
|Covid-19-provtagning                                                                                                                  |2020-03-23T00:00:00Z|Suédois    |
|Covid-19 och graviditet                                                                                                               |2020-06-05T00:00:00Z|Suédois    |
|Lista över covid-19-simuleringsmodeller                                                                                               |2021-06-06T00:00:00Z|Suédois    |
|COVID-19                                                                                                                              |2020-02-22T00:00:00Z|Turque     |
|Mauritius'ta COVID-19 pandemisi                                                                                                       |2020-03-20T00:00:00Z|Turque     |
|COVID-19 pandemisi                                                                                                                    |2020-01-23T00:00:00Z|Turque     |
|Vatikan'da COVID-19 pandemisi                                                                                                         |2020-03-14T00:00:00Z|Turque     |
|Malezya'da COVID-19 pandemisi                                                                                                         |2020-05-20T00:00:00Z|Turque     |
|Lesotho'da COVID-19 pandemisi                                                                                                         |2020-05-15T00:00:00Z|Turque     |
|Aruba'da COVID-19 pandemisi                                                                                                           |2021-05-22T00:00:00Z|Turque     |
|COVID-19 pandemisinden ölenler listesi (2021)                                                                                         |2021-01-13T00:00:00Z|Turque     |
|COVID-19 testi                                                                                                                        |2020-04-09T00:00:00Z|Turque     |
|COVID-19 pandemisinden ölenler listesi (Temmuz-Aralık 2020)                                                                           |2020-11-18T00:00:00Z|Turque     |
|Belize'de COVID-19 pandemisi                                                                                                          |2021-05-25T00:00:00Z|Turque     |
|Türkiye'de COVID-19 pandemisinin sağlık sektörüne etkisi                                                                              |2020-10-23T00:00:00Z|Turque     |
|Bhutan'da COVID-19 pandemisi                                                                                                          |2020-05-15T00:00:00Z|Turque     |
|Kıbrıs Cumhuriyeti'nde COVID-19 pandemisi                                                                                             |2020-03-16T00:00:00Z|Turque     |
|Kuala Lumpur'da COVID-19 pandemisi                                                                                                    |2021-05-31T00:00:00Z|Turque     |
|Güney Afrika Cumhuriyeti'nde COVID-19 pandemisi                                                                                       |2020-05-16T00:00:00Z|Turque     |
|Portekiz'de COVID-19 pandemisi                                                                                                        |2020-03-20T00:00:00Z|Turque     |
|Hindistan'da COVID-19 pandemisi                                                                                                       |2020-05-09T00:00:00Z|Turque     |
|Ülke ve bölgelere göre COVID-19 pandemisi                                                                                             |2020-03-25T00:00:00Z|Turque     |
|Bangladeş'te COVID-19 pandemisi                                                                                                       |2020-05-15T00:00:00Z|Turque     |
|Fransa'da COVID-19 pandemisi                                                                                                          |2020-03-21T00:00:00Z|Turque     |
|Pfizer-BioNTech COVID-19 aşısı                                                                                                        |2020-11-11T00:00:00Z|Turque     |
|Kuzey Amerika'da COVID-19 pandemisi                                                                                                   |2020-05-10T00:00:00Z|Turque     |
|Rusya'da COVID-19 pandemisi                                                                                                           |2020-03-15T00:00:00Z|Turque     |
|COVID-19 aşısı                                                                                                                        |2020-03-24T00:00:00Z|Turque     |
|Yemen'de COVID-19 pandemisi                                                                                                           |2020-04-14T00:00:00Z|Turque     |
|Cezayir'de COVID-19 pandemisi                                                                                                         |2020-03-15T00:00:00Z|Turque     |
|COVID-19 pandemisinden ölenler listesi (Ocak-Haziran 2020)                                                                            |2020-11-18T00:00:00Z|Turque     |
|Letonya'da COVID-19 pandemisi                                                                                                         |2020-04-24T00:00:00Z|Turque     |
|Ekvador'da COVID-19 pandemisi                                                                                                         |2020-05-15T00:00:00Z|Turque     |
|Birleşik Krallık'ta COVID-19 pandemisi                                                                                                |2020-03-17T00:00:00Z|Turque     |
|İzlanda'da COVID-19 pandemisi                                                                                                         |2020-05-12T00:00:00Z|Turque     |
|Suudi Arabistan'da COVID-19 pandemisi                                                                                                 |2020-05-15T00:00:00Z|Turque     |
|Saint Pierre ve Miquelon'da COVID-19 pandemisi                                                                                        |2020-05-15T00:00:00Z|Turque     |
|Oxford–AstraZeneca COVID-19 aşısı                                                                                                     |2021-04-19T00:00:00Z|Turque     |
|Türkiye'de COVID-19 aşılaması                                                                                                         |2021-02-06T00:00:00Z|Turque     |
|Asya'da COVID-19 pandemisi                                                                                                            |2020-05-10T00:00:00Z|Turque     |
|Norveç'te COVID-19 pandemisi                                                                                                          |2020-03-26T00:00:00Z|Turque     |
|Avrupa'da COVID-19 pandemisi                                                                                                          |2020-03-19T00:00:00Z|Turque     |
|İspanya'da COVID-19 pandemisi                                                                                                         |2020-03-17T00:00:00Z|Turque     |
|Türkiye'de bölgelere göre COVID-19 pandemisi                                                                                          |2020-09-06T00:00:00Z|Turque     |
|Türkiye'de COVID-19 pandemisi                                                                                                         |2020-03-11T00:00:00Z|Turque     |
|Kosova'da COVID-19 pandemisi                                                                                                          |2020-04-23T00:00:00Z|Turque     |
|Arnavutluk'ta COVID-19 pandemisi                                                                                                      |2020-04-13T00:00:00Z|Turque     |
|Bulgaristan'da COVID-19 pandemisi                                                                                                     |2020-03-18T00:00:00Z|Turque     |
|İstanbul'da COVID-19 pandemisi                                                                                                        |2020-07-16T00:00:00Z|Turque     |
|Çin'de COVID-19 pandemisi                                                                                                             |2020-04-10T00:00:00Z|Turque     |
|Bosna-Hersek'te COVID-19 pandemisi                                                                                                    |2020-04-15T00:00:00Z|Turque     |
|Kazakistan'da COVID-19 pandemisi                                                                                                      |2020-03-14T00:00:00Z|Turque     |
|Kuzey Kore'de COVID-19 pandemisi                                                                                                      |2020-04-18T00:00:00Z|Turque     |
|Singapur'da COVID-19 pandemisi                                                                                                        |2020-05-15T00:00:00Z|Turque     |
|Türkiye'de COVID-19 pandemisi zaman çizelgesi                                                                                         |2020-03-26T00:00:00Z|Turque     |
|Nepal'de COVID-19 pandemisi                                                                                                           |2020-05-14T00:00:00Z|Turque     |
|Diamond Princess gemisinde COVID-19 pandemisi                                                                                         |2020-05-25T00:00:00Z|Turque     |
|COVID-19 pandemisinin Mart 2020 kronolojisi                                                                                           |2020-03-17T00:00:00Z|Turque     |
|COVID-19 pandemisinin spor organizasyonlarına etkisi                                                                                  |2020-03-12T00:00:00Z|Turque     |
|COVID-19 protestoları                                                                                                                 |2021-01-10T00:00:00Z|Turque     |


<- [Accueil](/readme.md) ------ [Les outils de la dataviz](/Les_outils_dataviz.md) ->
