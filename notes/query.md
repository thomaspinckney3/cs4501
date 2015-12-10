## Query

### 1970 vision of keyword 
* problem: some words are really rare and easy to query, but something like "the" doesn't work really well  
* Besides "term frequency", "importance of words" became reference as well
e.g. .edu document is more preferred than .com document 


### 1970 score function 
* using TIF DIF 
* Term frequency: keyword appears in the document 
* Document frequency: fractions of documents in the universe contains keyword 
* score --> 0 for really frequent word

### prototype of Google 
* Rank the most cited document. who cited "your document" also matters 
* importance of page  ≈ sum of importance of pages linked to the page


### Ranking Factors 
* TIF DIF score
* click 
* time on page (negative sign: if user clicks one page and goes back really quick)


### Ranking and Recall //I feel confused about this part 
* recall set
* score functions happened in the recall time, ahead of query time 


### Bigrams
* indexing words together "the white house"
* prefer the result with the words appear to each other


### How to make judgement 
* linear combination of the scores 
* human pick the best result from thousands queries 
* find the score function that could return the "best" queries 
* problem: sample size is really small

###
* problem: some link get clicked by chance 
* solution: doing audition 
* rewards for link show up and get clicked on; punishment for link not get clicked on 
* problem: whether people click the best link among all the results? or they just pick the appealing one?  
* punish less for those lower in the page

### Machine Learning Model 
* someone clicks the link with same query, same link is presented 
* put different users in the different buckets 
* e.g.  people don't click domino when they search "domino", they click "domino hours" instead. The result of "domino hours" will substitute "domino" later
* spell correction works in a similar way
