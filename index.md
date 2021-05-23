## Welcome
My name is [Paul Stroet](https://paulstroet.netlify.app/) and currently I am writing my thesis under supervision of prof. [van de Wardt](http://www.marcvandewardt.com/) for the research master program [Business Data Science](https://businessdatascience.nl/home). On this page I post weekly updates about the progress of my thesis. 

### General idea
In a broad sense, I deploy advanced natural language processing (NLP) techniques to extract features from text data and consequently use these features in predictive modeling. This text data is web-scraped and in first instance concerns the parliamentary speeches from Belgium, but in a later stage I might expand on this with more countries. The choice for Belgium is made in consultation with prof. van de Wardt, as (1) this country has not yet been web-scraped (ie, not included in the [ParlSpeech V2 dataset](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/L4OAKN)) and therefore the dataset itself could serve as a valuable contribution to ParlSpeech, and (2) we have personality scores for the politicians in (among others) the Belgium parliament. 

### Data collection
In this section, I illustrate briefly how I collect my data. First, I take one random document from the list of [284 plenary sessions](https://www.dekamer.be/kvvcr/showpage.cfm?section=/cricra&language=nl&cfm=dcricra.cfm?type=plen&cricra=CRI&count=all&legislat=54) held in the 54th parliamentary term, say [plenary session 111](https://www.dekamer.be/doc/PCRI/pdf/54/ip111.pdf), and consequently use this document to elaborate on the steps needed to make the data operational. 

```R
// R code.
library(textreadr)
url <- "https://www.dekamer.be/doc/PCRI/html/54/ip111x.html"
txt <- read_html(url)
mystring <- paste(c(txt[1:length(txt)]), collapse = " ")
(nchar(mystring))
```

#### Starting rule

Now, the data is stored as one string in R and contains 231718 characters. The trick here is to extract only the information in this string which is of relevance, ie the spoken text of each politician. One can do so by first cleaning the html text into readable text (I use `BeautifulSoup` and consequently `gsub()` for this), subsetting the string (from the first character to the last character) into chunks of spoken text, and consequently concatenating these string subsets per politican. The criteria on which these subsets start with I call the `starting rule`. An example of such a `starting rule` are the digits a priori a politician starts to speak (eg 01.01 _before_ Laurette Onkelinx, 01.02 _before_ Barbara Pas, etc). One achieves this as follows.

```R
library(stringi)
extract_digits <- function(mystring){
  digits <- regex("
  \\d{2}\\.\\d{2}
  [\\ ]
  ", comments = TRUE)
  digits <- str_match_all(mystring, digits)
  dig <- NULL
  for(i in 1:dim(digits[[1]])[1]){
    dig <- c(dig, digits[[1]][i])
  }
}
relevant_digits <- extract_digits(mystring)
locate_digits <- stri_locate_all(pattern = a, mystring, fixed = TRUE)
text_subset <- NULL
for(i in 1:length(locate_digits)){
  if(i < length(locate_digits)){
    text_subset[i] <- str_sub(mystring, locate_digits[[i]][1], locate_digits[[i+1]][1]-1)
  }
  if(i == length(locate_digits)){
    text_subset[i] <- str_sub(mystring, locate_digits[[i]][1], nchar(mystring))
  }
}
text_subset <- data.frame(text_subset[1:length(text_subset)])
```

#### Stopping rule
Now, it is just a matter of finding a proper stopping rule for the subsets. For example, whenever the chairman of the session takes over the word, the spoken text of the politician talking before that must end. Let's keep track of how many non-spoken characters we remove by a `counter`. 

```R
stopping_rule <- function(e){
  counter <- sum(nchar(as.character(e[,])))
  
  f <- data.frame()
  for(i in 1:dim(e)[1]){
    if(grepl("De voorzitter", as.character(e[i,1]), fixed = TRUE)==TRUE){
      f[i,1] <- str_sub(e[i,1], 1, stri_locate_first(pattern = "De voorzitter", as.character(e[i,1]), fixed = TRUE, case_insensitive=F)[1]-1)
    }
    else{
      f[i,1] <- as.character(e[i,1])
    }
  }
  e <- f
  counter <- c(counter, sum(nchar(as.character(e[,]))))
    
  l1 <- list(e, counter)
  return(l1)
}
spoken_text <- stopping_rule(text_subset)
```

Note that this only traces the character location whenever 'De voorzitter' starts speaking in a particular string, and consequently removes whatever characters comes after this location. A critical reader might observe that the plenary sessions are alternately spoken in Dutch and French, and 'De voorzitter' solely traces the Dutch variant of chairman. Therefore, inclusion of chairman in French simply translates to 'Le président', and the code above is adjusted in a similar fashion to only retain the relevant information. 

#### Aggregate

Now, we concatenate these string subsets per politican, and save it as spoken_text_per_politician, so all spoken text from one politician is summarized into one string. This makes merging of the different plenary sessions in a later stadium easier. The result is a dataset with two variables (the unique politician, and all spoken text of that politician concatenated into one string) and 37 observations. So, 37 unique politicians contributed to the session in plenary session 111. Now, one can finetune the starting and stopping rules a bit further, simply iterate over all the other sessions, and this will result in the final dataset encapsulating all plenary sessions of parliamentary term 54. Then, the same ideas apply to (1) parliamentary term 55 and the committee sessions in a given parliamentary term. 

### Datasets
The dataset of this specific session is accessible via 'Download ZIP File' on the left, under the name of 'session_111' (in `.xlsx` format). Please contact me if you are interested in the complete dataset containing all spoken text of all plenary sessions, whether this concerns the 54th parliamentary term, any other term, or multiple terms. 

### Summary statistics parliamentary term 54

| Parliament   | Session   | Period       | File size (MB)    | Unique speakers |Average character count per speaker |
|:-------------|:----------|:-------------|:------------------|:----------------|:-----------------------------------|
| Belgium      | [Plenary](https://www.dekamer.be/kvvcr/showpage.cfm?section=/cricra&language=nl&cfm=dcricra.cfm?type=plen&cricra=CRI&count=all&legislat=54)   | 2014-2019    | 45                | 186             | 242929                             |
| Belgium      | [Plenary](https://www.dekamer.be/kvvcr/showpage.cfm?section=/cricra&language=nl&cfm=dcricra.cfm?type=plen&cricra=CRI&count=all&legislat=55)   | 2019-now     | 18                | 182             |  98592                             |
| Belgium      | [Committee](https://www.dekamer.be/kvvcr/showpage.cfm?section=/cricra&language=nl&cfm=dcricra.cfm?type=comm&cricra=cri&count=all)   | 2014-2019     | tba                | tba             |  tba                             |
| Belgium      | [Committee](https://www.dekamer.be/kvvcr/showpage.cfm?section=/cricra&language=nl&cfm=dcricra.cfm?type=comm&cricra=CRI&count=all&legislat=55)   | 2019-now     | 89                | tba             |  tba                             |

add flag

### More coming soon
I wrote the code in a generic way, ie it is easily adaptable to different formats and it has a quick and insightful debug function. Soon I will illustrate how this is done. Also, I will set forth the preliminary analysis of the text data, the modifications to the LDA algorithm I make and a clever way how to extract subsets of the data to fastly train and fine-tune the algorithm on, while maintaining performance when training on the whole training set. 

### Contact
For any queries and/or interest for collaborations on exciting projects, please reach me at p.stroet@businessdatascience.nl
