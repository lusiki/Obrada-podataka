---
title: "Obrada podataka"
author:
  name: Luka Sikic, PhD
  affiliation: Fakultet hrvatskih studija | [OP](https://github.com/BrbanMiro/Obrada-podataka)
subtitle: 'Predavanje 6: Preuzimanje podataka sa interneta (Webscraping II)'
output:
  html_document:
    theme: flatly
    highlight: haddock
    toc: yes
    toc_depth: 4
    toc_float: yes
    keep_md: yes
  pdf_document:
    toc: yes
    toc_depth: '4'
---




## Registracija i softwerski set-up

### Registracija

U ovom dijelu predavanja ćemo preuzeti ekonomske podatke (između ostalog) sa FRED API-ja. To zatijeva [registraciju](https://research.stlouisfed.org/useraccount/apikey) i pohranu [API ključa](https://research.stlouisfed.org/useraccount/apikey).

### "Vanjski" software

Koristiti ćemo [JSONView](https://jsonview.com/) browser ekstenziju koja omogućava pregled JSON output-a u Chrome i Firefox. (Nije nužno ali preporučeno!)

### R paketi 

- New: **jsonlite**, **httr**, **listviewer**, **usethis**, **fredr**
- Already used: **tidyverse**, **lubridate**, **hrbrthemes**, **janitor**

Prigodan način da instalirate i učitate sve prethodno pobrojane pakete ukoliko to niste već napravili.


```r
## učitaj i instaliraj pakete
if (!require("pacman")) install.packages("pacman")
```

```
## Warning: package 'pacman' was built under R version 4.0.3
```

```r
pacman::p_load(tidyverse, httr, lubridate, hrbrthemes, janitor, jsonlite, listviewer, usethis)
pacman::p_install_gh("sboysel/fredr") ## https://github.com/sboysel/fredr/issues/75
#tema
theme_set(hrbrthemes::theme_ipsum())
```



## Prisjetimo se...

U prvom dijelu preavanja o preuzimanju sadržaja sa interneta smo vidjeli da web stranice i web aplikacije mogu biti 1) na strani servera i 2) na strani klijenta. U tom dijelu smo pokazali kako preuzeti podatke koji su procesuirani na strani servera koristeći **rvest** paket. Ta se tehnika fokusira na CSS selektore [(i SelectorGadget)](http://selectorgadget.com/) i HTML tagove. Također smo vidjeli da webscraping nije egzaktna zanost već dijelom i umjetnost! Mnoštvo CSS opcija i fleksibilnost HTML-a čine preuzimanje podataka specifičnim za svaku pojedinu stranicu i vrijeme. To ne znači da opći principi ne funkcioniraju!

Fokus ovog dijela predavanja je druga kategorija: preuzimanje podataka koji su procesuirani na strani klijenta (**client-side**). Ovaj je pristup u najvećem broju slučajeva jednostavniji način za preuzimanje podataka sa web-a.To također ne znači da određena nijansa umjetnosti nije potrebna! Još jednom valja naglasiti **etičke** i **zakonske** aspekte preuzimanja web sadržaja...pogledajte u prvom dijelu predavanja!

## Strana klijenta (*Client-side*), API i API "izvori"


Webstranice i aplikacije koje su procesuirane na **strani klijenta**  zahtijevaju sljedeći pristup:

- Posjetite URL koji koristi predložak statičkog sadržaja (HTML tablice, CSS, itd). Taj predložak se sadržava podate "u sebi".
- U procesu otvaranja URL-a, browser šalje zahtijev (*request*) na (host) server.
- Ukoliko je zahtjev ispravan (*valid*), server će vratiti odgovor (*response*) koji poziva tražene podatke i dinamički ih procesuirati u browseru.
- Stranica koju vidite u browseru je stoga mješavina statičkog sadržaja (predloška) i dinamički generiranih informacija koje su procesuirane u browseru (i.e. *klijent*).

Cijelokupni proces slanja zahtjeva, odgovora i procesuiranja se odvija kroz **API** (or **A**pplication **P**rogram **I**nterface) (host) aplikacije.

### API

Ako prvi put čujete za API, pogledajte Zapier- ov pregled: [An Introduction to APIs](https://zapier.com/learn/apis/). Pregled je opširan ali ne morate proći sve detalje...Zaključak je da API predstavlja skup pravila i metoda koje omogućavaju interakciju različitih software-skih aplikacija u razmjeni informacija. To se ne odnosi samo na web servere i browser-e nego i na npr. neke pakete koje smo sveć koristili u ovom kolegiju.^[Zanimljivost: Brojn i paketi koje ćemo koristiti u ovom kolegiju (npr. **leaflet**, **plotly**, etc.) su samo skup *wrapper* funkcija koje komuniciraju sa API-jima i pretvaraju R kod u neki drugi jezik (npr. JavaScript).] Ključni koncepti su:

- **Server:** Računalo koje izvršava (*engl. run*).
- **Klijent:** Program koji izmjenjuje podatke sa serverom kroz API.
- **Protokol:** "Bonton" koji odrđuje način međusobne interakcije između računala (npr. HTTP).
- **Metode:** "Naredbe" koje klijenti koriste za komunikaciju sa serverom. Glavna naredba koju ćemo koristiti u ovom dijelu predavanja je  `GET` (i.e. "zamoli" server za informaciju), a neke druge su `POST`, `PUT` i `DELETE`.
- **Zahtjevi:** Ono što klijent traži od servera (vidi Metode!).
- **Odgovor:** Odgovor servera. Odgovor uključuje *Status Code* (npr. "404" *if not found*, ili "200" *if successfu*l),  *Header* (i.e. meta-informacije o odgovoru) i *Body* (i.e sadržaj onoga što zahtjevamo od servera).
- Etc.

### API "izvori"

Ključna točka u razumijevanju API-ja je da možemo pristupiti informacijama *direktno* iz API baze podataka ukoliko na ispravan način specificiramo URL-ove. Ti URL-ovi su ono što nazivamo API izvorima (*engl. API endpoints*).

API izvori su u mnogočemu sličini normalnim web URL-ovima koje stalno posjećjemo. Za početak, možete navigirati do njih u browser-u. Iako "normalne" web stranice prikazuju informacije u HTML kontekstu --- slike, video, GFI itd. --- API izvori su znatno manje "lijepi". Navigirajte browser do API izvora i vidjeti ćete samo hrpu neformatiranog teksta. Ono što uistinu vidite je najvjerojatnije [JSON](https://en.wikipedia.org/wiki/JSON) (**J**ava**S**cript **O**bject **No**tation) ili [XML](https://en.wikipedia.org/wiki/XML) (E**x**tensible **M**arkup **L**anguage). 

Sintaksa tih jezika (JSON i XML) vas ne treba zabrinjavati. Važno je da objekt u vašem browser-u --- koji učitava hrpu nestrukturiranog teksta --- zapravo ima vrlo precizno definiranu strukturu (i format). Taj objekt akođer sadžava informacije (podatke) koje je moguće vrlo jednostavno učitati u R (ili Python, Julia, etc.)). Potrebno je samo znati točan API izvor za podatke koje želimo!

Vrijeme je za nekoliko primjera...Započeti ćemo sa najjednostavnijim slučajem (bez API ključa i sa eksplicitnim API izvorom), a nakon toga nastaviti sa složenijim primjerima.


## Praktični primjer 1: Drveće u New York-u

[NYC Open Data](https://opendata.cityofnewyork.us/) je zanimljiva inicijativa. Njezina misija je *"make the wealth of public data generated by various New York City agencies and other City organizations available for public use*". Podatci koje možete preuzeti uključuju uhićenja, lokacije wifi spotova, oglase za posao, broj beskućnika, licenci za pse, popis wc-a u javnim parkovima...POgledajte popis dostupnih podataka kada stignete! U ovom primjeru ćemo preuzeti podatke o drveću [**2015 NYC Street Tree Census**](https://data.cityofnewyork.us/Environment/2015-Street-Tree-Census-Tree-Data/uvpi-gqnh).

Ovaj izvor koristimo na početku pošto nije potrebno postaviti API ključ unaprijed

I wanted to begin with an example from NYC Open Data, because you don't need to set up an API key in advance.^[Ipak, da biste smanjili opterećenje na server --- i.e. broj zahtjeva u vremenu --- najbolje je napraviti [registraciju](https://data.cityofnewyork.us/profile/app_tokens) za **NYC Open Data app token**. Za svrhe ovog predavanja ćemo napraviti jedan ili dva zahtjeva na server pa registracija nije potrebna.] Podatke ćemo preuzeti u nekoliko koraka:

- Otvorite [web stranicu](https://data.cityofnewyork.us/Environment/2015-Street-Tree-Census-Tree-Data/uvpi-gqnh) u browseru. 
- Odmah ćete vidjeti **API** tab. Kliknite na njega!. 
- Kopirajte [API izvor](https://data.cityofnewyork.us/resource/nwxe-4ae8.json) koji se pojavi u *pop-up* prozoru. 
- *Doatano:* Zalijepite taj izvor u novi tab u browser-u. Pojaviti će se hrpa JSON teksta, koji možete procesuirati sa JSONView browser ekstenziji koju smo instalirali na početku predavanja.

Pogledajte animirani prikaz:

![](../Foto/trees.gif)
Sada kada smo locirali API izvor, možemo učitati podatke u R. To ćemo napraviti pomoću`fromJSON()` funkcije iz **jsonlite** paketa([vidi!](https://cran.r-project.org/web/packages/jsonlite/index.html)). To će automatski prisiliti JSON *array* (vrsta objetka;slično kao data frame) objekt u R data frame. Ovdje ćemo taj data frame pretvoriti u tibble (pogledajte prethodno predavanje o manipulaciji sa podatcima) zbog funkcionalnosti koje taj objekt pruža.


```r
# library(jsonlite) ## već učitano
nyc_trees <- 
  fromJSON("https://data.cityofnewyork.us/resource/nwxe-4ae8.json") %>%
  as_tibble()
nyc_trees
```

```
## # A tibble: 1,000 x 45
##    tree_id block_id created_at tree_dbh stump_diam curb_loc status health
##    <chr>   <chr>    <chr>      <chr>    <chr>      <chr>    <chr>  <chr> 
##  1 180683  348711   2015-08-2~ 3        0          OnCurb   Alive  Fair  
##  2 200540  315986   2015-09-0~ 21       0          OnCurb   Alive  Fair  
##  3 204026  218365   2015-09-0~ 3        0          OnCurb   Alive  Good  
##  4 204337  217969   2015-09-0~ 10       0          OnCurb   Alive  Good  
##  5 189565  223043   2015-08-3~ 21       0          OnCurb   Alive  Good  
##  6 190422  106099   2015-08-3~ 11       0          OnCurb   Alive  Good  
##  7 190426  106099   2015-08-3~ 11       0          OnCurb   Alive  Good  
##  8 208649  103940   2015-09-0~ 9        0          OnCurb   Alive  Good  
##  9 209610  407443   2015-09-0~ 6        0          OnCurb   Alive  Good  
## 10 192755  207508   2015-08-3~ 21       0          OffsetF~ Alive  Fair  
## # ... with 990 more rows, and 37 more variables: spc_latin <chr>,
## #   spc_common <chr>, steward <chr>, guards <chr>, sidewalk <chr>,
## #   user_type <chr>, problems <chr>, root_stone <chr>, root_grate <chr>,
## #   root_other <chr>, trunk_wire <chr>, trnk_light <chr>, trnk_other <chr>,
## #   brch_light <chr>, brch_shoe <chr>, brch_other <chr>, address <chr>,
## #   zipcode <chr>, zip_city <chr>, cb_num <chr>, borocode <chr>,
## #   boroname <chr>, cncldist <chr>, st_assem <chr>, st_senate <chr>, nta <chr>,
## #   nta_name <chr>, boro_ct <chr>, state <chr>, latitude <chr>,
## #   longitude <chr>, x_sp <chr>, y_sp <chr>, council_district <chr>,
## #   census_tract <chr>, bin <chr>, bbl <chr>
```

**Komentar:** Primjetite da puni podatkovni skup sadržava skoro 700.000 drveća. U ovonm primjeru ćemo preuzeti samo mali dio toga, zbog *default* postavke API limita od 1000 redova. Ionako nije potrebno preuzeti sve podatke u ovom demonstrativnom primjeru. Važno je znati ([pročitajte API upute](https://dev.socrata.com/docs/queries/limit.html)) da ostatak podataka možete preuzeti tako da dodate `?$limit=LIMIT` u API izvor. Da biste učitali prvih pet redova:


```r
## !izvrši
fromJSON("https://data.cityofnewyork.us/resource/nwxe-4ae8.json?$limit=5")
```

Vratimo se primjeru sa drvećem i vizualizirajmo podatke koje smo preuzeli. Još jedan dodatni komentar je da `jsonlite::fromJSON()` funkcija automatski pretvara sav sadržaj u string (character ) pa je potrebno promijeniti kolone u numeričke prije vizualizacije.


```r
nyc_trees %>% 
  select(longitude, latitude, stump_diam, spc_common, spc_latin, tree_id) %>% 
  mutate_at(vars(longitude:stump_diam), as.numeric) %>% 
  ggplot(aes(x=longitude, y=latitude, size=stump_diam)) + 
  geom_point(alpha=0.5) +
  scale_size_continuous(name = "Stump diameter") +
  labs(
    x = "Longituda", y = "Latituda",
    title = "Uzorak drveća u New York-u",
    caption = "Izvor: NYC Open Data"
    )
```

![](06_WEBSCRAP_I_files/figure-html/nyc3-1.png)<!-- -->

Ovo bi bilo zanimljivje kada bi mapa sadržavala prave geolokacijske podatke New York-a. Prostorna analiza je zanimljivo istraživačko područje, a vizualizacija tog tipa bi zahtijvela zasabno predavanje...

Cilj ovog praktičnog primjera je pokazazati kako za preuzimanje API podataka nije uvijek potrebna registracija. To nije tipično jer najveći broj API sučelja dozvoljava pristup isključivo nakon registracije (*potreban je ključ!*). To je u najvećem broju slučajeva kada preuzimate javne podatke ( npr. sudski registar, HNB, etc.). Pogledajmo sada jedan primjer gdje je potrebna API registracija...


## Praktični primjer 2: FRED podatci

Drugi primjer uključuje preuzimanje podataka sa [**FRED API**](https://research.stlouisfed.org/docs/api/fred/). Potrebna je [registracija API ključa](https://research.stlouisfed.org/useraccount/apikey) ako želite provesti primjer sami. 


[FRED](https://fred.stlouisfed.org/) je ekonomska baza Federalne banke u St. Louis-u, podružnice Američke središnje banke. Stranica ima ugrađene alate za vizualizaciju podataka [poput ovog](https://fred.stlouisfed.org/series/GNPCA#0) za seriju BDP/per capita SAD-a od 1929. godine.

<iframe src="https://fred.stlouisfed.org/graph/graph-landing.php?g=mPCo&width=670&height=475" scrolling="no" frameborder="0"style="overflow:hidden; width:670px; height:525px;" allowTransparency="true"></iframe>


<iframe src="http://rpsychologist.com/d3/NHST/" scrolling= "yes"></iframe>

</br>

U ovoj aplikaciji ćemo preuzeti podatke na osnovi kojih je napravljen prethodni grafikon kroz **FRED API** kako biste naučili što se događa "u pozadini". Nakon toga ćemo preuzeti iste podatke kroz specifičan R paket za tu svrhu.


### Napravite sami

Njabolje je započeti sa [FRED API dokumentacijom](https://research.stlouisfed.org/docs/api/fred/). Na osnovi pročitanog je moguće zaključiti da se pripadajući API izvor nalazi pod [**series/observations**](https://research.stlouisfed.org/docs/api/fred/series_observations.html). 
Ovaj izvor "*gets the observations or data values for an economic data series*". U API dokumentaciji sa linka ćete pronaći više detalja, uključujući i parametre koji su dozvoljeni u API pozivu.^[API *parameteri* su nešto kao funkcijski *argumenti*. To su imputi (upute) koji karakteriziraju API zahtjev.] Parametri koje ovdje koristimo (i.e. kalibriramo):

- **file_type:** "json" (Nije nužno ali želimo JSON output.)
- **series_id:** "GNPCA" (Nužno. Serija koju tražimo.)
- **api_key:** "YOUR_API_KEY" (Nužno. Sada dohvatite vaš ključ.)

Sada ćemo podesiti ove parametreprema izvoru kako bismo pogledali podatke direktno u browser-u. Unesite [https://api.stlouisfed.org/fred/series/observations?series_id=GNPCA&api_key=<mark>YOUR_API_KEY</mark>&file_type=json](https://api.stlouisfed.org/fred/series/observations?series_id=GNPCA&api_key=YOUR_API_KEY&file_type=json), uz vaš specifični "YOUR_API_KEY". Trebali biste vidjeti nešto poput ovoga:

![](../Foto/fred-redacted.png)

U ovom trenutku zasigurno želite učitati JSON objekt direktno u radni prostor R koristeći `jsonlite::readJSON()` funkciju. To bi ujedno i funkcioniralo! Ipak, ovdje ćemo koristiti **httr** [paket](https://httr.r-lib.org/). Zašto? Zbog toga što **httr** ima mnoštvo funkcionalnosti koje omogućuju fleksibilnu i sigurnu interakciju sa web API-jem. 

Definirajmo prvo neke verijable poput puta (*path*) do API izvora i pripadajućih parametara. Spremiti ćemo sve u list-e.



```r
endpoint = "series/observations"
params = list(
  api_key= "YOUR_FRED_KEY", ## Unesite svoj ključ
  file_type="json", 
  series_id="GNPCA"
  )
```

Potom ćemo koristiti `httr::GET()` funkciju za zahtijev (i.e. download) podataka. To ćemo pripisati objektu `fred`.


```r
# library(httr) ## Učitano
fred <- 
  httr::GET(
    url = "https://api.stlouisfed.org/", ## Osnovni URL
    path = paste0("fred/", endpoint), ## API izvor
    query = params ## Popis parametara
    )
```

Pogkledajte`fred` objekt u konzoli. To što vidite je pravi API odgovor (i.e. response), uključujući *Status Code* i *Content*. Otprilike ovako::

```
## Response [https://api.stlouisfed.org/fred/series/observations?api_key=YOUR_API_KEY&file_type=json&series_id=GNPCA]
##   Date: 2020-11-11 00:06
##   Status: 200
##   Content-Type: application/json; charset=UTF-8
##   Size: 9.09 kB
```

Da bismno uzeli sadržaj (i.e. podatke)  iz ovog odgovora, koristiti ćemo `httr::content()` funkciju. Pošto već znamo da je riječ o JSON array-u, nožemo koristiti `jsonlite::fromJSON()` kao u prethodnom slučaju. Možemo očekivati da će taj objekt u R biti učitan kao lista, a za provjeru je moguće koristiti`str()` funkciju za provjeru objekta. Ovdje ima smisla ukazati na **listviewer**  [paket](https://github.com/timelyportfolio/listviewer) ::jsonedit()`koji omogućava interaktivni pregled podataka.^[Ugnježđene liste (*engl.nested lists*) su karakteristika JSON podataka. Ovo nije previše važno jer R ima podršku za procesuiranje takvih formata.]

```r
fred %>% 
  httr::content("text") %>%
  jsonlite::fromJSON() %>%
  listviewer::jsonedit(mode = "view")
```

<!--html_preserve--><div id="htmlwidget-63233e32b05948c14d17" style="width:100%;height:10%;" class="jsonedit html-widget"></div>
<script type="application/json" data-for="htmlwidget-63233e32b05948c14d17">{"x":{"data":{"realtime_start":"2020-11-11","realtime_end":"2020-11-11","observation_start":"1600-01-01","observation_end":"9999-12-31","units":"lin","output_type":1,"file_type":"json","order_by":"observation_date","sort_order":"asc","count":91,"offset":0,"limit":100000,"observations":{"realtime_start":["2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11"],"realtime_end":["2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11","2020-11-11"],"date":["1929-01-01","1930-01-01","1931-01-01","1932-01-01","1933-01-01","1934-01-01","1935-01-01","1936-01-01","1937-01-01","1938-01-01","1939-01-01","1940-01-01","1941-01-01","1942-01-01","1943-01-01","1944-01-01","1945-01-01","1946-01-01","1947-01-01","1948-01-01","1949-01-01","1950-01-01","1951-01-01","1952-01-01","1953-01-01","1954-01-01","1955-01-01","1956-01-01","1957-01-01","1958-01-01","1959-01-01","1960-01-01","1961-01-01","1962-01-01","1963-01-01","1964-01-01","1965-01-01","1966-01-01","1967-01-01","1968-01-01","1969-01-01","1970-01-01","1971-01-01","1972-01-01","1973-01-01","1974-01-01","1975-01-01","1976-01-01","1977-01-01","1978-01-01","1979-01-01","1980-01-01","1981-01-01","1982-01-01","1983-01-01","1984-01-01","1985-01-01","1986-01-01","1987-01-01","1988-01-01","1989-01-01","1990-01-01","1991-01-01","1992-01-01","1993-01-01","1994-01-01","1995-01-01","1996-01-01","1997-01-01","1998-01-01","1999-01-01","2000-01-01","2001-01-01","2002-01-01","2003-01-01","2004-01-01","2005-01-01","2006-01-01","2007-01-01","2008-01-01","2009-01-01","2010-01-01","2011-01-01","2012-01-01","2013-01-01","2014-01-01","2015-01-01","2016-01-01","2017-01-01","2018-01-01","2019-01-01"],"value":["1120.076","1025.091","958.378","834.291","823.156","911.019","992.537","1118.944","1177.572","1138.989","1230.22","1337.075","1574.74","1870.911","2187.818","2361.622","2337.63","2068.966","2048.293","2134.291","2121.201","2305.668","2493.148","2594.934","2715.067","2700.542","2893.97","2957.097","3020.083","2994.683","3201.683","3285.454","3371.35","3579.446","3736.061","3951.902","4208.08","4481.593","4604.613","4831.761","4980.667","4989.534","5156.41","5428.368","5746.389","5723.068","5697.677","6011.215","6293.525","6637.838","6868.092","6849.819","7011.223","6889.371","7199.441","7711.063","8007.532","8266.358","8549.125","8912.281","9239.186","9425.052","9406.669","9734.705","10000.831","10389.663","10672.832","11076.879","11556.745","12064.59","12647.632","13179.965","13327.458","13553.208","13953.961","14503.006","15006.043","15398.622","15748.3","15771.553","15359.37","15803.886","16081.66","16429.308","16722.335","17146.51","17647.607","17955.437","18421.034","18951.897","19338.371"]}},"options":{"mode":"view","modes":["code","form","text","tree","view"]}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

OPreuzeti objekt nije posebno složen. Ono što nas zanima je `fred$observations` pod-element. Sada ćemo izvršiti gornji kod i izvući element od interesa. To je moguće napraviti na više načina ali ovdje ćemo koristiti `purrr::pluck()` funkciju.


```r
fred <-
  fred %>% 
  httr::content("text") %>%
  jsonlite::fromJSON() %>%
  purrr::pluck("observations") %>% ## izvuci"$observations" element iz liste
  # .$observations %>% ## Alternativno...
  # magrittr::extract("observations") %>% ## Alternativno...
  as_tibble() ## Zbog formatiranja
fred
```

```
## # A tibble: 91 x 4
##    realtime_start realtime_end date       value   
##    <chr>          <chr>        <chr>      <chr>   
##  1 2020-11-11     2020-11-11   1929-01-01 1120.076
##  2 2020-11-11     2020-11-11   1930-01-01 1025.091
##  3 2020-11-11     2020-11-11   1931-01-01 958.378 
##  4 2020-11-11     2020-11-11   1932-01-01 834.291 
##  5 2020-11-11     2020-11-11   1933-01-01 823.156 
##  6 2020-11-11     2020-11-11   1934-01-01 911.019 
##  7 2020-11-11     2020-11-11   1935-01-01 992.537 
##  8 2020-11-11     2020-11-11   1936-01-01 1118.944
##  9 2020-11-11     2020-11-11   1937-01-01 1177.572
## 10 2020-11-11     2020-11-11   1938-01-01 1138.989
## # ... with 81 more rows
```


U redu! Sada smo povukli podatke i sve je spremno za vizualizaciju. Sjetite se da `jsonlite::fromJSON()` sve automatski prebacije u stringove pa ćemo prilagoditi datume (koristeći `lubridate::ymd()`) i prebaciti neke kolone u numeričke.


```r
# library(lubridate) ## Već učitano
fred <-
  fred %>%
  mutate_at(vars(realtime_start:date), ymd) %>%
  mutate(value = as.numeric(value)) 
```

Konačno...vizualizacija!


```r
fred %>%
  ggplot(aes(date, value)) +
  geom_line() +
  scale_y_continuous(labels = scales::comma) +
  labs(
    x="Datum", y="2012 USD (Milijarde)",
    title="Realni BDP u SAD", caption="Izvor: FRED"
    )
```

![](06_WEBSCRAP_I_files/figure-html/fred6-1.png)<!-- -->



### Doatak primjeru: Spremite API ključeve kao varijable u radnom prostoru


U gornjem primjeru je bilo potrebno unijeti vaš osobni "YOUR_FRED_KEY" API ključ. To nije baš sigurno jer implicira otkrivanje vašeg (privatnog) ključa.^[Isto vrijedi i za kompilirane R Markdown dokumente poput ovih predavanjas.] Postoji sigurniji način za rad sa API ključevim i lozinkama: Jednostavno ih spremite kao R [**environment variables**](https://stat.ethz.ch/R-manual/R-devel/library/base/html/EnvVar.html). Dva su načina:

1. Postavite *environment variabl* za tekuću R sesiju (*session*).
2. Postavite *environment variable* za sve R sesije.

Pregledajmo svaku opciju.

#### 1) Postavite *environment variabl* samo za tekuću R sesiju

Definiranje *environment variable* za tekuću R sesiju je jednostavno. Koristite `Sys.setenv()` funkciju iz base R, npr.:


```r
## postavite novu environmet variable MY_API_KEY. Samo za tekuću sesiju.
Sys.setenv(MY_API_KEY="xyxyzaqzazsda13e3243") 
```

Once this is done, you can then safely assign your key to an object --- including within an R Markdown document that you're going to knit and share --- using the `Sys.getenv()` function. For example:


```r
## Pripišite environment variable R objektu
my_api_key <- Sys.getenv("MY_API_KEY")
## Pregled
my_api_key
```

```
## [1] "xyxyzaqzazsda13e3243"
```

**Važno:** Iako je ovo jednostavno, valja primijetiti da `Sys.setenv()` dio trebate izvršiti u konzoli. *Nikada* nemojte uključivati osjetljive podatke poput `Sys.setenv()` poziva u R Markdown ili druge dokomente koje dijelite.^[Pošto je nova environment variable definirana samo u trajanju tekuće sesije R Markdown neće imati pristup do nje osim ako nije eksplicitno zadržana u skripti unutar dokumenta.] Cilj je sakriti ovakve podatke!Alternativna opcija isključuje da se sistemske varijable unose svaki put kada otvorite R. Ovo može biti posebno korisno za API izvore koje često koristite.

#### 2) Postavite *environment variable* za sve R sesije

Postavljanje R environment variable koja je dostupna u svim sesijama uključuje manipulaciju  `~/.Renviron` file-a. To je tekstualni file u vašem *home drectoriju* --- primjetite `~/` path --- koji R automatski učitava kada se podiže. Pošto je `~/.Renviron` obični tekstualni file, možete ga prilagoditi u bilo kojem text editor-u. Ipak, možda taj file morate prvo stvoriti, u slučaju da ga već nemate. Način za to u RStudio-u je korištenje `usethis::edit_r_environ()` funkcije. Izvršite sljdeći kod interaktivno:


```r
## otvorite .Renviron file. Ovdje dodajemo API ključeve koji vrijede u svim R sesijama.
usethis::edit_r_environ() 
```

Ovo će otvoriti vaš `~/.Renviron` file u novom RStudio prozoru, koji onda možete prilagoditi po potrebi. Postavite za primjer vaš FRED API ključ kao environment variable koja vrijedi u svim sesijama. Jednostavo dodajte donju liniju u  `~/.Renviron` file i spremite ga.^[Prijedlog je koristit neki intuitivan naziv poput "FRED_API_KEY".]

```
FRED_API_KEY="abcdefghijklmnopqrstuvwxyz0123456789" ## Zamijenite sa vašim ključem.
```

Kada ste spremili promjene, potrebno je restartati R sesiju tako da nova varijabla postane dostupna u svim sljedećim sesijama.


```r
## Opcionalno: Osvježite .Renviron file.  
readRenviron("~/.Renviron") ## Potrebno samo ako učitavate u novi R environment
```

**Izazov:** Kada ste podesili vaš `~/.Renviron` file, pokušajte preuzeti FRED podatke od prije... Ovaj put pozovite vaš FRED API kluč kao environment variable u vašoj listi parametara koristeći `Sys.getenv()`na ovaj način:


```r
params = list(
  api_key= Sys.getenv("FRED_API_KEY"), ## uzmi API direktno kao environment variable
  file_type="json", 
  series_id="GNPCA"
  )
```

Environment variable su važne za interakciju sa *cloud*-om pa je korisno znati raditi s njima.

### Koristite paket

Sjana stvar kod R je da vjerojatno postoji paket za to....! Ovdje vrijedi istaknuti **fredr** [paket](http://sboysel.github.io/fredr/index.html). Pokušajte sada put iste podatke preuzeti kroz paket!







