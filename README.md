# Project 3: OpenStreetMap Data Wrangling with SQL

**Name:** Raju Singh

**Map Area**: I have choosen Ahmedabad as it is near to my hometown metro city.

* Location: Ahmedabad, India
* [OpenStreetMap URL](https://www.openstreetmap.org/export#map=11/23.0217/72.5797)
* [MapZen URL](https://s3.amazonaws.com/metro-extracts.mapzen.com/ahmedabad_india.osm.bz2)

# 1. Data Audit
###Unique Tags
Looking at the XML file, I found that it uses different types of tags. So, I parse the Ahmedabad,India dataset using ElementTree and count number of the unique tags.
`mapparser.py` is used to count the numbers of unique tags.

* 'bounds': 1,
* 'member': 2288,
* 'nd': 639811,
* 'node': 550820,
* 'osm': 1,
* 'relation': 510,
* 'tag': 99409,
* 'way': 82308

### Patterns in the Tags
The `"k"` value of each tag contain different patterns. Using `tags.py`, I created  3 regular expressions to check for certain patterns in the tags.
I have counted each of four tag categories.

*  `"lower" : 97364`, for tags that contain only lowercase letters and are valid,
*  `"lower_colon" : 2004`, for otherwise valid tags with a colon in their names,
*  `"problemchars" : 7`, for tags with problematic characters, and
*  `"other" : 34`, for other tags that do not fall into the other three categories.

# 2. Problems Encountered in the Map
###Street address inconsistencies
The main problem we encountered in the dataset is the street name inconsistencies. Below is the old name corrected with the better name. Using `audit.py`, we updated the names.
 

* **Abbreviations** 
    * `Rd -> Road`
* **LowerCase**
    * `gandhi -> Gandhi`
* **Misspelling**
    * `socity -> Society`
* **Hindi names**
    * `rasta -> Road`
* **UpperCase Words**
    * `sbk -> SBK`

### City name inconsistencies
Using `audit.py`, we update the names

* **LowerCase**
	* `ahmedabad -> Ahmedabad`
* **Misspelling**
	* `Ahmadabad -> Ahmedabad`


#3. Data Overview
the csv files are obtained using ahmedabad_sample.osm because processing the ahmedabad_india.osm is taking too long.

### File sizes:
* `ahmedabad_india.osm: 109.2 MB`
* `ahmedabad_sample.osm 3.67 MB `
* `nodes_csv: 1.46 MB`
* `nodes_tags.csv: 7.34 KB`
* `ways_csv: 162 KB`
* `ways_nodes.csv: 506 KB`
* `ways_tags.csv: 101 KB`
* `ahmedabad.db: 34.2 MB`

###Number of nodes:
``` python
sqlite> SELECT COUNT(*) FROM nodes
```
**Output:**
```
 550820
 ```

### Number of ways:
```python
sqlite> SELECT COUNT(*) FROM ways
```
**Output:**
```
82308
```

###Number of unique users:
```python
sqlite> SELECT COUNT(DISTINCT(e.uid))          
FROM (SELECT uid FROM nodes UNION ALL SELECT uid FROM ways) e;
```
**Output:**
```
364
```

###Top contributing users:
```python
sqlite> SELECT e.user, COUNT(*) as num
FROM (SELECT user FROM nodes UNION ALL SELECT user FROM ways) e
GROUP BY e.user
ORDER BY num DESC
LIMIT 10;
```
**Output:**
```
[(u'uday01', 177284), 
(u'sramesh', 136707), 
(u'chaitanya110', 123131), 
(u'shashi2', 49502), 
(u'shravan91', 22667), 
(u'vkvora', 21961), 
(u'shiva05', 19669), 
(u'bhanu3', 12515), 
(u'Oberaffe', 7094), 
(u'kailashdhirwani', 5877)]
```

###Number of users contributing only once:
```python
sqlite> SELECT COUNT(*) 
FROM
    (SELECT e.user, COUNT(*) as num
     FROM (SELECT user FROM nodes UNION ALL SELECT user FROM ways) e
     GROUP BY e.user
     HAVING num=1) u;
```
**Output:**
```
95
```

# 4. Additional Data Exploration
###Common ammenities:
```python
sqlite> SELECT value, COUNT(*) as num
FROM nodes_tags
WHERE key='amenity'
GROUP BY value
ORDER BY num DESC
LIMIT 10;
```
**Output:**
```
place_of_worship =69
restaurant =49
hospital =33
bank =32
school =27
atm =24
fuel =23
fast_food =21
cafe =15
library =14
```

###Biggest religion:
```python
sqlite> SELECT nodes_tags.value, COUNT(*) as num
FROM nodes_tags 
    JOIN (SELECT DISTINCT(id) FROM nodes_tags WHERE value='place_of_worship') i
    ON nodes_tags.id=i.id
WHERE nodes_tags.key='religion'
GROUP BY nodes_tags.value
ORDER BY num DESC
LIMIT 1;
```
**Output:**
```
Hindu :	34
```
###Popular cuisines
```python
sqlite> SELECT nodes_tags.value, COUNT(*) as num
FROM nodes_tags 
    JOIN (SELECT DISTINCT(id) FROM nodes_tags WHERE value='restaurant') i
    ON nodes_tags.id=i.id
WHERE nodes_tags.key='cuisine'
GROUP BY nodes_tags.value
ORDER BY num DESC;
```
**Output:**
```
regional =7
vegetarian =3
indian =2
pizza =2
Punjabi,_SouthIndia,_Gujarati Thali =1
burger =1
```

# 5. Conclusion
The OpenStreetMap data of Ahmedabad is of fairly reasonable quality but the typo errors caused by the human inputs are significant. We have cleaned a significant amount of the data which is required for this project. But, there are lots of improvement needed in the dataset. The dataset contains very less amount of additional information such as amenities, tourist attractions, popular places and other useful interest. The dataset contains very old information which is now incomparable to that of Google Maps or Bing Maps.

So, I think there are several opportunities for cleaning and validation of the data in the future. 

### Additional Suggestion and Ideas

#### Control typo errors
* We can build parser which parse every word input by the users.
* We can make some rules or patterns to input data which users follow everytime to input their data. This will also restrict users input in their native language.
* We can develope script or bot to clean the data regularly or certain period.

#### More information
* The tourists or even the city people search map to see the basic amenities provided in the city or what are the popular places and attractions in the city or near outside the city. So, the users must be motivated to also provide these informations in the map.
* If we can provide these informations then there are more chances to increase views on the map because many people directly enter the famous name on the map.

# Files
* `README.md` : this file
* `ahmedabad_sample.osm`: sample data of the OSM file
* `audit.py` : audit street, city and update their names
* `data.py` : build CSV files from OSM and also parse, clean and shape data
* `database.py` : create database of the CSV files
* `mapparser.py` : find unique tags in the data
* `query.py` : different queries about the database using SQL
* `sample.py` : extract sample data from the OSM file
* `tags.py` : count multiple patterns in the tags

