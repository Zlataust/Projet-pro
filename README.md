# Projet-pro
Bibliothèques
# l'analyse des données
import numpy as np
import pandas as pd
# visualisation
import matplotlib.pyplot as plt 
import matplotlib as mpl 
import seaborn as sns
# avertissements
import time
import warnings
# gestion du système de fichiers
import os
# importations de packages
from wordcloud import WordCloud, STOPWORDS
# les paramètres
start_time=time.time()
color = sns.color_palette()
sns.set_style("dark")
warnings.filterwarnings("ignore")
%matplotlib inline
# accès aux fonctions
import sys
# encodage des données au format JSON
import json
Processus de travail:

    Import des données
        Création d'une requête pour obtenir les données via l’API
        Collecte de données

    Explorer les données
        Types de colonnes
        Récupération des champs nécessaires
        Application d'un filtre sur des champs nécessaires
        Suppression des doublons
        Analyse des valeur manquantes
        Création d'un fichier csv
        Analyse des critiques

    Création d'un WordCloud
        Collection des avis
        Création de nuages de mots
Import des données

J'utilise GraphQL pour collecter des données directement via l’API.
# Installation des packages nécessaires:
!pip install --pre gql[all]
Application:

Pour commencer on voit que yelp propose une console pour tester des requêtes : 
https://www.yelp.com/developers/graphiql. Si on va seulement dessus ça ne fonctionne pas, 
il faut un compte et activer la beta developper. Tant que cette étape n'est pas faite 
on ne peut pas avoir nos clefs pour nous connecter à cette API. 
(C'est-à-dire que tant qu'une requête simple ne fonctionne pas dans la console, la suite ne marchera pas). Exemple simple de requete :
from gql import gql, Client
from gql.transport.aiohttp import AIOHTTPTransport


# Je récupère ma clé, je vais la désactiver après l'avoir testé 
API_Key = "NZ4RTd0W29c2e_FVFTthsP2wIOEdfzPECo-oKU44N667sBDN-BctBuMCNWcABYGgogEff8x9zCzKR1rMRQXzObuhciYJYFdwagnV3o4f6pJcaVz0FoqYsGlF_Y68YHYx"

# Cindy : Ici on modifie l'url pour accèder à celle de yelp et on rajoute les autorisations
transport = AIOHTTPTransport(url="https://api.yelp.com/v3/graphql", headers={'Authorization' : "bearer {}".format(API_Key)})


# Create a GraphQL client using the defined transport
client = Client(transport=transport, fetch_schema_from_transport=True)

# Provide a GraphQL query
query = gql(
    """
{
    business(id: "garaje-san-francisco") {
        name
        id
        rating
        photos
    }
}
"""
)

# Erreur à cause du fait que ça tourne dans un notebook, je fais la modification suggérée (il est possible que ça ne fonctionne pas
# sur colab, le faire dans jupyter)
result = await client.execute_async(query)
print(result)
Ca fonctionne, il faut maintenant juste modifier la requête pour récupérer ce que je souhaite en utilisant : 
https://www.yelp.com/developers/graphql/query/search. Par exemple, je vais chercher des starbucks à new york 
pour savoir s'ils sont ouverts maintenant et collecter leurs coordonnées.
from gql import gql, Client
from gql.transport.aiohttp import AIOHTTPTransport


# Je récupère ma clé, je vais la desactiver après l'avoir testé puis tu devras mettre la tienne
API_Key = "NZ4RTd0W29c2e_FVFTthsP2wIOEdfzPECo-oKU44N667sBDN-BctBuMCNWcABYGgogEff8x9zCzKR1rMRQXzObuhciYJYFdwagnV3o4f6pJcaVz0FoqYsGlF_Y68YHYx"

# Cindy : Ici on modifie l'url pour accèder à celle de yelp et on rajoute les autorisations
transport = AIOHTTPTransport(url="https://api.yelp.com/v3/graphql", headers={'Authorization' : "bearer {}".format(API_Key)})

