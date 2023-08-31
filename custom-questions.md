## Vaishnav
What are the top 10 films that have been rented by customers in the Canada, and how many times have they been rented?
```sql
SELECT film.title AS movie, COUNT(*) AS rented FROM film
INNER JOIN inventory ON film.film_id = inventory.film_id
INNER JOIN rental ON inventory.inventory_id = rental.inventory_id
INNER JOIN customer ON rental.customer_id = customer.customer_id
INNER JOIN address ON customer.address_id = address.address_id
INNER JOIN city ON address.city_id = city.city_id
INNER JOIN country ON city.country_id = country.country_id
WHERE country.country = 'United States'
GROUP BY film.title
ORDER BY rented DESC
LIMIT 10;
```
_______

What is the average rental duration for the top 5 films that have been rented by customers in each country?
```sql
WITH films_by_country AS (
SELECT
country.country,
film.title AS movie,
film.rental_duration,
ROW_NUMBER() OVER (PARTITION BY country.country ORDER BY COUNT(*) DESC) AS row
FROM film
INNER JOIN inventory ON film.film_id = inventory.film_id
INNER JOIN rental ON inventory.inventory_id = rental.inventory_id
INNER JOIN customer ON rental.customer_id = customer.customer_id
INNER JOIN address ON customer.address_id = address.address_id
INNER JOIN city ON address.city_id = city.city_id
INNER JOIN country ON city.country_id = country.country_id
GROUP BY country.country, movie, film.rental_duration
)
SELECT country, AVG(rental_duration)
FROM films_by_country
WHERE row <= 5
GROUP BY country
ORDER BY country
LIMIT 500;
```
________

## Shreyash
Select 25 the actors firstname and lastname in one column named as name and their film names who have released in recent year 2022 order by name
```sql
SELECT title AS film, actor.first_name || ' ' || actor.last_name AS name
FROM actor
INNER JOIN film_actor ON actor.actor_id = film_actor.actor_id
INNER JOIN film ON film_actor.film_id = film.film_id
ORDER BY name
LIMIT 25;
```

________

Select the staffs firstname and lastname in one column and their whole payment
```sql
SELECT staff.first_name || ' ' || staff.last_name AS name, SUM(payment.amount) AS payment
FROM staff
INNER JOIN payment ON staff.staff_id = payment.staff_id
GROUP BY name
ORDER BY name
LIMIT 500;

```
________

## Sadid
Find all actors who acted in more than one language. Output should contain actor_name, language. Sorted by languages, actor_name.
```sql
SELECT
CONCAT(actor.first_name, ' ', actor.last_name) as actor_name,
language.name as language
FROM actor
INNER JOIN film_actor ON actor.actor_id = film_actor.actor_id
INNER JOIN film ON film_actor.film_id = film.film_id
INNER JOIN language ON film.language_id = language.language_id
GROUP BY actor_name, language.name
HAVING COUNT(DISTINCT language.name) > 1
ORDER BY language.name, actor_name
LIMIT 500;
```
_______

Find the category which was rented the most for each month in year 2005, if there is a tie find all categories.
Sorted by months and category names. Output should contain month, category, count.
You can ignore the month if there is no data present for it.
```sql
SELECT category.name AS category, COUNT(*) AS rented
FROM category
INNER JOIN film_category ON category.category_id = film_category.category_id
INNER JOIN film ON film_category.film_id = film.film_id
INNER JOIN inventory ON film.film_id = inventory.film_id
INNER JOIN rental ON inventory.inventory_id = rental.inventory_id
WHERE EXTRACT(YEAR FROM rental.rental_date) = 2005
GROUP BY category.name
ORDER BY COUNT(rental_id) DESC
LIMIT 500;
```
_________

## Tejas
Query to Find Actors Who Have Collaborated the Most.
```sql
WITH film_actor AS (
SELECT CONCAT(actor.first_name, ' ', actor.last_name) as name, film_id, actor.actor_id
FROM actor
INNER JOIN film_actor ON actor.actor_id = film_actor.actor_id
)
SELECT CONCAT(actor1.name, ' & ', actor2.name) as actor_pair, COUNT(*) as count_of_films_together
FROM film_actor AS actor1
INNER JOIN film_actor AS actor2 ON (actor1.film_id = actor2.film_id) AND (actor1.actor_id != actor2.actor_id)
GROUP BY actor_pair
ORDER BY COUNT(*) DESC
LIMIT 500;
```

________


