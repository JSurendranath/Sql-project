# Sql-project
Music store data analysis project SQL
drop table if exists employee;

CREATE TABLE IF NOT EXISTS employee(
employee_id INT PRIMARY KEY,
last_name varchar(50) NOT NULL,
first_name varchar(50) NOT NULL,
title varchar(50) NOT NULL,
reports_to int DEFAULT NULL,
level1 varchar(50) NOT NULL,
birthdate varchar(50),
hire_date varchar(50),
address varchar(50) NOT NULL,
city varchar(50) NOT NULL,
state varchar(50) NOT NULL,
country varchar(50) NOT NULL,
postal_code varchar(15) NOT NULL,
phone varchar(50) NOT NULL,
fax varchar(50) NOT NULL,
email varchar(100) NOT NULL
);

desc employee;

select * from employee;

drop TABLE IF EXISTS customer;

CREATE TABLE IF NOT EXISTS customer(
customer_id int PRIMARY KEY,
first_name varchar(50) NOT NULL,
last_name varchar(50) NOT NULL,
company varchar(50) NOT NULL,
address varchar(50) NOT NULL,
city varchar(50) NOT NULL,
state varchar(50) NOT NULL,
country varchar(50) NOT NULL,
postal_code varchar(15) NOT NULL,
phone varchar(50) NOT NULL,
fax varchar (50) NOT NULL,
email varchar(100) NOT NULL,
support_rep_id INT NOT NULL
);

desc customer;

select * from customer;

ALTER TABLE customer
ADD CONSTRAINT fk_employee_employee_id
FOREIGN KEY (support_rep_id)
REFERENCES employee (employee_id);

drop TABLE IF EXISTS invoice;

CREATE TABLE IF NOT EXISTS invoice (
invoice_id int ,
customer_id int NOT NULL,
invoice_date varchar(50),
billing_address varchar(50) NOT NULL,
billing_city varchar(50) NOT NULL,
billing_state varchar(50) NOT NULL,
billing_country varchar(50) NOT NULL,
billing_postal_code varchar(15) NOT NULL,
total int NOT NULL,
PRIMARY KEY (invoice_id),-- COMPOSITE PRIMARY KEY
CONSTRAINT fk_customer_customer_id FOREIGN KEY (customer_id) REFERENCES customer (customer_id)
);


desc invoice;
select * from invoice;


DROP TABLE IF EXISTS media_type;


CREATE TABLE IF NOT EXISTS media_type(
media_type_id INT PRIMARY KEY,
name varchar(50) NOT NULL
);

desc media_type;
select * from media_type;

DROP TABLE IF EXISTS genre;

CREATE TABLE IF NOT EXISTS genre(
genre_id int PRIMARY KEY,
name varchar(50) NOT NULL
);
desc genre;
select * from genre;

drop TABLE IF EXISTS playlist;

CREATE TABLE IF NOT EXISTS playlist(
playlist_id INT PRIMARY KEY,
name varchar(50) NOT NULL
);
desc playlist;
select * from playlist;

drop TABLE IF EXISTS artist;

CREATE TABLE IF NOT EXISTS artist(
artist_id INT PRIMARY KEY,
name varchar(50) NOT NULL
);

desc artist;
select* from artist;

DROP TABLE IF EXISTS playlist_track;

CREATE TABLE IF NOT EXISTS playlist_track(
playlist_id INT ,
track_id INT PRIMARY KEY
);

desc playlist_track;

select * from playlist_track;


ALTER TABLE playlist_track
ADD CONSTRAINT fk_playlist_playlist_id
FOREIGN KEY (playlist_id)
REFERENCES playlist (playlist_id);



drop TABLE IF EXISTS album;

CREATE TABLE IF NOT EXISTS album(
album_id INT PRIMARY KEY,
title varchar(500) NOT NULL,
artist_id INT Default NULL
);

desc album;
select * from album;



DROP TABLE IF EXISTS track;


CREATE TABLE IF NOT EXISTS track(
track_id INT PRIMARY KEY,
name varchar(50) NOT NULL,
album_id INT NOT NULL,
media_type_id INT NOT NULL,
genre_id INT NOT NULL,
composer varchar(50) NOT NULL,
milliseconds INT NOT NULL,
bytes INT NOT NULL,
unit_price DECIMAL(3,1) NOT NULL
);
desc track;
select * from track;

ALTER TABLE track
ADD CONSTRAINT fk_album_album_id
FOREIGN KEY(album_id)
REFERENCES album (album_id);

ALTER TABLE track
ADD CONSTRAINT fk_media_type_media_type_id
FOREIGN KEY(media_type_id)
REFERENCES media_type (media_type_id);

ALTER TABLE track
ADD CONSTRAINT fk_genre_genre_id
FOREIGN KEY(genre_id)
REFERENCES genre (genre_id);


DROP TABLE IF EXISTS invoice_line;


