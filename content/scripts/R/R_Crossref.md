---
editor_options:
  chunk_output_type: console
output: html_document
---

# CrossRef API in R

#### By Cyrus Gomes and Vincent Scalfani

Crossref API Documentation:
https://api.crossref.org/swagger-ui/index.html

These recipe examples were tested on October 10, 2022.

From our testing, we have found that the crossref metadata across
publishers and even journals can vary considerably. As a result, it can
be easier to work with one journal at a time when using the crossref API
(e.g., particulary when trying to extract selected data from records).

### Setup

Importing the necessary libraries and setting up the base api:

\`\`\`{r message=FALSE, warning=FALSE} library(tidyverse) \#essential
packages library(purrr) \#character manipulation library(httr) \#GET()
API requests library(jsonlite) \#converting to JSON

base_url \<- “https://api.crossref.org/works/”


    ### 1. Basic CrossRef API call

    #### Let's set up basic parameters

    ```{r}
    email <- "your_email@ua.edu" #Change this to your email
    mailto <- "?mailto="
    doi <- "10.1186/1758-2946-4-12" #example

    complete_data <- paste(base_url, doi, mailto, email, sep="") #combines it all in one line
    complete_data

#### Request data from CrossRef

\`\`\`{r} raw_api_data \<- GET(complete_data)\$content \# fetches
content from the data raw_api_data

    The raw data is in hexadecimal and is hard to read.

    #### Converting the data to JSON format (which is more readable)

    ```{r}
    api_data <- fromJSON(rawToChar(raw_api_data), flatten = TRUE)
    print(api_data)

Replace the above DOI value with a different DOI to customize.

#### Let’s see the reference part of CrossRef in a convenient tabular way

`` {r} api_data$message$`reference`[1:7,1:7]    # prints the table of data ``

#### Selecting some specific data

Getting the Journal Title

\`\``{r} #Get Journal Title api_data$message$`container-title\` \#
prints the title


    Getting the Article Title

    ```{r}
    api_data$message$`title` 

Getting the Article Author Names

`{r} for (au in api_data$message$author){   article_authors <- paste(api_data$message$author$given, api_data$message$author$family) } article_authors`

Getting the Bibliography References and saving them to a list

\`\`\`{r} \#Get bibliography references and save to list bib_refs \<-
list() \#declare a list for (ref in api_data$message$reference){
bib_refs \<- append(x = bib_refs,
api_data$message$reference\$unstructured) } bib_refs\[0:5\]


    #### Save JSON data to a file

    This is particularly useful for downstream testing or returning to results in the future (e.g., no need to keep requesting the data from crossref, save the results to a file)

    ```{r}
    #Save JSON data to a file
    jsonData <- toJSON(api_data)
    write (jsonData, "output.json")

#### Load JSON data from a file

`{r} loaded_data <- fromJSON("output.json") loaded_data                       #Load Json data from file`

### 2. CrossRef API call with a Loop

#### Setup API parameters

\`\`\`{r} base_url \<- “https://api.crossref.org/works/” email \<-
“your_email@ua.edu” \#Change this to your email mailto \<- “?mailto=”
doi \<- “10.1186/1758-2946-4-12” \#example


    #### Create a List of DOIs

    ```{r}
    #Create List of DOIs

    doi_list <- list('10.1021/acsomega.1c03250',                          
                 '10.1021/acsomega.1c05512',
                 '10.1021/acsomega.8b01647',
                 '10.1021/acsomega.1c04287',
                 '10.1021/acsomega.8b01834')

#### Request metadata for each DOI from CrossRef API and save to a list

`{r} doi_metadata <- list(list()) # a list of lists is used to hold the data from the 5 DOIs i <- 1 for (doi in doi_list){   doi_metadata[[i]] <-fromJSON(rawToChar(GET(paste(base_url, doi, mailto, email, sep=""))$content), flatten = TRUE)   i <- i+1   Sys.sleep(1)  # important to add a delay between API calls } # doi_metadata ## not shown here as it is long.`

Let’s select some specific data

