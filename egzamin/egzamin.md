Egzamin, Daniel Jakubek
---------------------------------------------
Wykorzystany sprzet:

|X|Informacje o komputerze                             |
|-----------------------|------------------------------|
| Procesor              | Intel Core i7-3687U 2.60 GHz |
| Ilość rdzeni          | 2 fizyczne, 4 logiczne       |
| Pamięć                | 7,88 GB                      |
| Dysk                  | SSD PLEXTOR PX-128M6M 128 GB |
| System operacyjny     | Linux Ubuntu Gnome 14.04.3   |
| Typ systemu           | 64 - bit                     |
| Model komputera       | Ultrabook DELL XPS 14        |
| MongoDB               | 3.0.7                        |
| Python                | 2.7.6                        |

**Zaimportowana baza [zabytki polski](http://otwartezabytki.pl/strony/pobierz-dane)**

Zliczyłem wszystkie rekordy
```sh
db.zabytki.count()
42346
```

**Wyświetlenie przykładowego rekordu**
------------------------------------------------------------------------------
```sh
db.zabytki.find().limit(1).skip(500)
{
  "_id": ObjectId("56ad2e6eed08f7ed69655275"),
  "id": 10925,
  "nid_id": "601275",
  "identification": "Kamienica",
  "common_name": "",
  "description": "",
  "categories": [
    "mieszkalny"
  ],
  "state": "unchecked",
  "register_number": "A/387/1 z 22.11.1993",
  "dating_of_obj": "190119-03",
  "street": "ul. Cieszkowskiego 16",
  "latitude": 53.1311236,
  "longitude": 18.0062492,
  "tags": [ ],
  "country_code": "PL",
  "fprovince": null,
  "fplace": null,
  "documents_info": null,
  "links_info": null,
  "main_photo": {
    "id": null,
    "relic_id": null,
    "author": null,
    "date_taken": null,
    "alternate_text": null,
    "file": {
      "url": "/assets/fallback/photo_default.png",
      "icon": {
        "url": "/assets/fallback/photo_icon_default.png"
      },
      "mini": {
        "url": "/assets/fallback/photo_mini_default.png"
      },
      "midi": {
        "url": "/assets/fallback/photo_midi_default.png"
      },
      "maxi": {
        "url": "/assets/fallback/photo_maxi_default.png"
      },
      "full": {
        "url": "/assets/fallback/photo_full_default.png"
      }
    },
    "file_full_width": null
  },
  "events": [ ],
  "entries": [ ],
  "links": [ ],
  "documents": [ ],
  "alerts": [ ],
  "descendants": [ ],
  "photos": [ ],
  "place_id": 90115,
  "place_name": "Bydgoszcz",
  "commune_name": "Bydgoszcz",
  "district_name": "Bydgoszcz",
  "voivodeship_name": "kujawsko-pomorskie",
  }
}
```

======================================================================

##Cztery agregacje korzystające z Aggregation Pipeline w JavaScript

-------------------------------------------------------------------------

**1. Wyświetlenie ilości zabytków w poszczególnych województwach**

```sh
db.zabytki.aggregate( {"$group" : {"_id" : "$voivodeship_name", "count" : {"$sum" : 1}}}, {"$sort":{"count":-1}})
```

**Wynik aggregacji**

```sh
{
  "result": [
    {
      "_id": "dolnośląskie",
      "count": 5222
    },
    {
      "_id": "wielkopolskie",
      "count": 4181
    },
    {
      "_id": "warmińsko-mazurskie",
      "count": 4072
    }
  ```
  **Przedstawiony graficznie**
  
  ![wykres1](wykres1.png)

------------------------------------------------------------------------------------------------------------


**2. Wyświetlenie ilości zabytków według datowania roku budowy obiektów**

```sh
db.zabytki.aggregate([ {"$group" : {"_id" : "$dating_of_obj", "ilosc" : {"$sum" : 1}}}, {"$sort":{"ilosc":-1}}, {"$limit" :25}])
```

**Cztery pierwsze pozycje wyniku**

```sh
{
      "_id": "",
      "ilosc": 7402
    },
    {
      "_id": "XVIII",
      "ilosc": 2106
    },
    {
      "_id": "1 poł. XIX",
      "ilosc": 1646
    },
    {
      "_id": "2 poł. XIX",
      "ilosc": 1587
    },
```

**Przedstawienie pierwszych 25 wyników w fromie wykresu**
![wykres2](wykres2.png)

Jak widać jest sporo zabytków w Polsce, których data budowy nie jest oszacowana

---------------------------------------------------------------------------------------

**3. Wyświetlenie ilości zabytków według kategorii**

```sh
db.zabytki.aggregate([ {"$group" : {"_id" : "$categories", "ilosc" : {"$sum" : 1}}}, {"$sort":{"ilosc":-1}}, {"$limit" :30}])
```

**Cztery pierwsze wyniki**
```
{
  "result": [
    {
      "_id": [
        "mieszkalny"
      ],
      "ilosc": 15009
    },
    {
      "_id": [
        "sakralny"
      ],
      "ilosc": 6733
    },
    {
      "_id": [ ],
      "ilosc": 6306
    },
    {
      "_id": [
        "dworski_palacowy_zamek"
      ],
      "ilosc": 4715
    },
    {
      "_id": [
        "park_ogrod",
        "dworski_palacowy_zamek"
      ],
      "ilosc": 1275
    },
```
**Przedstawienie pierwszych 25 wyników w fromie wykresu**


![wykres3](wykres3.png)

------------------------------------------------------------------------------

**4. Wyświetlenie ilości zabytków powiązanych tematycznie z obiektami sakralnymi oraz obiektów takich jak parki i ogrody**

```sh
db.zabytki.aggregate({$match: { categories: 'sakralny'}},{"$group" : {"_id" : "sakralny", "ilosc" : {"$sum" : 1}}})
```

**Wynik agregacji obiektów sakralnych:**

```sh
{
  "result": [
    {
      "_id": "sakralny",
      "ilosc": 9974
    }
  ],
  "ok": 1
}
```
-------------------------------------------------------------------




**Agregacja dotycząca wyświetlenia wszystkich zabytków powiązanych z ogrodami lub parkami**

```sh
db.zabytki.aggregate({$match: { categories: 'park_ogrod'}},{"$group" : {"_id" : "park_ogród", "ilosc" : {"$sum" : 1}}})

```
```sh
{
  "result": [
    {
      "_id": "park_ogród",
      "ilosc": 2151
    }
  ],
  "ok": 1
}
```
---------------------------------------------------------------------------

**A więc w Polsce mamy około 10 tyś obiektów powiązanych z tematyką sakralną oraz ponad 2 tyś zabytków w postaci parków lub ogrodów.**

Mogło być więcej, być może baza danych nie jest kompletnie uzupełniona przez jej twórców :)



