**NGS Coursework I**


**Question 1**

A fastqc report of the trimmed negative is shown below:
![perbase.PNG](https://www.dropbox.com/s/q2gz85f9wesenrv/perbase.PNG?dl=0&raw=1)
Low scores can be  observed at both the beginning and end of the read. The quality of calls on most platforms will degrade as the run progresses, so it is common to see base calls falling into the orange area towards the end of a read. However, the base calls falls abruptly at both ends. 
![n_content.PNG](https://www.dropbox.com/s/6ip211zo7cz86id/n_content.PNG?dl=0&raw=1)

Furthermore, the per base N content analysis suggests a high content of the ambiguous character N at both ends. For this reason, the file was remapped first by trimming the last 5 bases at the 5’ end and the last 4 bases at the 3’ end end to end using the default sensitive mode, then by using the --fast preset mode and last by using the --very-sensitive mode.The bowtie stats for the 2 alignements are shown below:
**--sensitive mode**
![normal.PNG](https://www.dropbox.com/s/51chflq0arzmvua/normal.PNG?dl=0&raw=1)

**--fast**
![fast.PNG](https://www.dropbox.com/s/3jp3y2n79tbtwnx/fast.PNG?dl=0&raw=1)

**--very-sensitive**
![very_Sens.PNG](https://www.dropbox.com/s/slnbjdzjg8qeru2/very_Sens.PNG?dl=0&raw=1)
The bowtie results show an increase of overall alignment rate  using the --very-sensitive-local mode(99.99% overall alignment rate) and a decrease using the --fast mode(99.94% overall alignment rate).


```sh
#Create Bowtie genome index
/s/software/anaconda/python3/bin/bowtie2-build ${st_path}/course_materials/genomes/AFPN02.1/AFPN02.1_merge.fasta AFPN02.1_merge

cd${st_path}
mkdir q1
cd q1


#Align with Bowtie 2 end-to-end and trim 5' end and 3' end using default
time /s/software/anaconda/python3/bin/bowtie2 --end-to-end -x ${st_path}/course_materials/genomes/AFPN02.1/AFPN02.1_merge -q -5 5 -3 4 ${st_path}/course_materials/fastq/trimmed_Negative.fq  -S Negative_5_3.sam 2> Negative_bowtie_stats_5_3.txt

#Convert .sam to .bam and flagstat analysis 
/s/software/samtools/v1.9/bin/samtools sort Negative_5_3.sam > Negative_5_3.bam
/s/software/samtools/v1.9/bin/samtools flagstat Negative_5 > Long_flagstat.txt

#Align with Bowtie 2 end-to-end using --fast mode
time /s/software/anaconda/python3/bin/bowtie2 --end-to-end -x ${st_path}/course_materials/genomes/AFPN02.1/AFPN02.1_merge -q -5 5 -3 4 ${st_path}/course_materials/fastq/trimmed_Negative.fq  -S Negative_5_3_fast.sam 2> Negative_bowtie_stats_5_3_fast.txt

#Convert .sam to .bam and flagstat analysis 
/s/software/samtools/v1.9/bin/samtools sort Negative_5_3_fast.sam > Negative_5_3_fast.bam
/s/software/samtools/v1.9/bin/samtools flagstat Negative_5_3_fast.bam > Negative_5_3_fast_flagstat.txt


#Align with Bowtie 2 end-to-end using --very-sensitive mode
time /s/software/anaconda/python3/bin/bowtie2 --very-fast -x ${st_path}/course_materials/genomes/AFPN02.1/AFPN02.1_merge -q -5 5 -3 4 ${st_path}/course_materials/fastq/trimmed_Negative.fq  -S Negative_5_3_very_sensitive.sam 2> Negative_bowtie_stats_5_3_very_sensitive.txt


#Convert .sam to .bam and flagstat analysis 
/s/software/samtools/v1.9/bin/samtools sort Negative_5_3_very_sensitive.sam > Negative_5_3_very_sensitive.bam
/s/software/samtools/v1.9/bin/samtools flagstat Negative_5_3_very_fast.bam > Negative_5_3_very_fast_flagstat.txt

#MultiQC analysis
/s/software/anaconda/python3/bin/multiqc . -f

```

**Question 2**



**fastqc analysis**

**Per base sequence quality**
![perbase_bq.PNG](https://www.dropbox.com/s/thr8ktloxtw9jnq/perbase_bq.PNG?dl=0&raw=1)
Compared to the Negative.fq file, the sequence quality falls in the yellow zone of accetable quality. As with the previous file, the quality drops at both ends.

**Per base N content**
![Ncontent_bq.PNG](https://www.dropbox.com/s/a12undvzuhbhmlh/Ncontent_bq.PNG?dl=0&raw=1)
As with the Negative.fq file, a higher N content can be observed in the first 5 bases at the 5' end and the last 4 bases at the 3' end.


Code used: 
```sh
#run fastqc
thoth.cryst.bbk.ac.uk> /s/software/fastqc/v0.11.8/FastQC/fastqc trimmed_BQ.fq

#re-trim BQfile
-q 5,4 -o /d/projects/u/sm003/cwrk/q2/re_trimmed_BQ.fq trimmed_BQ.fq

#run bowtie2
time /s/software/anaconda/python3/bin/bowtie2 --end-to-end -x ${st_path}/course_materials/genomes/AFPN02.1/AFPN02.1_merge -q ${st_path}/q2/re_trimmed_BQ.fq -S BQ.sam 2> BQ_reprocessed_bowtie_stats.txt

#sam to bam
/s/software/samtools/v1.9/bin/samtools sort BQ.sam > re_trimmed_BQ.bam

#Remapping using more relaxed parameters
time /s/software/anaconda/python3/bin/bowtie2 --local -D 20 -R 3 -N 1 -L 20 -i S,1,0.5 --mp 3 -x ${st_path}/course_materials/genomes/AFPN02.1/AFPN02.1_merge  -q ${st_path}/q2/re_trimmed_BQ.fq -S BQ_mp4.sam 2> BQ_mp4_bowtie_stats.txt

#sam to bam
/s/software/samtools/v1.9/bin/samtools sort BQ_mp4.sam > BQ_mp4.bam

#MultiQC analysis
/s/software/anaconda/python3/bin/multiqc . -f
```

Reprocessed BQ ststs using default mode
![reprocessed_stats.PNG](https://www.dropbox.com/s/n7ynu1zqa8xkhez/reprocessed_stats.PNG?dl=0&raw=1)

**Discussion**
The mapping error rate and the mapping coverage of the alignments obtained using bowtie2 in the default mode are:
* error rate:     4.632085e-02    # mismatches / bases mapped (cigar)
* mapping coverage:  82.83% : N/A)

In my opinion, a higher coverage is more important than lowering the error rate since by having a low coverage, important genes can be missed out in the bacterium we have sampled. 

The BQ file was remapped again using the --local alignment and a variation(more relaxed) version of the --very--sensitive mode. The parameters used are: 
* --local -D 20 -R 3 -N 1 -L 20 -i S,1,0.5 --mp 3 

A local alignment was performed to allow for soft clipping. The parameters used are those from the --very-sensitive mode preset with minor modifications:
* -N 1 = this was chosen to permit a mismatch in the seed. This is because we are looking to assembly the genome of bacteria that we sampled so it does not have to be very similar to the reference genome. 
* --mp 3 = the maximum penalty for a mismatch was reduced to 3(default is 6).

The new alignment gives the following statistics:

![errornew.PNG](https://www.dropbox.com/s/cf9twha4x8j9qhw/errornew.PNG?dl=0&raw=1)
![bq_mp4_coverage.PNG](https://www.dropbox.com/s/6nyhulhxxn72x6s/bq_mp4_coverage.PNG?dl=0&raw=1)

It can be observed that the mapping coverage has increased greatly from 82.83% to 98.64% and the error rate has increased marginally from 4.632085e-02 to 4.639276e-02 (0.007191e-2). 

**Question 3**

```sh
# Get mapped reads
/s/software/samtools/v1.9/bin/samtools view -F 4 BQ.sam > mapped_BQ.sam

#get unmapped reads
/s/software/samtools/v1.9/bin/samtools view -f 4 BQ.sam > unmapped_BQ.sam

#get unique mappings
/s/software/samtools/v1.9/bin/samtools view -b -q 10  BQ.sam > unique_BQ.sam
/s/software/samtools/v1.9/bin/samtools sort unique_BQ.sam >unique_BQ.bam
/s/software/samtools/v1.9/bin/samtools flagstat unique_BQ.bam > unique_BQ_flagstat.txt

#get mapped reads using bowtie
time /s/software/anaconda/python3/bin/bowtie2 --local -D 20 -R 3 -N 1 -L 20 -i S,1,0.5 --mp 3  --no-unal -x ${st_path}/course_materials/genomes/AFPN02.1/AFPN02.1_merge  -q ${st_path}/q2/re_trimmed_BQ.fq -S BQ_aligned.sam 2> BQ_aligned_bowtie_stats.txt

```













 

