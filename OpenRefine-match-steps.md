
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