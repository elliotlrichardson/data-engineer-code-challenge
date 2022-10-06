### Description
This repo contains my work for the WFP data engineering challenge. The goal of the challenge was to manipulate JSON data from an events platform that is ubiquitous in progressive spaces into flattened, queryable tables, as well as provide instructions for using those tables.

### My approach
To start, I examined the attendances dictionary to see what it contained and where there were nested dictionaries that would need to be flattened. Right off the bat, I saw that there were nested dictionaries for people, events, and timeslots which were three tables that I was supposed to create. Each of these dictionaries had primary keys called *id* so I decided that these tables should be constructed in a star schema model (see visual below). 
https://assets.website-files.com/5e6f9b297ef3941db2593ba1/614df58a1f10f92b88f95709_Screenshot%202021-09-24%20at%2017.46.51.png
My plan was to (1) loop through the records, (2) extract those nested dictionaries, (3) reformat them, (4) store all of the reformatted dictionaries in lists, and finally, (5) convert them into CSVs. I utilized several helper functions to help me accomplish these steps!
- First, I created helper functions to extract those nested dictionaries called get_person, get_event, and get_timeslot. These functions grabbed the nested dictionary from each attendance record, renamed the primary keys, converted UNIX timestamps to datetime strings, extracted useful information from any additional nested dictionaries, and then removed those nested dictionaries. These functions accomplished steps 2 and 3, extracting and reformatting the nested dictionaries.
- Next, I created a helper function that took the list of attendance records, looped through them, applied the get functions to each record, and stored the reformatted records in dictionaries. I chose to store them in dictionaries with the primary keys (person_id, event_id, and timeslot_id) as the keys and the records as the corresponding value so that it wouldn’t create thousands of duplicate records if the unique person, event, or timeslot had already been processed. However, the function returned only the dictionaries in one big list so that it would be easily convertible to a CSV. This accomplished steps 1 and 4, looping through the records and storing them in lists of dictionaries.
- In this process, I realized that the people and events tables would not be able to relate to one another without a flattened attendances table, so I created an additional helper function to process the full attendance record by renaming the primary key, converting UNIX timestamps, extracting primary keys and other useful information from nested dictionaries, and then removed those dictionaries so the record was flat (very similar to how the person, event, and timeslot records were processed above). This wasn’t part of my original plan but seemed important for the use of the other tables.
- Finally, I created a helper function for step 5, writing those lists of dictionaries (people, events, timeslots, and flat_attendances) to CSVs in the output folder. Then I defined main to apply each of these functions in turn.
- Running this script results in the creation of 4 CSVs in the output folder: people, events, timeslots, flat_attendances.
Column names etc
### How to use these tables
There are three “dimension” tables, containing all of the details about specific people, events and timeslots. The “fact” table is flat_attendances, which references the other three. If you just want information about an event, you could likely just query the events table, but most of the time, you will likely want to join these dimension tables to the attendance table to pull useful numbers. I’ll elaborate on a few examples below. 
## Examples
- How many people RSVP’d to an event with a given ID?
For this query, you only need the attendances table since you have the event’s primary key. You can do a simple query to count attendance_ids with a where statement filtering to the specific event. 
```
select count(attendance_id) 
from attendances 
where event_id = [your event id]
```
- What event had the most number of completed attendances?
For this query, you could just use the attendances table if an event_id was a useful result, but to get an event name, type, or other details like that, you’d to join the attendances and events tables. In that case, you can count attendance_ids grouped by the event and limit your results to the event with the most attendees. 
```
select 
    events.title, 
    count(attendance_id) as attendees 
from attendances 
left join events using(event_id) 
where attended = ‘TRUE’ 
order by attendees desc 
limit 1
```
- What timeslot for a given event had the most event attendances?
For this query, you will need to join the attendances and timeslots table. You can count attendees grouped by timeslot and filter to the specific event.
```
select 
    timeslot_id, 
    start_date, 
    end_date, 
    count(attendance_id) as attendees 
from attendances 
left join timeslots using(timeslot_id) 
where attended = ‘TRUE’ 
    and event_id = [your event id]
group by 1,2,3
order by 4 desc
```
### Limitations & next steps
I did not accomplish everything that I would have liked to if I wanted to present useful and clean table to an analyst. I completed this challenge with the assumption that I was creating source tables that would be staged later on (in dbt for example) according to the conventions and needs of the team. For example, there were string columns that were intended to be boolean columns, some nulls here and there, and column names that I would have preferred (i.e. created_date → created_at). That would be the next step in this process for me!
An oversight or misstep on my part was that this script was not test-driven. With the suggested time limit, I jumped right into manipulation and transformation without setting up testing infrastructure to ensure data quality first. Test-driven development is a growth area of mine and the lack of that skill may have affected the usefulness of a script like this. 
						
						 					
				
			
		

