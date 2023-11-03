# Partie 1
##### 1. Quel est le nombre d'exutoires pour les déchets `Dématérialisation` ?
Il y a 14 conseils pour les dématérialisations.

```sql

SELECT COUNT(*) FROM project WHERE label = 'Dématérialisation' ;

```

##### 2. Quel est le type d'exutoire pour les déchets de `Peinture à l'eau` ?
Le type d'exutoire est `'conseil'`
```sql
SELECT type FROM project WHERE nom_dechet = 'Peinture à l eau';

```

##### 3. Quels sont les déchets associés à l'exutoire id `3` ?

Ce sont bocal en verre, bouteille en verre, flacon en verre, pot de yaourt en verre, pot en verre. 
```sql
SELECT nom_dechet FROM project WHERE id_exutoire = 3;

```
##### 4. Quels sont les déchets qui ont comme famille `Verre` ?
Ce sont bocal en verre, bouteille en verre, flacon en verre, pot de yaourt en verre, pot en verre.
```sql
SELECT nom_dechet FROM project WHERE famille_dechet = 'Verre';

```
##### 5. Quelle est la description de l'exutoire `6` ?
La description est` "Ce déchet peut être composté. Il deviendra alors une ressource !!"`
``` sql 
SELECT description FROM project WHERE id_exutoire = 6;
```

##### 6. Combien de type de déchets sont concernés par les exutoires de type `Conseil` ?
Il ya 21 familles de déchet concernées par le type `Conseil`. 
```sql 
SELECT COUNT(DISTINCT famille_dechet) FROM project WHERE type = 'conseil';

```
##### 7. Quel est le type d'exutoire qui contient le plus de déchets différents ?

```sql    
SELECT type, COUNT(DISTINCT nom_dechet) AS nb_dechets
FROM project
GROUP BY type
ORDER BY nb_dechets DESC
LIMIT 1;
```
##### 8. Quel est le type d'exutoire le plus courant ?
C'est l'exutoire de `label= 'Dechetterie'` qui est le plus courant 
```sql 
SELECT label, COUNT(*) AS nb
FROM project
WHERE type = 'exutoire'
GROUP BY label
ORDER BY nb DESC
LIMIT 1;
```
##### 9. Quel est le type d'exutoire le plus courant par famille de déchet ?
Pour 'alimentation' : Conseil (Gaspillage alimentaire).  
Pour la famille 'bijoux' : Conseil (Don / Entraide / Revente).  
Pour la famille 'bois' : Exutoire (dechetterie)  
Pour la famille 'Culture loisirs' : Conseil (Dématérualisation)  
Pour la famille 'Divers' : Exutoire (Poubelle noire)  
Pour la famille 'electrique et éléctronique' : exutoire (dechetterie)  
Pour la famille 'emballage métal' : Exutoire (Poubelle verte / jaune)  
Pour la famille 'emballage plastique' : Exutoire (Poubelle verte / jaune)  
Pour la famille 'equipement de la maison' : Conseil (Don/Entraide/Revente)  
Pour la famille 'instrument de musique' : Conseil (Don / Entraide / Revente)  
Pour la famille 'médical' : Exutoire (Pharmacie)  
Pour la famille 'matériaux' : Exutoire (dechetterie)  
Pour la famille 'objet plastique' : exutoire (poubelle noire)  
Pour la famille 'outil manuel' : Conseil (don / entraide / revente)  
Pour la famille 'papier carton' : exutoire (poubelle verte/jaune )  
Pour la famille 'produit chimiques' : Exutoire (dechetterie)  
Pour la famille 'Sport' : Conseil (don / entraide / revente)  
Pour la famille 'textile' : exutoire (revnedeur ou installateur)  
Pour la famille 'végétaux' :  Exutoire (dechetterie)  
Pour la famille 'vaiselle' :  Exutoire (dechetterie)  
Pour la famille 'verre' : exutoire (borne à verre)  
Pour la famille 'vie domestique' : Exutoire (dechetterie)  

