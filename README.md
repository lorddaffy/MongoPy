#### Datasets:

[prizes.zip](https://github.com/lorddaffy/PyMongo/files/8526471/prizes.zip)
_____

[laureates.zip](https://github.com/lorddaffy/PyMongo/files/8526472/laureates.zip)

_____

**Apply MongoDB to search and analytics. Working with unprocessed data from the official nobelprize.org API,Exploring and answering questions about Nobel Laureates and prizes.**

_________________________

- Save a list, called db_names, of the names of the databases managed by our connected client.
- Similarly, save a list, called nobel_coll_names, of the names of the collections managed by the "nobel" database.
```
# Save a list of names of the databases managed by client
client=MongoClient()
db_names = client.list_database_names()
nobel_coll_names=client.nobel.list_collection_names()

```
_________________________

- Connect to the nobel database.
- Fetch one document from each of the prizes and laureates collections, and then take a look at the output in the console to see the format and type of the documents in Python.
- Since prize and laureate are dictionaries, you can use the .keys() method to return the keys (i.e. the field names). But it's often more convenient to work with lists of fields.
- Save a list of the fields present in the prize and laureate documents.

```
# Connect to the "nobel" database
db = client.nobel

# Retrieve sample prize and laureate documents
prize = db.prizes.find_one()
laureate = db.laureates.find_one()

# Print the sample prize and laureate documents
print(prize)
print(laureate)
print(type(laureate))

#Since prize and laureate are dictionaries, you can use the .keys() method to return the keys (i.e. the field names). But it's often more convenient to work with lists of fields.

#Use the list() constructor to save a list of the fields present in the prize and laureate documents.
# Get the fields present in each type of document
prize_fields = list(prize.keys())
laureate_fields = list(laureate.keys())

print(prize_fields)
print(laureate_fields)

```
_________________________

- Create a filter criteria to count laureates who died ("diedCountry") in the USA ("USA"). Save the document count as count.
- Create a filter to count laureates who died in the United States but were born ("bornCountry") in Germany.
- Count laureates who died in the USA, were born in Germany, and whose first name ("firstname") was "Albert".
```
# Create a filter for laureates who died in the USA
criteria = {"diedCountry": "USA"}

# Save the count of these laureates
count = db.laureates.count_documents(criteria)
print(count)


# Create a filter for laureates who died in the USA but were born in Germany
criteria = {'diedCountry': 'USA', 
            'bornCountry': 'Germany'}

# Save the count
count = db.laureates.count_documents(criteria)
print(count)

# Create a filter for Germany-born laureates who died in the USA and with the first name "Albert"
criteria = {'bornCountry': 'Germany', 
            'diedCountry': 'USA',
            'firstname': 'Albert'}

# Save the count
count = db.laureates.count_documents(criteria)
print(count)

```
_________________________

- How many laureates were born in "USA", "Canada", or "Mexico"? Save a filter as criteria and your count as count.
- How many laureates died in the USA but were not born there? Save your filter as criteria and your count as count.
```
# Save a filter for laureates born in the USA, Canada, or Mexico
criteria = { 'bornCountry': 
                { "$in": ['USA','Canada','Mexico']}
             }

# Count them and save the count
count = db.laureates.count_documents(criteria)
print(count)

# Save a filter for laureates who died in the USA and were not born there
criteria = { 'diedCountry': 'USA',
               'bornCountry': { "$ne":  'USA'}, 
             }

# Count them
count = db.laureates.count_documents(criteria)
print(count)

```
_________________________

- Save a filter criteria for laureates born in (bornCountry) "Austria" with a non-Austria prizes.affiliations.country.
- Save your count of laureates as count.
```
# Filter for laureates born in Austria with non-Austria prize affiliation
criteria = { 'bornCountry': 'Austria',
               'prizes.affiliations.country': { "$ne":  'Austria'}, 
             }

# Count the number of such laureates
count = db.laureates.count_documents(criteria)
print(count)

```
_________________________

- Use a filter document (criteria) to count the documents that don't have a "born" field.
- Use a filter document (criteria) to find a document for a laureate with at least three elements in its "prizes" array. In other words, does a third element exist for the array? Remember about the zero-based indexing!
```
# Filter for documents without a "born" field
criteria = {'born': {'$exists': False}}

# Save count
count = db.laureates.count_documents(criteria)
print(count)

# Filter for laureates with at least three prizes
criteria = {"prizes.2": {'$exists': True}}

# Find one laureate with at least three prizes
count = db.laureates.find_one(criteria)
# Print the document
print(count)

```
_________________________

- Return a set of all such countries as countries.
```
# Countries recorded as countries of death but not as countries of birth
countries = set(db.laureates.distinct('diedCountry')) - set(db.laureates.distinct('bornCountry'))
print(countries)
```
_________________________
- Determine the number of distinct countries recorded as part of an affiliation for laureates' prizes. Save this as count
```
# The number of distinct countries of laureate affiliation for prizes
count = len(db.laureates.distinct('prizes.affiliations.country'))
print(count)
```
_________________________

- Save a filter document criteria that, when passed to db.prizes.distinct, returns all prize categories shared by three or more laureates. That is, "laureates.2" must exist for such documents.
- Save these prize categories as a Python set called triple_play_categories.
- Confirm via an assertion that "literature" is the only prize category with no prizes shared by three or more laureates.
```
# Save a filter for prize documents with three or more laureates
criteria = {"laureates.2": {"$exists": True}}

# Save the set of distinct prize categories in documents satisfying the criteria
triple_play_categories = set(db.prizes.distinct("category", criteria))
assert set(db.prizes.distinct("category")) - triple_play_categories == {"literature"}

```
_________________________

- Save an $elemMatch filter unshared to count laureates with unshared prizes in categories other than ("not in") ["physics", "chemistry", "medicine"] in or after 1945.
- Save an $elemMatch filter shared to count laureates with shared (i.e., "share" is not "1") prizes in categories other than ["physics", "chemistry", "medicine"] in or after 1945.

```
# Save a filter for laureates with unshared prizes
unshared = {
    "prizes": {"$elemMatch": {
        "category": {"$nin": ["physics", "chemistry", "medicine"]},
        "share": "1",
        "year": {"$gte": "1945"},
    }}}

# Save a filter for laureates with shared prizes
shared = {
    "prizes": {"$elemMatch": {
        "category": {"$nin": ["physics", "chemistry", "medicine"]},
        "share": {"$ne": "1"},
        "year": {"$gte": "1945"},
    }}}

ratio = db.laureates.count_documents(unshared) / db.laureates.count_documents(shared)
print(ratio)

```
_________________________

- You won't need the $elemMatch operator at all for this exercise.
- Save a filter before to count organization laureates with prizes won before 1945. Recall that organization status is encoded with the "gender" field, and that dot notation is needed to access a laureate's "year" field within its "prizes" array.
- Save a filter in_or_after to count organization laureates with prizes won in or after 1945.

```
# Save a filter for organization laureates with prizes won before 1945
before = {
    'gender':'org',
    'prizes.year': {'$lt': "1945"},
    }

# Save a filter for organization laureates with prizes won in or after 1945
in_or_after = {
    'gender':'org',
    'prizes.year': {'$gte': "1945"},
    }

n_before = db.laureates.count_documents(before)
n_in_or_after = db.laureates.count_documents(in_or_after)
ratio = n_in_or_after / (n_in_or_after + n_before)
print(ratio)

```
_________________________

- Use a regular expression object to filter for laureates with "Germany" in their "bornCountry" value.
- Use a regular expression object to filter for laureates with a "bornCountry" value starting with "Germany".
- Use a regular expression object to filter for laureates born in what was at the time Germany but is now another country.
- Use a regular expression object to filter for laureates born in what is now Germany but at the time was another country.

```
from bson.regex import Regex

# Filter for laureates with "Germany" in their "bornCountry" value
criteria = {"bornCountry": Regex("Germany")}
print(set(db.laureates.distinct("bornCountry", criteria)))

# Filter for laureates with a "bornCountry" value starting with "Germany"
criteria = {"bornCountry": Regex('^Germany')}
print(set(db.laureates.distinct("bornCountry", criteria)))

# Fill in a string value to be sandwiched between the strings "^Germany " and "now"
criteria = {"bornCountry": Regex("^Germany " + '\(' + "now")}
print(set(db.laureates.distinct("bornCountry", criteria)))

#Filter for currently-Germany countries of birth. Fill in a string value to be sandwiched between the strings "now" and "$"
criteria = {"bornCountry": Regex("now" + ' Germany\)' + "$")}
print(set(db.laureates.distinct("bornCountry", criteria)))

```
_________________________

- Save a filter criteria that finds laureates with prizes.motivation values containing "transistor" as a substring. The substring can appear anywhere within the value, so no anchoring characters are needed.
- Save to first and last the field names corresponding to a laureate's first name and last name (i.e. "surname") so that we can print out the names of these laureates.

```
from bson.regex import Regex

# Save a filter for laureates with prize motivation values containing "transistor" as a substring
criteria = {'prizes.motivation': Regex('transistor')}

# Save the field names corresponding to a laureate's first name and last name
first, last = 'firstname', 'surname'
print([(laureate[first], laureate[last]) for laureate in db.laureates.find(criteria)])

```
_________________________

- First, use regular expressions to fetch the documents for the laureates whose "firstname" starts with "G" and whose "surname" starts with "S".
- Use projection and adjust the query to select only the "firstname" and "surname" fields.
- Iterate over the documents, and for each document, concatenate the first name and the surname fields together with a space in between to obtain full names.

```
# Use projection to select only firstname and surname
docs = db.laureates.find(
       filter= {"firstname" : {"$regex" : "^G"},
                "surname" : {"$regex" : "^S"}  },
   projection= ["firstname", "surname"]  )

# Iterate over docs and concatenate first name and surname
#full_names = [doc["firstname"] + " " + doc["surname"]  for doc in docs]
full_names=[]
for doc in docs:
   x=doc["firstname"]+' '+doc['surname']
   full_names.append(x)
# Print the full names
print(full_names)

```
_________________________

- Save a list of prizes (prizes), projecting out only the "laureates.share" values for each prize.
- For each prize, compute the total share as follows:
  - Initialize the variable total_share to 0.
  - Iterate over the laureates for each prize, converting the "share" field of the "laureate" to float and adding the reciprocal of it (that is, 1 divided by it) to total_share.

```

# Save documents, projecting out laureates share
prizes = db.prizes.find({}, ['laureates.share'])

# Iterate over prizes
for prize in prizes:
    # Initialize total share
    total_share = 0
    
    # Iterate over laureates for the prize
    for laureate in prize["laureates"]:
        # convert the laureate's share to float and add the reciprocal to total_share
        total_share += 1 / float(laureate["share"])
        
    # Print the total share    
    print(total_share)    

```
_________________________

- Complete the definition of all_laureates(prize). Within the body of the function:
  - Sort the "laureates" list of the prize document according to the "surname" key.
  - For each of the laureates in the sorted list, extract the "surname" field.
- Find the documents for the prizes in the physics category, sort them in chronological order (by "year", ascending), and only fetch the "year", "laureates.firstname", and "laureates.surname" fields.
- Print the year and the names of the laureates (use your all_laureates() function) for each prize document.
  
  ```
 from operator import itemgetter

def all_laureates(prize):  
  # sort the laureates by surname
  sorted_laureates = sorted(prize["laureates"], key=itemgetter("surname"))
  #print(sorted_laureates)
  # extract surnames
  surnames = [laureate["surname"] for laureate in sorted_laureates]
  
  # concatenate surnames separated with " and " 
  all_names = " and ".join(surnames)
  
  return all_names

# find physics prizes, project year and name, and sort by year
docs = db.prizes.find(
           filter= {"category": "physics"}, 
           projection= ["year", "laureates.firstname", "laureates.surname"], 
           sort= [("year", 1)])

# print the year and laureate names (from all_laureates)
for doc in docs:
  print("{year}: {names}".format(year=doc['year'], names=all_laureates(doc)))
  
  ```
_________________________
  
- Find the original prize categories established in 1901 by looking at the distinct values of the "category" field for prizes from year 1901.
- Fetch ONLY the year and category from all the documents (without the "_id" field).
- Sort by "year" in descending order, then by "category" in ascending order.

```
# original categories from 1901
original_categories = db.prizes.distinct('category', {'year':'1901'})
print(original_categories)

# project year and category, and sort
docs = db.prizes.find(
        filter={},
        projection ={'_id':0,'category':1,'year':1},
        sort=[('year',-1),('category',1)]
)

#print the documents
for doc in docs:
  print(doc)

```
_________________________

- Specify an index model that indexes first on category (ascending) and second on year (descending).
- Save a string report for printing the last single-laureate year for each distinct category, one category per line. To do this, for each distinct prize category, find the latest-year prize (requiring a descending sort by year) of that category (so, find matches for that category) with a laureate share of "1".

# Specify an index model for compound sorting
index_model = [('category', 1), ('year', -1)]
db.prizes.create_index(index_model)

# Collect the last single-laureate year for each category
report = ""
for category in sorted(db.prizes.distinct("category")):
    doc = db.prizes.find_one(
        {'category':category, "laureates.share": "1"},
        sort=[('year', -1)]
    )
    report += "{category}: {year}\n".format(**doc)

print(report)

```
_________________________

- Create an index on country of birth ("bornCountry") for db.laureates to ensure efficient gathering of distinct values and counting of documents
- Complete the skeleton dictionary comprehension to construct n_born_and_affiliated, the count of laureates as described above for each distinct country of birth. For each call to count_documents, ensure that you use the value of country to filter documents properly.

```
from collections import Counter

# Ensure an index on country of birth
db.laureates.create_index([('bornCountry', 1)])

# Collect a count of laureates for each country of birth
n_born_and_affiliated = {
    country: db.laureates.count_documents({
        'bornCountry': country,
        "prizes.affiliations.country": country
    })
    for country in db.laureates.distinct("bornCountry")
}
#print(n_born_and_affiliated)
five_most_common = Counter(n_born_and_affiliated).most_common(5)
print(five_most_common)

```
_________________________

- Save to filter_ the filter document to fetch only prizes with one or more quarter-share laureates, i.e. with a "laureates.share" of "4".
- Save to projection the list of field names so that prize category, year and laureates' motivations ("laureates.motivation") may be fetched for inspection.
- Save to cursor a cursor that will yield prizes, sorted by ascending year. Limit this to five prizes, and sort using the most concise specification.

```
from pprint import pprint

# Fetch prizes with quarter-share laureate(s)
filter_ = {'laureates.share': '4'}

# Save the list of field names
projection = ['category', 'laureates.motivation', 'year']

# Save a cursor to yield the first five prizes
cursor = db.prizes.find(filter_, projection).sort("year").limit(5)
pprint(list(cursor))

```
_________________________

- Complete the function get_particle_laureates that, given page_number and page_size, retrieves a given page of prize data on laureates who have the word "particle" (use $regex) in their prize motivations ("prizes.motivation"). Sort laureates first by ascending "prizes.year" and next by ascending "surname".
- Collect and save the first nine pages of laureate data to pages

```
from pprint import pprint

# Write a function to retrieve a page of data
def get_particle_laureates(page_number=1, page_size=3):
    if page_number < 1 or not isinstance(page_number, int):
        raise ValueError("Pages are natural numbers (starting from 1).")
    particle_laureates = list(
        db.laureates.find(
            {"prizes.motivation": {'$regex': "particle"}},
            ["firstname", "surname", "prizes"])
        .sort([('prizes.year', 1), ('surname', 1)])
        .skip(page_size * (page_number - 1))
        .limit(page_size))
    return particle_laureates

# Collect and save the first nine pages
pages = [get_particle_laureates(page_number=page) for page in range(1,9)]
pprint(pages[0])

```
_________________________

- Translate the cursor `cursor = (db.laureates.find({"gender": {"$ne": "org"}},["bornCountry", "prizes.affiliations.country"]).limit(3))` to an equivalent aggregation cursor, saving the pipeline stages to pipeline. Recall that the find collection method's "filter" parameter maps to the "$match" aggregation stage, its "projection" parameter maps to the "$project" stage, and the "limit" parameter (or cursor method) maps to the "$limit" stage.

```
# Translate cursor to aggregation pipeline
pipeline = [
    {"$match": {'gender': {"$ne": "org"}}},
    {'$project': {"bornCountry": 1, "prizes.affiliations.country": 1}},
    {"$limit":3}
]

for doc in db.laureates.aggregate(pipeline):
    print("{bornCountry}: {prizes}".format(**doc))

```
_________________________

- Save to pipeline an aggregation pipeline to collect prize documents as detailed above. Use Python's collections.OrderedDict to specify any sorting.
```
from collections import OrderedDict
from itertools import groupby
from operator import itemgetter

original_categories = set(db.prizes.distinct("category", {"year": "1901"})) 
#['physics', 'chemistry', 'medicine', 'literature', 'peace']


# Save an pipeline to collect original-category prizes
pipeline = [
    {'$match': {'category': {'$in': list(original_categories)}}},
    {'$project': {'category': 1, 'year': 1}},
    {'$sort': OrderedDict([('year',-1)])}
]
cursor = db.prizes.aggregate(pipeline)
for key, group in groupby(cursor, key=itemgetter("year")):
    missing = original_categories - {doc["category"] for doc in group}
    if missing:
        print("{year}: {missing}".format(year=key, missing=", ".join(sorted(missing))))
        
        
        db.prizes.distinct("category", {"year": "1901"})

```
_________________________

- Fill out pipeline to determine the number of prizes awarded (at least partly) to organizations.
```
# Count prizes awarded (at least partly) to organizations as a sum over sizes of "prizes" arrays.
#In the slides at the beginning of this lesson, we saw a two-stage aggregation pipeline to determine the number of prizes awarded in total. How many prizes were awarded (at least partly) to organizations?

pipeline = [
    {'$match': {'gender': "org"}},
    {"$project": {"n_prizes": {"$size": '$prizes'}}},
    {"$group": {"_id": None, "n_prizes_total": {"$sum": "n_prizes"}}}
]

print(list(db.laureates.aggregate(pipeline)))

```
_________________________
> What proportion of laureates won a prize while affiliated with an institution in their country of birth? Build an aggregation pipeline to get the count of laureates who either did or did not win a prize with an affiliation country that is a substring of their country of birth -- for example, the prize affiliation country "Germany" should match the country of birth "Prussia (now Germany)".

- Use $unwind stages to ensure a single prize affiliation country per pipeline document.
- Filter out prize-affiliation-country values that are "empty" (null, not present, etc.) -- ensure values are "$in" the list of known values.
- Produce a count of documents for each value of "affilCountrySameAsBorn" (a field we've projected for you using the $indexOfBytes operator) by adding 1 to the running sum.

```
key_ac = "prizes.affiliations.country"
key_bc = "bornCountry"
pipeline = [
    {"$project": {key_bc: 1, key_ac: 1}},

    # Ensure a single prize affiliation country per pipeline document
    {"$unwind": "$prizes"},
    {"$unwind": "$prizes.affiliations"},

    # Ensure values in the list of distinct values (so not empty)
    {"$match": {key_ac: {"$in": db.laureates.distinct(key_ac)}}},
    {"$project": {"affilCountrySameAsBorn": {
        "$gte": [{"$indexOfBytes": ["$"+key_ac, "$"+key_bc]}, 0]}}},

    # Count by "$affilCountrySameAsBorn" value (True or False)
    {"$group": {"_id": "$affilCountrySameAsBorn",
                "count": {"$sum": 1}}},
]
for doc in db.laureates.aggregate(pipeline): print(doc)


#{'count': 477, '_id': True}
#{'count': 261, '_id': False}

```
_________________________
> Countries of birth by prize categorySome prize categories have laureates hailing from a greater number of countries than do other categories. You will build an aggregation pipeline for the prizes collection to collect these numbers, using a $lookup stage to obtain laureate countries of birth.

- $unwind the laureates array field to output one pipeline document for each array element.
- After pulling in laureate bios with a $lookup stage, unwind the new laureate_bios array field (each laureate has only a single biography document).
- Collect the set of bornCountries associated with each prize category.
- Project out the size of each category's set of bornCountries.

```
pipeline = [
    # Unwind the laureates array
    {"$unwind": "$laureates"},
    {"$lookup": {
        "from": "laureates", "foreignField": "id",
        "localField": "laureates.id", "as": "laureate_bios"}},

    # Unwind the new laureate_bios array
    {"$unwind": "$laureate_bios"},
    {"$project": {"category": 1,
                  "bornCountry": "$laureate_bios.bornCountry"}},

    # Collect bornCountry values associated with each prize category
    {"$group": {'_id': "$category",
                "bornCountries": {"$addToSet": "$bornCountry"}}},

    # Project out the size of each category's (set of) bornCountries
    {"$project": {"category": 1,
                  "nBornCountries": {"$size": "$bornCountries"}}},
    {"$sort": {"nBornCountries": -1}},
]
for doc in db.prizes.aggregate(pipeline): print(doc)

```
_________________________

- In your aggregation pipeline pipeline, use the "gender" field to limit results to people (that is, not organizations).
- Count prizes for which the laureate's "bornCountry" is not also the "country" of any of their affiliations for the prize. Be sure to use field paths (precede a field name with "$") when appropriate.

```
pipeline = [
    # Limit results to people; project needed fields; unwind prizes
    {"$match": {"gender": {"$ne": "org"}}},
    {"$project": {"bornCountry": 1, "prizes.affiliations.country": 1}},
    {"$unwind": "$prizes"},
  
    # Count prizes with no country-of-birth affiliation
    {"$addFields": {"bornCountryInAffiliations": {"$in": ["$bornCountry", "$prizes.affiliations.country"]}}},
    {"$match": {"bornCountryInAffiliations": False}},
    {"$count": "awardedElsewhere"},
]

print(list(db.laureates.aggregate(pipeline)))

```
_________________________

- Construct a stage added_stage that filters for laureate "prizes.affiliations.country" values that are non-empty, that is, are $in a list of the distinct values that the field takes in the collection.
- Insert this stage into the pipeline so that it filters out single prizes (not arrays) and precedes any test for membership in an array of countries. - - Recall that the first parameter to <list>.insert is the (zero-based) index for insertion.

```
pipeline = [
    {"$match": {"gender": {"$ne": "org"}}},
    {"$project": {"bornCountry": 1, "prizes.affiliations.country": 1}},
    {"$unwind": "$prizes"},
    {"$addFields": {"bornCountryInAffiliations": {"$in": ["$bornCountry", "$prizes.affiliations.country"]}}},
    {"$match": {"bornCountryInAffiliations": False}},
    {"$count": "awardedElsewhere"},
]

# Construct the additional filter stage
added_stage = {"$match": {"prizes.affiliations.country": {"$in": db.laureates.distinct("prizes.affiliations.country")}}}

# Insert this stage into the pipeline
pipeline.insert(3, added_stage)
print(list(db.laureates.aggregate(pipeline)))

```
_________________________

