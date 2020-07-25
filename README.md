-- Which tracks appeared in the most playlists?
  -- ANS: 2 Minutes To Midnight
SELECT Name, count(tracks.trackid)
FROM tracks
JOIN playlist_track
  ON tracks.TrackId = playlist_track.TrackId
  GROUP BY 1
  ORDER BY 2 DESC
  ;
  --How many playlist did they appear in?
  -- ANS:13
  
 SELECT Name,  tracks.trackid, count(tracks.trackid)
FROM tracks
JOIN playlist_track
  ON tracks.TrackId = playlist_track.TrackId
  GROUP BY 1
  ORDER BY 3 DESC
  LIMIT 1
  ;
  
 --  Which track generated the most revenue? which album? which genre?
 -- ANS: The Woman King, Battlestar Galactica, Season 3
 WITH Table_1 AS (
 SELECT tracks.Name, tracks.AlbumId,tracks.GenreId, albums.Title, tracks.TrackId, invoice_items.InvoiceId -- sum(invoices.total)
FROM tracks
JOIN albums
  ON trackS.AlbumId= albums.AlbumId
  JOIN invoice_items
      ON tracks.TrackId = invoice_items.TrackId
	  )
SELECT Table_1.Name, Table_1.Genreid,Table_1.Title, Table_1.TrackiD,Table_1.InvoiceId,  sum (invoices.total) 
	FROM Table_1 
         JOIN invoices
	     ON Table_1.InvoiceId = invoices.InvoiceId
         GROUP BY 4
		 ORDER BY 6 DESC
		 ;
	-- Nested JOINs but not efficient (See best answer below)
	SELECT tracks.Name, tracks.AlbumId,genres.name, albums.Title, tracks.TrackId, invoice_items.InvoiceId, sum (invoices.total) 
	FROM 
	genres JOIN 
	(
	invoices JOIN
	(
	invoice_items JOIN
	(
	tracks JOIN albums 
	             ON tracks.albumid = albums.albumId
				)
	    ON tracks.TrackId = invoice_items.TrackId
		)
		ON invoices.InvoiceId = invoice_items.InvoiceId
		)
		ON genres.GenreId = tracks.GenreId
		 GROUP BY 5
		 ORDER BY 7 DESC
		;
 
  --  Which track generated the most revenue? which album? which genre?
 -- ANS: The Woman King | Battlestar Galactica, Season 3 | Science Fiction

 SELECT tracks.Name, albums.Title, genres.name, sum(invoice_items.unitprice) As 'Total Sales'
FROM tracks
JOIN albums
  ON trackS.AlbumId= albums.AlbumId
  JOIN invoice_items
      ON tracks.TrackId = invoice_items.TrackId
	  JOIN genres
	     ON tracks.GenreId = genres.GenreId
		 JOIN invoices 
		    ON invoice_items.InvoiceId= invoices.InvoiceId
			-- Note : In the last JOIN above, the 'tracks' table has no common column with the 'invoices' table, so I used the 'invoices_items' table to bring in the 'invoices' table  
GROUP BY tracks.TrackId
ORDER BY 4 DESC ;

-- Which countries have the highest sales revenue? What percent of total revenue does each country make up?
-- USA and Canada with 22.5% and 13.1% of total revenue respectively 
WITH Bucket AS (SELECT sum(total) AS 'Total_Revenue'
FROM invoices)
select BillingCountry, ROUND(SUM(total), 1) As 'Country_Sales' , ROUND(Bucket.Total_Revenue,1) AS 'Total_Revenue', ROUND(SUM(invoices.total)*100/Bucket.Total_Revenue,1) AS '% of Total_Revenue'
FROM invoices
CROSS JOIN Bucket
GROUP BY BillingCountry
ORDER BY 2 DESC; 

-- How many customers did each employee support, what is the average revenue for each sale, and what is their total sale?

SELECT employees.EmployeeId, employees.LastName,COUNT (customers.CustomerId), AVG (invoices.Total), SUM(invoices.Total)
  FROM employees
  JOIN customers
    ON employees.EmployeeId = customers.SupportRepId
	JOIN invoices
	  ON customers.CustomerId = invoices.CustomerId
	  GROUP BY 1
	  ORDER BY 2 DESC 
	;


 
