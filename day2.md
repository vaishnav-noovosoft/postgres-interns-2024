## Joins 
_________

1) How can you produce a list of the start times for bookings by members named 'David Farrell'?
    ```sql
    SELECT starttime
    FROM cd.bookings bks
    INNER JOIN cd.members mems
    ON mems.memid = bks.memid
    WHERE mems.firstname = 'David' AND
    mems.surname = 'Farrell'
    ```
   ![start time](./images/day2/start-time.png)

2) How can you produce a list of the start times for bookings for tennis courts, for the date '2012-09-21'? Return a list of start time and facility name pairings, ordered by the time.
   ```sql
   SELECT bks.starttime AS start, flts.name AS name
   FROM cd.bookings bks
   INNER JOIN cd.facilities flts
   ON flts.facid = bks.facid
   WHERE
   bks.starttime >= '2012-09-21' AND
   bks.starttime < '2012-09-22' AND
   flts.name IN ('Tennis Court 1', 'Tennis Court 2')
   ORDER BY bks.starttime
   ```
   ![start time tennis](./images/day2/start-time-tennis.png)

3) How can you output a list of all members who have recommended another member? Ensure that there are no duplicates in the list, and that results are ordered by (surname, firstname).
   ```sql
   SELECT DISTINCT recs.firstname, recs.surname
   FROM cd.members mems
   INNER JOIN cd.members recs
   ON recs.memid = mems.recommendedby
   ORDER BY surname, firstname
   ```
   ![list recommended](./images/day2/recommend-list.png) 

4) How can you output a list of all members, including the individual who recommended them (if any)? Ensure that results are ordered by (surname, firstname).
   ```sql
   SELECT mems.firstname memfname, mems.surname memssname, recs.firstname recfname, recs.surname recsname
   FROM cd.members mems
   LEFT JOIN cd.members recs
   ON mems.recommendedby = recs.memid
   ORDER BY mems.surname, mems.firstname
   ```
   ![list recommender](./images/day2/recommender-list.png)
   
5) How can you produce a list of all members who have used a tennis court? Include in your output the name of the court, and the name of the member formatted as a single column. Ensure no duplicate data, and order by the member name followed by the facility name.
   ```sql
   SELECT DISTINCT mems.firstname || ' ' || mems.surname member, flts.name facility
   FROM cd.members mems
   INNER JOIN cd.bookings bks
   ON mems.memid = bks.memid
   INNER JOIN cd.facilities flts
   ON bks.facid = flts.facid
   WHERE flts.name IN ('Tennis Court 1', 'Tennis Court 2')
   ORDER BY member, facility
   ```
   ![members used tennis](./images/day2/mems-tennis.png)

6) How can you produce a list of bookings on the day of 2012-09-14 which will cost the member (or guest) more than $30? Remember that guests have different costs to members (the listed costs are per half-hour 'slot'), and the guest user is always ID 0. Include in your output the name of the facility, the name of the member formatted as a single column, and the cost. Order by descending cost, and do not use any subqueries.
   ```sql
    SELECT mems.firstname || ' ' || mems.surname AS member, flts.name AS facility,
        CASE 
	        WHEN mems.memid = 0 THEN
	            bks.slots*flts.guestcost
		    ELSE
			    bks.slots*flts.membercost
	    END AS cost
	    FROM cd.facilities flts
		    INNER JOIN cd.bookings bks
		        ON bks.facid = flts.facid
		    INNER JOIN cd.members mems
		        ON mems.memid = bks.memid
	    WHERE bks.starttime >= '2012-09-14' AND bks.starttime < '2012-09-15' AND
		    ((mems.memid = 0 AND bks.slots*flts.guestcost > 30) OR (mems.memid != 0 AND bks.slots*flts.membercost > 30))
	    ORDER BY cost DESC
   ```
   ![Three Join 2](./images/day2/three-join-2.png)

7) Produce a list of all members, along with their recommender, using no joins.
   ```sql
   SELECT DISTINCT mems.firstname || ' ' || mems.surname AS member, 
   (
    SELECT rec.firstname || ' ' || rec.surname AS recommender
    FROM cd.members rec
    WHERE mems.recommendedby = rec.memid
   )
   FROM
   cd.members mems
   ORDER BY member, recommender
   ```
   ![sub](./images/day2/sub.png)   

