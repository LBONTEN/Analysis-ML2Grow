# Blueprint POC AI

## Table of Content

- [Blueprint POC AI](#blueprint-poc-ai)
  - [Table of Content](#table-of-content)
  - [Introduction](#introduction)
    - [SparQL Prefixes](#sparql-prefixes)
  - [Models](#models)
    - [NER - Named Entity Recognition](#ner---named-entity-recognition)
    - [BERTopic](#bertopic)
    - [Embed](#embed)
    - [Zeroshot](#zeroshot)
  - [Apache Airflow architecture](#apache-airflow-architecture)
  - [DAGs - Directed Acyclic Graphs](#dags---directed-acyclic-graphs)
    - [NER](#ner)
    - [BERTopic Retrain](#bertopic-retrain)
    - [BERTopic Transform](#bertopic-transform)
    - [Embed](#embed-1)
    - [Zeroshot](#zeroshot-1)
  - [Data Flow Diagrams](#data-flow-diagrams)
  - [Triplestore Mapping](#triplestore-mapping)
  - [Conclusion](#conclusion)

## Introduction

This project lays out the architecture of the ML2Grow AI project. This project was instantiated in order to leverage the power of AI in current and future LBLOD projects.
This blueprint was meant to lay bare this architecture to make potential improvements apparent.
[...]

### SparQL Prefixes

Throughout this document we will make use of prefixes in order to abbreviate some links that will occur, to make this more readable.  
These prefixes are explained here:

Prefix | Full link
---|---
ext | <http://mu.semte.ch/vocabularies/ext/>  
rdf | <http://www.w3.org/1999/02/22-rdf-syntax-ns#>  
info | <http://data.lblod.info/>

## Models

The project uses multiple models throughout the project to achieve various functionalities. These models and their functionalities will be listed out here.
[...]

### NER - Named Entity Recognition

[NER](https://pypi.org/project/flair/) is used to recognize various entities within sentences. For example, using the sentence "I love Berlin.", the model will recognize that Berlin is most likely a location. This can be useful for tagging entities within sentences. So these can be used for linking to related topics and such.
[...]

### BERTopic

[BERTopic](https://pypi.org/project/bertopic/) is used for identifying topics within a certain article. For example, [...].

### Embed

This models links an embedding vector to a document based on its content. This vector can later be used to perform elastic search.

### Zeroshot

This model will be used to perform classifications on BBC Topics. These BBC topics are a way of classifying documents based on the content and matter addressed in the document.

## Apache Airflow architecture

[Apache Airflow](https://airflow.apache.org/docs/) is a framework used to deploy Big Data Networks that can be used amongst multiple projects. There are certain containers that contain various scripts that can be run from CLI. These scripts perform various important tasks such as saving and loading models or data.
[...]

## DAGs - Directed Acyclic Graphs

[DAGs](https://airflow.apache.org/docs/apache-airflow/stable/concepts/dags.html) [...]  
DAGS configure the containers that run the various tasks of the project.

### NER

![ner tasks](dag-tasks/ner.dot.svg)

[Link to the repository containing the scripts](https://github.com/lblod/poc-ai-airflow-ner)
During these tasks. We first load the NER model. Then we perform NER related stuff.

1. [Load](https://github.com/lblod/poc-ai-airflow-ner/blob/master/scripts/load.py)  
    During this script we load the data from the triplestore and export it to a json file. This file can then later be used to perform transformations. This script loads following query

    ```sparql
        PREFIX prov: <http://www.w3.org/ns/prov#>
        PREFIX dct: <http://purl.org/dc/terms/>
        PREFIX soic: <http://rdfs.org/sioc/ns#>
        PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>


        SELECT DISTINCT ?thing ?text WHERE {
            ?thing a <http://rdf.myexperiment.org/ontologies/base/Submission>; 
            prov:generated/dct:hasPart ?part.    
            ?part soic:content ?text.  
            FILTER NOT EXISTS {  ?thing ext:ingestedml2GrowSmartRegulationsNer "1" }
        }
    ```

2. [NER](https://github.com/lblod/poc-ai-airflow-ner/blob/master/scripts/ner.py)  
    Afterwards we use the data to run the NER model on. The data gets processed and the results are written to a json file on disk.

3. [Save](https://github.com/lblod/poc-ai-airflow-ner/blob/master/scripts/save.py)  
    Finally the results are persisted in the triplestore. The results are saved in the triplestore like this.

    _Example_  
    ![Example](./triples/NER.dot.svg)

    Predicate | Description
    ---|---
    rdf:type |  A constant value being ext:Ner. Representing the type of the subject.
    ext:start   | The start position of the word. Relative to [...].
    ext:end | The end position of the word. Relative to [...].
    ext:word    | The word that was guessed on by the AI model.
    ext:entity  | Can be either "Location", "Person" or "Organization". This is the value guessed by the AI model.

    Additionally we also add a predicate to the file that was used to generate the NER with.

    Predicate | Description
    ---|---
    ext:hasNer | A link to the NER associated with the document. A document can relate to many NERs.
    ext:ingestedml2GrowSmartRegulationsNer | Tag indicating it has already been ingested for future runs of the model.

### BERTopic Retrain

![BERTopic Retrain](dag-tasks/bertopic_retrain.dot.svg)  

[Link to the repository containing the scripts](https://github.com/lblod/poc-ai-airflow-bertopic)  

1. [Load](https://github.com/lblod/poc-ai-airflow-bertopic/blob/master/scripts/load.py)
    Script that loads the following query.

    ```sparql
        PREFIX prov: <http://www.w3.org/ns/prov#> 
        PREFIX dct: <http://purl.org/dc/terms/>  
        PREFIX soic: <http://rdfs.org/sioc/ns#>  
        PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>  
        
        SELECT DISTINCT ?thing ?part ?text WHERE {
            ?thing a <http://rdf.myexperiment.org/ontologies/base/Submission>; 
            prov:generated/dct:hasPart ?part.    
            ?part soic:content ?text.  
        }
    ```

2. [Retrain & Save](https://github.com/lblod/poc-ai-airflow-bertopic/blob/master/scripts/retrain.py)  
    [...]  

3. Restart  
    [...]  

4. [Transform](https://github.com/lblod/poc-ai-airflow-bertopic/blob/master/scripts/transform.py)  
    [...]

5. Save
   1. [Transform](https://github.com/lblod/poc-ai-airflow-bertopic/blob/master/scripts/save_transform.py)

        _Example_  
        ![Example](./triples/BERTopic-transforms.dot.svg)

        Predicate | Description
        ---|---
        ext:HasTopic | URI to the linked topic.
        ext:ingestedByMl2GrowSmartRegulationsTopics | Tag indicating it has been ingested by a ML2Grow model.

        Predicate | Description
        ---|---
        rdf:type | TopicScore
        ext:TopicURI | URI to the linked topic
        ext:score | score of the linked topic

   2. [Topics](https://github.com/lblod/poc-ai-airflow-bertopic/blob/master/scripts/save_topics.py)

        _Example_  
        ![Example](./triples/BERTopic-topics.dot.svg)

        Predicate | Description
        ---|---
        rdf:type | isTopic
        ext:relevant_words | The relevant words found by the model. A Topic can have multiple relevant words.
        ext:count | The count of [...]
        ext:topic_label | The label of the topic

### BERTopic Transform

![BERTopic Transform](dag-tasks/bertopic_transform.dot.svg)

[Link to the repository containing these scripts](https://github.com/lblod/poc-ai-airflow-bertopic)

1. [Load](https://github.com/lblod/poc-ai-airflow-bertopic/blob/master/scripts/load.py)  
    Initiates the task by loading data from the triplestore.

    ```sparql
        PREFIX prov: <http://www.w3.org/ns/prov#> 
        PREFIX dct: <http://purl.org/dc/terms/>  
        PREFIX soic: <http://rdfs.org/sioc/ns#>  
        PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>  
        
        SELECT DISTINCT ?thing ?part ?text WHERE {
            ?thing a <http://rdf.myexperiment.org/ontologies/base/Submission>; 
            prov:generated/dct:hasPart ?part.    
            ?part soic:content ?text.  
        }
    ```

2. [Transform](https://github.com/lblod/poc-ai-airflow-bertopic/blob/master/scripts/transform.py)  
    [...]  

3. [Save](https://github.com/lblod/poc-ai-airflow-bertopic/blob/master/scripts/save_transform.py)
    This script saves the same data as the number 3.1 of [the previous section](#bertopic-retrain).
`1
### Embed

![embed tasks](dag-tasks/embed.dot.svg)

[Link to the repository containing the scripts](https://github.com/lblod/poc-ai-airflow-embed)

1. [Load](https://github.com/lblod/poc-ai-airflow-embed/blob/master/scripts/load.py)
  
    ```sparql
        PREFIX prov: <http://www.w3.org/ns/prov#>
        PREFIX dct: <http://purl.org/dc/terms/>
        PREFIX soic: <http://rdfs.org/sioc/ns#>
        PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>


        SELECT DISTINCT ?thing ?text WHERE {
            ?thing a <http://rdf.myexperiment.org/ontologies/base/Submission>;
            prov:generated/dct:hasPart ?part.    
            ?part soic:content ?text.  
            FILTER NOT EXISTS {  ?thing ext:ingestedByMl2GrowSmartRegulationsEmbedding "1" }

        }
    ```

2. [Embed](https://github.com/lblod/poc-ai-airflow-embed/blob/master/scripts/embed.py)  
    [...]

3. [Save](https://github.com/lblod/poc-ai-airflow-embed/blob/master/scripts/save.py)

    _Example_  
    ![Example](./triples/Embed.dot.svg)

    Predicate | Description
    ---|---
    ext:searchEmbedding | Embedding Vector linked to the file.
    ext:ingestedByMl2GrowSmartRegulationsEmbedding | Tag indicating it has been ingested by a ML2Grow model.

### Zeroshot

![zeroshot tasks](dag-tasks/zeroshot.dot.svg)

[Link to the repository containing the scripts](https://github.com/lblod/poc-ai-airflow-zeroshot)

1. Load  
   1. [Zeroshot](https://github.com/lblod/poc-ai-airflow-zeroshot/blob/master/scripts/load.py)

        ```sparql
            PREFIX prov: <http://www.w3.org/ns/prov#>
            PREFIX dct: <http://purl.org/dc/terms/>
            PREFIX soic: <http://rdfs.org/sioc/ns#>
            PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>
            SELECT DISTINCT ?thing ?text WHERE {
                ?thing a <http://rdf.myexperiment.org/ontologies/base/Submission>; 
                prov:generated/dct:hasPart ?part.    
                ?part soic:content ?text.  
                FILTER NOT EXISTS {  ?thing ext:ingestedMl2GrowSmartRegulationsBBC "1" }
            }
        ```

   2. [Taxonomy](https://github.com/lblod/poc-ai-airflow-zeroshot/blob/master/scripts/load.py)

        ```sparql
            PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>

            SELECT ?nl WHERE {
                <http://data.lblod.info/ML2GrowClassification> ?o ?taxo.
                ?taxo ext:nl_taxonomy ?nl
            }
        ```

2. [ZS_BBC](https://github.com/lblod/poc-ai-airflow-zeroshot/blob/master/scripts/zeroshot.py)  
    [...]

3. [Save](https://github.com/lblod/poc-ai-airflow-zeroshot/blob/master/scripts/save.py)

    _Example_  
    ![Example](./triples/Zeroshot.dot.svg)

    Predicate | Description
    ---|---
    rdf:type | One of many values
    ext:score | Score of the models prediction

    _File_
    Predicate | Description
    ---|---
    ext:BBC_scoring | Link to the BBC Scoring instance. A document can have multiple of these instances.
    ext:ingestedMl2GrowSmartRegulationsBBC | Tag indicating it has been ingested by the zeroshot model.

## Data Flow Diagrams

The Data Flow in the project goes a little like this.
[...]

## Triplestore Mapping

SparQL triplestores are used for persisting most data. Though through Airflow there are also mentions of postgreSQL. The use of PostgreSQL can be an inconvenience for the purpose of linking this project to other LBLOD projects.
[...]

## Conclusion

[...]