CREATE TABLE IF NOT EXISTS invoice_line(
invoice_line_id INT PRIMARY KEY,
invoice_id INT Default NULL,
track_id INT NOT NULL,
unit_price DECIMAL(3,1) not null,
quantity INT NOT NULL
);
desc invoice_line;

select * from invoice_line;


ALTER TABLE invoice_line
ADD CONSTRAINT fk_playlist_track_playlist_id
FOREIGN KEY (track_id)
REFERENCES playlist_track (playlist_id_id);

ALTER TABLE invoice_line
ADD CONSTRAINT fk_invoice_invoice_id
FOREIGN KEY (invoice_id)
REFERENCES invoice (invoice_id);

--- Questions 1 : Easy


/* Q1: Who is the senior most employee based on job title? */

select * from employee;
ORDER BY levels desc
limit 1

/* Q2: Which countries have the most Invoices? */

select * from invoice;
select count(*) as c, billing_country from invoice
group by billing_country
order by c desc;

/* Q3: What are top 3 values of total invoice? */

select total from invoice;
order by total desc
limit 3;

/* Q4: Which city has the best customers? We would like to throw a promotional Music Festival in the city we made the most money. 
Write a query that returns one city that has the highest sum of invoice totals. 
Return both the city name & sum of all invoice totals */

select SUM(total) as invoice_total, billing_city from invoice
group by billing_city
order by invoice_total desc;

/* Q5: Who is the best customer? The customer who has spent the most money will be declared the best customer. 
Write a query that returns the person who has spent the most money.*/

SELECT customer.customer_id, first_name, last_name, SUM(total) AS total_spending
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
GROUP BY customer.customer_id
ORDER BY total_spending DESC
LIMIT 1;

/*   Method 2  */

select customer.customer_id, customer.first_name, customer.last_name, SUM(invoice.total) as total
from customer
join invoice on customer.customer_id = invoice.customer_id
group by customer.customer_id
order by total desc
limit 1;


-- Moderate

/* Q1: Write query to return the email, first name, last name, & Genre of all Rock Music listeners. 
Return your list ordered alphabetically by email starting with A. */

SELECT 
    e.first_name, e.last_name,e.email,
    g.name
FROM employee e
INNER JOIN genre g
having name = 'Rock'
ORDER BY e.email;

/* Q2: Let's invite the artists who have written the most rock music in our dataset. 
Write a query that returns the Artist name and total track count of the top 10 rock bands. */

SELECT 
    t.name,
    g.name
FROM track t
INNER JOIN genre g
ON t.genre_id = g.genre_id
having g.name = 'Rock'
ORDER BY t.name
limit 10;

select * from track;
/* Q3: Return all the track names that have a song length longer than the average song length. 
Return the Name and Milliseconds for each track. Order by the song length with the longest songs listed first. */

SELECT name,milliseconds FROM track
WHERE milliseconds > (
	SELECT AVG(milliseconds) AS avg_track_length
	FROM track )
ORDER BY milliseconds DESC;


/* Question Set 3 - Advance */

/* Q1: Find how much amount spent by each customer on artists? 
Write a query to return customer name, artist name and total spent */

WITH best_selling_artist AS (
	SELECT artist.artist_id AS artist_id, artist.name AS artist_name, SUM(invoice_line.unit_price*invoice_line.quantity) AS total_sales
	FROM invoice_line
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN album ON album.album_id = track.album_id
	JOIN artist ON artist.artist_id = album.artist_id
	GROUP BY 1
	ORDER BY 3 DESC
	LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name, SUM(il.unit_price*il.quantity) AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album alb ON alb.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
GROUP BY 1,2,3,4
ORDER BY 5 DESC;



/* Q2: We want to find out the most popular music Genre for each country. We determine the most popular genre as the genre 
with the highest amount of purchases. Write a query that returns each country along with the top Genre. For countries where 
the maximum number of purchases is shared return all Genres. */

WITH popular_genre AS 
(
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id, 
	ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
    FROM invoice_line 
	JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
	JOIN customer ON customer.customer_id = invoice.customer_id
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN genre ON genre.genre_id = track.genre_id
	GROUP BY 2,3,4
	ORDER BY 2 ASC, 1 DESC
)
SELECT * FROM popular_genre WHERE RowNo <= 1;



/* Q3: Write a query that determines the customer that has spent the most on music for each country. 
Write a query that returns the country along with the top customer and how much they spent. 
For countries where the top amount spent is shared, provide all customers who spent this amount. */


WITH Customter_with_country AS (
		SELECT customer.customer_id,first_name,last_name,billing_country,SUM(total) AS total_spending,
	    ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo 
		FROM invoice
		JOIN customer ON customer.customer_id = invoice.customer_id
		GROUP BY 1,2,3,4
		ORDER BY 4 ASC,5 DESC)
SELECT * FROM Customter_with_country WHERE RowNo <= 1

