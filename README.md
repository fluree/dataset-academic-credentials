## The Academic Credentials Dataset (the ACD)
This is a demo dataset that describes a fictional ecosystem of Academic Credentials (degrees, certificates, etc.) and the related Institutions, Learners, and Employers. 
This dataset uses JSON-LD and a Context Map to represent the data as a semantic graph. This just means that some data objects will point to other data objects and some of the object properties and values are namespaced with IRIs so that they can be understood in any context.
___
### Load the ACD into Fluree
1. First, you'll need a running instance of Fluree. If you need help with this, the fastest path is [Fluree's Introduction documentation](https://next.developers.flur.ee/docs/learn/tutorial/introduction/#running-fluree)
2. Now that we have a Fluree instance, let's create a new ledger for our dataset. We'll send the transaction to the `/fluree/create` endpoint with the following curl command. You should receive a `201 Created` in response.
```sh
  curl -H "Content-Type:application/json" --data "@resources/00_create_ledger.jsonld" localhost:58090/fluree/create
```
3. Last thing to do is transact our dataset into our fresh ledger. We'll send this POST to the `/fluree/transact` endpoint.
```sh
  curl -H "Content-Type:application/json" --data "@resources/01_dataset.jsonld" localhost:58090/fluree/transact
```

### Ready to roll!
Here are some queries you can send to confirm you haven't missed anything. We need to send queries to the `/fluree/query` endpoint.
1. Check the summary statistics
```sh
  curl -H "Content-Type:application/json" --data "@resources/queries/summary_stats.json" localhost:58090/fluree/query
``` 
expected result (3 Learners, 3 Institutions, 2 Employers, 1 Achievement, 4 Courses, and 0 Degrees):
```json
[[3,3,2,1,4,0]]
```
2. Find Freddy
```sh
  curl -H "Content-Type:application/json" --data "@resources/queries/find_freddy.json" localhost:58090/fluree/query
```
expected result:
```json
[
  {
    "id": "acd:learners/freddyyeti1",
    "http://www.w3.org/1999/02/22-rdf-syntax-ns#type": [
      "clr:Profile"
    ],
    "acd:profileType": "Learner",
    "clr:dateOfBirth": "2016-11-24",
    "schema:email": "freddyyeti1@asu.edu",
    "schema:familyName": "Yeti",
    "schema:givenName": "Freddy"
  }
]
```
3. View all of Freddy's credentials
```sh
  curl -H "Content-Type:application/json" --data "@resources/queries/freds_creds.json" localhost:58090/fluree/query
```
expected result:
```json
[
  {
    "id": "acd:credentials/1",
    "http://www.w3.org/1999/02/22-rdf-syntax-ns#type": [
      "clr:AchievementSubject"
    ],
    "acd:learner": {
      "id": "acd:learners/freddyyeti1"
    },
    "clr:achievement": {
      "id": "acd:achievements/MAT117"
    },
    "clr:source": {
      "id": "acd:institutions/wssu"
    }
  }
]
```
Here's a nice graph of these results:
```mermaid
graph LR
    mm(1) -->|is named| Mothman
    mm -->|lives in| wv
    wv(2) -->|is named| wvn(West Virginia)
    mm -->|first sighted on| d(November 12, 1966)
```
---
### What's in the box?
The /resources folder contains JSON files, each of which contains the data that will be sent as transactions to the Fluree ledger you'd like to populate.
The dataset defined in these files uses the [Comprehensive Leaner Network](https://www.imsglobal.org/spec/clr/v2p0) Standard for data schema and vocabulary.

#### Concepts
The CLR defines several terms it's concerned with in [its specification](https://www.imsglobal.org/spec/clr/v2p0#terminology) but we'll zoom in on just a handful of subject types and define them more specifically for our purposes:
- __Institution__: Any educational organization (e.g. - school, college, university, online training, apprentice programs) that teaches skills and concepts to learners. May also be referred to as an "Issuer" because, along with educating learners, Institutions also issue academic credentials to Learners that have earned them.
- __Learner__: An individual that is recognized by one or more Institutions and has earned zero or more academic achievements.
- __Employer__: A business entity that is interested in hiring Learners. May also be referred to as a "Verifier" because Employers usually want to verify the validity of a Learner's academic credentials during the hiring process.

And here are a few additional important concepts featured in this dataset:
- __Course__: A description of an educational course which may be offered as distinct instances which take place at different times or take place at different locations, or be offered through different media or modes of study. An educational course is a sequence of one or more educational events and/or creative works which aims to build knowledge, competence or ability of learners. [1]
- __Achievement__: A specific academic achievement that is recognized by an Institution. Some examples of an achievement might be a Bachelor's Degree, an apprenticeship, and an online certification.
- __Result__: The outcome of an academic achievement for a particular Learner. This could be a letter grade, a percentage score, a Pass or Fail, etc. A Result may also capture other metadata about a particular Learner's achievement such as dates of enrollment, instructor, commendations, etc.
___

