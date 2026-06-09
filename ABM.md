# Agent-Based Modelling

## Introduction

Agents are autonomous systems which can perform operations. Sometimes, learning over time is required for a system to be an agent, but it is not always the case.

Agent-based models simulate phenomena through actions of autonomous agents.
These agents can be either persons, organizations/groups.

Agent-based models are useful when wanting to study how micro-scale rules generate macro-scale patterns.
It is the bottom-up approach to simulation.
Such models are usually used for prediction of behavior of complex phenomena.
Emergence is significant in the process, because macro-scale state changes are expected to emerge from micro-scale agent behaviors.
Rules defined are usually simple decision making rules or heuristics.

## Elements of ABM

Agent-based models usually consist of:
1. various agents specified at various scales of granularity
2. decision-making rules or heuristics
3. learning rules or adaptive processes
4. an interaction topology
5. an environment

## Comments
Idea of such models is quite old and agent-based models were designed in simple ways already in the 1940s, by John von Neumann.

For situations with large linearity, usually different models are used, because simulational approaches have their faults in being computationally intensive.
Though, ABM models are flexible, and allow checking hypothetical scenarios easily.

The models need to be designed carefully to be usable for prediction.

Classic heuristics can also be replaced with usage of LLM models.

## Direction for Usage with Existing Data

Despite being focused on simulation and prediction, such models can also be used to check hypotheses.
A way to compare different states can be made, and simulations can be ran under different conditions, and basing on which model is more similar to the genuine data, conclusions about the genuine data can be drawn.
That approach is similar to how predictive models are evaluated/validated, but here it would be used to make conclusions about available data.

Sources:
https://en.wikipedia.org/wiki/Agent-based_model

# Idea for Usage with CAC and ALMA

CAC (events)
```csv
person_id	event_place,event_place_parent,event_place_id,event_place_parent_id,date_start,date_end,event_metadata
2033185,Kolegium Większe {Uniwersytetu Krakowskiego},Uniwersytet Krakowski (Akademia Krakowska),7613.0,8309.0,1517-04-23,1534-11-27,{'position': 'kolegiat', 'event_type': 'trwani...'}
```

CAC, ALMA (names)
```csv
final_id,alma_born,alma_died,first_polish_pub,cac_id,alma_name
3,3,1600.0,1700.0,None,2014948.0,Abrek Andrzej (16..-1700)
```

ALMA (publications)
```csv
ALMA_id,autors_list,title	subtitle,publication_city,date_start,date_end,record_type,all_names,all_names_final_id,contributing_persons,contributing_organizations,genre,subjects_people,subjects_topics,subjects_places
208,991000470979705067,None,Kalendarz Polski Ruski i Astronomiczno-Gospod...,na rok ... [...] /,[a: Polska | d: Kraków.],1806-01-01,1832-12-31,Tekst,[Koch, Rudolf Bogumił (1771-post 1854), Ryszko...],(1362, 1283, 2604),[(Ryszkowski, Franciszek Ksawery (1746-1809), ...)],	[(Drukarnia Antoniego Gröbla Wdowy i Sukcesoró...)],[Kalendarze, Kalendarze],None,None,None
```

## Basic idea
Have agents be scholars, each with state which would be responsible for choosing topics of new publications.
As decision-making rules, either there would be randomness, probability-based heuristics dependent on domains of scholars who had contact with a particular scholar earlier (it would require recording when scholars first contacted other scholars, and when scholars first published in some domain, and how much scholars have published in these domains until certain years, what publications specific scholars have already published).
If a bottom-up construction using that data would be enough to obtain simulation results where topic clustering among researchers would be "similar" to genuine data, then it would be revealed that these conditions had significant influence on scholars.

## Obstacles
Most data entries from ALMA do not have topics set in `subjects_topics`.
Those entries which have subjects listed, almost all have distinct topics, and these topics have various levels of granularity.
```json
['Wielkanoc', 'Zjawiska paranormalne', 'Algebra', 'Junosza (herb)', 'Bogowie rzymscy', 'Twarz', 'Bezpieczeństwo i wojskowość', 'Maszyny i narzędzia rolnicze', 'Prawo', 'Język francuski',
...
]
```

A way to compare when clustering of topics among researchers at particular time is similar, would be required.

Starting a model from scratch would waste most data, it could not be build using it, and could be used only for comparison.

Data is sparse, for periods of a few months there would be only a few entries, making periods longer makes the dataset less granular.

Topics are likely not dependent only on scholars, but also on general historical prevalence at certain year.

## How to overcome these obstacles
Topic names would have to be re-generated from titles, which are almost always present, to some more uniform list of topics.
More directly, re-derive usable topics, with similar granularity, from titles of publications using LLM, and a list of earlier topics prepared earlier.
It can be done using Ollama and a relatively small model.
Another approach could be generating embeddings of titles, and clustering these embeddings, ang taking back titles for topics.

Comparison between historical and generated data would be done by comparing topic distribution for each time period.
