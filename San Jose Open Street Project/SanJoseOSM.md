#Cleaning and Exploring the Open Street Data for San-Jose Metro Area
2016/04

Python, ETL, data wrangling, XML Parsing, Json, MongoDB, NoSql

## Summary
This project cleans and explores the Open Street Map (OSM) data at the San-Jose metro area where I currently live. I downloaded the OSM data from MapZen. The Open street map is the largest open source transportation dataset created by individual contributions. Despite of its resourcefulness, it contains quite many human-made errors. In this project, I am focused on understanding the quality of open street map data from the point of view as an urban planner and transportation researcher. I explored the data and cleaned some of the data in python and transformed them into json. I also shared results of my exploration on the fixed json data in MongoDB.


## The Python File Description
**Sample.osm** is created by start.py which extract 1/20 data from the original file for testing. 
 
Data audit files are under `data_audit` folder.

There are two data cleaning and transforming files:  

- One cleaning the data and saved the file to a new xml file: `cleanning_OSM_data_to_a_new_XML.py`  
- The other the cleaning the data and save it to a json file: `creating_json_data.py`. Note when we transform the data to json, we set each node or way as one observation and add attributes to the observation.
  
  

## 1.Data Quality and Cleaning

I first examined the data quality of this project. I am focused on four areas: street activities (residence and amenities), street system, the bicycle network and the public transportation system. Below are the problems of data quality and my suggestions.

### 1.1 The Street Activity Data

#### Data Quality: Incomplete

My first finding is that the OSM’s address data is very incomplete. For city planner and transportation professionals, the address data of most neighborhoods in San Jose Area at OSM is not good enough for research or data analysis.

When I audit my sample file using audit_postcode.py to see how many addresses are there in each neighborhood/zipcode. I notice there is one neighborhood having data larger than all the other areas combined.

 ```
 {'95054': 4, '95050': 1, '95037': 6, '95035': 2, '95014': 386, '95128': 3, '95124': 1, '95125': 3, '95126': 2, '95127': 6, '95070': 7, '95123': 1, '94085': 1, '94086': 1, '94087': 5, '95002': 1, '95131': 1, '95113': 1, '95135': 1, '95134': 1, '95140': 2, '95051': 3}  
 ```
 

As a result, I went to openstreetmap website and compared the data in neighborhood 95014 and other areas. I found that the 95014 neighborhood is an outlier as most of residential properties and amenities are clearly marked, while other neighborhoods do not have this information.

#### Usage suggestion:

The data variation between 95014 and other neighborhood indicates the address information at OSM is very incomplete. I suggest not use San Jose OSM data for urban amenity/resident location analysis.

### 1.2 The Street System Data
#### Data Quality: The Wrong Tag

In contrary to street activity information, the OSM maintains quite a good set of data in terms of road system in San Jose area. However, it is hard for me to get any query result using keywords like postcode or county. I examined the data and found out this is because the key value for street-related information is not clean. In the current dataset, the country, postcode and uid of most streets are “TIGER:country”, “TIGER: `zip_left`” or “TIGER: `zip_right`”, “TIGER:uuid”. Since many street data of the open street map were imported from the Tiger dataset[1], these data are marked with “tiger”[2] at the value “k”. It is inconsistent with the standard tag suggested by OSM. For researchers who don’t know this but want to analyze the road system by simply querying the county, postcode and etc., this would cause some problem, as they cannot simply search the data by using “county” or “zipcode”.

#### Usage suggestion and Cleanning Solution

- Update the keys by removing the string of "TIGER"  in the value of k and make it more consistent with OSM map key standard. See `cleanning_OSM_data_to_a_new_XML.py`

- While wrangling these data and tranform it to json, I also add these fields to a dictionary called highway and set node[‘highway’]=highway. See`cleanning_OSM_data_to_a_new_XML.py`

### 1.3 The Cycleway Data

#### Data quality: Inconsistency and Inacuracy

OSM provide quite a lot of data on the cycleway information. It is something that transportation researchers would be very excited about. However, the data needs further validation and the accuracy requires certain improvement.

##### Inconsistency problem

The bicycle information are tagged at two tags. One is called “bicycle” with a value of “yes” or “no” indicating whether a bicycle has a right of way[3]. The other is called “cycleway” with value such as “lane”, “track” or “share” indicating its format of bike infrastructure on the street[4].

To examine the accuracy of data, I did a cross validation using `audit_bicycle_crosscheck.py` to see whether the data in bicycle field and cycleway field is consistent. The code returns the value for “bicycle”, “cycleway” and “element id” of the elements that have both tags. Some of the results are as follows.

```		yes no 8941958  
		yes lane 8942625  
		yes track 26274651  
		yes no 233802747  
		yes no 234183465  
		yes no 234318879  
		...
```


 We can see many elements having the bicycle value as yes, but the cycleway value as no. This is confusing and require further examination and validation.

##### The unnecessary “no” value of the cycleway:

I also simple query on the tag cycleway (audit_cycleway.py) and found out the “no” group that indicates there is no cycleway in this road.

`{'track': 13, 'lane': 66, 'no': 17}`

 This is redundant and causing confusing, as applications like opencyclemap adds cycleway data from OSM data tagged with the field of cycleway. The no value induces problem.  For example, South Mary Avenue at Sunnyvale has no cycle lane but tagged as “k”=”cycleway”, “v”=”no”. The “no” value for cycleway makes the avenue appear at opencyclemap.

#### Usage Suggestion and Cleaning Solution

##### Usage   

Only take the cycleway into account only when the value is not “no”. Some can double check at this website <http://mijndev.openstreetmap.nl/~ligfietser/fiets/index.html>

##### Cleaning 
Remove the cycleway tag if the k is cycleway and the value is no, as it does not provide any extra information. See `cleanning_OSM_data_to_a_new_XML.py`;

### 1.4 Public Transportation

There is very limited information on public transportation. There are no `public_transportation` tag[5] in this entire dataset (using `audit_public_transport`) and the amenity with the value of bus_stop is only 1 throughout the dataset (using amenity.py). This part of data is incomplete and requires further contribution.

###1.5 Others: Street name, Postcode and City Name etc.

####Data Quanlity: Inaccurate
I found Street name end with abbreviation “St”, “Blvd” or lower-case Street(see audit script at audit_street.py). Using similar methods, I found problematic value for the addr:postcode and addr:city fields.

Result of some of street name audit of sample.osm 

```'Ave': {'1425 E Dunne Ave'},
'Blvd': {'McCarthy Blvd'},
'Rd': {'Mt Hamilton Rd'},
'ave': {'wilcox ave'}}
```

#### Cleaning Solution
Change abbreviation to full name like Street, Boulevard, and change the postcode to five digit standard digits.

see `cleanning_OSM_data_to_a_new_XML.py`;

##2. Overview of the data in MangoDB


`>use test`


db.osm.find().count()
```

db.runCommand ( { distinct: "osm", key: "created.user" } )['values'].length
```
db.osm.find({"type":"node"}).count()
```

```
```

db.osm.aggregate([{$group:{_id:"$created.user",count:{$sum:1}}},{$sort:{count:-1}},{$limit:5}])
```

## 3. Additional thoughts

What else could we find using at OSM San Jose?



```
```

```
```
db.osm.aggregate([{$match:{"amenity": {"$exists":1},'address.postcode':{"$exists":1}}},{$group:{_id:"$address.postcode",count:{$sum:1}}},{$sort:{count:-1}},{$limit:5}])
```

 
```
```

```
```

The neighborhood 94087 seems to have some good bicycle infasturcture. 


