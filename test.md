draft:
## How to analyze a 10GB dataset in R without all the fuss.


You have a dataset that is large and reasonably well structured and formatted.

The result is code that is elegant, concise, readable and fast.

Your folder should contain three files: 

 - the dataset (usually a .csv file)
 -  the .R script
 - the html output of Markdown 

You can use 

Here are my tips:

- use the `data.table` package (or `dplyr` if you have less than 1 million rows)
- avoid intermediate variables - use pipes( `%>%`)  or chaining( `[...]`) instead

Another thing that comes naturally from this is that you have no real reason to save your workspace - the only data frame in it will be your original dataset which is usually to large to save anyways.


Basic WorkFlow:
1.  use `fread` to read your data in memory
2.  Make a list of questions you'd like to know about the data.
3.  For each of your questions, use `data.table` to aggregate, group and modify your data
4.  pipe the results from 2 and output your result as a table or plot

```{r}
fread("data.csv")

for (i in questions) {
	data[group by, aggregate, filter][...][...] %>%
		ggplot(aes(...)) OR head()
		repeat or move to the next question.}

rmakrdown::render(analysis.R)
	

```


library(dplyr)
library(data.table)

> system.time(flights <- rbind(fread("1987.csv"),
+                  fread("1988.csv"),
+                  fread("1989.csv"),
+                  fread("1990.csv"),
+                  fread("1991.csv"),
+                  fread("1992.csv"),
+                  fread("1993.csv"),
+                  fread("1994.csv"),
+                  fread("1996.csv"),
+                  fread("1997.csv"),
+                  fread("1998.csv"),
+                  fread("1999.csv"),
+                  fread("2000.csv"),
+                  fread("2001.csv"),
+                  fread("2002.csv"),
+                  fread("2003.csv"), 
+                  fread("2004.csv"), 
+                  fread("2005.csv"),
+                  fread("2006.csv"),
+                  fread("2007.csv"),
+                  fread("2008.csv")))
Read 5202096 rows and 29 (of 29) columns from 0.467 GB file in 00:00:06
Read 5041200 rows and 29 (of 29) columns from 0.453 GB file in 00:00:06
Read 5270893 rows and 29 (of 29) columns from 0.474 GB file in 00:00:06
Read 5076925 rows and 29 (of 29) columns from 0.457 GB file in 00:00:06
Read 5092157 rows and 29 (of 29) columns from 0.459 GB file in 00:00:06
Read 5070501 rows and 29 (of 29) columns from 0.457 GB file in 00:00:06
Read 5180048 rows and 29 (of 29) columns from 0.467 GB file in 00:00:06
Read 5351983 rows and 29 (of 29) columns from 0.497 GB file in 00:00:08
Read 5411843 rows and 29 (of 29) columns from 0.503 GB file in 00:00:07
Read 5384721 rows and 29 (of 29) columns from 0.501 GB file in 00:00:08
Read 5527884 rows and 29 (of 29) columns from 0.515 GB file in 00:00:08
Read 5683047 rows and 29 (of 29) columns from 0.531 GB file in 00:00:08
Read 5967780 rows and 29 (of 29) columns from 0.559 GB file in 00:00:09
Read 5271359 rows and 29 (of 29) columns from 0.494 GB file in 00:00:08
Read 6488540 rows and 29 (of 29) columns from 0.584 GB file in 00:00:09
Read 7129270 rows and 29 (of 29) columns from 0.624 GB file in 00:00:12
Read 7140596 rows and 29 (of 29) columns from 0.625 GB file in 00:00:09
Read 7141922 rows and 29 (of 29) columns from 0.626 GB file in 00:00:12
Read 7453215 rows and 29 (of 29) columns from 0.655 GB file in 00:00:10
Read 7009728 rows and 29 (of 29) columns from 0.642 GB file in 00:00:10
   user  system elapsed 
150.186  86.780 301.489 

> dim(flights)
[1] 118207534        29

> mem_used()
16.1 GB

> class(flights)
[1] "data.table" "data.frame"

> object_size(flights)
16.1 GB

#example 1:

> system.time(flights[, .(num = .N), by = TailNum][order(-num)])
   user  system elapsed 
  1.039   0.239   1.294 
  
#example 2:
  > system.time(flights[, .(n = .N, delay = mean(ArrDelay, na.rm = TRUE)), 
+         by = Origin][n >10000][order(-delay)] %>% head())
   user  system elapsed 
  1.537   1.090   2.797 
