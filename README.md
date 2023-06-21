# chq-carts-leaflet

* Goal of this project
	* measure transit access
	* visualize it in an easy to see way
* Methodology
	* Preparing the data
		* Conceptualizing "transit access"
		* Many different methods
	* Visualizing the data
		* Leaflet.js - used code from other project
		* Added color coding based on frequency 

## Project Background

The goal of this project is to analyze the quality of public transit in Chautauqua County, NY, and then visualize it in an intuitive way.

My connection to Chautauqua County is that I grew up here, and my family still lives here. My connection to transit is that I'm passionate about making transit better. I'm hoping that this analysis can help do that.

## Methodology 

### Preparing the data

The first challenge in analyzing the quality of public transit is to determine what qualifies as "good" public transportation. 

I think of good transit as transit that is useful. The utility of transit should be reflected in its ridership. All things equal, we should strive to increase ridership on a transit network.

Jarrett Walker, a public transit consultant, has a good blog post outlining ingredients for high ridership transit. I hope to evaluate CARTS on all of these criteria, but for this project I'm focusing on ridership and access. Specifically, I want to measure the number of people who are within a 1 mile buffer of frequent transit service - that is, service that runs 15 mins or better.

### Enter GTFS

To calculate this number, I downloaded the GTFS data for CARTS (Chautauqua Area Regional Transportation System). 

Then, I uploaded all of the files into a local MySQL database that I ran using MAMP.

From there, I used the built in SQL query window to run a join that took data from `stops.txt` (which contains the list of stops) and combined it with the `stop_times.txt` (which contains the actual trips). I used the third line to count the number of distinct trips that a stop sees. As far as I understand, this captures the number of transit vehicles that stop at that stop. I was only interested in trips during the morning rush hour (which I defined as 8am to 10am), so I used a `WHERE` clause to filer only those trips. Here's what my first table looked like:

|stop_id                                  |stop_name            |num_trips|
|-----------------------------------------|---------------------|---------|
|364759                                   |Junction             |18       |
|STOP_cf45d44f-1408-4095-8d41-acb100f6c296|Walmart              |6        |
|STOP_0bb67a21-f462-47cf-b583-bbf05609b1e2|Save-A-Lot           |5        |
|STOP_aaa24966-5097-4b44-a581-47bfc1e3e7ac|Dunkirk City Hall    |5        |
|364746                                   |Doughty & Lincoln    |4        |
|364767                                   |2nd & Foote          |4        |

I don't have the exact query I used - but I did save the results of the query as a view. Here's the code that underlies that view.


```
select 
	`carts_gtfs`.`stops_txt`.`stop_id` AS `stop_id`,
	`carts_gtfs`.`stops_txt`.`stop_name` AS `stop_name`,
	 count(distinct `carts_gtfs`.`stop_times_txt`.`trip_id`) AS `num_trips` 
	from (`carts_gtfs`.`stops_txt` 
	join `carts_gtfs`.`stop_times_txt` 
	on((`carts_gtfs`.`stops_txt`.`stop_id` = `carts_gtfs`.`stop_times_txt`.`stop_id`))) 
	where (`carts_gtfs`.`stop_times_txt`.`departure_time` between '08:00:00' and '10:00:00') 
	group by `carts_gtfs`.`stops_txt`.`stop_id`,`carts_gtfs`.`stops_txt`.`stop_name` 
	order by count(distinct `carts_gtfs`.`stop_times_txt`.`trip_id`) desc
```
Next, I saved the results of the above query as a view, and performed the following operation:

```
SELECT morning_rush_frequencies.stop_id, 
morning_rush_frequencies.stop_name, 
stops_txt.stop_desc, 
morning_rush_frequencies.num_trips, stops_txt.stop_lat,
stops_txt.stop_lon FROM morning_rush_frequencies 
JOIN stops_txt 
ON morning_rush_frequencies.stop_id = stops_txt.stop_id
ORDER BY num_trips DESC
```
to get the following table:

|stop_id                                  |stop_name            |stop_desc|num_trips|stop_lat |stop_lon  |
|-----------------------------------------|---------------------|---------|---------|---------|----------|
|364759                                   |Junction             |Jamestown|18       |42.096881|-79.237982|
|STOP_cf45d44f-1408-4095-8d41-acb100f6c296|Walmart              |         |6        |42.453782|-79.312534|
|STOP_aaa24966-5097-4b44-a581-47bfc1e3e7ac|Dunkirk City Hall    |         |5        |42.483357|-79.334410|
|STOP_0bb67a21-f462-47cf-b583-bbf05609b1e2|Save-A-Lot           |         |5        |42.484678|-79.329572|
|364743                                   |Lakeside Community   |         |4        |42.497036|-79.309830|
|364754                                   |Don's Motel          |         |4        |42.482139|-79.355922|
|364767                                   |2nd & Foote          |         |4        |42.097477|-79.234428|

### Visualizing the data