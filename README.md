# Capstone Project README

## Overview
This README provides an introduction to the Capstone Project for the Google Analytics Certification Training completed in May 2023. The project focuses on analyzing Divvy bike sharing data using SQL, Excel, and R. The goal of the project is to understand the differences in bike usage patterns between members and casual riders.

## Data Collection and Preparation
The script begins by loading the necessary packages, including `tidyverse`, `lubridate`, and `ggplot2`. The [Divvy bike sharing data](https://divvy-tripdata.s3.amazonaws.com/index.html) for each month from May 2022 to April 2023 is loaded and stored into separate dataframes. To ensure consistency, the data is checked for column name consistency, and any inconsistencies are addressed by renaming the columns.

## Data Wrangling
In the data wrangling phase, all the monthly dataframes are combined into a single dataframe called `all_trips`. The latitude and longitude fields, which are not required for this analysis, are removed. Additional columns are created, including the date, month, day, year, day of the week, and ride length for each ride.

## Data Cleaning
The data is further cleaned by converting the ride length from a factor to a numeric field, enabling numerical calculations. The script also eliminates "bad" data, such as entries with negative ride lengths.

## Descriptive Analysis
The script performs descriptive analysis on the `ride_length` column, providing a summary of the data and comparing member and casual users. Ride lengths are aggregated by user type and day of the week to calculate average ride times. Additionally, the script groups the data by user type and weekday, calculating the number of rides and average duration.

## Data Visualization
To visualize the findings, the script utilizes `ggplot2` to create bar charts that display the number of rides and average ride duration by rider type and weekday.

---

The primary objective of this Capstone Project is to analyze the usage patterns of Divvy bikes among members and casual riders. The script accomplishes this by collecting and preparing the data, performing data wrangling and cleaning, conducting descriptive analysis, and visualizing the results.



    required_packages <- c("lubridate", "tidyverse", "ggplot2", "hms", "flexdashboard", "purrr", "readr")
    for (package in required_packages) {
      if (!require(package, character.only = TRUE)) {
        install.packages(package)
        library(package, character.only = TRUE)
      }
    }


    ## Loading required package: flexdashboard

Get a list of file names in the “data” directory. Read and combine all
CSV files into a single dataframe

    file_list <- list.files("data/", pattern = "\\.csv$", full.names = TRUE)
    all_trips <- bind_rows(lapply(file_list, read_csv))

    ## Rows: 634858 Columns: 13
   ### Data Wrangling:

    colnames(all_trips)

    ##  [1] "ride_id"            "rideable_type"      "started_at"        
    ##  [4] "ended_at"           "start_station_name" "start_station_id"  
    ##  [7] "end_station_name"   "end_station_id"     "start_lat"         
    ## [10] "start_lng"          "end_lat"            "end_lng"           
    ## [13] "member_casual"

Rename columns for better readability. Add columns for dates and time

    all_trips <- all_trips %>%
      rename(
        trip_id = ride_id,
        bikeid = rideable_type,
        start_time = started_at,
        end_time = ended_at,
        from_station_name = start_station_name,
        from_station_id = start_station_id,
        to_station_name = end_station_name,
        to_station_id = end_station_id,
        start_lat = start_lat,
        start_lng = start_lng,
        end_lat = end_lat,
        end_lng = end_lng,
        usertype = member_casual
      ) %>%
      mutate(
        date = as.Date(start_time),
        month = month(date),
        day = day(date),
        year = year(date),
        day_of_week = weekdays(date)
      )

### Data Cleaning:

The data is further cleaned by adding and converting the ride length
from a factor to a numeric field, allowing for numerical calculations.
The script also removes “bad” data, such as entries where the ride
length is negative.

Removing unused columns

    all_trips <- all_trips %>%  
      select(-c(start_lat, start_lng, end_lat, end_lng))

Converting the ride length from a factor to a numeric field

    all_trips$start_time <- ymd_hms(all_trips$start_time)

    ## Warning: 31 failed to parse.

    all_trips$end_time <- ymd_hms(all_trips$end_time)

    ## Warning: 23 failed to parse.

Inspect the new table that has been created

    colnames(all_trips)  #List of column names

    ##  [1] "trip_id"           "bikeid"            "start_time"       
    ##  [4] "end_time"          "from_station_name" "from_station_id"  
    ##  [7] "to_station_name"   "to_station_id"     "usertype"         
    ## [10] "date"              "month"             "day"              
    ## [13] "year"              "day_of_week"

    nrow(all_trips)  #How many rows are in data frame?

    ## [1] 5859061

    dim(all_trips)  #Dimensions of the data frame?

    ## [1] 5859061      14

    head(all_trips)  #See the first 6 rows of data frame.

    ## # A tibble: 6 × 14
    ##   trip_id       bikeid start_time          end_time            from_station_name
    ##   <chr>         <chr>  <dttm>              <dttm>              <chr>            
    ## 1 EC2DE40644C6… class… 2022-05-23 23:06:58 2022-05-23 23:40:19 Wabash Ave & Gra…
    ## 2 1C31AD03897E… class… 2022-05-11 08:53:28 2022-05-11 09:31:22 DuSable Lake Sho…
    ## 3 1542FBEC8304… class… 2022-05-26 18:36:28 2022-05-26 18:58:18 Clinton St & Mad…
    ## 4 6FF598529245… class… 2022-05-10 07:30:07 2022-05-10 07:38:49 Clinton St & Mad…
    ## 5 483C52CAAE12… class… 2022-05-10 17:31:56 2022-05-10 17:36:57 Clinton St & Mad…
    ## 6 C0A3AA5A614D… class… 2022-05-04 14:48:55 2022-05-04 14:56:04 Carpenter St & H…
    ## # ℹ 9 more variables: from_station_id <chr>, to_station_name <chr>,
    ## #   to_station_id <chr>, usertype <chr>, date <date>, month <dbl>, day <int>,
    ## #   year <dbl>, day_of_week <chr>

    summary(all_trips)  #Statistical summary of data. Mainly for numerics

    ##    trip_id             bikeid            start_time                    
    ##  Length:5859061     Length:5859061     Min.   :2022-05-01 00:00:06.00  
    ##  Class :character   Class :character   1st Qu.:2022-07-03 11:12:29.25  
    ##  Mode  :character   Mode  :character   Median :2022-08-28 12:44:56.50  
    ##                                        Mean   :2022-09-19 13:39:54.05  
    ##                                        3rd Qu.:2022-11-08 06:30:15.00  
    ##                                        Max.   :2023-04-30 23:59:05.00  
    ##                                        NA's   :31                      
    ##     end_time                      from_station_name  from_station_id   
    ##  Min.   :2022-05-01 00:05:17.00   Length:5859061     Length:5859061    
    ##  1st Qu.:2022-07-03 11:38:52.00   Class :character   Class :character  
    ##  Median :2022-08-28 13:07:18.00   Mode  :character   Mode  :character  
    ##  Mean   :2022-09-19 13:59:04.64                                        
    ##  3rd Qu.:2022-11-08 06:44:06.25                                        
    ##  Max.   :2023-05-03 10:37:12.00                                        
    ##  NA's   :23                                                            
    ##  to_station_name    to_station_id        usertype              date           
    ##  Length:5859061     Length:5859061     Length:5859061     Min.   :2022-05-01  
    ##  Class :character   Class :character   Class :character   1st Qu.:2022-07-03  
    ##  Mode  :character   Mode  :character   Mode  :character   Median :2022-08-28  
    ##                                                           Mean   :2022-09-18  
    ##                                                           3rd Qu.:2022-11-08  
    ##                                                           Max.   :2023-04-30  
    ##                                                                               
    ##      month             day             year      day_of_week       
    ##  Min.   : 1.000   Min.   : 1.00   Min.   :2022   Length:5859061    
    ##  1st Qu.: 5.000   1st Qu.: 8.00   1st Qu.:2022   Class :character  
    ##  Median : 7.000   Median :15.00   Median :2022   Mode  :character  
    ##  Mean   : 6.945   Mean   :15.67   Mean   :2022                     
    ##  3rd Qu.: 9.000   3rd Qu.:23.00   3rd Qu.:2022                     
    ##  Max.   :12.000   Max.   :31.00   Max.   :2023                     
    ## 

Adding the ride length column

    all_trips$ride_length <- difftime(all_trips$end_time,all_trips$start_time)

Convert “ride\_length” from Factor to numeric so we can run calculations
on the data

    is.factor(all_trips$ride_length)

    ## [1] FALSE

    all_trips$ride_length <- as.numeric(as.character(all_trips$ride_length))
    is.numeric(all_trips$ride_length)

    ## [1] TRUE

Removing negative ride\_length values, creating a new version of the
dataframe (v2) since data is being removed

    all_trips_v2 <- all_trips[!(all_trips$ride_length < 0),]

This script first removes rides that are either shorter than 1 minute or
longer than 24 hours. Such durations are likely due to users forgetting
to end their trips.

    all_trips_v2 <- all_trips_v2 %>%
      filter(ride_length >= 1 & ride_length <= 60 * 24)

Afterwards, the script converts the ride length from minutes to hours.
The result is rounded to two decimal places.

    all_trips_v2$ride_length <- round(all_trips_v2$ride_length / 60, 2)

### Descriptive Analysis and Data Visualization::

Ride Length Analysis

    summary(all_trips_v2$ride_length)

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   0.020   5.050   8.430   9.543  13.280  24.000

Boxplot of ride length by user type:

    ggplot(all_trips_v2, aes(x = usertype, y = ride_length, fill = usertype)) +
      geom_boxplot() +
      labs(
        x = "User Type",
        y = "Ride Length (hours)",  
        title = "Ride Length by User Type"
      ) +
      scale_fill_manual(values = c("lightblue", "lightgreen")) +
      theme_minimal() +
      theme(
        axis.title.x = element_text(size = 12),
        axis.title.y = element_text(size = 12),
        axis.text = element_text(size = 10),
        plot.title = element_text(size = 14, face = "bold")
      )

![Ride Length (hours) plot](Notebook_files/figure-markdown_strict/unnamed-chunk-15-1.png)

The data presents an interesting distinction in ride durations between
casual and member users. Casual users, on average, tend to enjoy longer
rides, typically around 10 hours. They show a wide variance, with some
rides extending from slightly over 5 hours to just below 15 hours.
Conversely, member users generally prefer shorter rides. The following
plot shows the average hourly usage in more details

    # Calculate the average ride length by user type
    avg_ride_length <- all_trips_v2 %>%
      group_by(usertype) %>%
      summarise(avg_ride_length = mean(ride_length, na.rm = TRUE))

    # Create a bar plot with custom color palette
    ggplot(avg_ride_length, aes(x = usertype, y = avg_ride_length, fill = usertype)) +
      geom_bar(stat = "identity", width = 0.5) +
      labs(
        x = "User Type",
        y = "Average Ride Length (hours)",
        title = "Average Ride Length by User Type"
      ) +
      scale_fill_manual(values = c("#FF8C00", "#4C79A6")) +
      theme_minimal() +
      theme(
        legend.position = "none",
        axis.title.x = element_text(size = 12),
        axis.title.y = element_text(size = 12),
        axis.text = element_text(size = 10),
        plot.title = element_text(size = 14, face = "bold")
      )

![](Notebook_files/figure-markdown_strict/unnamed-chunk-16-1.png)

Determine the day and hour that reflect the highest usage in the Divvy
bike data, you can analyze the data based on the number of rides by day
of the week and hour of the day.

    # Calculate the number of rides by day of the week and hour of the day
    usage_by_day_hour <- all_trips_v2 %>%
      group_by(day_of_week, hour = hour(start_time)) %>%
      summarise(ride_count = n(), .groups = "drop") %>%
      arrange(day_of_week, hour)

    # Find the day and hour with the highest usage
    max_usage <- usage_by_day_hour %>%
      filter(ride_count == max(ride_count))

    # Plot the number of rides by day of the week and hour of the day
    ggplot(usage_by_day_hour, aes(x = hour, y = ride_count, group = day_of_week, color = day_of_week)) +
      geom_line(size = 1.5) +
      geom_point(size = 3, shape = 21, fill = "white") +
      scale_color_brewer(palette = "Set1") +
      labs(
        x = "Hour of the Day",
        y = "Ride Count",
        title = "Divvy Bike Usage by Day and Hour"
      ) +
      theme_minimal() +
      theme(
        legend.position = "top",
        legend.title = element_blank(),
        axis.title.x = element_text(size = 12),
        axis.title.y = element_text(size = 12),
        axis.text = element_text(size = 10),
        plot.title = element_text(size = 14, face = "bold")
      )

    ## Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
    ## ℹ Please use `linewidth` instead.
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

![](Notebook_files/figure-markdown_strict/unnamed-chunk-17-1.png)

    #cat("Day with the highest usage:", max_usage$day_of_week, "\n")
    #cat("Hour with the highest usage:", max_usage$hour, "\n")
    # Day with the highest usage: Wednesday 
    # Hour with the highest usage: 5  pm

This plot provides an overview of the bike usage patterns throughout the
week, segmented by the hour of the day. The lines for each day represent
the number of rides taken during each hour, allowing us to observe the
peaks and troughs of usage.

The data suggests that bike usage patterns change significantly
depending on the day of the week and the hour of the day. Some hours
show consistently high bike usage across all days of the week, while
others show variable usage.

Interestingly, the day with the highest overall usage is Wednesday, and
the hour with the most rides is 5 pm. This might indicate that a
significant number of users are using the service for commuting
purposes, as this hour typically aligns with the end of a standard
workday.

However, it would be beneficial to look at the data more granularly to
understand if this pattern holds across all days, or if there are
specific days where the usage at 5 pm is particularly high. This
information could be used to manage the bike fleet more effectively and
ensure high availability during peak usage times.

    library(scales)

    ## 
    ## Attaching package: 'scales'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     discard

    ## The following object is masked from 'package:readr':
    ## 
    ##     col_factor

    # Define a vector of month names
    month_names <- c("January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December")

    ride_count_by_month_user <- all_trips_v2 %>%
      drop_na(month, usertype) %>%
      group_by(month, usertype) %>%
      summarise(ride_count = n(), .groups = "drop")

    # Convert month from numeric to factor with specified levels and labels
    ride_count_by_month_user$month <- factor(ride_count_by_month_user$month, levels = 1:12, labels = month_names)
    # Convert ride count to thousands
    ride_count_by_month_user$ride_count <- ride_count_by_month_user$ride_count / 1000

    ggplot(ride_count_by_month_user, aes(x = month, y = ride_count, fill = usertype)) +
      geom_bar(stat = "identity", position = "dodge") +
      labs(
        x = "Month",
        y = "Ride Count",
        fill = "User Type",
        title = "Monthly Ride Count by User Type (in 'k')"
      ) + 
      scale_y_continuous(labels = function(x) paste(x, "k"))+
      theme_minimal() +
      theme(
        legend.position = "top",
        legend.title = element_blank(),
        axis.text.x = element_text(angle = 45, hjust = 1),
        axis.title.x = element_text(size = 12),
        axis.title.y = element_text(size = 12),
        axis.text = element_text(size = 10),
        plot.title = element_text(size = 14, face = "bold")
      )

![](Notebook_files/figure-markdown_strict/ride_count_by_month_user-1.png)

The bar chart presents the ride count per month, divided by user type,
for both casual and member riders. The ride count is plotted on the
y-axis, while the x-axis represents each month of the year. This data
visualization allows us to examine and compare the monthly ride patterns
between the two types of users.

We observed that member rides outnumber casual rides every month. This
suggests that members use the service more consistently throughout the
year. For both user types, ride counts tend to increase from January to
July, peak in the summer months (June, July, and August), and then
gradually decrease towards the end of the year. Ride Length Analysis

Calculate the ride count by date

    # Calculate the ride count by month and year
    ride_count_by_month_year <- all_trips_v2 %>%
      drop_na(date) %>%
      mutate(year_month = format(date, "%Y-%m")) %>%
      group_by(year_month) %>%
      summarise(ride_count = n()) %>%
      ungroup()

    # Convert ride count to thousands
    ride_count_by_month_year$ride_count <- ride_count_by_month_year$ride_count / 1000

    # Find the highest and lowest ride count
    max_ride_count <- max(ride_count_by_month_year$ride_count)
    min_ride_count <- min(ride_count_by_month_year$ride_count)

    # Find the month/year with the highest and lowest ride count
    highest_month_year <- ride_count_by_month_year %>%
      filter(ride_count == max_ride_count) %>%
      pull(year_month)
    lowest_month_year <- ride_count_by_month_year %>%
      filter(ride_count == min_ride_count) %>%
      pull(year_month)

    # Print the highest and lowest ride month/year
    cat("Highest bike ride month/year: ", highest_month_year, "\n")

    ## Highest bike ride month/year:  2022-07

    cat("Lowest bike ride month/year: ", lowest_month_year, "\n")

    ## Lowest bike ride month/year:  2022-12

    # Create a line plot with smoothed line and filled points
    ggplot(data = ride_count_by_month_year, aes(x = year_month, y = ride_count, group = 1)) +
      
      geom_point(aes(fill = year_month), size = 3, shape = 21, color = "black") +
      labs(
        x = NULL,
        y = "Ride Count (in thousands)",
        title = "Divvy Bike Ride Monthly Count Over Time"
      ) +
      geom_line(color = "blue", size = 1.5, alpha = 0.7) +
      scale_y_continuous(labels = function(x) paste(x, "k")) +
      scale_fill_manual(values = rainbow(length(unique(ride_count_by_month_year$year_month)))) +
      theme_minimal() +
      theme(
        axis.text.x = element_blank(),
        axis.title.x = element_blank(),
        axis.title.y = element_text(size = 12),
        axis.text = element_text(size = 10),
        plot.title = element_text(size = 14, face = "bold")
      )

![](Notebook_files/figure-markdown_strict/unnamed-chunk-18-1.png)

The data indicates that in May 2022, there were approximately 502,000
rides, while in July 2022, the count rose to about 657,000 rides, which
was the highest throughout the observed period. On the contrary, the
lowest number of rides was in December 2022 with approximately 169,000
rides.

The data points in the plot are filled with different colors for each
month, and the points are connected by a blue line to show the trend of
ride count over time. The highest point on the plot corresponds to July
2022, which was the month with the highest ride count, while the lowest
point corresponds to December 2022, the month with the lowest ride
count.

From this visualization, one can infer the fluctuations in bike ride
counts over the year. For example, the count tends to peak in summer
(July) and drop in winter (December). These trends can be useful for
understanding the seasonality of bike usage and planning accordingly.

    # write.csv(all_trips_v2, file = '~/github.io/data/cleaned_data.csv')