``` sql
SELECT famille_dechet, type, label COUNT(*) AS nb_occurrences
FROM project
GROUP BY famille_dechet, type, label
ORDER BY famille_dechet, nb_occurrences DESC;

```
##### 10. Quels sont les labels d'exutoires associés à chaque famille de déchets ?
Voir question précdente. 
##### 11. Indiquer une clé qui permettrait de s'assurer que les données soient réparties de manière uniforme sur un cluster distribué.
Une clé sur le type d'exutoire (Exutoire / Conseil )
##### 12. Indiquer une clé qui permettrait de ne traiter les requêtes ne concernant que les déchets sur un seul noeud.
Une clé UUID sur l'id dechet 


# Partie 2

##### 1. Quels sont les communes qui ont eu un indice de qualité de l'air `Moyen` ?
Pérignat-lès-Sarliève, Pont-du-Château, Aubière, Aulnat, Beaumont, Blanzat, Orcines, Le Cendre, Cébazat, Durtol, Gerzat...
```javascript
db.collection.distinct("properties.lib_zone", {
  "properties.lib_qual": "Moyen"
})
```
##### 2. Quelle est le code de qualité de l'air maximal sur la commune de `Saint-Genès-Champanelle` ?
le code est 3 
```javascript
db.collection.find({
  "properties.lib_zone": "Saint-Genès-Champanelle"
}).sort({ "properties.code_qual": -1 }).limit(1)
```
##### 3. Quelles sont les différentes communes observées ?

Aubière,Aulnat,Beaumont,Blanzat,Ceyrat,Chamalières,Châteaugay,Clermont-Ferrand,Cournon-d'Auvergne,Cébazat,Durtol, Gerzat,Le Cendre,Lempdes,Nohanent,Orcines,Pont-du-Château,Pérignat-lès-Sarliève,Romagnat,Royat,Saint-Genès-Champanelle

```javascript
db.collection.distinct("properties.lib_zone")
```

##### 4. Quelle est le code qualité moyen par commune ?
Le code qualité moyen est de 2 pour la majorité des communes mais 2.1 pour les communes suivantes : Saint-Genès-Champanelle,Ceyrat, Royat, Durtol, Chamalières, Orcines
```javascript
db.collection.aggregate([
  {
    $group: {
      _id: "$properties.lib_zone",
      code_qual_moyen: { $avg: "$properties.code_qual" }
    }
  },
  {
    $project: {
      _id: 0,
      lib_zone: "$_id",
      code_qual_moyen: 1
    }
  }
])
```
##### 5. Quelle est la qualité de l'air la plus récente dans un rayon de 1 km de la position GPS `45.77441539864761, 3.0890499134717686` ?
L'utilisation de `createIndex` permet de créer un index géospatiale pour le champ géometry

Et donc la qualité de l'air la plus récente dans un rayon de 1 km est `"Moyen"`

```javascript
db.collection.createIndex({ "geometry": "2dsphere" })

db.collection.find({
  "geometry": {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [3.0890499134717686, 45.77441539864761]
      },
      $maxDistance: 1000, 
    }
  }
}).sort({ "properties.date_ech": -1 }).limit(1)
```
##### 6. Combien y-a t'il de relevé d'indice différent pour les communes dont la lettre commence par `C` ?

Il y a 6 relevés pour les communes dont la lettre commence par C.
``` javascript
db.collection.distinct("properties.lib_zone", {
  "properties.lib_zone": /^C/i
}).length
```
##### 7. Pour chaque libellé de qualité de l'air, quelles sont les communes qui lui sont associé ?