8) The Produce a list of costly bookings exercise contained some messy logic: we had to calculate the booking cost in both the WHERE clause and the CASE statement. Try to simplify this calculation using subqueries. For reference, the question was:
   How can you produce a list of bookings on the day of 2012-09-14 which will cost the member (or guest) more than $30? Remember that guests have different costs to members (the listed costs are per half-hour 'slot'), and the guest user is always ID 0. Include in your output the name of the facility, the name of the member formatted as a single column, and the cost. Order by descending cost.
   ```sql
   SELECT member, facility, cost from (
	SELECT m.firstname || ' ' || m.surname AS member, f.name AS facility,
  	CASE
  		WHEN m.memid = 0 THEN
  			b.slots*f.guestcost
  		ELSE
  			b.slots*f.membercost
  	END AS cost
  	FROM
  		cd.members m
  		INNER JOIN cd.bookings b
  			ON m.memid = b.memid
  		INNER JOIN cd.facilities f
  			ON f.facid = b.facid
  	WHERE b.starttime::date = '2012-09-14'
   ) AS bookings
   WHERE cost > 30
   ORDER BY cost DESC;
   ```
   ![TJ sub](./images/day2/tjsub.png)

## Aggregates
_____________
1) Count the number of facilities
   ```sql
   SELECT COUNT(facid) FROM cd.facilities
   ```
   ![count](./images/day2/count.png)

2) Count the number of expensive facilities
   ```sql
   SELECT COUNT(facid) FROM cd.facilities WHERE guestcost >= 10;
   ```
   ![count expensive facilities](./images/day2/count2.png)

3) Count the number of recommendations each member makes.
   ```sql
   SELECT recommendedby, COUNT(*)
   FROM cd.members
   WHERE recommendedby IS NOT NULL
   GROUP BY recommendedby
   ORDER BY recommendedby
   ```
   ![count 3](./images/day2/count3.png)

4) List the total slots booked per facility
   ```sql
   SELECT facid, SUM(slots) AS "Total Slots" FROM cd.bookings
   GROUP BY facid
   ORDER BY facid
   ```
   ![Fachours](./images/day2/fachours.png)

5) Produce a list of the total number of slots booked per facility in the month of September 2012. Produce an output table consisting of facility id and slots, sorted by the number of slots.
   ```sql
   SELECT FACID, SUM(SLOTS) AS "Total Slots"
   FROM CD.BOOKINGS
   WHERE STARTTIME >= '2012-09-01' AND STARTTIME < '2012-10-01'
   GROUP BY FACID
   ORDER BY "Total Slots"
   ```
   ![Fachours by month](./images/day2/fachoursbymonth.png)

6) List the total slots booked per facility per month
   ```sql
   SELECT FACID, EXTRACT(MONTH FROM STARTTIME) AS MONTH, SUM(SLOTS) AS "Total Slots"
   FROM CD.BOOKINGS
   WHERE EXTRACT(YEAR FROM STARTTIME) = 2012
   GROUP BY FACID, MONTH
   ORDER BY FACID, MONTH;
   ```
   ![Fachours by month](./images/day2/fachoursbymonth2.png)

7) Find the count of members who have made at least one booking
   ```sql
   select count(*) from 
	(select distinct memid from cd.bookings) as mems
   ```
   ![Members1](./images/day2/members.png)

8) List facilities with more than 1000 slots booked
   ```sql
   select facid, sum(slots) as "Total Slots"
        from cd.bookings
        group by facid
        having sum(slots) > 1000
        order by facid
   ```
   ![Fachours 1a](./images/day2/fachours1a.png)

9) Find the total revenue of each facility
   ```sql
   select facs.name, sum(slots * case
			when memid = 0 then facs.guestcost
			else facs.membercost
		end) as revenue
	from cd.bookings bks
	inner join cd.facilities facs
		on bks.facid = facs.facid
	group by facs.name
   order by revenue;
   ```
   ![Facrev](./images/day2/facrev.png)

10) Find facilities with a total revenue less than 1000
   ```sql
   select name, revenue from (
	select facs.name, sum(case 
				when memid = 0 then slots * facs.guestcost
				else slots * membercost
			end) as revenue
		from cd.bookings bks
		inner join cd.facilities facs
			on bks.facid = facs.facid
		group by facs.name
	) as agg where revenue < 1000
    order by revenue;
   ```
   ![Facrev 2](./images/day2/facrev2.png)
   
11) Output the facility id that has the highest number of slots booked
   ```sql
   select facid, sum(slots) as "Total Slots"
	from cd.bookings
	group by facid
    order by sum(slots) desc
    limit 1;
   ```
   ![Fachours 2](./images/day2/fachours2.png)
   
