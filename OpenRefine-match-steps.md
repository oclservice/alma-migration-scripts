
First, run both Chatham and Windsor files through the following steps.

1. Create column ISBN10 based on column 020$a with GREL
value.match(/\b([\dX]{10})\b.*/)[0]

2. Create column ISBN13 based on column 020$a with GREL
replace(value.match(/\b(978\d{10}|[\d-]{17})\b.*/)[0],'-','')

3. Create column ISSN based on column 020$a with GREL
value.match(/\bS?(\d{4}-\d{4})\b.*/)[0]

4. Create column ISBN-normalized based on column ISBN13 with GREL
if(isNonBlank(value),value,with('978'+cells['ISBN10'].value[0,9],v,v+((10-(sum(forRange(0,12,1,i,toNumber(v[i])*(1+(i%2*2)) )) %10)) %10).toString()[0] ))

5. On Chatham file
* Rename 001 into BIB
* Rename 245 into Title

6. On Windsor file, create column Chatham-BIBs based on column ISBN-normalized with GREL
forEach(cell.cross("Chatham records","ISBN-normalized"),r,r.cells.BIB.value).join("|")

7. Do the same thing on ISSN:
forEach(cell.cross("Chatham records","ISSN"),r,r.cells.BIB.value).join("|")

8. Combine the two found BIBs columns with GREL
if(isNonBlank(value),value,cells['Chatham-ISSN-BIBs'].value)

9. Create a Chatham title column based on the combined Chatham BIBs column with GREL
forEach(cell.cross("Chatham records","BIB"),r,r.cells.Title.value).join("|")

After the college has confirmed duplicates

1. Reload duplicates file in OpenRefine
2. Split multi valued cells in the Chatham BIBs
3. **Fill down** the Windsor BIBs column (this will copy the Windsor BIB id for lines where there are multiple correspondances)
4. Switch to row mode and reorder rows by Chatham BIB. Make the new sort permanent.
5. **Blank down** the Chatham BIB column. This will blank all duplicate lines based on Chatham BIBs
6. Facet by blank on Chatham BIB, then remove matching rows. This should remove all duplicates.
7. Add a column based on Windsor BIB named Matchpoint with GREL "(dedup20211028)W" + cells['Windsor BIB'].value
8. Export as CSV - this will be the file to feed to the Python routine



Chatham record with bib ID 6393 needed to be manually fixed, as it had 100 with indicators 20 - converted to 10