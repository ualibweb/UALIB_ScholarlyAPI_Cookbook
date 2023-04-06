---
editor_options:
  chunk_output_type: console
output: html_document
---

# Pubchem API in R

#### By Vishank Patel

Pubchem API Documentation:
<https://pubchemdocs.ncbi.nlm.nih.gov/programmatic-access>

Mathematica PubChem documentation:
<https://reference.wolfram.com/language/ref/service/PubChem.html>

These recipe examples were tested on April 19, 2022.

Attribution: This tutorial was adapted from supporting information in:

Scalfani, V. F.; Ralph, S. C. Alshaikh, A. A.; Bara, J. E. Programmatic
Compilation of Chemical Data and Literature From PubChem Using Matlab.
Chemical Engineering Education, 2020, 54, 230.
<https://doi.org/10.18260/2-1-370.660-115508> and
<https://github.com/vfscalfani/MATLAB-cheminformatics>)

### Setup

Importing the necessary libraries and setting up the base api:

\`\`\`{r message=FALSE, warning=FALSE} library(tidyverse) \#essential
packages library(dplyr) \#tibbles (R data_frames) library(purrr)
\#character manipulation library(httr) \#GET() API requests
library(jsonlite) \#converting to JSON library(knitr) \#including
graphics library(imager) \#including images

api \<- ‘https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/’


    ### 1. PubChem Similarity

    Search for chemical structures in PubChem via a Fingerprint Tanimoto Similarity Search.

    #### Get compound image

    ```{r}
    compoundID <- "2734162"
    CID_URL <- paste(api,"cid/",compoundID,"/PNG",sep = "")  #paste concatenates strings 

    include_graphics(CID_URL)

Replace the above CID value (CID_SS_query) with a different CID to
customize.

#### Retrieve InChI and SMILES

Retrieve InChI

\`\`\`{r} inchi_url \<-
paste(api,“cid/”,compoundID,“/property/inchi/TXT”,sep = ““)

raw_inchi \<- rawToChar(GET(inchi_url)$content); #"$content” filters the
http response from the output and only returns the required output data
inchi \<- raw_inchi %\>% gsub(“”,““,.); \#”.” refers to raw_inchi in
gsub inchi


    Retrieve Isomeric SMILES

    ```{r}
    IS_url <- paste(api,"cid/",compoundID,"/property/IsomericSMILES/TXT", sep = "");

    raw_IS <- rawToChar(GET(IS_url)$content);
    IS <- raw_IS %>% gsub("\n","",.);
    IS

#### Perform a Similarity Search

Search for chemical structures by Similarity Search (SS), (2D Tanimoto
threshold 95% to 1-Butyl-3-methyl-imidazolium; CID = 2734162)

\`\`\`{r} api \<- “https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/”;
SS_url \<- paste(api,‘fastsimilarity_2d/cid/’,compoundID,
‘/cids/JSON?Threshold=95’,sep = ““);

raw_output \<- GET(SS_url)\$content; \#getting the unicode output
raw_result \<- rawToChar(raw_output); \#converts the output from unicode
to text CIDs1_ls \<- fromJSON(raw_result); \#creates a list of lists
from the JSON data


    ```{r}
    head(CIDs1_ls)

Working with list of lists is hard, so we convert the same into a
tibble. The code is adopted from:
https://www.r-bloggers.com/2015/11/accessing-apis-from-r-and-a-little-r-programming/

