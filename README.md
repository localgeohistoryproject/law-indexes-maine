# Law Indexes: Maine

[![DOI](https://zenodo.org/badge/737969155.svg)](https://zenodo.org/doi/10.5281/zenodo.10457133)

## Summary

As part of efforts to expand the Local Geohistory Project, which aims to educate users and disseminate information concerning the geographic history and structure of political subdivisions and local government, this repository has been created to disseminate law index data for the State of Maine. This index currently covers private and special laws only from 1820 through 1957.

Private and special laws concern one or several individuals, entities, or localities, unlike public or general laws, which apply to all similarly situated individuals, entities, and localities within a jurisdiction. Throughout New England, private and special laws were often the primary method used to alter municipal and county boundaries and forms of government.

The data is released as individual tab-separated values (TSV) files by volume. The Detail column may contain additional Prefix information that is repeated from prior entries.

This repository does not contain the full text of the laws, nor does it currently contain links to the full text. Because the index was created using OCR technology, it may contain uncaptured errors.

This is not an official publication of the State of Maine or any of its instrumentalities, and is not affiliated with or endorsed by any government agency at any level.

## Harmful Content

The terms used in the law indexes were drawn from Source Works that were written before current diversity, equity, and inclusion principles were developed, and have not been updated to reflect modern standards. Some terms may:[^1]

- reflect racist, sexist, ableist, misogynistic/misogynoir, and xenophobic opinions and attitudes;
- be discriminatory towards or exclude diverse views on sexuality, gender, religion, and more;
- include graphic content of historical events such as violent death, medical procedures, crime, wars/terrorist acts, natural disasters, and more; or
- demonstrate bias and exclusion in the subjects documented.

## Using tab-separated values (TSV) files

TSV files do not use quotation marks to escape fields containing tabs; therefore, the **String delimiter** option in LibreOffice Calc or the **Text qualifier** option in Microsoft Excel must be left blank to ensure the file imports accurately.

These files use Unix-style [line endings](https://en.wikipedia.org/wiki/Newline#Representations) (LF), which may have to be adjusted in applications that expect Windows-style line endings (CR LF).

A header row containing column names is included. When importing into a relational database system, like PostgreSQL, it may be necessary to remove this header line, particularly when using the [PostgreSQL COPY function](https://www.postgresql.org/docs/16/sql-copy.html) in the Text Format.

## Source Works

A complete list of the Source Works used in this repository is available under [Source Works.md](Source%20Works.md). Prior to inclusion, Source Works were examined to determine whether they fell under the public domain in the United States, using the following guides:

Hirtle, Peter B. "Copyright Services: Copyright Term and the Public Domain." *Cornell University Library.* 1 January 2026. <https://guides.library.cornell.edu/copyright/publicdomain>.

U.S. Copyright Office. *Compendium of U.S. Copyright Office Practices,* 3rd ed. 2021. § 313.6(C). <https://www.copyright.gov/comp3/docs/compendium.pdf#page=83>.

## Methodology

The following is a general methodology for how the law indexes were derived from the Source Works.

- Once images are acquired, they will generally go through one of the following:
  - For images personally digitized from hard copy, the quality will be improved using ScanTailor, and then combined as PDFs.
  - For JPEG 2000 files acquired, they are converted to PDFs.
  - For PDF files acquired from Online Versions, the original image files may optionally be extracted and then re-combined as PDFs to remove any OCR.
- The PDFs are then OCR'd using [Amazon Textract](https://aws.amazon.com/pm/textract/). The files are uploaded using the [Bulk Document Uploader](https://docs.aws.amazon.com/textract/latest/dg/bulk-uploader-best-practices.html) and processed using the Detect Document Text API. This process returns a ZIP container for each file containing a text file, a CSV file, and a JSON file.
  - **Warning:** If the uploaded PDF is too large, it may not return the JSON file in the ZIP container, requiring the document to be broken down and re-uploaded in smaller pieces in order to follow the next steps.
- The JSON files are run through a PHP script, which finds any LINE BlockType entries, and returns the Page, Text, and the X and Y coordinates for the Geometry Polygon for each line in a delimited file. To simplify the coordinates, they are multiplied by 1000 and then rounded as integers. (In future steps, these LINE BlockType rows will be referred to as data points.)
  - **Note:** In tabular data sources, the LINE BlockType often corresponds with a cell, rather than an entire row. This helps to quickly categorize data points by location.
- The delimited file data points are imported into a PostgreSQL table.
- A preliminary attempt is made to group each data point into contiguous lines on the page, and to determine the type of data contained in each data point. For source works with multiple columns, the column number is also guessed.
- A PHP script converts the data points for each page into color-coded SVGs that represent the data points positionally in a similar manner to how they appeared on the original document.
- The SVGs are reviewed to ensure each data point is accurately categorized, that the data points appear in the correct order on the page, and for blatant OCR errors. In some cases, data points are split where Amazon Textract did not correctly separate them. Because these indexes often use positional prefixes in lieu of repeating text, these prefixes are split off into separate data points where found.
  - **Note:** The accuracy of the OCR was, in general, not manually reviewed.
- For certain data types, data integrity checks are done to ensure the data makes sense (e.g., whether years are digits and fall in the correct time period).
- The data points are combined and reorganized using a Python script to create the final report. As part of this reorganization, it is determined where repeating data points like subjects and positional prefixes start and end.

[^1]: Some content in this section was derived from the following work: National Archives and Records Administration. *NARA's Statement on Potentially Harmful Content.* 17 June 2022. Archived 16 January 2025 by Wayback Machine. <https://web.archive.org/web/20250116215713/https://www.archives.gov/research/reparative-description/harmful-content>.
