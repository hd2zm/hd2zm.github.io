---
layout: project_single
title:  "Analyzing New York Subway Data to Optimize WomenTechWomenYes's Street Team Efficiency"
slug: "NYC-MTA"
---

**This is the first project I completed as part of Metis Data Science Bootcamp. I worked with a team of 4 people: Joel Porcaro, Leo Lillard, and Jay Cheong. We worked together to deliver a project and a presentation.**

# Introduction

This was an email we got from a fictional client. 

>   As we mentioned, we are interested in harnessing the power of data and analytics to optimize the effectiveness of our street team work, which is a significant portion of our fundraising efforts.
>    WomenTechWomenYes (WTWY) has an annual gala at the beginning of the summer each year. As we are new and inclusive organization we try to do double duty with the gala both to fill our event space with individuals passionate about increasing the participation of women in technology, and to concurrently build awareness and reach.
>    To this end we place street teams at entrances to subway stations. The street teams collect email addresses and those who sign up are sent free tickets to our gala.
>    Where we’d like to solicit your engagement is to use MTA subway data, which as I’m sure you know is available freely from the city, to help us optimize the placement of our street teams, such that we can gather the most signatures, ideally from those who will attend the gala and contribute to our cause.
>    The ball is in your court now—do you think this is something that would be feasible for your group? From there we can explore what kind of an engagement would make sense for all of us.

# Project Overview

Our goal is to use New York's Metropolitan Transportation Authority (MTA) data to analyze which stations are the best to place WTWY's street teams in. We're breaking this problem into 3 parts

* What resources does WTWY have for the street team? 
* What time and day are optimal for most subway station traffic?
* What station generates highest traffic?

# Resources and Assumptions

WTWY is a non for profit. So it is very limited in resources. Here are the assumptions we made. 

* WTWY can place at most 2 people per station
* WTWY can only occupy 2 hour time blocks
* WTWY can have a street team at most 3 days per week


# Data Wrangling and Cleaning

We accessed all MTA data from this [website](http://web.mta.info/developers/turnstile.html). Looking at the [field description](http://web.mta.info/developers/resources/nyct/turnstile/ts_Field_Description.txt), we noticed that the stations keep track of turnstile counts rather than users. We got the users for each time period by grouping each station turnstile device (a station can have more than 1 turnstile device, with each device set to a different turnstile counter) and calculating the difference of turnstile counts. This gave us the number of users per time period for each station turnstile device. 

# Optimal Time and Day

We loaded in MTA data from 5/25/2019 to 6/28/2019. As of now, we wanted to see a pattern with a small time frame before loading in more data. 

In addition, the data shows users in intervals of 4 hours per day. So we have users at midnight, 4 am, 8 am, 12 pm, 4 pm, and 8pm. 

**For optimization, we focused on morning rush hours (8am-12pm) and afternoon rush hours (4pm-8pm). Those have the highest traffic out of other hour intervals.**

Below are the average number of users taking the metro during that time frame. 

![image]({{site.url}}/images/projects/NYC-MTA/Date_Timeseries.png)

There are two takeaways

* Afternoon rush hour traffic exceeds morning traffic by a lot
* Some days have lower traffic than others. Our guess is that those are weekends and there's not many people using the subway then. We dug deeper to plot the average number of users taking the metro by day. 

![image]({{site.url}}/images/projects/NYC-MTA/Day_Timeseries.png)

As predicted, Saturday and Sunday have the lowest users. The users then shoot back up on Monday and continue for each weekday. 

**Given that WTWY has limited resources, we excluded turnstile data from Saturday and Sunday.**

# Optimal Station

Below are the top 10 stations with highest traffic during morning rush hour and afternoon rush hour (excluding Saturday and Sunday).

| Morning Rush Hour     | Afternoon Rush Hour |
| ---      | ---       |
| ![image]({{site.url}}/images/projects/NYC-MTA/10_Stations_Morning_Rush_Bar.png) | ![image]({{site.url}}/images/projects/NYC-MTA/10_Stations_Afternoon_Rush_Bar.png)         |

While these averages are good, we also wanted to see how much the data varies. Do we have any strong outliers per station? Below are boxplots for the top 10 stations during morning and afternoon rush hours. 

### Morning Rush Hour
![image]({{site.url}}/images/projects/NYC-MTA/10_Stations_Morning_Rush_Box.png)

### Afternoon Rush Hour
![image]({{site.url}}/images/projects/NYC-MTA/10_Stations_Afternoon_Rush_Box.png)

So there isn't much variabilitiy within our data, which is good. Although some data points have a lot of outliers. We also noticed that for morning and afternoon rush, the top 5 stations are the same (34 ST-PENN STA, GRD CNTRL-42 ST, 34 ST-HERALD SQ, TIMES SQ-42 ST, 23 ST). Those are the stations we should focus on. 

# Conclusion

If WTWY can get 4-10 volunteers for only 1 day, they should follow these recommendations
* Assign 2 volunteers to each station in order of importance: 34 ST-PENN STA, GRD CNTRL-42 ST, 34 ST-HERALD SQ, TIMES SQ-42 ST, 23 ST
* Take a 2 hour time slot between 4pm - 8pm 
* Choose either Tuesday, Wednesday, or Thursday

# Appendix

Below is a bar graph zoomed in on the 5 stations per day.

![image]({{site.url}}/images/projects/NYC-MTA/5_Stations_Day_Bar.png)

Below is a time series graph of the 5 stations per day.

![image]({{site.url}}/images/projects/NYC-MTA/5_Stations_Day_Timeseries.png)