Résultats :
```
"_id" : "Moyen",
"communes" : [
"Chamalières",
"Cournon-d'Auvergne",
"Romagnat",
"Aubière",
"Gerzat",
"Orcines",
"Blanzat",
"Ceyrat",
"Pérignat-lès-Sarliève",
"Clermont-Ferrand",
"Le Cendre",
"Cébazat",
"Pont-du-Château",
"Durtol",
"Châteaugay",
"Beaumont",
"Nohanent",
"Saint-Genès-Champanelle",
"Lempdes",
"Royat",
"Aulnat" ]

"_id" : "Dégradé",
"communes" : [
"Durtol",
"Ceyrat",
"Royat",
"Chamalières",
"Saint-Genès-Champanelle",
"Orcines"
]
```

```javascript
db.collection.aggregate([
  {
    $group: {
      _id: "$properties.lib_qual",
      communes: { $addToSet: "$properties.lib_zone" }
    }
  }
])
```

##### 8. Quelle est la concentration moyenne de `PM10` dans la région `Auvergne-Rhône-Alpes` le `22 octobre 2023` ?
la concentration moyenne est de 7.74 : `"concentration_moyenne_PM10" : 7.739757142857142`

```javascript
db.collection.aggregate([
  {
    $match: {
      "properties.date_ech": "2023-10-22T00:00:00Z",
      "properties.nom_division": "Clermont Auvergne Métropole"
    }
  },
  {
    $group: {
      _id: null,
      concentration_moyenne_PM10: { $avg: "$properties.conc_pm10" }
    }
  }
])
```
##### 9. Afficher l'évolution de la concentration de `NO2` dans la commune de `Clermont-Ferrand` à travers le temps.
Résulats : 
```javascript
{
    "_id" : {
        "date" : "2023-10-17T00:00:00Z"
    },
    "moyenne_NO2" : 50.732014285714286
}
{
    "_id" : {
        "date" : "2023-10-18T00:00:00Z"
    },
    "moyenne_NO2" : 26.73774285714286
}
{
    "_id" : {
        "date" : "2023-10-19T00:00:00Z"
    },
    "moyenne_NO2" : 39.2756
}
{
    "_id" : {
        "date" : "2023-10-20T00:00:00Z"
    },
    "moyenne_NO2" : 30.77321904761905
}
{
    "_id" : {
        "date" : "2023-10-21T00:00:00Z"
    },
    "moyenne_NO2" : 5.952457142857143
}
{
    "_id" : {
        "date" : "2023-10-22T00:00:00Z"
    },
    "moyenne_NO2" : 35.02343333333334
}
{
    "_id" : {
        "date" : "2023-10-23T00:00:00Z"
    },
    "moyenne_NO2" : 37.163247619047624
}
{
    "_id" : {
        "date" : "2023-10-24T00:00:00Z"
    },
    "moyenne_NO2" : 39.599004761904766
}
{
    "_id" : {
        "date" : "2023-10-25T00:00:00Z"
    },
    "moyenne_NO2" : 33.593490476190475
}
```

```javascript
db.collection.aggregate([

  {
    $group: {
      _id: { date: "$properties.date_ech" },
      moyenne_NO2: { $avg: "$properties.conc_no2" }
    }
  },
  {
    $sort: {
      "_id.date": 1
    }
  }
])
```
##### 10. Quelle est la commune qui a l'indice de concentration d'ozone le plus faible sur la période observée ?

La commune avec le plus faible indice de concentration d'ozone est la commune de Durtol 

```javascript
db.collection.aggregate([
  {
    $match: {
      "properties.code_o3": 3
    }
  },
  {
    $sort: {
      "properties.conc_o3": 1
    }
  },
  {
    $limit: 1
  },
  {
    $project: {
      _id: 0,
      lib_zone: "$properties.lib_zone",
      conc_o3: "$properties.conc_o3"
    }
  }
])

```
##### 11. Quelle serait la clé de sharding à utiliser pour pouvoir s'assurer que tous les enregistrements d'une même commune soient traités par le même noeud ?
On doit  choisir une clé de sharding qui garantit que les données d'une même commune sont distribuées sur le même fragment. Dans ce cas, la clé de sharding appropriée serait le nom de la commune, c'est-à-dire `"properties.lib_zone"`.