# Provide a GraphQL query
query = gql(
     """
{
  search(term: "Starbucks",
         location: "New York") {
    business {
      hours {
        is_open_now
      }
      coordinates {
          longitude
          latitude
      }      
    }
  }
}"""
)

# Erreur à cause du fait que ça tourne dans un notebook, je fais la modification suggérée (il est possible que ça ne fonctionne pas
# sur colab, le faire dans jupyter)
result = await client.execute_async(query)
print(result)
Les données sont renvoyées en dictionnaire qu'il faut après modifier pour avoir un dataframe avec les colonnes que je souhaite.
# importing ensemble de données "review"
review = pd.read_json("yelp_academic_dataset_review.json", lines=True, orient="records", encoding="utf8", 
                      nrows=100000, chunksize=100000)
for r in review:
    data_review=r
end_time=time.time()
print("Took",end_time-start_time,"s")
# information sur dtypes
data_review.info()
# info sur data-colonnes
data_review.head()
Explorer les données
Regardons les avis des utilisateurs
# application d'un filtre pour récupérer juste les avis insatisfaits
review_mask=data_review['stars']<=1
filtered_review = data_review[review_mask]
print(filtered_review)
# suppression des doublons
filtered_review.drop_duplicates(inplace = True)
# analyse des valeurs manquantes
count = filtered_review.isna().sum()
print(count)
# faire un dataframe de soumission
submit = filtered_review
# enregistrer le dataframe de soumission
submit.to_csv('filtered_review.csv', index = False)
Regardons les meilleurs utilisateurs en fonction du nombre d'avis qu'ils ont donnés
user_top=data_review.groupby('user_id').agg({'review_id':['count'],'date':['min','max'],
                                'useful':['sum'],'funny':['sum'],'cool':['sum'],
                               'stars':['mean']})
user_top=user_top.sort_values([('review_id','count')],ascending=False)
print("                              Top 10 Users in Yelp")
user_top.head(10)
Répartition des notes
# obtenir la distribution des notes
x=data_review['stars'].value_counts()
x=x.sort_index()

# plot visualisation
plt.figure(figsize=(8,4))
ax= sns.barplot(x.index, x.values, alpha=0.8)
plt.title("Star Distribution")
plt.ylabel('Critique', fontsize=12)
plt.xlabel('Star', fontsize=12)

# ajouter des étiquettes de texte
rects = ax.patches
labels = x.values
for rect, label in zip(rects, labels):
    height = rect.get_height()
    ax.text(rect.get_x() + rect.get_width()/2, height + 5, label, ha='center', va='bottom')

# visualisation de distribution
plt.show()
Il y a 32304 de 100000 personnes qui ont donné moins de 4 stars. Ca te dire, plus que 30% des clients ne sont pas satisfaits.
Toutes les critiques sont-elles uniques?
data_review.review_id.is_unique
user=filtered_review.groupby('user_id').agg({'review_id':['count'], 'stars':['mean']})
user=user.sort_values([('review_id','count')],ascending=False)
print("                              Top 10 Users in Yelp")
user.head(10)
Création d'un WordCloud
# collection de 50 avis 
review_final = filtered_review.text[:50] + '...'
review_final
# création d'un fond de nuages de mots
mpl.rcParams['figure.figsize']=(15.0,15.0)    
mpl.rcParams['font.size']=10                
mpl.rcParams['savefig.dpi']=100             
mpl.rcParams['figure.subplot.bottom']=.1 
# éliminer des stopwords
stopwords = set(STOPWORDS)

wordcloud = WordCloud(
                          background_color='black',
                          stopwords=stopwords,
                          max_words=200,
                          max_font_size=40, 
                          random_state=42
                         ).generate(str(review_final))
# création de nuages
print(wordcloud)
fig = plt.figure()
plt.imshow(wordcloud)
plt.axis('off')
plt.show()
fig.savefig("word1.png", dpi=900)

Conclusion : je suis arrivée assez vite à collecter des données directement via l’API


