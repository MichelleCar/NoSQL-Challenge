# NoSQL-Challenge
The UK Food Standards Agency evaluates various establishments across the United Kingdom, and gives them a food hygiene rating. You've been contracted by the editors of a food magazine, Eat Safe, Love, to evaluate some of the ratings data in order to help their journalists and food critics decide where to focus future articles.

![Critics-heroGIF_1619159531](https://user-images.githubusercontent.com/115101031/214083673-5d492180-345c-433b-9feb-aec0638dbdba.gif)

## An Exploration of NoSQL Databases Using MongoDB

MongoDB stands at the root of the construction of many applications by industry giants.  Companie including Forbes, Toyota, Vodafone, and Verizon, to name only a few of the more than 35,000 customers, have built applications using this powerhouse NoSQL technology.

NoSQL databases are particularly ideal for specific data models and have flexible schemas for building modern applications. They are widely recognized for their ease of development, functionality, and performance at scale. They use a variety of data models for accessing and managing data. NoSQL databases are optimized for applications that require large data volume, low latency, and flexible data models, which are achieved by relaxing some of the data consistency restrictions of other databases.

MongoDB is a document database with a high degree of scalability and flexibility.  In application code, data is represented often as an object or JSON-like document because it is an efficient and intuitive data model for developers. Document databases make it easier for developers to store and query data in a database by using the same document model format that they use in their application code. The flexible, semistructured, and hierarchical nature of documents and document databases allows them to evolve with applications’ needs. The document model works well with catalogs, user profiles, and content management systems where each document is unique and evolves over time. 

![what-is-mongodb-pnoifkiu0jygbjg2lw88ndjisu83aao1ajszea97so](https://user-images.githubusercontent.com/115101031/214097830-5b2d634a-cc35-4057-b05d-a151b5163a73.png)

### Advantages of MongoDB
Here are some of the advantages offered by MongoDB:
* It is easy to install, use and schema-less database.
* Due to it is the ability of a schema-less database, the code which we create defines the schema.
* Data is stored in Binary JSON format, which is key-value pair, no joins complexity is needed.
* It uses RAM to store data; this makes faster access to the data.
* It supports replication; if the primary server goes down during transaction, then the secondary server handles the transaction without human interaction.
* It is cost-effective because it reduces cost on hardware and storage.
* It can save a lot of data which will help in faster query processing.

### Disadvantages of MongoDB
* MongoDB doesn’t support joins like a relational database. Yet one can use joins functionality by adding by coding it manually. But it may slow execution and affect performance.
* MongoDB stores key names for each value pair. Also, due to no functionality of joins, there is data redundancy. This results in increasing unnecessary usage of memory.
* You can have a document size, not more than 16MB.
* You cannot perform the nesting of documents for more than 100 levels.


Sources:
* https://www.mongodb.com/who-uses-mongodb
* https://aws.amazon.com/nosql/
* https://stackoverflow.com/questions/5244437/pros-and-cons-of-mongodb
* https://www.adathedev.co.uk/2011/02/thoughts-on-mongodb-from-sql-server-dev.html


## Project Details and Observations
Using the Pymongo Python library, Pretty Print, Pandas, and Jupyter Notebook, the challenge was divided into three parts:
1) Part 1 - Database and and Jupyter Notebook Set Up
2) Part 2 - Update the Database
3) Part 3 - Exploratory Analysis

#### Part 1: Database and and Jupyter Notebook Set Up
* The process begins with activating our MongoDB service in the background using the Terminal: brew services start mongodb-community@6.0
* Then importing the JSON data provided: **mongoimport --type json -d uk_food -c establishments --drop --jsonArray establishments.json**
* Once the data is imported, using the mongosh terminal, we named our database (*use uk_food*) and our collection (*establishmnets*)
* After initiating Jupyter Notebook, we imported the Pymongo and Pretty Print libraries, created an instance of the Mongo Client, and assigned our database and collections to a variable.

Key thoughts:
* Because you are using a server that sits in the background, it is important to constantly verify that your server and notebook are communicating.  Printing lists of our databases <**print(mongo.list_database_names())**>, collections <**print(db.list_collection_names())**>, and documents <**pprint(db.establishments.find_one())**> allowed us to confirm the database was active and that the notebook was successfully pulling data.

#### Part 2: Update the Database
The amagazine editors have some requested modifications for the database before we can perform any queries or analysis.
1. Add a new restaurant ("Penang Flavours") to the database (*insert_one*)
2. Update the Business Type ID to reflect that it is of the type "Restaurant/Cafe/Canteen"
 
      **query = {'BusinessType': 'Restaurant/Cafe/Canteen'}
      
        fields = {'BusinessTypeID': 1, 'BusinessType': 1}
      
        results = list(establishments.find(query, fields))
        
        Using '$set' to change the Business Type ID**
        
3. The magazine is not interested in establishments in Dover, so we will remove all documents containing the "Dover" local authority (*994 entries*, using **'delete_many'**)
4. Some of the number values are stored as strings, when they should be stored as numbers.  Using 'update_many' we converted longitude and latitude to decimals using, <**{'$set': {"geocode.latitude": {'$toDecimal': "$geocode.latitude"}}}**>

Key thoughts:
* Because the server is live, any changes are made in real time.  That means that going back to a previous step and trying to make changes will result in errors.  For example, after deleting all the records with local authority in "Dover", I noticed a small element I needed to make earlier in the code.  After making hte change, any instructions related to searching and deleting the "Dover" documents no longer worked.  I had to reinstall my JSON data to ensure that Dover was refreshed/reinstalled into the database to successfully run my code.
* MongoDB has a rich lobrary of operators...it was important to know which one to use for a specific task 

#### Part 3: Exploratory Analysis
Eat Safe, Love has specific questions they need answered, which will help them find the locations they wish to visit and those they wish to avoid.

After importing our dependencies, creating our instance of the Mongo client, assigning our database and collections to variable, and verifying that data is callable, we proceeded to some analysis.
1. Using our query <**"query1 = {'scores.Hygiene': {'$eq': 20}}**> to find hygiene scores equal to 20, we found that 41 establishments.
2. In our next query, we found 34 establishments in London with a rating of 4, using: 
                 <**query2 = {'LocalAuthorityName': {'$regex': 'London'}, 'RatingValue': {'$gte': '4'}}**>
                 
3. Next we searched for the top 5 establishments within 0.01 degrees of our newly added restaurant "Penang Flavours" with the lowest hygiene scores...these seemed like options we might want to avoid.  After converting our longitude and latitude to decimals, the following query gave us the best results:

**degree_search = 0.01
latitude = 51.49014200
longitude = 0.08384000

query3 = {'geocode.latitude': {'$gte': latitude - degree_search, '$lte': latitude + degree_search}, 
         'geocode.longitude': {'$gte': longitude - degree_search, '$lte': longitude + degree_search},
         'RatingValue': '4'
        }

*Sort by hygiene score*
sort = [('scores.Hygiene', 1)]

*Limit to top 5 establoshments*
limit = 5**

4. Finally, we searched how many restaurants had a hygiene score of 0, grouped them by jurisdiction, and sorted them from highest to lowest (number of establishments).  We created a data pipeline that:

** *Matches establishments with a hygiene score of 0*
match_query = {'$match': {'scores.Hygiene': {'$eq': 0}}}

*Groups the matches by Local Authority*
group_query = {'$group': {'_id': "$LocalAuthorityName", 'count': { '$sum': 1 }}}

*Sorts the matches from highest to lowest*
sort_values = {'$sort': { 'count': -1 }}

*Put the pipeline together*
pipeline = [match_query, group_query, sort_values]**
                 






