# Analysis ML2Grow


## poc-ai-airflow-bertopic

### load (endpoint, query)
Executes the query and loads it into a dataframe

### retrain
Retrains the model on the topics

### save-topics
clears topics and then saves them

```sparql
    PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>
    
    DELETE {{
        GRAPH <http://mu.semte.ch/application> {{
        <{topic_base_uri}> ext:relevant_words ?words;  ext:count ?count; ext:topic_label ?label .
        
        {joined}
        }}
    }}
    WHERE {{
        GRAPH <http://mu.semte.ch/application> {{
        <{topic_base_uri}> ext:relevant_words ?words;  ext:count ?count; ext:topic_label ?label .
        
        {joined}
        }}
    }}
```

```sparql
        PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>

        INSERT {{
            GRAPH <http://mu.semte.ch/application> {{

                <{topic_base_uri}> a ext:isTopic; ext:relevant_words {", ".join(relevant_word_uri)} ; ext:count {v['count']} ; ext:topic_label \"{v['topic_label']}\" . 

                {joined}
            }}
        }}
```

### save-transform
Takes BERTopic output and saves it to a sparql endpoint


```sparql
    PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>

            DELETE{{
        GRAPH <http://mu.semte.ch/application> {{
    <{record['thing']}> ext:HasTopic ?topic_link ; ext:ingestedByMl2GrowSmartRegulationsTopics ?srt . 
    ?topic_link ext:TopicURI ?topic_uri ; ext:score ?topic_score .
        }}
        }}
                    WHERE {{
        GRAPH <http://mu.semte.ch/application> {{
        <{record['thing']}> ext:HasTopic ?topic_link ; ext:ingestedByMl2GrowSmartRegulationsTopics ?srt . 
        ?topic_link ext:TopicURI ?topic_uri ; ext:score ?topic_score .
        }}
    }}
```

```sparql
    PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>
    INSERT {{
    GRAPH <http://mu.semte.ch/application> {{
    <{record['thing']}> ext:HasTopic <{new_uri}> ; ext:ingestedByMl2GrowSmartRegulationsTopics "1" .

    <{new_uri}> a ext:TopicScore;
    ext:TopicURI <http://data.lblod.info/ML2GrowTopicModeling/topic/{record['topic']}>; 
    ext:score {record['probability']} .
            }}
    }}
```

### transform

## poc-ai-bertopic

initial loading of the model. Using Roberta as the embedding model.
```topic_model = BERTopic.load("/models/topic.model", embedding_model=model)```

Uses this model for data processing and fetching. See specifications for this model:
    - <https://pypi.org/project/bertopic/>


## poc-ai-embed


## poc-ai-ner
named entity recognition

- <https://pypi.org/project/flair/>

## poc-ai-text-generation


## poc-ai-airflow-dags

- <https://airflow.apache.org/docs/apache-airflow/stable/concepts/dags.html>

Directed Acyclic Graphs
used to configure and run stuffs

Sparql queries for loading data in config folder.

## poc-ai-airflow-embed

### scripts/save.py

```sparql
      PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>
      
       DELETE{{
       GRAPH <http://mu.semte.ch/application>{{
       <{file_reference}> ext:searchEmbedding ?embed; ext:ingestedByMl2GrowSmartRegulationsEmbedding ?sre . 
      }}
      }}
      WHERE{{
       <{file_reference}> ext:searchEmbedding ?embed; ext:ingestedByMl2GrowSmartRegulationsEmbedding ?sre .            
      }}
```

```sparql
    PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>
    
    INSERT {{
        GRAPH <http://mu.semte.ch/application> {{
            <{file_reference}> ext:searchEmbedding \"{list(record["embedding"])}\".
            <{file_reference}> ext:ingestedByMl2GrowSmartRegulationsEmbedding "1".
        }}
    }}
```

## poc-ai-airflow-ner
cli scripts for Named Entity Recognition

### scripts/save.py

```sparql
    PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>
    
    
    DELETE{{
     GRAPH <http://mu.semte.ch/application>{{
     <{file_name}> ext:hasNer ?b ; ext:ingestedml2GrowSmartRegulationsNer ?srn .
    ?b ext:entity ?ner_type; ext:end ?end; ext:start ?start; ext:word ?word .
    }}
    }}
    WHERE{{
     <{file_name}> ext:hasNer ?b ; ext:ingestedml2GrowSmartRegulationsNer ?srn .
    ?b ext:entity ?ner_type; ext:end ?end; ext:start ?start; ext:word ?word .
    
    }}
```

## poc-ai-airflow-zeroshot

### scripts/save.py

```sparql
      PREFIX ext: <http://mu.semte.ch/vocabularies/ext/>
  
      DELETE {{
      GRAPH <http://mu.semte.ch/application> {{
       <{file_name}> ext:BBC_scoring ?bbc .
       <{file_name}> ext:ingestedMl2GrowSmartRegulationsBBC ?sbr .
       ?bbc ext:score ?bbc_score .
          }}
      }}
      
      WHERE {{
       <{file_name}> ext:BBC_scoring ?bbc .
       <{file_name}> ext:ingestedMl2GrowSmartRegulationsBBC ?sbr .
       ?bbc ext:score ?bbc_score .
  }}
```
