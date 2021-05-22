## Welcome
My name is [Paul Stroet](https://paulstroet.netlify.app/) and currently I am writing my thesis under supervision of prof. [van de Wardt](http://www.marcvandewardt.com/) for the research master program [Business Data Science](https://businessdatascience.nl/home). On this page I post weekly updates about the progress of my thesis. 

### General idea
In a broad sense, I deploy advanced natural language processing (NLP) techniques to extract features from text data and consequently use these features in predictive modeling. This text data is web-scraped and in first instance concerns the parliamentary speeches from Belgium, but in a later stage I might expand on this with more countries. The choice for Belgium is made in consultation with prof. van de Wardt, as (1) this country has not yet been web-scraped (ie, missing in the [ParlSpeech V2 dataset](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/L4OAKN)) and therefore the dataset itself could serve as a valuable contribution to ParlSpeech, and (2) we have personality scores for the politicians in (among others) the Belgium parliament. 

### Data collection
In this section, I illustrate briefly how I collect my data. First, I take one random document from the list of [284 plenary sessions](https://www.dekamer.be/kvvcr/showpage.cfm?section=/cricra&language=nl&cfm=dcricra.cfm?type=plen&cricra=CRI&count=all&legislat=54) held in the 54th parliamentary term, say [plenary session 111](https://www.dekamer.be/doc/PCRI/pdf/54/ip111.pdf). 

```R
// R code.
library(textreadr)
url <- "https://www.dekamer.be/doc/PCRI/html/54/ip111x.html"
txt <- read_html(url)
mystring <- paste(c(txt[1:length(txt)]), collapse = " ")
(nchar(mystring))
```

Now, the data is stored as one string in R and contains 231718 characters. The trick here is to extract only the information in this string which is of relevance, ie the spoken text of each politician. One can do so by subsetting the string (from the first character to the last character) into chunks of spoken text, and consequently concatenating these string subsets per politican, so all spoken text from one politician is summarized into one string. This makes merging of the different plenary sessions in a later stadium easier. The criteria of these subsets are the digits a priori a politician starts to speak (eg 01.01 _before_ Laurette Onkelinx, 01.02 _before_ Barbara Pas, etc). One achieves this as follows.

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

Now, it is just a matter of finding a proper stopping rule for the subsets. For example, whenever the chairman of the session takes over the word, the spoken text of the politician talking before that must end. 

```R
// R code.
library(textreadr)
url <- "https://www.dekamer.be/doc/PCRI/html/54/ip111x.html"
txt <- read_html(url)
mystring <- paste(c(txt[1:length(txt)]), collapse = " ")
(nchar(mystring))
```

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/paulstroet/bds_thesis/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