\`\`\`{r} CIDs1_df \<- do.call(what = “rbind”, args = lapply(CIDs1_ls,
as.data.frame)) \#converts the list of lists to a dataframe \#lapply
applies its second argument (as.data.frame) to every element of the list
in the first argument (result1_ls) \#rbind binds all the dataframes
together \#do.call feeds in all the arguments (dataframes to be binded
together) to the rbind command

CIDs1_df \<- tibble(CIDs1_df) \# converting the dataset into a tibble
(from dplyr) CIDs1_df \#displaying the first few elements of the data


    In the above SS_url value, you can adjust to the desired Tanimoto threshold (i.e., 97, 90, etc.)

    #### Retrieve Identifier and Property Data

    Create an identifier/property dataset from Similarity Search results.

    Retrieve the following data from CID hit results: InChI, Isomeric SMILES, MW, Heavy Atom Count, Rotable Bond Count, and Charge

    ```{r}
    short_CIDs <- CIDs1_df$CID[1:25] #taking the first 25 CIDs from the similarity search results

    #initializing the tibble
    similarity_results_tibble <- tibble();
    similarity_results_tibble <- add_column(similarity_results_tibble,
                                 Compound_ID = "",
                                 InChi = "",
                                 IsoSMI = "",
                                 MW = "",
                                 Heavy_Atom_Count = "",
                                 Rotatable_Bond_Count = "",
                                 Charge = ""
                                 );


    for (CID in short_CIDs) {
      
      #define the api calls:
      api = 'https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/';
      CID_InChI_url = paste(api,'cid/',toString(CID),'/property/InChI/TXT',sep = "");
      CID_IsoSMI_url = paste(api,'cid/',toString(CID),'/property/IsomericSMILES/TXT',sep="");
      CID_MW_url = paste(api,'cid/',toString(CID),'/property/MolecularWeight/TXT',sep="");
      CID_HeavyAtomCount_url = paste(api,'cid/',toString(CID),'/property/HeavyAtomCount/TXT',sep="");
      CID_RotatableBondCount_url = paste(api,'cid/',toString(CID),'/property/RotatableBondCount/TXT',sep="");
      CID_Charge_url = paste(api,'cid/',toString(CID),'/property/Charge/TXT',sep="");
      
      
      #downloading the data
      inchi_temp <- rawToChar(GET(CID_InChI_url)$content) %>% gsub("\n","",.);
      Sys.sleep(1)       # adding a delay for the PubChem server
      isoSMI_temp <- rawToChar(GET(CID_IsoSMI_url)$content) %>% gsub("\n","",.);
      Sys.sleep(1)
      mw_temp <- rawToChar(GET(CID_MW_url)$content) %>% gsub("\n","",.);
      Sys.sleep(1)
      heavy_atom_count_temp <- rawToChar(GET(CID_HeavyAtomCount_url)$content) %>% gsub("\n","",.);
      Sys.sleep(1)
      rotatable_bond_count_temp <- rawToChar(GET(CID_RotatableBondCount_url)$content) %>% gsub("\n","",.);
      Sys.sleep(1)
      charge_temp <- rawToChar(GET(CID_Charge_url)$content) %>% gsub("\n","",.);
      Sys.sleep(1)

      #Appending the data in a tibble
      similarity_results_tibble <- similarity_results_tibble %>%
        add_row(
          Compound_ID = toString(CID),
          InChi = inchi_temp,
          IsoSMI = isoSMI_temp,
          MW = mw_temp,
          Heavy_Atom_Count = heavy_atom_count_temp,
          Rotatable_Bond_Count = rotatable_bond_count_temp,
          Charge = charge_temp
        )

    }

    similarity_results_tibble

We will now export the generated dataframe as a tab separated text file.
The file will be saved in the present working directory.

`{r} write.table(similarity_results_tibble, file = "Data/R_Similarityq_results.txt", sep = "\t", row.names = TRUE, col.names = NA);`

#### Retrieve Images of CID Compounds from Similarity Search

Create the results png:

\`\`\`{r message=FALSE, warning=FALSE}
png(“Data/Similarity_plot_save.png”, width = 2096, height = 2096);
\#Initialize the results png

par(mfrow=c(5,5),mar=c(.1,.1,.1,.1)) \#setup plot parameters

for(CID in short_CIDs\[1:25\]){ \#define the url CID_URL \<-
paste(api,“cid/”,toString(CID),“/PNG”,sep = ““); download.file(CID_URL,
destfile =”Data/tempfile.png”, method = “libcurl”); Sys.sleep(1)  
im \<- load.image(“Data/tempfile.png”) plot(im,axes = FALSE) }

dev.off(); \#save the png


    Display the results:

    ```{r}
    include_graphics("Data/Similarity_plot_save.png")

### 2. PubChem SMARTS

Search for chemical structures in PubChem via a SMARTS substructure
query and compile results

`{r} #defining the PubChem base API api <- "https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/";`

#### Define SMARTS Queries

\`\`\`{r} \# view pattern syntax at:
https://smartsview.zbh.uni-hamburg.de/ \# these are vinyl imidazolium
substructure searches

SmartsQ \<-
c(“\[CR0H2\]\[n+\]1\[cH1\]\[cH1\]n(\[CR0H1\]=\[CR0H2\])\[cH1\]1”,“\[CR0H2\]\[n+\]1\[cH1\]\[cH1\]n(\[CR0H2\]\[CR0H1\]=\[CR0H2\])\[cH1\]1”,“\[CR0H2\]\[n+\]1\[cH1\]\[cH1\]n(\[CR0H2\]\[CR0H2\]\[CR0H1\]=\[CR0H2\])\[cH1\]1”);


    Add your own SMARTS queries to customize. You can add as many as desired within a cell array.

    #### Perform a SMARTS Query Search

    ```{r}
    #generate URLs for SMARTS query searches

    SmartsQ_url= c();
    for (Smarts_query in SmartsQ) {
      SmartsQ_url <- append(SmartsQ_url, 
                            values = paste(api,"fastsubstructure/smarts/",Smarts_query,"/cids/JSON",sep = ""));
    }

    #perform substructure searches for each query link in SMARTSq_url

    hit_CIDs_results_seperate =c();
    for (url in SmartsQ_url) {
      hit_CIDs_ls_temp <- fromJSON(rawToChar(GET(url)$content))
      hit_CIDs_df_temp <- tibble(do.call(what = "rbind",
                                         args = lapply(hit_CIDs_ls_temp, as.data.frame)))
      hit_CIDs_results_seperate <- append(hit_CIDs_results_seperate, hit_CIDs_df_temp)
    }

    #create the final result by appending each of the returned CID list outputs 
    hit_CIDs_results <- c();
    hit_CIDs_results <- append(hit_CIDs_results,
                                     values = c(hit_CIDs_results_seperate[1]$CID,
                                                hit_CIDs_results_seperate[2]$CID, 
                                                hit_CIDs_results_seperate[3]$CID))
    length(hit_CIDs_results)

