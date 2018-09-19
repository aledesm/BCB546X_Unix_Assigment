## UNIX Assignment
### Alejandro Ledesma

#### 1. Data Inspection

We have different files:

##### 1.1 fang_et_al_genotypes.txt, which consist in a SNP data set including maize, teosinte and tripsacum.
This file is txt format file with very long lines and encoding type file ASCII text. It contains 2783 lines, 2744038 words, and 11051939 characters. The file size is 11 Mb and contains 986 columns.

##### 1.2 snp_position.txt
This file is in format .txt with encoding type ASCII text. It contains 984 lines, 13198 words and 82763 characters. The file size is 84 kb.

##### 1.3 Transpose.awk
This is a .awk format file with 4 kb the encoding type os awk script, ACSII text. It has 15 lines, 46 words and 353 characters.

##### 1.4 UNIX_Assignment.md
This is a md format file with 56 kb of size. This file contain the information about the assignment in a readable format pdf.

##### 1.5 UNIX_Assignment.pdf
This is a .md format file with 8 kb of size. the encoding type is ASCII format. This file contain the information about the assignment in a readable format md (mark down)

##### Commands used in data inspection:
- Size of file: $ du -h (file name)
- type of encode: $ file *
- number of lines, words and characters: $ wc *
- number of columns: $ awk -F "\t" '{print NF; exit}' (file name)
- Observe uniq groups $ sort -k3,3 fang_et_al_genotypes.txt | cut -f 3 | uniq -c 



#### 2. Processing of the data

##### 2.1 Extract data from fang_et_al_genotypes.txt file for Maize and Teosinte:

For maize (Group = ZMMIL, ZMMLR, and ZMMMR)

$ grep -E "(ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt > maize_genotypes.txt

For Teosinte (Group = ZMPBA, ZMPIL, and ZMPJA)

 $ grep -E "(ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt > teosinte_genotypes.txt


##### 2.2 Verify that the split of the files went ok:
$ wc teosinte_genotypes.txt maize_genotypes.txt
  975   961350  3873338 teosinte_genotypes.txt
1573  1550978  6240114 maize_genotypes.txt
2548  2512328 10113452 total


##### 2.3 Extract the header from the genotype file and add it to the extracted files
$ grep "Group" fang_et_al_genotypes.txt > header.txt
$ cat header.txt maize_genotypes.txt > maize_header.txt
$ cat header.txt teosinte_genotypes.txt > teosinte_header.txt


##### 2.4 Add a common column (SNP ID) by removing the Sample ID and JG_OTU
$ cut -f 3-968 maize_header.txt > maize_cut.txt
$ cut -f 3-968 teosinte_header.txt > teosinte_cut.txt


##### 2.5 Transpose the data to have SNP-ID in the first column

$ awk -f transpose.awk maize_cut.txt > transposed_maize.txt
$ awk -f transpose.awk teosinte_cut.txt > transposed_teosinte.txt


##### 2.6 Create a new file with only columns 1 (SNP ID), 3 (Chromosome number) and 4 (SNP position) from the file snp.postion.txt 
$ grep -v "^#" snp_position.txt | cut -f 1,3,4 > snp_position_cut.txt


##### 2.7 Now we will remove the headers from the files
$ grep -v "Group" transposed_maize.txt > maize_no_header.txt
$ grep -v "Group" transposed_teosinte.txt > teosinte_no_header.txt
$ grep -v "SNP_ID" snp_position_cut.txt > snp_no_header.txt


##### 2.8 Now, we will sort the files to be able to join the files
$ sort -k1,1 maize_no_header.txt > maize_sorted.txt
$ sort -k1,1 teosinte_no_header.txt > teosinte_sorted.txt
$ sort -k1,1 snp_no_header.txt > snp_sorted.txt

##### 2.9 Then we need to check if every file was sorted correctly.
$ echo$? maize_sorted.txt
$ echo$? teosinte_sorted.txt
$ echo$? snp_sorted.txt

If it is sorted will show 0 but if itis unsorted will show 1


##### 2.10 So, now that they files are sorted, we need to join them

$ join -t $'\t' -1 1 -2 1 snp_sorted.txt maize_sorted.txt > maize_joined.txt
$ join -t $'\t' -1 1 -2 1 snp_sorted.txt teosinte_sorted.txt > teosinte_joined.txt
 
 
##### 2.11 Once both files are joined we need to sort both files by chromosome number
 
$ sort -k2,2n maize_joined.txt > maize_sorted_by_chr.txt
$ sort -k2,2n teosinte_joined.txt > teosinte_sorted_by_chr.txt


##### 2.12 Create a file where the missing data are shown by question mark (?) symbol

$ sed 's/?/?/g' maize_sorted_by_chr.txt > maize_question_mark.txt
$ sed 's/?/?/g' teosinte_sorted_by_chr.txt > teosinte_question_mark.txt


##### 2.13 Create a file where missing data are shown by dash mark (-) symbol

$ sed 's/?/-/g' maize_sorted_by_chr.txt > maize_dash_mark.txt
$ sed 's/?/-/g' teosinte_sorted_by_chr.txt > teosinte_dash_mark.txt


##### 2.14 Create separate files for each chromosome

$ for i in {1..10}; do awk '$2=='$i'' maize_sorted_by_chr.txt > maize_chr"$i"_question_mark.txt; done
$ for i in {1..10}; do awk '$2=='$i'' maize_dash_mark.txt > maize_chr"$i"_dash_mark.txt; done

$ for i in {1..10}; do awk '$2=='$i'' teosinte_sorted_by_chr.txt > teosinte_chr"$i"_question_mark.txt; done
$ for i in {1..10}; do awk '$2=='$i'' teosinte_dash_mark.txt > teosinte_chr"$i"_dash_mark.txt; done


##### 2.15 Sort files with SNP's ordered based on increasing position values and missing data encoded by symbol (?)

$ for i in {1..10}; do sort -k3,3n maize_chr"$i"_question_mark.txt > maize_chr"$i"_question_mark_final.txt; done

$ for i in {1..10}; do sort -k3,3n teosinte_chr"$i"_question_mark.txt> teosinte_chr"$i"_question_mark_final.txt; done


##### 2.16 Sort files with SNP's ordered based on decreasing position values and missing data encoded by symbol (-)

$ for i in {1..10}; do sort -k3,3nr maize_chr"$i"_dash_mark.txt > maize_chr"$i"_dash_mark_final.txt; done

$ for i in {1..10}; do sort -k3,3nr teosinte_chr"$i"_dash_mark.txt > teosinte_chr"$i"_dash_mark_final.txt; done


##### 2.17 Create 1 file with all SNPs with unknown positions in the genome (these need not be ordered in any particular way)

$ grep "unknown" maize_sorted_by_chr.txt > maize_unknown_position.txt
$ grep "unknown" teosinte_sorted_by_chr.txt > teosinte_unknown_position.txt


##### 2.18 Create 1 file with all SNPs with multiple positions in the genome (these need not be ordered in any particular way)

$ grep "multiple" maize_sorted_by_chr.txt > maize_multiple_position.txt
$ grep "multiple" teosinte_sorted_by_chr.txt > teosinte_multiple_position.txt

##### 2.19 Realocate the files in their particular folder
Once all the files were created, I just create new folders and realocated the files inside the specific folder, the command that I used were:
$ mkdir
$ mv

Also I used the wild card * (to move multiple files at the same time)














