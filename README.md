# DENV-metadata-dates
un código de R para configurar fechas a utilizar en estudios de epidemiología molecular

1 asd
```r

## 1: configurar el directorio ##

setwd("C:/Users/USUARIO/Documents/dengue_2024/compilacion_2024/analisis")
getwd()
as.data.frame(dir())
```

## 2: leer el archivo de metadata ##

d1 <- read.csv("DENV1/denv1_metadata_2.tsv", header=T, sep="\t")
head(d1)

## 3: leer el archivo de los tips seteados en el árbol filogenético generado ##

d1t <- read.csv("DENV1/denv1_tips.txt", header=F)
head(d1t)
dim(d1t)

## 4: identificar las entradas de la metadata compatibles con los tipos de árbol obtenido ##

d1.2 <- d1[d1$strain %in% d1t$V1, ]
dim(d1.2)

setdiff(d1t$V1,d1.2)
setdiff(d1.2,d1t$V1)

head(d1.2)
sort(unique(d1.2$date))

## 5: identificar las entradas que no presentan fechas de muestreo  o presentan fechas en otro formato ##

library(lubridate)

r1 <- grep("/",d1.2$date,value = F)
r2 <- grep("/",d1.2$date,value = F, invert = T)
d1.3 <- d1.2[r1,]
d1.4 <- d1.2[r2,]
d1.3$date <- dmy(d1.3$date)
head(d1.3)
head(d1.4)
tail(d1.4)

d1.5 <- rbind(d1.3, d1.4)
dim(d1.5)
tail(d1.5)
c <- d1.5

## 6: estimar fechas en formato Y-M-D ##
library(tidyr)
we <- separate(c,"date",c("year2","month","day"), sep="-")
head(we)
dim(we)
tail(we)

## 7: identificar las columnas con fechas incompletas ##

we1 <- we[we$year2 %in% NA, ]
we1$year2 <- we1$year
we2 <- we[!we$year2 %in% NA, ]
we3 <- rbind(we1,we2)
dim(we3)
head(we3)

## 8: correr en siguiente comando para rellenar las fechas faltantes ##

## format : 2022-NA-NA ##
x <- we3[we3$month %in% NA & we3$day %in% NA, ]
## format : 2022-01-NA ##
y <- we3[!we3$month %in% NA & we3$day %in% NA, ]
## format : 2022-01-15 ##
z <- we3[!we3$month %in% NA & !we3$day %in% NA, ]

x$month <- ifelse(x$month %in% NA , "06","")
x$day <- ifelse(x$day %in% NA , "15","")
y$day <- ifelse(y$day %in% NA , "15","")

as.data.frame(names(x))
x1 <- data.frame(x[,1:6],date=paste0(x$year,"-",x$month,"-",x$day),x[,10:16])
y1 <- data.frame(y[,1:6],date=paste0(y$year,"-",y$month,"-",y$day),y[,10:16])
z1 <- data.frame(z[,1:6],date=paste0(z$year,"-",z$month,"-",z$day),z[,10:16])

complete <- rbind(x1,z1)
dim(complete)
head(complete)

## 9: finalmente generar la tabla final con la data configurar ##

write.table(complete,"DENV1/denv1_metadata_3.tsv", row.names=FALSE, sep="\t", quot=F)
