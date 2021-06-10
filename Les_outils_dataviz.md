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
Palladio est un outil de datavisualition qui permet de visualiser des données de différentes façons. Il est possible de créer des cartes afin de voir les données sous forme de points. Les relations entre les points peuvent être reliées par des lignes, avec des sortes d’arc de lignes représentant le flux de la relation. Il est également possible de  visualiser les relations entre deux dimensions des données. Les informations graphiques seront affichées sous forme de nœuds reliés par des lignes. L’affichage des liens et des étiquettes peut être activé ou désactivé.
Pour nos jeux de données en français, nous avons voulu tester la deuxième méthode de visualisation, afin de voir de quelle manière les données allaient être représentées sur le graphique.

![Palaldio_1](https://user-images.githubusercontent.com/74506096/121567571-04e75b00-ca1f-11eb-8de5-c32e96092437.png)
![Palaldio_2](https://user-images.githubusercontent.com/74506096/121567654-1f213900-ca1f-11eb-98ba-6ea3e71c5ad8.png)

Sur ces deux images, nous pouvons voir les résultats des données que nous avons trouvées en français. Chaque titre contient une date de publication qui lui correspond et aucun lien n’est fait entre elles. Ce qui aurait été pertinent pour notre travail, c’est de pouvoir séparer les dates par mois et années afin de voir quelles sont les articles correspondant au covid-19 qui ont été rédigé les mêmes années ou les mêmes mois de la même année.
On ne peut pas faire d’analyse critique avec l’outil Palladio. Il est préférable d’utiliser un outil qui nous permet de faire des timelines afin de pouvoir




## Les outils recommandés pour le format de timeline
### Timeline JS
### Matplotlib
