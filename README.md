[![Build Status](https://travis-ci.org/jhu-digital-manuscripts/gcme.png?branch=master)](https://travis-ci.org/jhu-digital-manuscripts/gcme)

# Introduction

This project refashions a website which allowed tagged Middle English text to be searched. See https://middleenglish.library.jhu.edu/about for more information.

# Terminology

Consider `may{*mouen@v3%pr_1*}`

* Word: `may`
* Lemma or headword: `mouen` 
* Tagged lemma: `mouen@v3%pr_1`
* Tag: v3%pr_1

# Data

The underlying data is lines of text oragnized in a heirarchy of groups.
The toplevel groups correspond to works by Chaucer and Gower. The bottom groups
are something like a chapter in a book. Each group has a short identifier and a title.
The file abbr2title.lut in a strange fashion specifies this structure together with the ids and titles.

Ordered lines of text are stores in the texts directory as .cat files.
A line has each of the original words associated with a tagged lemma.
Each .cat file is part of 2-4 groups. The mapping from groups to .cat files is specified in abbr2file.txt.

# Java tool

A java tool in gcme-tool transforms the data into static files for the Ember UI and
documents for Elasticsearch. From gcme-tool run `mvn package` in order to produce
and executable jar in target.

To test it out try running something like `java -jar target/gcme-tool-0.0.1-SNAPSHOT-shaded.jar ../data/ info`
to see the structure of the texts printed out.

# Elasticsearch indices

## line

The line index allows lines of text to be searched for by word or by tagged lemma.
The text field contains the raw words of the line and uses the simple analyzer
which ignores case and handles punctuation. The tag_lemma_text field contains
the tagged lemmas for the words of the line and uses a custom anaylyzer which
ignores case and tokenizes based on whitespace. The id is the identifier
assigned to the line. The raw_number is the number assigned to the line.
It is either an integer or an integer followed by some letters. The number field
is the integer extracted from raw_number. The group is an array of 2-4 identifiers
for all of the groups containing the line in order from toplevel to parent.
For example a line in the Knight's tale would have group `["Ch", "CT", "Frag1", "KnT"]`.


| Field          | Type    | Cardinality |
| -------------- | ------- | ----------- |
| id             | keyword | 1           | 
| number         | integer | 1           |
| raw_number     | keyword | 1           |
| group          | keyword | 2-4         |
| text           | text    | 1           |
| tag_lemma_text | text    | 1           |


## dict

The dict index allows a definition for a tagged lemma to be looked up.
The tagged lemma is associated with its word forms as well as a dictionary definition.
Completion can be done on the tagged lemma as well as its word forms using the .suggest
subfields. The word forms have been normalized to lower case.

| Field             | Type       | Cardinality |
| ----------------- | ---------- | ----------- |
| word              | keyword    | 1*          |
| word.suggest      | completion | 1*          | 
| tag_lemma         | keyword    | 1           |
| tag_lemma.suggest | completion | 1           |
| definition        | text       | 1           |


# Ember UI

The ember UI uses static files generated by the Java tool and Elasticsearch.

# Deployment

Install Elasticsearch 6.2 or later which should be available at http://localhost:9200.

In order to generate data for Elasticsearch and Ember, from the deploy directory run
`./gen_data.sh`. It will put some files in the gcme-ember/public and write out
ndjson files for Elasticsearch. If that is successful, run `./update_indicies.sh`
which will delete, create, and then update the dict and line indicies in Elasticsearch.

To build the ember UI, first install ember 3.2 or later and its prerequisites.
Then in gcme-ember, run `ember build --environment=production`.
To deploy ember, ensure that elasticsearch at http://localhost:9200/_search is available as `/es`/
and copy dist/* to an appropriately configured web server.

