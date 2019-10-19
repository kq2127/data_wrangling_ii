Data Wrangling II
================
Kristal Quispe
10/19/2019

## Get some data

read in the NSDUH
data

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"

drug_use_xml = read_html(url)

#You can also copy and paste the url into the read_html. Tables usually works to get tables from websites, not alwasy though. 

table_list = drug_use_xml %>% html_nodes(css ="table")


(table_list) %>% .[[1]]
```

    ## {html_node}
    ## <table class="rti" border="1" cellspacing="0" cellpadding="1" width="100%" summary="This is a table containing 16 columns.">
    ## [1] <caption>Table 1 – <span class="boldital">Marijuana Use in the Past  ...
    ## [2] <thead><tr>\n<th class="left" scope="col">State</th>\r\n<th scope="c ...
    ## [3] <tfoot><tr>\n<td scope="row" colspan="16">NOTE: State and census reg ...
    ## [4] <tbody>\n<tr>\n<th scope="row">Total U.S.</th>\r\n<td width="6%" cla ...

``` r
#Here the (...) is being pipeted into the . (period) and of that we are pulling out a single html node (the first table) from a list of 15 objects. Dot(.) is a place holder for the list of 15 objects. 


tabl_marj = 
  table_list[[1]] %>% 
  html_table() %>% 
  slice(-1) %>%  #Caption at the bottom of the table has been assigned as the first row, repeating over and over for each columns. So use slice to take out the first row.  
  as_tibble()
#as tibble makes 
```

## get harry potter data

``` r
hpsaga_html = 
  read_html("https://www.imdb.com/list/ls000630791/")
```

``` r
hp_movie_names =
  hpsaga_html %>% 
  html_nodes(".lister-item-header a") %>% 
  html_text() #converts html to text. Now you have a vector with just the titles. 
hp_movie_runtime = 
  hpsaga_html %>% 
  html_nodes(".runtime") %>% 
  html_text()

hp_movie_money = 
  hpsaga_html %>% 
  html_nodes(".text-small:nth-child(7) span:nth-child(5)") %>% 
  html_text()

hp_df = 
  tibble(
    title = hp_movie_names,
    runtime = hp_movie_runtime,
    money = hp_movie_money
  )
```

## get napoleon

``` r
url = "https://www.amazon.com/product-reviews/B00005JNBQ/ref=cm_cr_arp_d_viewopt_rvwer?ie=UTF8&reviewerType=avp_only_reviews&sortBy=recent&pageNumber=2"

dynamite_html = read_html(url)

review_titles = 
  dynamite_html %>%
  html_nodes(".a-text-bold span") %>%
  html_text()

review_stars = 
  dynamite_html %>%
  html_nodes("#cm_cr-review_list .review-rating") %>%
  html_text()
#this includes the css selector for ratings on amazon

review_text = 
  dynamite_html %>%
  html_nodes(".review-text-content span") %>%
  html_text()

reviews = tibble(
  title = review_titles,
  stars = review_stars,
  text = review_text
)
```

## Use API

``` r
nyc_water_df = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.csv") %>% 
  content()
```

    ## Parsed with column specification:
    ## cols(
    ##   year = col_double(),
    ##   new_york_city_population = col_double(),
    ##   nyc_consumption_million_gallons_per_day = col_double(),
    ##   per_capita_gallons_per_person_per_day = col_double()
    ## )

``` r
#GET is one of the four API cores, that calls the data. content pulls the data and parses it assuming csv. 
```

``` r
nyc_water_df_json = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.json") %>% 
  content("text") %>%
  jsonlite::fromJSON() %>%
  as_tibble()

#the json format is messier. Its more flexible way of exposirintg data, but requires a bit mroe parsing.  In comparison to the csv file. This looks like a data rectangle. 
```

``` r
brfss_data = 
  GET("https://data.cdc.gov/api/views/waxm-p5qv/rows.csv?accessType=DOWNLOAD") %>% 
  content("parsed")
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character(),
    ##   Year = col_double(),
    ##   Locationabbr = col_double(),
    ##   Sample_Size = col_double(),
    ##   Data_value = col_double(),
    ##   Confidence_limit_Low = col_double(),
    ##   Confidence_limit_High = col_double(),
    ##   Display_order = col_double(),
    ##   LocationID = col_double()
    ## )

    ## See spec(...) for full column specifications.

``` r
#using an api is the most reproducible way of having the most up to date dataset
```

``` r
poke = 
  GET("http://pokeapi.co/api/v2/pokemon/1") %>%
  content()

poke$name
```

    ## [1] "bulbasaur"

``` r
poke$abilities
```

    ## [[1]]
    ## [[1]]$ability
    ## [[1]]$ability$name
    ## [1] "chlorophyll"
    ## 
    ## [[1]]$ability$url
    ## [1] "https://pokeapi.co/api/v2/ability/34/"
    ## 
    ## 
    ## [[1]]$is_hidden
    ## [1] TRUE
    ## 
    ## [[1]]$slot
    ## [1] 3
    ## 
    ## 
    ## [[2]]
    ## [[2]]$ability
    ## [[2]]$ability$name
    ## [1] "overgrow"
    ## 
    ## [[2]]$ability$url
    ## [1] "https://pokeapi.co/api/v2/ability/65/"
    ## 
    ## 
    ## [[2]]$is_hidden
    ## [1] FALSE
    ## 
    ## [[2]]$slot
    ## [1] 1