Query to Find Countries with the Longest Average Address Street(string) Length.
```sql
SELECT country.country as country, LENGTH(address.address) AS street_text_length
FROM country
INNER JOIN city ON country.country_id = city.country_id
INNER JOIN address ON city.city_id = address.city_id
ORDER BY LENGTH(address.address) DESC
LIMIT 500;
```

__________

## Harshit
Determine which customer watch how many movies of particular actor.
```sql
SELECT
CONCAT(customer.first_name, ' ', customer.last_name) as customer_name,
CONCAT(actor.first_name, ' ', actor.last_name) as actor_name,
COUNT(*) as times_rented
FROM customer
INNER JOIN rental ON customer.customer_id = rental.customer_id
INNER JOIN inventory ON rental.inventory_id = inventory.inventory_id
INNER JOIN film ON inventory.film_id = film.film_id
INNER JOIN film_actor ON film.film_id = film_actor.film_id
INNER JOIN actor ON film_actor.actor_id = actor.actor_id
GROUP BY customer_name, actor_name
ORDER BY COUNT(*) DESC
LIMIT 500;
```
________

List out top 10 most revenue generated district to release animated film with average price running on that district.
```sql
SELECT
address.district,
SUM(payment.amount) as revunue
FROM address
INNER JOIN store ON address.address_id = store.address_id
INNER JOIN staff ON store.store_id = staff.store_id
INNER JOIN payment ON staff.staff_id = payment.staff_id
INNER JOIN rental ON payment.rental_id = rental.rental_id
INNER JOIN inventory ON rental.inventory_id = inventory.inventory_id
INNER JOIN film ON inventory.film_id = film.film_id
INNER JOIN film_category ON film.film_id = film_category.film_id
INNER JOIN category ON film_category.category_id = category.category_id
WHERE category.name = 'Animation'
GROUP BY address.district
ORDER BY revunue DESC
LIMIT 500;
```
_________

## Pulkit
Find the Indian Staff from whom customer rented film language id 1(i.e. English).
```sql
SELECT DISTINCT CONCAT(staff.first_name, ' ', staff.last_name) AS staff_name
FROM address
INNER JOIN staff ON address.address_id = staff.address_id
INNER JOIN payment ON staff.staff_id = payment.staff_id
INNER JOIN rental ON payment.rental_id = rental.rental_id
INNER JOIN inventory ON rental.inventory_id = inventory.inventory_id
INNER JOIN film ON inventory.film_id = film.film_id
INNER JOIN language ON film.language_id = language.language_id
WHERE language.name = 'English'
ORDER BY staff_name
LIMIT 500;
```
_________

Find the Location of stores where film was rented whose release year is 2006.
```sql
SELECT DISTINCT CONCAT(address.district, ', ', address.address) AS store_location
FROM address
INNER JOIN store ON address.address_id = store.address_id
INNER JOIN staff ON store.store_id = staff.store_id
INNER JOIN rental ON staff.staff_id = rental.staff_id
INNER JOIN inventory ON rental.inventory_id = inventory.inventory_id
INNER JOIN film ON inventory.film_id = film.film_id
WHERE film.release_year = 2006
ORDER BY store_location
LIMIT 500;
```
___________

## Nandini
List out all films
by highest rental rate  and classify the films labelled as "Inappropriate for children under 13" with ratings PG-13,
"General" with ratings as G and " Restricted for Age below 18" with ratings as R
```sql
SELECT
film.title AS film,
CASE
WHEN film.rating = 'G' THEN 'General Audiences'
WHEN film.rating = 'PG' THEN 'Parental Guidance Suggested'
WHEN film.rating = 'PG-13' THEN 'Parents Strongly Cautioned'
WHEN film.rating = 'R' THEN 'Restricted to Viewers Over the Age of 17'
WHEN film.rating = 'NC-17' THEN 'No One 17 and Under Admitted'
ELSE 'OTHER'
END AS description
FROM film
ORDER BY film
LIMIT 500;
```
__________

List the top 3 actors having done maximum sci-fi movies and with maximum rental rates.
```sql
SELECT CONCAT(actor.first_name, ' ', actor.last_name) AS actor_name, COUNT(*) AS number_of_scifi_movie
FROM actor
INNER JOIN film_actor ON actor.actor_id = film_actor.actor_id
INNER JOIN film ON film_actor.film_id = film.film_id
INNER JOIN film_category ON film.film_id = film_category.film_id
INNER JOIN category ON film_category.category_id = category.category_id
WHERE category.name = 'Sci-Fi'
GROUP BY actor_name
ORDER BY number_of_scifi_movie DESC
LIMIT 3;
```