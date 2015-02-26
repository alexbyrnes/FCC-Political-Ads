###Issue Ads Methodology

The data was extracted from 44,806 contracts uploaded by television stations to the Federal Communications Commission during the 2014 US midterm election season.  The files were uploaded between July 1, 2014 and January 1, 2015. Coverage date is based on the "Contract Dates" field on contracts in the ["common" format](https://github.com/alexbyrnes/FCC-Political-Ads_The-Code/blob/master/pdfs/K27DX_14080331353324.pdf?raw=true).  Contracts that are not in the common format, or have low resolution or legibility problems are not included. 

Documents included are identified by a legible "Contract" header at the top of the page, and valid dates in the "Contract Dates" field.  Categorization as "Non-Candidate Issue Ads" is done by the stations.

[The full data set](https://github.com/alexbyrnes/FCC-Political-Ads) and [code used to extract the documents](https://github.com/alexbyrnes/FCC-Political-Ads_The-Code).

###Reproducing Results

SQL query:
```sql
SELECT 'Non-Candidate Issue Ad' as Category, count(*) as Count, d.day as Date FROM contract c INNER JOIN (SELECT date_trunc('day', dd)::date as day FROM generate_series('2014-06-01'::timestamp, '2014-12-01'::timestamp, '1 day'::interval) dd) d ON d.day BETWEEN c.contract_from AND c.contract_to INNER JOIN polfile p ON p.id = c.id AND p.url LIKE '%/Non-Candidate Issue Ads/%' GROUP BY Date
UNION
SELECT 'Non-Candidate Issue Ad' as Category, count(*) as Count, d.day as Date FROM contract c INNER JOIN (SELECT date_trunc('day', dd)::date as day FROM generate_series('2014-06-01'::timestamp, '2014-12-01'::timestamp, '1 day'::interval) dd) d ON d.day BETWEEN c.contract_from AND c.contract_to INNER JOIN polfile p ON p.id = c.id AND p.url NOT LIKE '%/Non-Candidate Issue Ads/%' GROUP BY Date
```

[Raw Data](https://github.com/alexbyrnes/FCC-Political-Ads_The-Code/blob/master/issue_ads.tsv)
