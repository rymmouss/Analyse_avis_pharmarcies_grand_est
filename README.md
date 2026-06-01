# Analyse-avis-pharmarcies-grand-est

# Scraping et Analyse des Avis Google Maps : Pharmacies du Grand Est

## Présentation du projet
Ce dépôt contient le code réalisé lors de mon stage. Le but du projet était de construire une base de données propre recensant les pharmacies de la région Grand Est, et d'y associer les avis clients récoltés sur Google Maps pour pouvoir analyser les notes globales, avis clients, et reponses des proprietaires.

Le script Python (Jupyter Notebook) s'occupe de toute la chaîne de traitement : le nettoyage des données publiques, le croisement avec notre fichier interne, le géocodage, et enfin le web scraping automatisé.

## Fonctionnement du script
Le fichier `analyse_pharmacies_grand_est.ipynb` est divisé en 5 grandes étapes :

1. Filtrage géographique (GeoPandas)
Le script charge un fichier de données OpenStreetMap et utilise les contours GeoJSON officiels des départements pour ne garder que les pharmacies situées dans le Grand Est. Les coordonnées sont converties du format Mercator (EPSG:3857) vers WGS84 (Latitude/Longitude).

2. Croisement des données
La base obtenue est croisée (merge) avec un fichier de référence interne (`ListingPharmaciesGrandEstfeb2026.xlsx`) en utilisant le code postal et la ville pour identifier les doublons et les pharmacies manquantes.

3. Géocodage (API Nominatim)
Pour les pharmacies dont il manque les coordonnées GPS, le script interroge l'API open source Nominatim pour récupérer la latitude et la longitude exactes (avec un délai d'attente configuré pour ne pas surcharger l'API).

4. Web Scraping (Selenium)
Un navigateur Chrome est piloté automatiquement pour chercher chaque pharmacie sur Google Maps. Le script intègre :
- La validation des fenêtres de cookies.
- Un système de scoring (nom, ville, code postal) pour vérifier qu'il clique sur le bon résultat.
- La récupération de la note globale, du nombre d'avis, et du détail des derniers commentaires (texte, date, auteur, réponse du propriétaire).
- Un système de traitement par lots (batchs) avec des sauvegardes intermédiaires pour éviter de tout perdre si le navigateur plante.

5. Nettoyage final
Les différents lots sont fusionnés, les doublons sont supprimés, et les enseignes trop génériques sont exclues pour générer le fichier final.

## Structure des fichiers
```text
pharmacies-grand-est/
├── analyse_pharmacies_grand_est.ipynb    # Script principal
├── pharmacies_point.csv                  # Données brutes OSM (à ajouter localement)
├── ListingPharmaciesGrandEstfeb2026.xlsx # Fichier de référence interne (à ajouter localement)
├── pharmacies_grand_est.csv              # Export intermédiaire 
├── avis_google_temp_N.csv                # Sauvegardes toutes les 10 pharmacies
├── avis_google_X_Y_propre.csv            # Export par lot après scraping
├── avis_google_X_Y_presentation.csv      # Lot nettoyé prêt à l'analyse
└── pharmacies_avis_FINAL.csv / .xlsx     # Fichiers de données finaux
```

Dictionnaire des données (Fichier Final)
Le script produit un dataset pharmacies_avis_FINAL pret pour l'analyse, contenant les variables suivantes :

pharmacie : Nom de l'officine.

ville : Commune d'implantation.

code_postal : Code postal a 5 chiffres.

latitude / longitude : Coordonnees spatiales WGS84.

note_moyenne : Note globale de la pharmacie sur Google Maps (sur 5).

nombre_avis : Volume total d'avis recenses.

auteur : Pseudonyme du client ayant redige l'avis.

note_avis : Note specifique laissee par le client.

date : Date de publication de l'avis.

avis_texte : Contenu textuel de l'avis client.

reponse_owner : Reponse apportee par la pharmacie a l'avis client.

## Comment tester ou relancer le script

 **Note sur la confidentialité des données :** *Le fichier source de référence (`ListingPharmaciesGrandEstfeb2026.xlsx`) étant un document interne, il n'est pas partagé dans ce dépôt public. Les instructions ci-dessous sont fournies à titre indicatif pour démontrer la reproductibilité du code.*

Si vous disposiez des fichiers sources, voici les étapes à suivre pour exécuter le pipeline :

**Prérequis :** Avoir Python 3.8+ et Google Chrome installés.

1. **Les bibliothèques à installer :** Il faut d'abord installer les packages nécessaires en tapant ceci dans votre terminal :
```bash
pip install pandas geopandas shapely selenium webdriver-manager requests openpyxl
```

2. Les fichiers de données : placer les deux fichiers sources (pharmacies_point.csv et ListingPharmaciesGrandEstfeb2026.xlsx) dans le même dossier que le notebook.

3. Lancement du code : ouvrez le notebook analyse_pharmacies_grand_est.ipynb. Dans la partie 8 (Boucle par lot), le script est configuré pour ne traiter que 100 pharmacies à la fois en commençant par le début du fichier. Vous pouvez modifier ces deux variables pour tester le code rapidement, ce qui évite de lancer le script sur des milliers de lignes d'un coup.

```python
BATCH_SIZE = 100  # Nombre de pharmacies a scraper en meme temps
START = 0    # Ligne de depart dans le fichier
```
