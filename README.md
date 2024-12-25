# P2-WCS2024-Outil_recommandation


# Introduction
Dans le cadre de ma formation Data Analyst, nous avons eu le challenge de créer un outil de recommandation de films. Nous avions comme ressources les bases de données IMDb et TMDb.
Nous avions un délai de 7 semaines que nous avons répartis en 3 sprint selon la méthode Agile (scrum) :  
        Sprint 1 : Etude de marché sur le cinéma dans la région de la Creuse (CNC,Insee)  
        Sprint 2 : Appropriation, exploration des données et nettoyage (Pandas, Matplotlib, Seaborn)  
        Sprint 3 : Machine learning et recommandations (scikit-learn) + Affinage, interface et présentation  

# Missions et Livrables Attendus
## Missions
- Faire une présentation pour présenter notre travail (démarche, outils expliquer votre démarche, difficultés ...)
- Présenter les indicateurs statistiques et KPI pertinents sur les films.
- Créer un système de recommandation de film en utilisant des algorithmes de machine learning et faire une démonstration de ces recommandations sur des films proposés en séance par le client.
## Livrables
- Un notebook contenant l’exploration et le nettoyage des données ainsi que les visualisations.
- Un dashboard présentant les KPI pertinents.
- Un notebook pour l’étape Système de recommandation

# Déroulement de nos sprints :
## Sprint 1 : 
Comme indiqué dans l'introduction, la première étape de notre projet a été de réaliser une étude de marché sur le cinéma dans la région de la Creuse. Lors de cette étude nous nous sommes rendus compte que la majorité de la population était âgée de plus de 40 ans. Nous avons donc choisi de cibler des films sortis entre 1990 et 2010 afin de remémorer à notre public creusois leurs plus belles années de jeunesse. 

## Sprint 2 : 
Lors de ce second sprint, nous avons réalisé l'EDA de toutes les bases de données mises à notre disposition. Nous avons donc exploré, analysé, mergé et filtré nos dataframes pour nous approprier les données.
Nous avons ensuite identifier des KPI pertinents pour réaliser notre dashboard.
Voici quelques exemples de codes et visualisations: 

```python
# Chargement des données :

import pandas as pd
df_tmdb = pd.read_csv('/Users/Laetitia/Documents/Wild/Projet 2/Base donnée/tmdb_full.csv')
df_tb = pd.read_csv('/Users/Laetitia/Documents/Wild/Projet 2/Base donnée/title.basics.tsv',sep='\t')
df_tr = pd.read_csv('/Users/Laetitia/Documents/Wild/Projet 2/Base donnée/title.ratings.tsv', sep='\t')
df_tc = pd.read_csv('/Users/Laetitia/Documents/Wild/Projet 2/Base donnée/title.crew.tsv, sep='\t')
df_ta = pd.read_csv('/Users/Laetitia/Documents/Wild/Projet 2/Base donnée/title.akas.tsv, sep='\t')
df_nb = pd.read_csv('/Users/Laetitia/Documents/Wild/Projet 2/Base donnée/name.basics.csv')

# Quelques nettoyages :
df_tmdb.drop(df_tmdb_v2[df_tmdb_v2['adult'] == True ].index, inplace=True) #Suppression des films pour adultes
df_tb = df_tb[df_tb['titleType']=='movie']    #On décide de ne garder que les types films (on supprime les séries, les clips, etc)
df_tmdb = df_tmdb[(df_tmdb['runtime'] >= 60) & (df_tmdb['runtime'] < 300)]
df_tmdb.rename(columns ={'imdb_id':'tconst'},inplace = True)

# Quelques jointures :
df = pd.merge(df_tmdb,df_tb,how="left",on="tconst")
df = pd.merge(df,df_tb, how='left', on='tconst')

# Exemples de KPI :
## Budget moyen par genres :
df_tmdb.dropna(subset=['genres'],inplace=True)
df_tmdb = df_tmdb_.assign(genres=df_tmdb_['genres'].str.split(', ')).explode('genres')
df_budget_moyen = df_tmdb.groupby("genres").agg(budget_moyen=("budget","mean"))

## Evolution quantité de films :
df_tmdb['release_date']=pd.to_datetime(df_tmdb['release_date'])
df_tmdb['Year']=df_tmdb['release_date'].dt.year
df_tmdb['Year'].value_counts()

### Visualisation matplotlib/seaborn : 
    import seaborn as sns
    import matplotlib.pyplot as plt
    fig, ax = figsize = (20,5)
    plt.suptitle("Nombre de films par année")
    sns.histplot(df_tmdb,x='Year',kde=True )
    sns.set_style("dark", {'grid.linestyle': ':'})
    plt.xlabel("Année")
    plt.ylabel("Nombre de films")

```


Création des fonctions utiles pour les recommandations avec Machine learning : 
```python
import pandas as pd
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import NearestNeighbors

# Recherche films par titre
def recherche_par_titre(df, morceau_titre):
    morceau_titre = morceau_titre.lower()  
    films_trouves = df[df['title'].str.lower().str.contains(morceau_titre, na=False)]
    return films_trouves

# Recommandation avec pondération des genres e
def recommandation_films_par_genre(df, id_film, k=5):
    genre_columns = ['Action', 'Adventure', 'Animation', 'Biography', 'Comedy', 'Crime', 'Documentary', 
                     'Drama', 'Family', 'Fantasy', 'History', 'Horror', 'Music', 'Musical', 'Mystery', 
                     'News', 'Reality-TV', 'Romance', 'Sport', 'Talk-Show', 'Thriller', 'War', 'Western', 
                     'Science_Fiction']

    # Pondération genres
    df[genre_columns] = df[genre_columns] * 2

    features = ['note', 'Nb_votes', 'date_sortie'] + genre_columns

    scaler = StandardScaler()
    X = scaler.fit_transform(df[features])

    nn = NearestNeighbors(n_neighbors=k + 10, algorithm='ball_tree')  # Trouver plus de voisins pour appliquer le filtrage
    nn.fit(X)

    distances, indices = nn.kneighbors([X[id_film]])
    indices_sans_cible = [i for i in indices[0] if i != id_film]

    # Obtenir les genres du film cible
    genres_cible = df.loc[id_film, genre_columns]

    # Filtrer les films similaires entre 1990 et 2010 et ayant au moins un genre commun
    films_similaires = df.iloc[indices_sans_cible]
    films_similaires = films_similaires[
        (films_similaires['date_sortie'] >= 1990) &
        (films_similaires['date_sortie'] <= 2010) &
        (films_similaires[genre_columns].apply(lambda x: (x & genres_cible).any(), axis=1))
    ]

    return films_similaires.head(k)[['title', 'date_sortie']]
```

Fonction principale pour la recommandation : 
```python
morceau_titre = input("Entrez un morceau de titre de film : ")
films_trouves = recherche_par_titre(df, morceau_titre)

if films_trouves.empty:
    print("Aucun film ne correspond à votre recherche.")
else:
    print("Films trouvés :")
    print(films_trouves[['title', 'date_sortie']])

    try:
        id_film = int(input("Entrez l'index du film que vous souhaitez choisir : "))
        if id_film not in films_trouves.index:
            print("Index invalide. Veuillez réessayer.")
        else:
            print(f"Recommandation de GenZ pour '{df.loc[id_film, 'title']}' :")
            recommandations = recommandation_films_par_genre(df, id_film)
            print(recommandations)
    except ValueError:
        print("Entrée invalide. Veuillez entrer un numéro d'index.")
```

