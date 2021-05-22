## Welcome
My name is [Paul Stroet](https://paulstroet.netlify.app/) and currently I am writing my thesis under supervision of prof. [van de Wardt](http://www.marcvandewardt.com/) for the research master program [Business Data Science](https://businessdatascience.nl/home). On this page I post weekly updates about the progress of my thesis. 

### General idea
In a broad sense, I deploy advanced natural language processing (NLP) techniques to extract features from text data and consequently use these features in predictive modeling. This text data is web-scraped and in first instance concerns the parliamentary speeches from Belgium, but in a later stage I might expand on this with more countries. The choice for Belgium is made in consultation with prof. van de Wardt, as (1) this country has not yet been web-scraped (ie, missing in the [ParlSpeech V2 dataset](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/L4OAKN)) and therefore the dataset itself could serve as a valuable contribution to ParlSpeech, and (2) we have personality scores for the politicians in (among others) the Belgium parliament. 

### Data collection
In this section, I illustrate briefly how I collect my data. First, I take one random document from the list of 284 plenary sessions held in the 54th parliamentary term, say plenary session 111. 

```R
// R code.
library(textreadr)
url <- "https://www.dekamer.be/doc/PCRI/html/54/ip111x.html"
txt <- read_html(url)
mystring <- paste(c(txt[1:length(txt)]), collapse = " ")
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
