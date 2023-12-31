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

temp2023 <- read.csv("2023-av-temp2.csv", skip = 4)



precip2019 <- read.csv("2019-precip.csv", skip = 4)

precip2020 <- read.csv("2020-precip.csv", skip = 4)

precip2021 <- read.csv("2021-precip.csv", skip = 4)

precip2022 <- read.csv("2022-precip.csv", skip = 4)

precip2023 <- read.csv("2023-precip2.csv", skip = 4)


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







note: FOR CLEANING THE WNV DATA
note: Load the necessary libraries

library(dplyr)
library(readr)

note: Load the data

wnv_data <- read_csv('WNV data request.csv')

note: Remove rows with NA values in 'County' or 'Diagnosis Year' columns

wnv_data <- na.omit(wnv_data, cols = c("County", "Diagnosis Year"))


note: Select only the 'County' and 'Diagnosis Year' columns

wnv_data <- select(wnv_data, County, `Diagnosis Year`)

note: Create a new column 'Human Cases' that contains the count of each duplicate
wnv_data <- wnv_data %>% 
  mutate(`Human Cases` = 1) %>% 
  group_by(County, `Diagnosis Year`) %>% 
  summarise(`Human Cases` = sum(`Human Cases`))


note: List of all Wyoming counties
wy_counties <- c('Albany', 'Big Horn', 'Campbell', 'Carbon', 'Converse', 'Crook', 'Fremont', 'Goshen', 'Hot Springs', 'Johnson', 'Laramie', 'Lincoln', 'Natrona', 'Niobrara', 'Park', 'Platte', 'Sheridan', 'Sublette', 'Sweetwater', 'Teton', 'Uinta', 'Washakie', 'Weston')


note: List of all years in the data
years <- unique(wnv_data$`Diagnosis Year`)

note: Insert missing Wyoming counties for each year and give them a value of 0 for 'Human Cases'
for (county in wy_counties) {
  for (year in years) {
    if (!any(wnv_data$County == county & wnv_data$`Diagnosis Year` == year)) {
      wnv_data <- rbind(wnv_data, data.frame(County = county, `Diagnosis Year` = year, `Human Cases` = 0))
    }
  }
}

note: Sort the dataframe by 'County' and 'Diagnosis Year'

wnv_data <- arrange(wnv_data, County, `Diagnosis Year`)

note: Rename the columns

colnames(wnv_data) <- c('Name', 'year', 'Cases')

note: Add the word 'County' after each county in the 'Name' column

wnv_data$Name <- paste(wnv_data$Name, 'County')


note: read in wnv data

all_wnv_data <- read.csv("wnv_data_2.csv")

note: combine all data

all_data <- merge(all_climate_data, all_wnv_data, by = c("Name", "year"))

total_cases <- group_by(all_data, year) %>% 
  summarise(totalcases = sum(Cases))



ggplot(data = all_data) + 
  geom_smooth(mapping = aes(x = year, y = Cases, color = Name))
  
note:get rid of zeros

filtered_counties <- group_by(all_data, Name)%>%
  filter(any(Cases != 0))

jit_filt_counties <- mutate(filtered_counties, Cases_jit = jitter(Cases, factor = 1))

ggplot(data = jit_filt_counties) + 
  geom_smooth(mapping = aes(x = year, y = Cases_jit, color = Name))+
  labs(x = "Year", y = "Cases", color = "WY Counties")

note:conversion factor for plotting purposes

conv <- 40
ggplot(data = all_data) + 
  geom_smooth(data = total_cases, mapping = aes(x = year, y = totalcases, color = "Cases")) +
  geom_smooth(mapping = aes(x = year, y = Temp - conv, color = "Temp")) +
  scale_y_continuous(sec.axis = sec_axis(~.+conv, name = "Temp F"))

ggplot(data = all_data) +
  geom_point(mapping = aes(x = Temp, y = Cases))

map_sf <- us_counties()
ggplot(data = map_sf) +
  geom_sf(fill = "grey") +
  coord_sf() +
  theme_minimal()

map_wy <- map_sf[map_sf$state_abbr == "WY",]

data_2023 <-all_data %>% filter(year == 2023)

map_2023 <- merge(map_wy, data_2023, by.y = "Name", by.x = "namelsad")

ggplot(data = map_2023) +
  geom_sf(fill = "grey") +
  coord_sf() +
  theme_minimal()

centroids <- st_centroid(map_2023$geometry)
centroids_df <- as.data.frame(st_coordinates(centroids))
centroids_df$Cases <- map_2023$Cases

ggplot(data = map_2023) +
  geom_sf(mapping = aes(fill = Temp)) +
  geom_text(data = centroids_df, aes(x = X, y = Y, label = Cases))+
  coord_sf() +
  theme_minimal()+
  scale_fill_gradient(low = "blue2", high = "tomato", name = "Average Temp")+
  labs(x = NULL, y = NULL)


