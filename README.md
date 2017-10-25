### The FCC Political Ad Data Archive


This archive contains data extracted from documents disclosed during the 2014 US midterm elections on political advertisements.  The original files are available at [stations.fcc.gov](https://stations.fcc.gov/).

In July 2014, the Federal Communications Commission began requiring all broadcast TV stations to disclose their [public inspection files](https://stations.fcc.gov/about-station-profiles/) online, including documents on political ads.  These files are available as they are produced during the election.  However, most of the files are in image-based PDF format, which makes comprehensive study of them very difficult.  

The purpose of this project is to make the documents machine-readable with optical character recognition (OCR) software and other open source tools so the data for this and future elections is available to a wider audience including individuals, nonprofits, and news organizations.  Over 80,000 documents have been extracted, 130,000 identified by type, and all 260,000 by date, url, and other metadata.  Some documents were scanned at a low resolution or have legibility problems, or are in a format specific to a few stations and have not been extracted.


#### Common Document Types: Invoices, Orders, and Contracts

The three most prevalent and usable documents are invoices, orders, and contracts.  The data extraction was focused on these forms because they tell the most about the groups taking out the advertisements, and the transactions that occurred.  Around half of each form type are in a text-based format, which makes extraction much less difficult.  See [Schema and Field Documentation](#schema-and-field-documentation) below or the [schema](schema.tsv), which includes all columns.

#### Examples of Forms that have been identified but not extracted

* Credit Advisory
* Political Issue Ad form
* Political Inquiry
* Record of Request for Broadcast
* Candidate Advertising Change Checklist
* Issue Advertising Checklist
* Federal Candidate Certification
* Advertising Authorized By a Candidate form

(A full list of document types is available in the polfiles table.)

#### How to Import the Archive into a Database

The archive is available in PostgreSQL and MySQL formats, and the main tables: Invoice, Orders, and Contract are also available in tab-delimited text.

PostgreSQL

    createdb fcc
    gunzip fcc_pg.tar.gz
    pg_restore -d fcc fcc_pg.tar
  
MySQL

    echo "CREATE DATABASE fcc" | mysql
    gunzip fcc_mysql.sql.gz
    mysql -D fcc < fcc_mysql.sql


#### Methodology

Each image-based form is identified by headers on the page.  The size of the header and offset is recorded to predict how much the page shifted or was reduced or enlarged when it was scanned.  Based on this information, and the PDF bounding box -- the dimensions of the image -- a template is applied to the page and each area where a field is expected to appear is cropped into a small image with a margin to allow for inaccuracy in the initial prediction.  Each of these images is cropped by a computer vision algorithm to cut the box around the field precisely.

The images go through a series of processes to remove artifacts that may affect OCR accuracy.  These processes do not work for all images universally.  Some forms have certain defects and some do not.  A process that improves accuracy for some images will reduce it for others.  Because of this, the images are put through up to 54 combinations of noise-reduction filters and validated against known values for the field.

The resulting data is much more accurate than output from OCR software without validation, but some defects remain so the fields with more unpredictable values like product, advertiser, and advertiser address are cleaned up in OpenRefine.

Advertiser Address, which generally contains the agency -- a media company that makes the purchase on behalf of a campaign or group -- is highly redundant.  It contains only around 1300 unique companies for 80,000 invoices, orders, and contracts, so an effort has been made to reduce them to a bare minimum of recognizable names.  Product and Advertiser have been reduced as much as possible, but the range of these fields is every political group and every campaign for local, state, and federal races so there is some variability left in those fields (Example, Webber/G/D and Alan Webber for Governor).  The aggregate-ability of the data will improve over time.  Searching or aggregating on a candidate's last name, or standard variations for group names is safe.


### Schema and Field Documentation

The list of all tables, fields, and column types are in the [schema](schema.tsv).

#### Main fields

The fields and field names are taken directly from the documents, and the authoritative source for the fields and the meaning of the fields remains the original document, and the originating station.

##### Derived fields

*id*: The primary key throughout the database is the id field, which is taken from the station call sign and a numeric id embedded in the FCC RSS feeds.  It uniquely identifies every document in the database and exists in all of the main tables: polfile, invoice, contract and orders.

*advertiser* and *product*: The advertiser and product fields are used by the stations to identify the entity (campaign or political group) for which the ad was taken out.  These fields may have a strict delineation at some stations, but they are generally used interchangeably. *Note: In the orders table, the field names follow the printed forms and product is called "product_desc."*

*advertiser_address_name*: The entity named in the "Advertiser Address" (or "Agency Name" in orders) is almost invariably not the campaign or political group.  It is generally a media company acting on behalf of the group.  The advertiser_address_name is the company name taken from the address and cleaned and standardized.  


##### Other fields

demographic: The demographic targeted by the ad.  Annotation differs slightly among stations, but the format is "Adults," "Women," or "Men" (alternately "RA"/"RW"/"RM") with an age range, or "Households"/"RHH."  

special_handling: Special instructions for the transaction: "Cash in Advance," "Do Not Mail" etc.

"contract_from"/*contract_to*: Dates range when the contract is in force.
*flight_from*/*flight_to*: Date range when the ads are scheduled to air.

Miscellaneous identifiers:  There are a number of identifiers such as idb_no, order_no, deal_no, revision, among others that may be used to check one form against another, or find forms that were never uploaded to the FCC.

stations table: The stations table contains a list of all stations with the state in which they are registered.

The full list of fields including billing_cycle, billing_calendar, pages can be found in the [schema](schema.tsv).


#### License

The license is an unrestricted creative commons public domain license (CC0 1.0).  The data may be used for any purpose without restriction, however, please cite Alex Byrnes and, if possible, link to the data in your work.


#### Code

The code used for this project is available in a [separate repository](https://github.com/alexbyrnes/FCC-Political-Ads_The-Code).


#### Bug Reports

Please fill out a bug report or feature request for any errors or suggestions you have.  If you do any work that can be integrated back into the archive, either code or translation tables, it would be greatly appreciated.


#### Thanks

Thank you to the makers of great software.  PostgreSQL, Python, Linux, Tesseract, OpenRefine, and many others, and to the people hard at work in transparency and open data.

