# SQL-Music-Store-Analysis
# Digital Music Store Database Analysis (SQL)

## 📌 Project Overview
This project focuses on an end-to-end data analysis of a **Digital Music Store** database. The goal is to analyze the store's playlist, track, and customer transaction dataset to help the business understand its growth, identify its most valuable markets, map customer music preferences, and recognize top-performing artists. 

Originally structured for PostgreSQL, this repository contains a **fully migrated and optimized MySQL version** of the project, complete with data definition adjustments and advanced analytical queries.

---

## 🛠️ Database & Tools Used
* **Database Management System:** MySQL
* **Editor/IDE:** MySQL Workbench
* **Dataset:** 11 relational CSV tables (Artists, Albums, Tracks, Customers, Invoices, Employees, Genres, etc.)

---

## 📐 Relational Database Schema
The database consists of 11 tables structured relationally. To make the imported CSV tables work cleanly as a relational schema, primary keys were designated across the following tables:
* `employee` (primary key: `employee_id`)
* `customer` (primary key: `customer_id`)
* `invoice` (primary key: `invoice_id`)
* `invoice_line` (primary key: `invoice_line_id`)
* `track` (primary key: `track_id`)
* `genre` (primary key: `genre_id`)
* `media_type` (primary key: `media_type_id`)
* `playlist` (primary key: `playlist_id`)
* `artist` (primary key: `artist_id`)
* `album2` (primary key: `album_id`)
* `playlist_track` (composite join table)

---

## 🚀 How to Set Up the Database in MySQL Workbench

Since the raw repository database files are optimized for PostgreSQL, use the following CSV import process to set this up in MySQL Workbench:

1. **Download the Data:** Clone this repository or download and extract `music store data.zip` to retrieve the 11 CSV files.
2. **Create Database:** Open MySQL Workbench, open a new query tab, and execute:
   ```sql
   CREATE DATABASE music_store;
   USE music_store;
Import CSVs: Right-click the music_store database schema in the left panel, select Table Data Import Wizard, and import each CSV file one by one (MySQL will auto-detect columns).
Define Primary Keys: Run the primary key schema queries provided in the database_setup.sql file in this repository to establish the database integrity.
📊 Business Questions & SQL Solutions
🟢 Level 1: Basic Operations
Q1: Who is the senior-most employee based on job title?
SELECT first_name, last_name, title, levels 
FROM employee 
ORDER BY levels DESC 
LIMIT 1;
Q2: Which countries have the most invoices?
SELECT billing_country, COUNT(*) AS invoice_count 
FROM invoice 
GROUP BY billing_country 
ORDER BY invoice_count DESC;
Q3: What are the top 3 highest transaction totals in the store?
SELECT total 
FROM invoice 
ORDER BY total DESC 
LIMIT 3;
🟡 Level 2: Intermediate Joins & Subqueries
Q4: Find all Rock Music listeners (Email, First Name, Last Name, and Genre).
Useful for targeting marketing campaigns specifically to Rock listeners.
SELECT DISTINCT customer.email, customer.first_name, customer.last_name, genre.name AS genre_name
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
JOIN invoice_line ON invoice.invoice_id = invoice_line.invoice_id
JOIN track ON invoice_line.track_id = track.track_id
JOIN genre ON track.genre_id = genre.genre_id
WHERE genre.name = 'Rock'
ORDER BY customer.email;
Q5: Who are the top 10 Rock Artists in terms of track count?
SELECT artist.name, COUNT(track.track_id) AS number_of_songs
FROM track
JOIN album2 ON album2.album_id = track.album_id
JOIN artist ON artist.artist_id = album2.artist_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name = 'Rock'
GROUP BY artist.artist_id, artist.name
ORDER BY number_of_songs DESC
LIMIT 10;
Q6: Return all song tracks that are longer than the overall average song length.
SELECT name, milliseconds
FROM track
WHERE milliseconds > (
    SELECT AVG(milliseconds) 
    FROM track
)
ORDER BY milliseconds DESC;
🔴 Level 3: Advanced Business Intelligence (CTEs & Window Functions)
Q7: Find the absolute most popular music genre for each country.
Calculated by the highest number of track purchases. If there is a tie, all top genres are shown.
WITH popular_genre AS (
    SELECT 
        COUNT(invoice_line.quantity) AS purchases, 
        customer.country, 
        genre.name AS genre_name, 
        genre.genre_id,
        ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
    FROM invoice_line 
    JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
    JOIN customer ON customer.customer_id = invoice.customer_id
    JOIN track ON track.track_id = invoice_line.track_id
    JOIN genre ON genre.genre_id = track.genre_id
    GROUP BY customer.country, genre.name, genre.genre_id
)
SELECT * 
FROM popular_genre 
WHERE RowNo <= 1;
Q8: Identify the top-spending customer in each country.
Calculated using a CTE and partitioning the sum of customer purchases to find the top VIP customer per country.
WITH customer_with_country AS (
    SELECT 
        customer.customer_id, 
        customer.first_name, 
        customer.last_name, 
        invoice.billing_country, 
        SUM(invoice.total) AS total_spending,
        ROW_NUMBER() OVER(PARTITION BY invoice.billing_country ORDER BY SUM(invoice.total) DESC) AS RowNo 
    FROM invoice
    JOIN customer ON customer.customer_id = invoice.customer_id
    GROUP BY customer.customer_id, customer.first_name, customer.last_name, invoice.billing_country
)
SELECT * 
FROM customer_with_country 
WHERE RowNo <= 1;
💡 Key Business Takeaways
Dominant Markets: The store’s primary transactional volume is concentrated in key geographic countries, heavily guiding where marketing spend should be focused.
The Power of Rock: "Rock" is overwhelmingly the most popular music genre across multiple regions. This makes hosting featured Rock campaigns or partnering with top-performing artists a highly lucrative option.
VIP Tracking: Using partitioning window functions, we successfully extracted the highest-spending customers in every country, allowing the business to initiate exclusive loyalty programs or targeted offers for high-lifetime-value (LTV) shoppers.

***
