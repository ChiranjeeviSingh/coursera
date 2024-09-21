**Data Engineering Activity Assignment -2**

Chiranjeevi Singh Marutla, 14259974, <c.marutla@ufl.edu>

**1\. Extracting the year, title, and critic_scores from the Rotten Tomatoes CSV:**

For this task, I explored running loops inside the awk command to help us deal with the commas inside the ‘synopsis’ column which is present just before the critic score column which makes extracting the score tricky.

awk 'BEGIN { FS=","; OFS=","; inside_quotes=0 }

{

for (i = 1; i <= NF; i++) {

if ($i ~ /^"/) inside_quotes = 1;

if ($i ~ /"$/) inside_quotes = 0;

if (inside_quotes) {

$(i) = $(i) "," $(i+1);

for (j = i+1; j < NF; j++) $j = $(j+1);

NF--;

i--;

}

}

print $2, $3, $5

}' /blue/cis6930/share/data-loading/rotten-tomatoes-movies.csv > extracted_data.csv

**2\. Remove the header row and blank lines:**

Removed header lines using tail -n + 2 as explained in the class. Used sed to handle blank lines which are completely empty rows.

tail -n +2 extracted_data.csv > cleaned_data.csv

sed -i '/^$/d' cleaned_data.csv

**3\. Filter rows with missing critic_score**

Additionally removed the rows which don’t have critic rows as we want to take the average later on. If not we have to handle null cases when we write the query.

awk -F',' '($3 != "") { print $0 > "final_data.csv" } ($3 == "") { print $0 > "removed_data.csv" }' cleaned_data.csv

**4\. Create SQLite3 database and import the data**

Below is the straightforward task of creating the rotten DB and creating a table for scores. Named the table scores as our aim is the take the average of scores.

\[c.marutla@login10 c.marutla\]$ **sqlite3 rotten.db**

SQLite version 3.26.0 2018-12-01 12:34:55

Enter ".help" for usage hints.

**sqlite> CREATE TABLE** scores (

...> title TEXT,

...> year INTEGER,

...> critic_score INTEGER

...> );

**sqlite> .mode csv**

**sqlite> .import final_data.csv scores**

**5\. Verify the data:**

Simple command to see couple of rows in our table before calculations.

**sqlite> SELECT \* FROM scores LIMIT 4;**

"Black Panther",2018,96

"Avengers: Endgame",2019,94

"Mission: Impossible -- Fallout",2018,97

"Mad Max: Fury Road",2015,97

sqlite>

**6\. Query the average critic scores by year:**

Simple SQL query to filter out average score greater than 60 and the output.

SELECT year, AVG(critic_score) AS avg_score

FROM scores

GROUP BY year

HAVING avg_score > 60

ORDER BY year ASC;

**sqlite> SELECT year, AVG(critic_score) AS avg_score**

**...> FROM scores**

**...> GROUP BY year**

**...> HAVING avg_score > 60**

**...> ORDER BY year ASC;**

1919|99.0

1921|100.0

1922|97.0

1925|98.0

1927|97.2857142857143

1928|98.0

1929|93.0

1930|96.8

1931|98.1666666666667

1932|94.2

1933|94.75

1934|98.5

1935|97.8666666666667

1936|100.0

1937|97.5714285714286

1938|96.4444444444444

1939|96.8333333333333

1940|98.45

1941|97.6923076923077

1942|95.5

1943|98.0

1944|99.0

1945|97.6666666666667

1946|95.0

1947|96.0

1948|98.625

1949|99.6

1950|98.3076923076923

1951|96.9090909090909

1952|97.875

1953|97.6923076923077

1954|97.0

1955|96.2727272727273

1956|95.6666666666667

1957|97.0

1958|93.8181818181818

1959|97.1052631578947

1960|94.5555555555556

1961|96.0

1962|95.875

1963|95.8571428571429

1964|98.0

1965|92.25

1966|98.25

1967|98.1428571428571

1968|95.3846153846154

1969|92.6666666666667

1970|92.6666666666667

1971|93.25

1972|95.8

1973|93.25

1974|95.8333333333333

1975|96.0

1976|94.7142857142857

1977|93.1666666666667

1978|94.7142857142857

1979|98.0909090909091

1980|86.0

1981|88.5

1982|95.3333333333333

1983|89.0

1984|97.2

1985|96.3333333333333

1986|87.4

1987|95.1666666666667

1988|97.1875

1989|94.5

1990|92.5714285714286

1991|95.4166666666667

1992|93.125

1993|91.5

1994|91.4

1995|91.1111111111111

1996|95.0

1997|91.75

1998|86.0

1999|90.6875

2000|85.5625

2001|80.8

2002|92.625

2003|84.4666666666667

2004|85.8666666666667

2005|81.9230769230769

2006|80.625

2007|91.5151515151515

2008|90.4130434782609

2009|93.0

2010|88.7083333333333

2011|89.3584905660377

2012|92.0833333333333

2013|91.9795918367347

2014|92.2878787878788

2015|92.3809523809524

2016|92.4084507042254

2017|91.7105263157895

2018|93.2692307692308

2019|92.7638888888889

2020|94.3414634146341

**sqlite>.exit**

**Used .exit to come out of sqlite.**
