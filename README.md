## The Academic Credentials Dataset (the ACD)

This is a demo dataset that describes a fictional ecosystem of Academic Credentials (degrees, certificates, etc.) and the related Institutions, Learners, and Employers.

This dataset uses JSON-LD and a Context Map to represent the data as a semantic graph. This just means that some data objects will point to other data objects and some of the object properties and values are namespaced with IRIs so that they can be understood in any context.

```mermaid
graph LR
  as(Assertion) -->|recipient| r(Learner)
  as -->|achievement| ac(Achievement)
  ac -->|creator| inst(Institution)
  c(Credential) -->|credentialSubject| as
  c -->|issuer| inst
```

In general, Fluree sends JSON responses. You can pretty-print these with the [jq](https://www.baeldung.com/linux/jq-command-json) command-line utility by appending `| jq` to the commands below.

___

### Load the ACD into Fluree
#### 1. Run Fluree
First, we'll need [a running instance of Fluree](https://next.developers.flur.ee/docs/learn/tutorial/introduction/#running-fluree).

#### 2. Clone ACD Repo
We'll also want to clone this repo and change into its directory.
```sh
  git clone https://github.com/fluree/dataset-academic-credentials.git && cd ./dataset-academic-credentials
```

#### 3. Create Ledger
Now that we have a Fluree instance and the dataset, let's create a new ledger for our dataset. We'll send the transaction in [resources/00_create_ledger.jsonld](resources/00_create_ledger.jsonld) to the `/fluree/create` endpoint with the following curl command. You should receive a `201 Created` in response.

```sh
curl -H "Content-Type:application/json" --data "@resources/00_create_ledger.jsonld" localhost:58090/fluree/create
```

#### 4. Transact Data
Last thing to do is transact our dataset into our fresh ledger. 
We'll send the other jsonld files one at a time in a POST to the `/fluree/transact` endpoint.

[resources/01_profiles.jsonld](resources/01_profiles.jsonld)
```sh
curl -H "Content-Type:application/json" --data "@resources/01_profiles.jsonld" localhost:58090/fluree/transact
```

[resources/02_achievements.jsonld](resources/02_achievements.jsonld)
```sh
curl -H "Content-Type:application/json" --data "@resources/02_achievements.jsonld" localhost:58090/fluree/transact
```

[resources/03_credentials.jsonld](resources/03_credentials.jsonld)
```sh
curl -H "Content-Type:application/json" --data "@resources/03_credentials.jsonld" localhost:58090/fluree/transact
```


### Ready to roll!

Here are some queries you can send to confirm you haven't missed anything. We need to send queries to the `/fluree/query` endpoint.

1. Check the summary statistics with [resources/queries/summary_stats.json](resources/queries/summary_stats.json)

```sh
curl -H "Content-Type:application/json" --data "@resources/queries/summary_stats.json" localhost:58090/fluree/query
``` 

  expected result (3 Learners, 10 Institutions, 2 Employers, 51 Achievements, 43 Courses, 1 Degree, 12 Licenses, and 13 Awards):


```json
[[3,10,2,51,43,1,12,13]]
```

1. Find Freddy with [resources/queries/find_freddy.json](resources/queries/find_freddy.json)

```sh
curl -H "Content-Type:application/json" --data "@resources/queries/find_freddy.json" localhost:58090/fluree/query
```

expected result:

```json
[
  {
    "id": "acd:learners/freddyyeti1",
    "type": "clr:Profile",
    "schema:email": "freddyyeti1@fluree.edu",
    "schema:familyName": "Yeti",
    "schema:givenName": "Freddy",
    "acd:profileType": "Learner",
    "clr:dateOfBirth": "2016-11-24"
  }
]
```

3. View all of Freddy's credentials with [resources/queries/freds_creds.json](resources/queries/freds_creds.json)

```sh
curl -H "Content-Type:application/json" --data "@resources/queries/freds_creds.json" localhost:58090/fluree/query
```

expected result:

```json
[
  {
    "id": "acd:credentials/1",
    "type": [
      "acd:Credential",
      "clr:AchievementCredential",
      "vc:VerifiableCredential"
    ],
    "clr:name": "Recognition of Course Completion",
    "clr:issuanceDate": "2013-08-10",
    "clr:issuer": {
      "id": "acd:institutions/fu"
    },
    "clr:credentialSubject": {
      "id": "acd:assertions/1"
    }
  }
]
```

---
### What's in the box?

The /resources folder contains JSON files, each of which contains the data that will be sent as transactions to the Fluree ledger you'd like to populate.

The dataset defined in these files uses the [Comprehensive Learner Network](https://www.imsglobal.org/spec/clr/v2p0) Standard for data schema and vocabulary.

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

#### Use Cases

This dataset supports the following use cases:

1. A Learner that has earned academic achievements from multiple Institutions would like to aggregate them all and export to a standard format in order to apply for a job.
2. An Institution would like to analyze the demographics and GPAs of Learners that have achieved a Mathematics degree since it's inception so that a baseline trend can be established for comparing outcomes of future accessibility and inclusivity efforts.
3. An Employer would like to request the Institutions that issued a specific set of academic credentials to prove the credentials are valid and belong to the Learner that is applying for a job with the Employer.