We will shorten the list to the first 25 results for time
considerations. This limit can be increased.

`{r} hit_CIDs_results <- hit_CIDs_results[1:25]; hit_CIDs_results`

## Retrieve Identifier and Property Data

Create an identifier/property dataset from the SMARTS substructure
search results

Retrieve the following data for each CID: InChI, Canonical SMILES, MW,
IUPAC Name, Heavy Atom Count, Covalent Unit Count, Charge

\`\`\`{r}

smarts_results_tibble \<- tibble(); smarts_results_tibble \<-
add_column(smarts_results_tibble, Compound_ID = ““, InChi =”“, CanSMI
=”“, MW =”“, IUPACName =”“, Heavy_Atom_Count =”“, Covalent_Unit_Count
=”“, Charge =”” );

for (CID in hit_CIDs_results) {

\#define the api calls: api =
‘https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/’; CID_InChI_url =
paste(api,‘cid/’,toString(CID),‘/property/InChI/TXT’,sep = ““);
CID_CanSMI_url =
paste(api,‘cid/’,toString(CID),‘/property/CanonicalSMILES/TXT’,sep=”“);
CID_MW_url =
paste(api,‘cid/’,toString(CID),‘/property/MolecularWeight/TXT’,sep=”“);
CID_IUPACName_url =
paste(api,‘cid/’,toString(CID),‘/property/IUPACName/TXT’,sep=”“);
CID_HeavyAtomCount_url =
paste(api,‘cid/’,toString(CID),‘/property/HeavyAtomCount/TXT’,sep=”“);
CID_CovalentUnitCount_url =
paste(api,‘cid/’,toString(CID),‘/property/CovalentUnitCount/TXT’,sep=”“);
CID_Charge_url =
paste(api,‘cid/’,toString(CID),‘/property/Charge/TXT’,sep=”“);

\#downloading the data inchi_temp \<-
rawToChar(GET(CID_InChI_url)$content) %>% gsub("\n","",.);  Sys.sleep(1) # adding a delay for the PubChem server  canSMI_temp <- rawToChar(GET(CID_CanSMI_url)$content)
%\>% gsub(“”,““,.); Sys.sleep(1) mw_temp \<-
rawToChar(GET(CID_MW_url)$content) %>% gsub("\n","",.);  Sys.sleep(1)  iupac_name_temp <- rawToChar(GET(CID_IUPACName_url)$content)
%\>% gsub(”“,”“,.); Sys.sleep(1) heavy_atom_count_temp \<-
rawToChar(GET(CID_HeavyAtomCount_url)$content) %>% gsub("\n","",.);  Sys.sleep(1)  covalent_unit_count_temp <- rawToChar(GET(CID_CovalentUnitCount_url)$content)
%\>% gsub(”“,”“,.); Sys.sleep(1) charge_temp \<-
rawToChar(GET(CID_Charge_url)\$content) %\>% gsub(”“,”“,.); Sys.sleep(1)

\#Appending the data in a tibble smarts_results_tibble \<-
smarts_results_tibble %\>% add_row( Compound_ID = toString(CID), InChi =
inchi_temp, CanSMI = canSMI_temp, MW = mw_temp, IUPACName =
iupac_name_temp, Heavy_Atom_Count = heavy_atom_count_temp,
Covalent_Unit_Count = covalent_unit_count_temp, Charge = charge_temp )

}

smarts_results_tibble


    Rearrange the result's columns:

    ```{r}
    smarts_results_tibble2 <- smarts_results_tibble[c("CanSMI","IUPACName","Compound_ID","InChi","MW","Heavy_Atom_Count","Covalent_Unit_Count","Charge")]

Exporting the data as a tabbed text file

\`\`\`{r}

write.table(smarts_results_tibble2, file = “Data/R_Smartsq_results.txt”,
sep = “, row.names = TRUE, col.names = NA);


    #### Retrieve Images of CID Compounds from SMARTS query match

    Create the results png:

    ```{r message=FALSE, warning=FALSE}
    png("Data/Smarts_plot_save.png", width = 2096, height = 2096); #Initialize the results png

    par(mfrow=c(5,5),mar=c(.1,.1,.1,.1)) #setup plot parameters

    for(CID in hit_CIDs_results){
      #define the url
      CID_URL <- paste(api,"cid/",toString(CID),"/PNG",sep = "");
      download.file(CID_URL, destfile = "Data/tempfile.png", method = "libcurl");
      Sys.sleep(1)      
      im <- load.image("Data/tempfile.png")
      plot(im,axes = FALSE)
    }

    dev.off();  #save the png

Display the results:

`{r} include_graphics("Data/Smarts_plot_save.png")`
