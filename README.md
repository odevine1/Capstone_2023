# Capstone_2023
Class MICR-4321-01 Microbiology Capstone
Group Culex Cartographers 
Contributors Olivia Devine (odevine1) Jena Tozzi (jenatozzi) Duke Dickson (DukeD22)
Community partners Wyoming Public Health, Teton Weed and Pest, Sean Harrington, Erin Bentley

Final R Script

temp2019 <- read.csv("2019-av-temp.csv", skip = 4)

temp2020 <- read.csv("2020-av-temp.csv", skip = 4)

temp2021 <- read.csv("2021-av-temp.csv", skip = 4)

temp2022 <- read.csv("2022-av-temp.csv", skip = 4)

temp2023 <- read.csv("2023-av-temp.csv", skip = 4)



precip2019 <- read.csv("2019-precip.csv", skip = 4)

precip2020 <- read.csv("2020-precip.csv", skip = 4)

precip2021 <- read.csv("2021-precip.csv", skip = 4)

precip2022 <- read.csv("2022-precip.csv", skip = 4)

precip2023 <- read.csv("2023-precip.csv", skip = 4)


note: create year column to be able to combine data, double brackets define column name

temp2019[["year"]] <- 2019

temp2020[["year"]] <- 2020

temp2021[["year"]] <- 2021

temp2022[["year"]] <- 2022

temp2023[["year"]] <- 2023

precip2019[["year"]] <- 2019

precip2020[["year"]] <- 2020

precip2021[["year"]] <- 2021

precip2022[["year"]] <- 2022

precip2023[["year"]] <- 2023

note: combine files by temp and precip, only 1 for temp, 1 for precip

tempcombine <- rbind(temp2019, temp2020, temp2021, temp2022, temp2023)

precipcombine <- rbind(precip2019, precip2020, precip2021, precip2022, precip2023)

note: get rid of columns we dont need 

tempreduced <- tempcombine[,c("Name", "Value", "year")]

precipreduced <- precipcombine[,c("Name", "Value", "year")]

note: rename columns so we can combine data

colnames(tempreduced) <- c("Name", "Temp", "year")

colnames(precipreduced) <- c("Name", "Precip", "year")

all_climate_data <- merge(tempreduced, precipreduced, by = c("Name", "year"))