12) List the total slots booked per facility per month, part 2
   ```sql
   select facid, extract(month from starttime) as month, sum(slots) as slots
	from cd.bookings
	where
		starttime >= '2012-01-01'
		and starttime < '2013-01-01'
	group by rollup(facid, month)
    order by facid, month; 
   ```
   ![Facours by month 2](./images/day2/fachoursbymonth3.png)
   
13) List the total hours booked per named facility
   ```sql
   select facs.facid, facs.name,
	trim(to_char(sum(bks.slots)/2.0, '9999999999999999D99')) as "Total Hours"

	from cd.bookings bks
	inner join cd.facilities facs
		on facs.facid = bks.facid
	group by facs.facid, facs.name
    order by facs.facid;
   ```
   ![Fachours 3](./images/day2/fachours3.png)
   
14) List each member's first booking after September 1st 2012
   ```sql
   select mems.surname, mems.firstname, mems.memid, min(bks.starttime) as starttime
	from cd.bookings bks
	inner join cd.members mems on
		mems.memid = bks.memid
	where starttime >= '2012-09-01'
	group by mems.surname, mems.firstname, mems.memid
    order by mems.memid;  
   ```
   ![NBooking](./images/day2/nbooking.png)

15) Produce a list of member names, with each row containing the total member count
   ```sql
   select (select count(*) from cd.members) as count, firstname, surname
	from cd.members
    order by joindate
   ```
   ![Count Members](./images/day2/countmembers.png)
   
16) Produce a numbered list of members
   ```sql
   select row_number() over(order by joindate), firstname, surname
	from cd.members
    order by joindate   
   ```
   ![Num members](./images/day2/nummembers.png)
   
17) Output the facility id that has the highest number of slots booked, again
   ```postgresql
   select facid, total from (
	select facid, sum(slots) total, rank() over (order by sum(slots) desc) rank
        	from cd.bookings
		group by facid
	) as ranked
	where rank = 1
   ```
   ![Fachours 4](./images/day2/fachours4.png)
   
18) Rank members by (rounded) hours used
   ```sql
   select firstname, surname,
	((sum(bks.slots)+10)/20)*10 as hours,
	rank() over (order by ((sum(bks.slots)+10)/20)*10 desc) as rank

	from cd.bookings bks
	inner join cd.members mems
		on bks.memid = mems.memid
	group by mems.memid
    order by rank, surname, firstname; 
   ```
   ![Rank members](./images/day2/rankmembers.png)
   
19) Find the top three revenue generating facilities
   ```postgresql
   select name, rank from (
	select facs.name as name, rank() over (order by sum(case
				when memid = 0 then slots * facs.guestcost
				else slots * membercost
			end) desc) as rank
		from cd.bookings bks
		inner join cd.facilities facs
			on bks.facid = facs.facid
		group by facs.name
	) as subq
	where rank <= 3
    order by rank; 
   ```
   ![Facrev 3](./images/day2/facrev3.png)
   
20) Classify facilities by value
   ```postgresql
   select name, case when class=1 then 'high'
		when class=2 then 'average'
		else 'low'
		end revenue
	from (
		select facs.name as name, ntile(3) over (order by sum(case
				when memid = 0 then slots * facs.guestcost
				else slots * membercost
			end) desc) as class
		from cd.bookings bks
		inner join cd.facilities facs
			on bks.facid = facs.facid
		group by facs.name
	) as subq
    order by class, name; 
   ```
   ![Classify](./images/day2/classify.png)
   
21) Calculate the payback time for each facility
   ```postgresql
   select 	facs.name as name,
	facs.initialoutlay/((sum(case
			when memid = 0 then slots * facs.guestcost
			else slots * membercost
		end)/3) - facs.monthlymaintenance) as months
	from cd.bookings bks
	inner join cd.facilities facs
		on bks.facid = facs.facid
	group by facs.facid
    order by name;    
   ```
   ![Payback](./images/day2/payback.png)

22) Calculate a rolling average of total revenue
   ```postgresql
   select 	dategen.date,
	(
		select sum(case
			when memid = 0 then slots * facs.guestcost
			else slots * membercost
		end) as rev

		from cd.bookings bks
		inner join cd.facilities facs
			on bks.facid = facs.facid
		where bks.starttime > dategen.date - interval '14 days'
			and bks.starttime < dategen.date + interval '1 day'
	)/15 as revenue
	from
	(
		select 	cast(generate_series(timestamp '2012-08-01',
			'2012-08-31','1 day') as date) as date
	)  as dategen
    order by dategen.date;
   ```
   ![Rolling Avg](./images/day2/rollingavg.png)