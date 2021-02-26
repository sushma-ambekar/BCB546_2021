# UNIX Assignment

## Data Inspection

### Attributes of `fang_et_al_genotypes`

#### Here is what I used for data inspection 

``` 
$ wc fang_et_al_genotypes.txt
$ cut -f 1-20 fang_et_al_genotypes.txt | column -t | head -n 5
$ awk '{ print $3 }' fang_et_al_genotypes.txt | uniq -c 
```
To check the number of columns, we can use this command:
``` 
$ awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
```
Awk in itself is a simple programming language. In this one liner, it distinguishes tab as our field separator, then executes the command: print 'number of fields' then 'exit' for our data file.

#### By inspecting this file I learned that:

1. There are a total of 2783 lines, 2744038 characters and 11.5 megabytes
2. The file has 986 columns of data 
3. These are the different groups in column 3 and their occurances: 
22 TRIPS
15 ZDIPL
9 ZPERR
17 ZLUXR
10 ZMHUE
900 ZMPBA
34 ZMPJA
75 ZMXCH
69 ZMXCP
7 ZMXNO
4 ZMXNT
41 ZMPIL
6 ZMXIL
1256 ZMMLR
27 ZMMMR
290 ZMMIL

### Attributes of `snp_position`

#### Here is what I used for data inspection 
```
$ wc snp_position.txt 
$ ls -l -h snp_position.txt 
$ cut -f 1-20 snp_position.txt | column -t | head -n 5
$ awk -F "\t" '{print NF; exit}' snp_position.txt 
```

#### By inspecting this file I learned that:

1. This file has 984 lines of data
2. The size of the file is 81K
3. The file has 15 columns of data 

### Data Processing

To select only the samples with the desired maize groups, I used this command: 
``` 
$ egrep -w "Group|ZMMIL|ZMMLR|ZMMMR" fang_et_al_genotypes.txt > maize.txt
```
For teosinte, similarly:
```
$egrep -w "Group|ZMPBA|ZMPIL|ZMPJA" fang_et_al_genotypes.txt > teosinte.txt
```

Then, I transposed the files to switch the SNP_ID row to column using the awk script given:
```
$ awk -f transpose.awk teosinte.txt > transposed_teosinte_genotypes.txt
$ awk -f transpose.awk maize.txt > transposed_maize_genotypes.txt
```
To remove the headers to avoid errors while joining, I used the tail command and directed into a new file. I also sorted these files in preparation for join:
```
$ tail -n +4 transposed_maize_genotypes.txt > transposed_headerless_maize_genotypes.txt
$ tail -n +4 transposed_teosinte_genotypes.txt > transposed_headerless_teosinte_genotypes.txt 
$ sort -k1,1 transposed_headerless_maize_genotypes.txt > sorted_maize_genotypes.txt
$ sort -k1,1 transposed_headerless_teosinte_genotypes.txt > sorted_teosinte_genotypes.txt
```
Realising I could have achieved this with one command, I did that for the snp_position.txt file:
```
$ tail -n +2 snp_position.txt | sort k1,1 > sorted_snp_position.txt
```

In between creating these files, I checked the file using the followinf command with the values of column and rows as needed
```
$ cut -f 1-5 file_name.txt | column -t| head -n 15
```
Using join, I joined the two files required to be joined while simultaneously keeping unpairable lines from file 2 using -a and replacing missing data with ? using -e:
```
$ join -1 1 -2 1 -t $'\t' -a 2 -e ? sorted_snp_position.txt sorted_maize_genotypes.txt > maize_all_chromosomes.txt 
$ join -1 1 -2 1 -t $'\t' -a 2 -e ? sorted_snp_position.txt sorted_teosinte_genotypes.txt > teosinte_all_chromosomes.txt
```

##### All further maize and teosinte files can be found in the directories, maize_files and teosinte_files

For maize_files:

I first selected just the required columns with cut command (i.e, 1,3,4,16-1574). Then, I used awk to select the required chromosome number from column 2 and place it in files called maize_chr(1-10).txt while also arranging the positions in increasing order with sort -k 3 -n. These files can be found in the directory increasing_position_files.
```
$ cut -f 1,3,4,16-1574 maize_all_chromosomes.txt > maize_filtered.txt
$ awk '$2 == 1' maize_filtered.txt | sort -k 3 -n > maize_chr1.txt
```
Next, I utilized the same strategy for selecting chrosmomes, but did an reverse sort with sed to replace ? to - and placed them in files maize_chr(1-10)_rev.txt
These files can be found in the directory decreasing_position_files.
```
$ awk '$2 == 1' maize_filtered.txt | sort -k 3nr | sed 's/?/-/g' > maize_chr1_rev.txt
```
Finally, to select for data with unknown or multiple chromosomes, I used awk to select for such data in column 2 and created the files, maize_unknown.txt and maize_multiple.txt respectively.
```
$ awk '$2 == "unknown"' maize_filtered.txt > maize_unknown.txt
$ awk '$2 == "multiple"' maize_filtered.txt > maize_multiple.txt
```
Similarly, I processed all files for the teosinte group and placed them in similar directories.
To add headers to the files to recognize the columns, I used the following command that edits the file and overwrites the original file:
```
$ echo -e "SNP_ID Chromosome Position Genotype" | cat - filename.txt > /tmp/out && mv /tmp/out filename.txt
```

#### Github
To commit and push the files I had been working on to my repository, I used the following commands
```
$ git status
$ git add .
$ git commit -am "comment"
$ git push origin
