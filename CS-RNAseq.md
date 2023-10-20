# interaction samples
```bash
# (base) kang1234@celia-PowerEdge-T640 Thu Oct 19 15:40:43 ~
mv SchunterC_RNASeq_CPOS-221130-CS-15577b CS_RNASeq
# (base) kang1234@celia-PowerEdge-T640 Thu Oct 19 15:36:09 ~/CS_RNAseq/FastQC
for i in *.zip;do unzip ${i};done
# (base) kang1234@celia-PowerEdge-T640 Thu Oct 19 15:39:32 ~/CS_RNAseq/primary_seq
ll *.fastq.gz|perl -alne '@a=split /\_/,$F[-1];$c=$a[0]."_R".$a[1];print "$F[-1]\t$c"' >../data_list.txt
# (base) kang1234@celia-PowerEdge-T640 Thu Oct 19 15:42:40 ~/CS_RNAseq
find -name "fastqc_data.txt"|perl -ne 'chomp;print "$_\t"'|perl -alne 'print "cat $_ > all.samples.fastqc_data.txt"' >1.sh
sh 1.sh
less Overrep_seq.txt|perl -alne '$i++;$nm=Over.$i;print ">$nm\n$_"' > Overrep_seq.txt.1
mv Overrep_seq.txt.1 Overrep_seq.txt

less data_list.txt|cut -f 2|perl -alne '@a=split /\./, $_;$b=$a[0].".fq.gz";print "$_\t$b" ' >data_list.txt.1
mv data_list.txt.1 data_list.txt

# (base) kang1234@celia-PowerEdge-T640 Thu Oct 19 15:50:12 ~/CS_RNAseq
cp ~/New_caledonia/quality_control.pl ./
mv FastQC fastqc1

vi quality_control.pl # this script still needs to be revised to rename the raw fastq file
# (base) kang1234@celia-PowerEdge-T640 Thu Oct 19 16:06:23 ~/CS_RNAseq
nohup perl quality_control.pl --input data_list.txt --raw_dir primary_seq --trim_dir ~/software/Trimmomatic-0.39 --kraken_lib ~/software/kraken2/library --fastqc ~/software/FastQC/fastqc >quality_control.process 2>&1 &
# 5458
```
