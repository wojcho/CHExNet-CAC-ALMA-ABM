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

# Elaboration of Idea

## General Idea
First, obtain topics through semantic soft clustering on titles.
Each publication would have a vector, where levels of beloning to clusters would be stored, vectors would be of same length for each publication with values between 0 and 1.
Then, using that data, construct ABM models, which attempt to recreate properties of historical data.
Through [ablation tests](https://en.wikipedia.org/wiki/Ablation_(artificial_intelligence)), compare different versions of a model.
A model would have part responsible for causing agents to be created, be removed, and selected to coauthor publications in specific topics.
Frequency of these events would be adjusted, similarly simulation step.
What would differ for these models would be:
- function deciding which agents are selected to publish new papers at a specific step,
- function which agents use to decide in which topic to coauthor a new publication after being selected for publishing at a specific step.
It would be later checked, if these simple rules allow to recreate global patterns.
Performance would be checked by comparing with properties of historical data, such as topic clustering in network of publications and contacts similar to CHExNet.

## Decision Functions on Topics - How Models Would Differ
In ablation tests, components of models would be added or removed, and performance checked.
Decision functions would return new topic vector, and take as parameters environmental variables.
Heuristics/components which would be tested in various combinations, as only influence, or combined with others:
- Uniform random vector of topic probabilities
- Reliant on probability of choosing topics at certain year
- Reliant on past topics of publishing researchers
- Reliant on topics of past coauthors of publishing researchers

Signature of these functions could be treated as `def decide_topic(current_date: date, publishing_researchers: list[Researcher]) -> np.array`, but not all functions would use all arguments.
Second function on the above list would be effectively curried with constant value or function obtained from historical data: `topic_probabilities_at_dates`.

## Decision Functions on Group selection
- Random subset selection
- Constant probability from historical data
- Reliant on past coauthorship
`def decide_collaborate(active_researchers: list[Researcher], publications: list[Publications]) -> list[list[Researcher]]`

## Data Structures: Agent - Researcher, Publication - at Specific Time Step
Researcher has:
- ID
- date of becoming a researcher
- vector of past publication IDs

Publication has:
- ID
- list of researcher IDs
- topic vector, always of same length, with values between 0 and 1

Alternatively, instead of IDs, pointers (object references) could be used.

## Simulation for Each Step
- Add new researchers, based on probability of that happening in real historical data during step interval depending on age `how_many_new_researchers(step_duration: duration, date_now: date) -> int`
- Remove researchers from among existing ones, based on probability of that happening in real historical data between during step interval `def researcher_quit_probability(step_duration, first_publication_date: date, date_now: date) -> float`
- Choose groups of researchers which would publish new publications `def decide_collaborate(active_researchers: list[Researcher], publications: list[Publications]) -> list[list[Researcher]]`
- Decide what would be topics of new publications using a decision function `def decide_topic(current_date: date, publishing_researchers: list[Researcher]) -> np.array`

Simulation would generate results stored in similar way as historical data is stored - Pandas dataframes, for comparison with historical data.

## Comparison of Models and Historical Data
- Topic Distribution Through Time - Does the model reproduce which topics were popular at a given time?
  - calculate topic distribution per decade
  - compare with historical distribution
- Scholar Topic Specialization - Are scholars similarly specialized?
  - amount of topics they published in
  - entropy of their topic distribution
- Coauthor Topic Similarity - If adding coauthor influence improves this metric, that is evidence that the mechanism matters.
  - for every coauthor pair compute similarity of their cumulative topic vectors
  - compare mean similarity, similarity distribution
- Topic Assortativity of the Collaboration Network - Do scholars collaborate mostly with people in similar domains?
  - collaboration graph aggregated
  - nodes are scholars, edges are coauthorships
  - each scholar has vector of average publication topic
  - measure assortativity, were collaborations with authors who had focused on similar topics
