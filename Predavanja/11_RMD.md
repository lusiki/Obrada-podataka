---
title: "Obrada podataka"
author:
  name: Luka Sikic, PhD
  affiliation: Fakultet hrvatskih studija | [OP](https://github.com/BrbanMiro/Obrada-podataka)
subtitle: 'Predavanje 11: Komunikacija i dijeljenje rezultata'
output:
  html_document:
    code_folding: hide
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




## Software podrška

U ovom predavanju se pretpostavlja da ste već upoznati sa osnovama R programskog jezika i ekosistema te da imate instaliran R + R Studio (ili alternativno [Visual Studio Code](https://code.visualstudio.com/)). Ukoliko su ti uvjeti zadovoljeni, za rad sa [R Markdown](https://vimeo.com/178485416)-om je potrebno instalirati još nekoliko paketa: 

- [tinytex](https://yihui.name/tinytex/)

`install.packages(c('tinytex','rmarkdown'))`
`tinytex::install_tinytex()`

- nekoliko paketa koji su neophodni ili olakšavaju rad u R Markdown-u

`install.packages(c("rmarkdown", "knitr", "kableExtra","stargazer", "plotly", "knitr","bookdown"))`
