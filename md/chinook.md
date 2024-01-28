## Udacity’s SQL Chinook Music Database
The SQL queries that I used during the project and the corresponding questions can be found below:

### Who is writing the rock music?
```sql
SELECT
    a.ArtistId,
    a.Name,
    Count(t.TrackId) AS Songs
FROM
    Artist a
    JOIN Album al ON a.ArtistId = al.ArtistId
    JOIN Track t ON al.AlbumId = t.AlbumId
    JOIN Genre g ON g.GenreId = t.GenreId
WHERE
    g.Name = 'Rock'
GROUP BY
    a.ArtistId,
    a.Name
ORDER BY
    Songs DESC
LIMIT
    10;
```

### Which artist has earned the most according to the InvoiceLines? Which customer spent the most on this artist

```sql
WITH artist_earning AS (
    SELECT
        a.ArtistId,
        a.Name,
        SUM((il.Quantity * il.UnitPrice)) AS AmountSpent
    FROM
        Artist a
        JOIN Album al ON a.ArtistId = al.ArtistId
        JOIN Track t ON al.AlbumId = t.AlbumId
        JOIN InvoiceLine il ON t.TrackId = il.TrackId
    GROUP BY
        a.ArtistId,
        a.Name
    ORDER BY
        AmountSpent DESC
    LIMIT
        1
)
SELECT
    e.Name,
    SUM((il.Quantity * il.UnitPrice)) AS AmountSpent,
    c.CustomerId,
    c.FirstName,
    c.LastName
FROM
    Customer c
    JOIN Invoice i ON c.CustomerId = i.CustomerId
    JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
    JOIN Track t ON t.TrackId = il.TrackId
    JOIN Album al ON al.AlbumId = t.AlbumId
    JOIN artist_earning e on al.ArtistId = e.ArtistId
GROUP BY
    e.Name,
    c.CustomerId,
    c.FirstName,
    c.LastName
ORDER BY
    AmountSpent DESC;

```

### The most popular music Genre for each country
```sql
WITH tb AS (
    SELECT
        SUM(il.Quantity) AS Purchases,
        i.BillingCountry AS Country,
        g.Name,
        g.GenreId
    FROM
        Invoice i
        JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
        JOIN Track t ON t.TrackId = il.TrackId
        JOIN Genre g ON g.GenreId = t.GenreId
    GROUP BY
        i.BillingCountry,
        g.Name,
        g.GenreId
    ORDER BY
        Country
),
max_pruchase AS (
    SELECT
        Country,
        MAX(Purchases) as max_purchase
    FROM
        tb
    GROUP BY
        Country
)
SELECT
    Purchases,
    tb.Country,
    Name,
    GenreId
FROM
    tb
    JOIN max_pruchase mp ON tb.Purchases = mp.max_purchase
    AND tb.Country = mp.Country
```

### The highest amount of money spent on music per country
```sql
WITH tb as (
    SELECT
        i.BillingCountry as Country,
        SUM(i.Total) as TotalSpent,
        c.FirstName,
        c.LastName,
        c.CustomerId
    FROM
        Invoice i
        JOIN Customer c ON i.CustomerId = c.CustomerId
    GROUP BY
        Country,
        c.CustomerId
    ORDER BY
        Country
),
max_pruchase AS (
    SELECT
        Country,
        MAX(TotalSpent) as max_spent
    FROM
        tb
    GROUP BY
        Country
)
SELECT
    tb.Country,
    TotalSpent,
    FirstName,
    LastName,
    CustomerId
FROM
    tb
    JOIN max_pruchase mp ON tb.TotalSpent = mp.max_spent
    AND tb.Country = mp.Country
```
