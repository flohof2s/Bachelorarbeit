# Webster, J. und Watson, R. T. (2002)

„Analyzing the Past to Prepare for the Future: Writing a Literature Review“. In: MIS Quarterly
26.2, S. xiii–xxiii.

## Beginning my article:

- motivate the topic
- provide working definition of key variables
- articulate the papers contributions
  - provide new theoretical understanding that helps explain previously results
  - noting that little research has addressed this topic
  - providing calls from well-respected academics to examine this topic
  - bringing togethter previously-disparate streams of work to help shed light on a phenenom
  - suggesting important implications for practice

### 2nd chapter:

- elaborate definitions of key variables
- set boundaries of the work
  - levels of analysis
  - temporal and contextual limitations
  - scope of the review
  - implicit values

Good example: Griffiths (1999) paper on "technology feauters"

## Identifying the Relevant Literature

1. Start with leading jounals to find major contributions
2. Go backward by reviewing the citations for the articles identified in step 1 to determine prior articles
   you should consider.
3. Go forward by using the Web of Science3 (the electronic version of the Social Sciences Citation Index)
   to identify articles citing the key articles identified in the previous steps. Determine which of these
   articles should be included in the review.

## Structuring the Review

- Compile concept matrix as I read each article
- If a concept has different meandings outside the scope, add second dimension to differentiate the concepts

## Tone

- in general, be fault tolerant in choosing literature
- if a "common" error emerges, point it out
- Use present tense
  - When attributing a statement or idea to a preson, use past tense

## Theoretical Development in Your article

- a review should itentify critical knowledge gaps

# Tree of Thoughts: Deliberate Problem Solving

with Large Language Models
Shunyu Yao
https://arxiv.org/pdf/2305.10601

Wie funktioniert ToT?

1. Thought decomposition: Problem in kleine, sinnvolle Denkschritte zerlegen, die einzelne Knoten in einem Suchbaum darstellen
2. Thought Generator: Aus einer Wurzel k neue Knoten generieren. Es gibt dazu 2 Strategien:
   1. Sample: Unabhängige Outputs von dem selben Prompt generieren zu lassen. Funktioniert gut, wenn der Gedankenraum groß ist (kreativ)
   2. Propose: Im Prompt die Anzahl der neuen Thoughts begrenzen. Funktioniert gut in limitierten Gedankenräume
3. State Evaluotor: Die generierten Knoten müssen nun nach einer gewissen Heuristik evaluiert werden. Dies soll das LM übernehmen. Dazu gibt es 2 Strategien: 1. Value: Jeder Zustand wird einzeln von dem LM bewertet und einen Wert zugewiesen (z.B. 1-10) 2. Vote: Alle Zustände werden miteinander verglichen und der beste soll ausgewählt werden
