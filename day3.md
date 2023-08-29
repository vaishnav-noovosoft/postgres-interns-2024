### Day 3

## Data Manipulation
____________________

1) Insert some data into a table
    ```sql
   INSERT INTO cd.facilities
    VALUES(9, 'Spa', 20, 30, 100000, 800)
   ```
   ![Insert](./images/day3/insert.png)

2) Insert multiple rows of data into a table
   ```sql
   INSERT INTO CD.FACILITIES
    VALUES (9, 'Spa', 20, 30, 100000, 800),
    (10, 'Squash Court 2', 3.5, 17.5, 5000, 80);
   ```
   ![Insert Multiple](./images/day3/insert2.png)

3) Insert calculated data into a table
   ```sql
   INSERT INTO CD.FACILITIES
	SELECT(SELECT MAX(FACID) FROM CD.FACILITIES)+1, 'Spa', 20, 30, 100000, 800
   ```
   ![Insert 3](./images/day3/insert3.png)

4) Update some existing data
   ```sql
   UPDATE CD.FACILITIES
   SET INITIALOUTLAY=10000
   WHERE FACID=1
   ```
   ![Update](./images/day3/UPDATE.png)

5) Update multiple rows and columns at the same time
   ```sql
   UPDATE cd.facilities
	SET
		membercost=6,
		guestcost=30
	WHERE
		facid in (0,1);
   ```
   ![update multiple](./images/day3/updatemultiple.png)

6) Update a row based on the contents of another row
   ```sql
   update cd.facilities facs
    set
        membercost = facs2.membercost * 1.1,
        guestcost = facs2.guestcost * 1.1
    from (select * from cd.facilities where facid = 0) facs2
    where facs.facid = 1;
   ```
   ![Update Calculated](./images/day3/updatecalc.png)

7) Delete all bookings
   ```sql
   DELETE FROM cd.bookings
   ```
   ![Delete all records](./images/day3/delete.png)

8) Delete a member from the cd.members table
   ```sql
   DELETE FROM cd.members
	WHERE memid=37;
   ```
   ![Delete record](./images/day3/deletewh.png)


## Strings
_______

1) Format the names of members
   ```sql
   SELECT surname || ', ' || firstname name FROM cd.members
   ```
   ![Concat](./images/day3/concat.png)

2) Find facilities by a name prefix
   ```sql
   SELECT * FROM cd.facilities WHERE name LIKE 'Tennis%'
   ```
   ![like](./images/day3/like.png)

3) Perform a case-insensitive search
   ```sql
   SELECT * FROM cd.facilities WHERE UPPER(name) LIKE 'TENNIS%'
   ```
   ![case](./images/day3/case.png)

4) Find telephone numbers with parentheses
   ```sql
   SELECT memid, telephone FROM cd.members WHERE telephone ~ '[()]';
   ```
   ![Regex](./images/day3/reg.png)

5) Pad zip codes with leading zeroes
   ```sql
   SELECT lpad(cast(zipcode as char(5)), 5, '0') zip FROM cd.members ORDER BY zip;
   ```
   ![pad](./images/day3/pad.png)

6) Count the number of members whose surname starts with each letter of the alphabet
   ```sql
   SELECT SUBSTR(mems.surname, 1, 1) AS letter, COUNT(*) as count
	FROM cd.members mems
	GROUP BY letter
	ORDER BY letter
   ```
   ![Padding](./images/day3/substr.png)

7) Clean up telephone numbers
   ```sql
   SELECT memid, translate(telephone, '-() ', '') as telephone
	FROM cd.members
	ORDER BY memid
   ```
   ![translate](./images/day3/translate.png)

## Date

--------

1) Produce a timestamp for 1 a.m. on the 31st of August 2012
   ```sql
   SELECT timestamp '2012-08-31 01:00:00';
   ```
   ![Timestamp](./images/day3/timestamp.png)

2) Subtract timestamps from each other
   ```sql
   SELECT timestamp '2012-08-31 01:00:00' - timestamp '2012-07-30 01:00:00' as interval
   ```
   ![Interval](./images/day3/interval.png)

3) Generate a list of all the dates in October 2012
   ```sql
   SELECT GENERATE_SERIES(timestamp '2012-10-01', timestamp '2012-10-31', interval '1 day') as ts;
   ```
   ![Series](./images/day3/series.png)

4) Get the day of the month from a timestamp
   ```sql
   SELECT extract(day from timestamp '2012-08-31');
   ```
   ![Extract](./images/day3/extract.png)

5) Work out the number of seconds between timestamps
   ```sql
   SELECT CAST(EXTRACT(EPOCH FROM (timestamp '2012-09-02 00:00:00' - '2012-08-31 01:00:00')) AS integer) AS date_part;
   ```
   ![Interval 2](./images/day3/interval2.png)

6) Work out the number of days in each month of 2012
   ```sql
   SELECT EXTRACT(month FROM cal.month) AS month,
	(cal.month + interval '1 month') - cal.month AS length
	FROM
	(
		SELECT generate_series(timestamp '2012-01-01', timestamp '2012-12-01', interval '1 month') AS month
	) cal
   ORDER BY month;
   ```
   ![Days in month](./images/day3/daysinmonth.png)

7) Work out the number of days remaining in the month
   ```sql
   SELECT (date_trunc('month', ts.testts) + interval '1 month')
		- date_trunc('day', ts.testts) as remaining
	FROM (SELECT timestamp '2012-02-11 01:00:00' as testts) ts
   ```
   ![Days remaining](./images/day3/daysremaining.png)

8) Work out the end time of bookings
   ```sql
   SELECT starttime, starttime + slots*(interval '30 minutes') endtime
	FROM cd.bookings
	ORDER BY endtime DESC, starttime DESC
	LIMIT 10;
   ```
   ![End times](./images/day3/endtimes.png)

9) Return a count of bookings for each month
   ```sql
   SELECT date_trunc('month',  starttime) AS month, count(*)
	FROM cd.bookings
	GROUP BY month
	ORDER BY month
   ```
   ![Booking sper month](./images/day3/bookingspermonth.png)

10) Work out the utilisation percentage for each facility by month
   ```sql
   SELECT name, month,
    ROUND((100 * slots) /
        CAST(
            25 * (CAST((month + interval '1 month') AS date)
                  - CAST(month AS date)) AS numeric),
         1) AS utilisation
    FROM (
     SELECT facs.name AS name,
            DATE_TRUNC('month', starttime) AS month,
               SUM(slots) AS slots
        FROM cd.bookings bks
        INNER JOIN cd.facilities facs
        ON bks.facid = facs.facid
         GROUP BY facs.facid, month
    ) AS inn
    ORDER BY name, month;
   ```
   ![Utilisation per month](./images/day3/utilizationpermonth.png)