=============================================================================

###Powyższe agregacje wykonane w innym języku niż JavaScript.
--------------------------------------------------------------
Do wykonania tego zadania wykorzystałem język programowania PYTHON (2.7.6 )

**Agregacja 1 (Wyświetlenie ilości zabytków w poszczególnych województwach)**

```sh
# -*- coding: utf-8 -*-
import pymongo
import json
from bson.son import SON
from pymongo import MongoClient
client = MongoClient()

db = client['GEOZABYTKI']
collection = db['zabytki']

pipeline = [
  { "$group": {"_id": "$voivodeship_name", "count" : {"$sum" : 1}}},
  {"$sort":{"count":-1}}
]

zapytanie = db.zabytki.aggregate(pipeline)
for doc in zapytanie:
   print(doc)
```

**Wynik**

![prt1.png](prt1.png)


Widać że są pewne problemy z kodowaniem polskich znaków, nawet po dodaniu a na początku pliku odpowedniego kodowania.

---------------------------------------------------
**Agregacja2 (Wyświetlenie ilości zabytków według datowania roku budowy obiektów)**

```sh
# -*- coding: utf-8 -*-
import pymongo
import json
from bson.son import SON
from pymongo import MongoClient
client = MongoClient()

db = client['GEOZABYTKI']
collection = db['zabytki']

pipeline = [
  {"$group" : {"_id" : "$dating_of_obj", "ilosc" : {"$sum" : 1}}},
  {"$sort":{"ilosc":-1}}, {"$limit" :25}
]

zapytanie = db.zabytki.aggregate(pipeline)
for doc in zapytanie:
   print(doc)

```

**Wynik**

![prt2.png](prt2.png)


**Agregacja3 (Wyświetlenie ilości zabytków według kategorii)**

```sh
# -*- coding: utf-8 -*-
import pymongo
import json
from bson.son import SON
from pymongo import MongoClient
client = MongoClient()

db = client['GEOZABYTKI']
collection = db['zabytki']

pipeline = [
  {"$group" : {"_id" : "$categories", "ilosc" : {"$sum" : 1}}}, 
  {"$sort":{"ilosc":-1}}, {"$limit" :30}
]

zapytanie = db.zabytki.aggregate(pipeline)
for doc in zapytanie:
   print(doc)

```

**Wynik**

![prt3.png](prt3.png)

**Agregacja4 (Agregacja obiektów sakralnych)**

```sh
# -*- coding: utf-8 -*-
import pymongo
import json
from bson.son import SON
from pymongo import MongoClient
client = MongoClient()

db = client['GEOZABYTKI']
collection = db['zabytki']

pipeline = [
  {"$match" : {"categories": 'sakralny'}}, 
  {"$group" : {"_id" : "sakralny", "ilosc" : {"$sum" : 1}}}
]

zapytanie = db.zabytki.aggregate(pipeline)
for doc in zapytanie:
   print(doc)
```

**Wynik**

![prt4.png](prt4.png)

--------------------------------------------------------------------------------------
**Agregacja5 (Agregacja parków/ogrodów)**

```sh
# -*- coding: utf-8 -*-
import pymongo
import json
from bson.son import SON
from pymongo import MongoClient
client = MongoClient()

db = client['GEOZABYTKI']
collection = db['zabytki']

pipeline = [
  {"$match": { "categories": 'park_ogrod'}},
  {"$group" : {"_id" : "park_ogród", "ilosc" : {"$sum" : 1}}}
]

zapytanie = db.zabytki.aggregate(pipeline)
for doc in zapytanie:
   print(doc)
```

**Wynik**

![prt5.png](prt5.png)
