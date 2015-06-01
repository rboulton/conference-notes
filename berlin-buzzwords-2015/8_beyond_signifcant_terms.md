# Beyond Significant terms - André Lynum

http://berlinbuzzwords.de/session/beyond-significant-terms

# Application

 - Openly accessible data "intelligence"
 - low resources
 - unusual sources and languages

# Significant terms

 - find the "commonly uncommon"
 - get the word frequency ratio for each term in the whole dataset ("background")
 - pick positive deviation from the ratio

 - Use significant terms to find good terms to include in the summaries

# Single document significant terms

 - Incude "Pareto" style score (http://en.wikipedia.org/wiki/Pareto_analysis)
  - Use document term vector against background
  - External background with IDF weighting
  - custom score: TF * IDF(domain) * IDF(general)

  - terms aggragation gives all the info you need to get IDF
  - get TF from term vectors

# Entity extraction

 - Finding things in text
 - And finding where they are in the text

 - no training data, not all in english
 
 - use high recall heuristics to get lost of possible entities
 - find significant terms in a document
 - and use cheap knowledge sources to fix it up

 - eg, use capitalised terms, and use DBpedia to help filter

# Trneding keywords on small data

 - increasing recent usage
 - with few documents, windowing approach fails
 - trends spike on immediate activity, far too soon

 So, smooth things out.

# Guided filtering

 - make filters for special topics
 - allow users to guide the filters
   - based on filtering output of significant terms
