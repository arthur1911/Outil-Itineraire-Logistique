# Outil de Visualisation d'Itinéraires Logistiques

## Description

Cet outil calcule différents types d'itinéraires entre un point de départ et une destination en prenant en compte plusieurs modes de transport de marchandises. 
Il permet de visualiser les différents chemins sur une carte interactive ainsi que la durée, le prix et les émissions de CO2 de chaque itinéraire.

## Méthodologie

L'algorithme repose sur la théorie des graphes et la minimisation de chemin. Le code prend en entrée le point de départ, la destination et la quantité transportée (en tonnes). Voici les étapes de la méthodologie :

### 1. Collecte des nœuds

- **Aéroports** : Coordonnées GPS des 100 plus grands aéroports d'Europe, collectées manuellement.
- **Ports maritimes** : Coordonnées des 30 plus grands ports d'Europe.
- **Ports fluviaux** : Coordonnées des 15 principaux ports fluviaux en France.
- **Gares de fret ferroviaire** : Liste des 224 gares collectées manuellement grâce à une base de données.
- **Points de départ et d'arrivée** : Coordonnées GPS obtenues via l'API Google Maps pour n'importe quel lieu ou adresse.

### 2. Création des arêtes

Les nœuds sont reliés par des arêtes bidirectionnelles :

- **Train** : Relié uniquement entre gares sur la même ligne ferroviaire.
- **Barge** : Relié uniquement entre ports fluviaux sur la même rivière.
- **Autres** : Tous les autres nœuds sont connectés entre eux.

Les distances associées aux arêtes varient en fonction du mode de transport :

- **Camion** : Distance à vol d'oiseau × 1,3 (irrigularités de la route).
- **Train** : Distance à vol d'oiseau × 1,1 (trajet ferroviaire).
- **Avion** : Distance à vol d'oiseau.
- **Bateau** : Utilisation d'une "autoroute maritime" construite manuellement.
- **Barge** : Suivi précis des rivières principales grâce à des coordonnées GPS collectées manuellement.

### 3. Ajout des poids

Les arêtes sont pondérées pour construire 3 types d'itinéraires :

- **Durée**
- **Prix**
- **Émissions de CO2**

#### Durée

Le temps est calculé en divisant la distance par la vitesse moyenne de chaque mode de transport :

- Camion : 80 km/h
- Train : 60 km/h
- Avion : 800 km/h
- Bateau : 25 km/h
- Barge : 15 km/h

Des délais de chargement et déchargement sont ajoutés lors des changements de mode :

- Camion : 0 h
- Train : 3 h
- Avion : 2 h
- Bateau : 24 h
- Barge : 1 h

#### Prix

Le prix de chaque arête est déterminé à partir des ratios suivants, issus d'une étude néerlandaise menée par le Netherlands Institute for Transport Policy Analysis ([KiM](https://www.kimnet.nl/binaries/kimnet/documenten/notities/2023/03/30/kostenkengetallen-voor-het-goederenvervoer/Cost+figures+for+freight+transport_def.pdf)) :

- Camion : 0,1 €/t-km
- Train : 0,02 €/t-km
- Avion : 0,19 €/t-km
- Bateau : 0,003 €/t-km
- Barge : 0,02 €/t-km

Le prix est calculé en multipliant le ratio correspondant par le poids de la marchandise et la distance de l'arête concernée.

#### Émissions de CO2

Les émissions de CO2 suivent les ratios définis par le Framework [GLEC](https://www.smartfreightcentre.org/en/our-programs/emissions-accounting/global-logistics-emissions-council/calculate-report-glec-framework/) :

- Camion : 0,1 kgCO2/t-km
- Train : 0,03 kgCO2/t-km
- Avion : 1 kgCO2/t-km
- Bateau : 0,005 kgCO2/t-km
- Barge : 0,02 kgCO2/t-km

### Poids supplémentaire

Comme la décision de choisir un itinéraire ou un autre est multi-factorielle, le code intègre un quatrième poids qui combine linéairement les 3 autres poids définis ci-dessus. Cela permet de définir un chemin minimisant à la fois la durée, le prix et le coût écologique.

### 5. Minimisation de chemin

Une fois que les nœuds, les arêtes et les poids ont été créés et que le graphe est complet, le code utilise l'algorithme de Dijkstra pour trouver le chemin qui minimise l'un des poids choisis par l'utilisateur. Celui-ci peut sélectionner parmi les options suivantes :

- **Itinéraire minimisant les émissions de CO2** : Le chemin reliant le point de départ à la destination en minimisant le coût écologique.
- **Itinéraire le plus rapide** : Le chemin minimisant la durée du trajet.
- **Itinéraire le plus économique** : Le chemin minimisant le prix du trajet.
- **Itinéraire optimal** : Le chemin minimisant simultanément le prix, les émissions de CO2 et la durée grâce au quatrième poids.
- **Itinéraire route** : Le chemin proposé par Google Maps reliant le point de départ à la destination par la route. Il sert de moyen de comparaison pour les autres itinéraires.




