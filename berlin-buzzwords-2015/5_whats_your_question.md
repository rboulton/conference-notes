# What's your question - Grant Ingersol

Question answering
 - Watson playing jeopardy
 - Giving answers, rather than links

Gave some eaxmples of question answering using Solr
 - get back short answers to questions
 - but based on lots of supporting content

Doesn't always work.

Oriented around "fact based questions".

## Taming text

No system will be perfect

Most NLP work is "grunt" work; getting data to the right place in the right format

- So start with simple, with tried and tested algorithms, then iterate.

## A system

 - indexing wikipedia documents
   - sentence detection (OpenNLP)
   - NER (OpenNLP)
   - Indexing (Solr)

 - Questions:
   - Parse Query (OpenNLP)
   - Determine Answer Type (OpenNLP)
   - Generate Query (custom)
   - Execute Search (Solr)
   - Rank Passages (custom)

Custom code described in chapter 2 of "Taming Text".

## Passage search

 - Use a custom query parser: take in user's query, classify it to find the answer type, generate solr query
 - get passages of text that match keywords and the expected answer type
 - need to know where matches occur

## Answer type

 - Could define lots of types; eg, "find a person", "location", "organisation", "time point", "duration", "money"
 - Train a classifier using annotated questions

 - Simple scoring approach: (from AT&T in 90s)

  - identify and rank passages
  - look for named entity presence, term matches

## User intent

 - Can use the query classification for determining user intent, as well as question answering

 - Demo
   - an "FAQ" system; labelled into categories
   - Use categories as answer type for training classifier
   - Do a best-bet based on category of question
   - Could show different facets based on category, or do other things

## Next steps

 - extract the answers from the sentence
 - de-duplication / ranking of answers
 - multiple data sources; authority voting
 - better query handling, more answer types, richer NLP processing
 - Feedback loops using signals

# Questions

 - (I missed the question)
   - Using payloads, put the POS tag information into core index along with terms
