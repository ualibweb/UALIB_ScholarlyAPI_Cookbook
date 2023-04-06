---
output:
  html_document:
    keep_md: true
title: Chronicling America API Tutorial in R
---

By Cyrus Gomes, Vincent Scalfani, and Adam M. Nguyen

These recipe examples were tested on March 24, 2023.

**Documentation:**

[LOC Chronicling America API
Documentation](https://chroniclingamerica.loc.gov/about/api/)

See the bottom of the document for information on R and package
versions.

**Attribution:** We thank Professor Jessica Kincaid (UA Libraries, Hoole
Special Collections) for the use-cases. All data was collected from the
Library of Congress, Chronicling America: Historic American Newspapers
site, using the API.

Note that the data from the Alabama state intelligencer, The age-herald,
and the Birmingham age-herald were contributed to Chronicling America by
The University of Alabama Libraries:
https://chroniclingamerica.loc.gov/awardees/au/

# Setup

First let us setup the required packages and base url for the
Chronicling America API. If you have not already installed a package,
run “install.packages(‘package name’)”, replacing ‘package name’ with
the name of the package you require.

`{r message=FALSE, warning=FALSE} library(tidyverse)  # ggplot2 & essential pacakges library(dplyr)      # tibbles library(purrr)      # turning into character library(httr)       # GET() library(jsonlite)   # converting to JSON library(stringr)    # string removal`

Setting the base url for the API:

`{r message=FALSE, warning=FALSE} # First store string of the API's base url base_url <- "https://chroniclingamerica.loc.gov/"`

# 1. Basic API request

The Chronicling America API identifies newspapers and other records
using LCCNs. We can query the API once we have the LCCN for the
newspaper and even ask for particular issues and editions. For example,
the following link lists newspapers published in the state of Alabama,
from which the LCCN can be obtained:
https://chroniclingamerica.loc.gov/newspapers/?state=Alabama

Here is an example with the Alabama State Intelligencer:

\`\`\`{r message=FALSE, warning=FALSE} \# Below we will show a basic API
call step by step requested_url \<- paste0(base_url,
“lccn/sn84023600.json”) \# Concatenates stored api base url and the rest
of our requested url requested_url \# Display requested_url

# Retrieve raw Data from requested_url

raw_data \<- GET(requested_url) \# Retrieve data from url and presents
user with information on the response raw_data \# Display response

json_data \<- rawToChar(raw_data\$content) \# Converts raw data to JSON
format json_data

data \<- fromJSON(json_data) \# Converts to usable format

data


    Indexing into the json output allows data to be extracted using key names as demonstrated below:

    ```{r message=FALSE, warning=FALSE}
    data$name                

`{r message=FALSE, warning=FALSE} data$publisher`

Moving on to another publication, we can get the 182nd page (seq-182) of
the Evening Star newspaper published on November 19, 1961.

`{r message=FALSE, warning=FALSE} data <- fromJSON(rawToChar(GET(paste(base_url, "lccn/sn83045462/1961-11-19/ed-1/seq-182.json", sep=""))$content), flatten = TRUE) # Outputs a list of links in various formats data`

`{r message=FALSE, warning=FALSE} # Download and save the PDF url <- data$pdf # Grabs the url of the pdf url download.file(url, destfile = "file.pdf", mode = "wb") # Downloads the file and saves the pdf`

# 2. Frequency of “University of Alabama” mentions

For this next example, querying the API is relatively the same process,
therefore we will follow the same steps. Feel free to refer to example 1
for information on a step-by-step basis.

\`\`\`{r , warning=FALSE, message=FALSE} \# API Query data \<- fromJSON(
rawToChar( GET( paste0( base_url,
“search/pages/results/?state=Alabama&proxtext=(University%20of%20Alabama)&rows=500&format=json”)
)\$content) , flatten = TRUE)

# Display the first item of data

data\$items\[1, \]


    ```{r , warning=FALSE, message=FALSE}
    # Display the length of the items in the data
    nrow(data$items)

\`\`\`{r , warning=FALSE, message=FALSE} \# create a list of dates from
each item record dates \<- list() \# Create empty list

# Iterate through the data to retrieve the dates of each item

for (items in
1:nrow(data$items)){  dates <- append(x = dates, data$items\[items,
\]\$date) }

# Dispaly first 10

unlist(dates\[1:10\])


    #### Converting the dates to years, so we can draw a histogram

    ```{r , warning=FALSE, message=FALSE}
    # Create empty list of years
    years <- list()

    # Iterate through dates to convert to Years
    for (i in 1:length(dates)){
      input <- dates[[i]]
      years[[i]]<- as.numeric(format(as.Date(as.character(input), format = "%Y%m%d"), "%Y"))
    }

\`\`\`{r , warning=FALSE, message=FALSE} \# We have to first unlist the
elements to feed the values for the histogram lst2 \<- unlist(years,
use.names = FALSE)

# Produce histogram

hist(lst2, \# Data main = “Publications in the following years”, \#
Title xlab = “Years”, \# x-label ylab = “Frequency”, \# y-label col =
“Plum” \# Plum Color )


    # 3. Sunday Comic Titles in the Age-herald

    The Age - Herald published comics every Sunday, we will try to extract the titles of those published on page 15 of the 17th October 1897 edition.

    ```{r , warning=FALSE, message=FALSE}
    # API Query
    request <- rawToChar(GET(paste(base_url, "lccn/sn86072192/1897-10-31/ed-1/seq-14/ocr.txt", sep=""))$content)
    request

    # Print the first 500 characters
    substr(request,1,500) 

There are a lot of unreadable characters, and this text is hard to read.
Let us simplify it. There is a lot of text here along with random
characters and non-interpretable characters. Our approach here to get
some of the titles will be to only keep uppercase letters and lines that
are at least 75% letters

\`\`\`{r , warning=FALSE, message=FALSE} \# We first remove all chars
other than capital letters, newlines, and spaces
string_alpha\<-gsub(“\[^A-Z\]”, ” “, request)

# Display string_alpha

string_alpha



    From what we see, we can still do better.
    ```{r , warning=FALSE, message=FALSE}
    # Iterate through split strings using function 'str_split()' as a means to split string_alpha by '\n'

    for (k in str_split(string_alpha, "\n")){
      strings <- k # Number of Strings
      # We then store the number of spaces, size and actual letters we need in a list
      spaces <- lengths(regmatches(k, gregexpr(" ", k))) # Find number of spaces in each string
      head(spaces, n=5)
      size <- nchar(k) # Find overall number of characters in string
      letters <- size-spaces # Find number of  letters in string
    }

`{r , warning=FALSE, message=FALSE} # Iterate through strings to produce strings we want to see for (i in 1:length(strings)){   if (is.na(letters[i]/size[i])){   }   else if((letters[i]/size[i])>=0.75){  #Set limitations and print the characters that we want to see     cat(strings[i])     cat("\n")   } }`

# 4. Industrialization keywords frequency in the Birmingham Age-herald

We will try to obtain the frequency of “Iron” on the front pages of the
Birmingham Age- herald newspapers from the year 1900 to 1920. (limited
to first 500 rows for testing here)

`{r , warning=FALSE, message=FALSE} # Query API requests <- fromJSON(rawToChar(GET(paste(base_url, "search/pages/results/?state=Alabama&lccn=sn85038485&dateFilterType=yearRange&date1=1900&date2=1920&sequence=1&andtext=Iron&rows=500&searchType=advanced&format=json", sep=""))$content), flatten = TRUE)`

\`\`\`{r , warning=FALSE, message=FALSE} \# Create empty list of dates
dates \<- list()

# Iterate through requests$items to detrieve dates for (item in 1:length(requests$items)){

dates \<- append(dates, requests$items[item]$date) }

# Display first 10 dates

unlist(dates\[1:10\])



    ```{r , warning=FALSE, message=FALSE}
    # Display length of dates
    length(dates)

\`\`\`{r , warning=FALSE, message=FALSE} \# Create empty list of years
years \<- list()

# Convert dates to representative years for histogram

for (i in 1:length(dates)){ input \<- dates\[\[i\]\] years\[\[i\]\]\<-
as.numeric(format(as.Date(as.character(input), format = “%Y%m%d”),
“%Y”)) }



    ```{r , warning=FALSE, message=FALSE}
    # Flatten years
    hist2 <- unlist(years, use.names = FALSE)

    # Produce histogram of publications in the 
    hist(hist2,
         main = "Publications in the following years", # Title
         xlim = c(1900,1922), # Limits of x
         xlab = "Years", # x-Label
         ylab = "Frequency", # y-Label
         col = "Plum", # Set color to Plum
         breaks = 20) # 20 breaks/bins

# R Session Info

\`\`\`{r sess info, warning=FALSE, message=FALSE} sessionInfo()

\`\`\`
