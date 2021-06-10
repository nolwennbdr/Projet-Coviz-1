# Les outils de dataviz

## Les outils non recommandés pour le format de timeline
### Flourish 
Dans le cadre de notre projet nous avons été amenés à produire un corpus de données sur les publications faites sur la pandémie Covid-19 sur Wikipédia dans diverses langues. Afin de rendre nos corpus de données plus exploitables nous avons besoin de faire une datavisualisation qui nous permettra de traduire nos données en graphiques compréhensibles.

C’est dans ce cadre que nous avons choisi l’outil Flourish afin de faire la représentation graphique de notre corpus. C’est un outil de datavisualisation en ligne qui permet de visualiser des corpus de données avec différents graphiques (graphiques linéaires, à barres, à secteurs ; cartes de projection, dispersion avec les nuages de points, carte 3D, Hiérarchie, carte de chaleur etc.) qui fonctionnent sur toutes les tailles d’écran que ça soit les écrans d’ordinateurs, tablettes et smartphones. Il est facile à utiliser car pas besoin de code pour visualiser, il suffit juste de disposer d’une feuille de données CSV.

![Flourish_1](https://user-images.githubusercontent.com/75143201/121436193-69071200-c980-11eb-812c-56878de7b948.png)

![Flourish_2](https://user-images.githubusercontent.com/75143201/121436211-73291080-c980-11eb-922e-63dcb320a2d4.png)

Nous avons visualisé une des langues (ici le français) de notre corpus de données afin d’avoir un aperçu sur la représentation des données et voir si ça allait correspondre à nos attentes. Nous avons testé plusieurs graphiques allant des graphiques linéaires aux graphiques chronologiques mais aucun n’est en adéquation avec les besoins du projet car les éléments indispensables d’une frise chronologique ne sont bien représentés. 

Comme nous pouvons les voir dans les images ci-dessus seules les années de création d’articles apparaissent alors que le jour et mois sont des éléments essentiels pour faire une frise chronologique, les titres sont entassés et impossible d’avoir une vue d’ensemble des articles de Wikipédia sur le Covid-19 et leurs évolutions au fil du temps. Les différentes tentatives en vain de faire une frise chronologique nous ont permis de voir que Flourish est un outil limité malgré ses nombreux avantages et n’est donc pas un outil adapté pour faire des timelines. 

### Palladio
## Les outils recommandés pour le format de timeline
### Timeline JS
TimelineJSest un outil open source créé par le Northwestern University Knight Lab, de l’université de Chicago et San Francisco. Il fait partie d’une suite d’outil gratuit permettant de créer des storytellinginteractifs destinésau journalisme, dans la veine du data-journalism. C’estun outil simple d’utilisation, qui permet de créer des frises chronologiques.Il fonctionne à partir d’un template Google Spreadsheet qui propose différents champsà renseigner.Son utilisation nécessite néanmoins la création d’un compte google.

Les champs disponibles:
- Date de d’entrée dans la chronologie. 
- Dates de fin (possibilité de créer des périodes entre deux dates)
- Date d'affichage
- Le titre et le corps du texte qui seront affichés sur chaque (champs non obligatoires, HTML autorisé)
- Possibilité d’ajout de médias à partir d'une variété de sources et a un support intégré pour Twitter, Instagram, Flickr, Google Maps, DropBox, DocumentCloud, Wikipedia, SoundCloud, Storify, iframes, les principaux sites vidéo  
- Possibilité de groupé lesdatessur plusieurs niveaugrâce à des descripteurs renseignés dans la colonne «group». 

Cet  outil  peut  également  être  utilisé  avec  un  fichier  JSON et le  rendu des visualisations peut être amélioré avec du CSS.

### Matplotlib