`{r} # Get Article titles for (item in 1:length(doi_metadata)){   print(doi_metadata[[item]]$message$title) }`

`{r} #Get Author affiliations for each Article for (item in 1:length(doi_metadata)){   for (au in 1:length(doi_metadata[[item]]$message$author)){     print(doi_metadata[[item]]$message$author$affiliation[[1]]$name)   } }`

### 3. Crossref API call for journal information

#### Setup API parameters

`{r} jbase_url <- "https://api.crossref.org/journals/" # the base url for api calls email <- "your_email@ua.edu" # Change this to be your email mailto <- "?mailto=" issn <- "1471-2105"  # issn for the journal BMC Bioinformatics`

#### Request journal data from crossref API

`{r} jour_data <- fromJSON(rawToChar(GET(paste(jbase_url, issn, mailto, email, sep=""))$content), flatten = TRUE)  jour_data`

### 4. Crossref API - Get article DOIs for a journal

#### Setup API parameters

`{r} jbase_url <- "https://api.crossref.org/journals/" # the base url for api calls email <- "your_email@ua.edu" # Change this to be your email mailto = "&mailto=" issn <- "1471-2105"  # issn for the journal BMC Bioinformatics journal_works2014 <- "/works?filter=from-pub-date:2014,until-pub-date:2014&select=DOI" # query to get DOIs for 2014`

#### Request DOI data from crossref API

`{r} doi_data = fromJSON(rawToChar(GET(paste(jbase_url, issn, journal_works2014, mailto, email, sep=""))$content), flatten = TRUE) doi_data`

By default, 20 results are displayed. Crossref allows up to 1000
returned results using the rows parameter. To get all 619 results, we
can increase the number of returned rows.

`{r} rows <- "&rows=700" doi_data_all <- fromJSON(rawToChar(GET(paste(jbase_url, issn, journal_works2014, rows, mailto, email, sep=""))$content), flatten = TRUE)`

#### Extract DOIs

`{r} dois_list = list() for (i in doi_data_all$message$items){   dois_list <- append(x = dois_list, doi_data_all$message$items$DOI) } length(dois_list)`

`{r} #display the first 20 print (dois_list[0:20])`

What if we have more than 1000 results in a single query?

For example, if we wanted the DOIs from BMC Bioinformatics for years
2014 through 2016?

`{r} jbase_url <- "https://api.crossref.org/journals/" # the base url for api calls email <- "your_email@ua.edu" # Change this to be your email mailto <- "&mailto=" issn <- "1471-2105"  # issn for the journal BMC Bioinformatics journal_works2014_2016 <- "/works?filter=from-pub-date:2014,until-pub-date:2016&select=DOI" # query to get DOIs for 2014-2016 doi_data2 <- fromJSON(paste(jbase_url, issn, journal_works2014_2016, mailto, email, sep=""), flatten = TRUE) doi_data2`

Here we see that the total results is over 1000 (total-results: 1772).
An additional parameter that we can use with crossref API is called
“offset”. The offset option allows us to select sets of records and
define a starting position (e.g., the first 1000, and then the second
set of up to 1000.)

\`\`\`{r} doi_list2 \<- list() \#declare a list rows \<- “&rows=1000”

temp \<- fromJSON(paste(jbase_url, issn, journal_works2014_2016, mailto,
email, sep=““), flatten = TRUE) numResults\<-
temp$message$`total-results` Sys.sleep(1)

end \<- as.integer(numResults/1000)+1

for (n in 0:end){ \# list(range(int(numberOfResults/1000)+1)) = \[0,1\]
query \<- fromJSON(paste(jbase_url, issn, journal_works2014_2016, rows,
“&offset=”, as.character(1000\*n), mailto, email, sep=““), flatten =
TRUE) Sys.sleep(1) for (doi in query$message$items){ doi_list2 \<-
append(x = doi_list2, query$message$items\$DOI) } }


    ```{r}
    length(doi_list2)

`{r} #how results 1000 through 1020 print(doi_list2[1000:1020])`
