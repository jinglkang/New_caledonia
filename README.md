# New caledonia
## 1. Quality control
```bash
# (base) kang1234@celia-PowerEdge-T640 Mon Mar 13 13:43:31 ~/New_caledonia
vi quality_control.pl
# (base) kang1234@celia-PowerEdge-T640 Mon Mar 13 13:58:25 ~/New_caledonia/rawdata
ll *.fastq.gz|perl -alne '@a=split /\_/,$F[-1];$c=$a[0]."_R".$a[1];print "$F[-1]\t$c"' >../data_list.txt
# (base) kang1234@celia-PowerEdge-T640 Mon Mar 13 14:03:37 ~/New_caledonia
nohup perl quality_control.pl --input data_list.txt --raw_dir rawdata --trim_dir ~/software/Trimmomatic-0.39 --kraken_lib ~/software/kraken2/library --fastqc ~/software/FastQC/fastqc >quality_control.process 2>&1 &
# [1] 8534

# Overrep_seq.txt is null at quality_control.pl line 51, <FIL> line 4667331.
find -name "fastqc_data.txt"|perl -ne 'chomp;print "$_\t"'|perl -alne 'print "cat $_ > all.samples.fastqc_data.txt"' >1.sh
sh 1.sh
# It's empty; it's right; revise the "quality_control.pl" to jump the first round of quality control
less all.samples.fastqc_data.txt|perl -alne 'print if /^[ATCG]{8,}/'|cut -f 1|sort -u > Overrep_seq.txt

# run "quality_control.pl" again
# (base) kang1234@celia-PowerEdge-T640 Thu Mar 16 15:43:27c
less data_list.txt|cut -f 2|perl -alne '@a=split /\./, $_;$b=$a[0].".fq.gz";print "$_\t$b" ' >data_list.txt.1
mv data_list.txt.1 data_list.txt
nohup perl quality_control.pl --input data_list.txt --raw_dir rawdata --trim_dir ~/software/Trimmomatic-0.39 --kraken_lib ~/software/kraken2/library --fastqc ~/software/FastQC/fastqc >quality_control.process 2>&1 &
# [1] 2412
```
## 2. de novo assembly
```bash
# the individual: 66 inds in total
# ClunB: 6 inds; DaruB: 16 inds; DaruG: 16 inds; ZlepB: 10 inds; ZlepG: 17 inds
# ClunB10;ClunB3;ClunB5;ClunB6;ClunB8;ClunB9;
# DaruB10;DaruB11;DaruB12;DaruB18;DaruB19;DaruB1;DaruB20;DaruB25;DaruB26;DaruB27;DaruB28;DaruB3;DaruB5;DaruB6;DaruB8;DaruB9;
# DaruG10;DaruG11;DaruG12;DaruG18;DaruG19;DaruG1;DaruG20;DaruG25;DaruG26;DaruG27;DaruG28;DaruG3;DaruG5;DaruG6;DaruG8;DaruG9;
# ZlepB12;ZlepB14;ZlepB18;ZlepB19;ZlepB20;ZlepB3;ZlepB5;ZlepB6;ZlepB7;ZlepB8;
# ZlepG10;ZlepG12;ZlepG14;ZlepG15;ZlepG16;ZlepG17;ZlepG18;ZlepG19;ZlepG21;ZlepG22;ZlepG3;ZlepG4;ZlepG5;ZlepG6;ZlepG7;ZlepG8;ZlepG9

# transfer data to HPC
# jlkang@hpc2021 Mon Aug 28 09:28:49 /lustre1/g/sbs_schunter/Kang
mkdir New_caledonia
# (base) kang1234@celia-PowerEdge-T640 Mon Aug 28 09:30:41 ~/New_caledonia/kraken
nohup scp -r ClunB* jlkang@hpc2021-io1.hku.hk:/lustre1/g/sbs_schunter/Kang/New_caledonia/ > nohup_ClunB.out 2>&1
nohup scp -r DaruB* jlkang@hpc2021-io1.hku.hk:/lustre1/g/sbs_schunter/Kang/New_caledonia/ > nohup_DaruB.out 2>&1
nohup scp -r DaruG* jlkang@hpc2021-io1.hku.hk:/lustre1/g/sbs_schunter/Kang/New_caledonia/ > nohup_DaruG.out 2>&1
nohup scp -r ZlepB* jlkang@hpc2021-io1.hku.hk:/lustre1/g/sbs_schunter/Kang/New_caledonia/ > nohup_ZlepB.out 2>&1
nohup scp -r ZlepG* jlkang@hpc2021-io1.hku.hk:/lustre1/g/sbs_schunter/Kang/New_caledonia/ > nohup_ZlepG.out 2>&1
# control + z
bg

# jlkang@hpc2021 Mon Aug 28 10:32:43 /lustre1/g/sbs_schunter/Kang/New_caledonia
ll Clun*_1.fq.gz|perl -alne 'my ($nm)=$F[-1]=~/(.*)_1\.fq\.gz/; my $fq2=$nm."_2.fq.gz";$fq1_t.=$F[-1].",";$fq2_t.=$fq2.",";END{$fq1_t=~s/\,$//;$fq2_t=~s/\,$//;print "$fq1_t\n$fq2_t"}'
# ClunB10_1.fq.gz,ClunB3_1.fq.gz,ClunB5_1.fq.gz,ClunB6_1.fq.gz,ClunB8_1.fq.gz,ClunB9_1.fq.gz
# ClunB10_2.fq.gz,ClunB3_2.fq.gz,ClunB5_2.fq.gz,ClunB6_2.fq.gz,ClunB8_2.fq.gz,ClunB9_2.fq.gz
Trinity --full_cleanup --seqType fq --max_memory 490G --CPU 38 --output Clun_trinity --left ClunT_1.fq.gz --right ClunT_2.fq.gz --SS_lib_type FR
```

```script_Clun.cmd
#!/bin/bash
#SBATCH --job-name=Clun_trinity        # 1. Job name
#SBATCH --mail-type=BEGIN,END,FAIL    # 2. Send email upon events (Options: NONE, BEGIN, END, FAIL, ALL)
#SBATCH --mail-user=jlkang@hku.hk     #    Email address to receive notification
#SBATCH --partition=amd               # 3. Request a partition
#SBATCH --qos=normal                  # 4. Request a QoS
#SBATCH --nodes=1                     #    Request number of node(s)
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=32
#SBATCH --mem-per-cpu=6G
#SBATCH --time=7-00:00:00             # 7. Job execution duration limit day-hour:min:sec
#SBATCH --output=%x_%j.out            # 8. Standard output log as $job_name_$job_id.out
#SBATCH --error=%x_%j.err             #    Standard error log as $job_name_$job_id.err

# print the start time
date
Trinity --full_cleanup --seqType fq --max_memory 190G --CPU 32 --output Clun_trinity --left ClunT_1.fq.gz --right ClunT_2.fq.gz --SS_lib_type FR
# print the end time
date
```

```script_cat.cmd
#!/bin/bash
#SBATCH --job-name=Cat_fq        # 1. Job name
#SBATCH --mail-type=BEGIN,END,FAIL    # 2. Send email upon events (Options: NONE, BEGIN, END, FAIL, ALL)
#SBATCH --mail-user=jlkang@hku.hk     #    Email address to receive notification
#SBATCH --partition=amd               # 3. Request a partition
#SBATCH --qos=normal                  # 4. Request a QoS
#SBATCH --nodes=1                     #    Request number of node(s)
#SBATCH --ntasks=6
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G
#SBATCH --time=7-00:00:00             # 7. Job execution duration limit day-hour:min:sec
#SBATCH --output=%x_%j.out            # 8. Standard output log as $job_name_$job_id.out
#SBATCH --error=%x_%j.err             #    Standard error log as $job_name_$job_id.err

# print the start time
date
cat Clun*_1.fq.gz > ClunT_1.fq.gz
cat Clun*_2.fq.gz > ClunT_2.fq.gz
cat Daru*_1.fq.gz > DaruT_1.fq.gz
cat Daru*_2.fq.gz > DaruT_2.fq.gz
cat Zlep*_1.fq.gz > ZlepT_1.fq.gz
cat Zlep*_2.fq.gz > ZlepT_2.fq.gz
# print the end time
```

```bash
# jlkang@hpc2021 Mon Aug 28 14:00:48 /lustre1/g/sbs_schunter/Kang/New_caledonia
module load trinity/2.14.0n
module load samtools
module load bowtie2
module load jellyfish
module load salmon
module load python

sbatch script_cat.cmd
sbatch script_Clun.cmd
```

```bash
# In SNORLAX
# Clun: 6 inds (ClunB10;ClunB3;ClunB5;ClunB6;ClunB8;ClunB9)
# (base) kang1234@celia-PowerEdge-T640 Fri Sep 15 10:55:10 ~/New_caledonia/kraken
cat Clun*_1.fq.gz > ClunT_1.fq.gz
cat Clun*_2.fq.gz > ClunT_2.fq.gz

export SHARED_DIR=$PWD
function docker_drap_cmd { echo "docker run --rm -e LOCAL_USER_ID=`id -u $USER` -u 1003:1003 -v $SHARED_DIR:$SHARED_DIR -w `pwd` sigenae/drap "; }
`docker_drap_cmd`runDrap -o Clun_runDrap -R1 ClunT_1.fq.gz -R2 ClunT_2.fq.gz -s FR --dbg trinity --dbg-mem 400 --no-trim --norm-mem 400 --no-rate --write
# (base) kang1234@celia-PowerEdge-T640 Fri Sep 15 11:13:44 ~/New_caledonia/kraken
`docker_drap_cmd`runDrap -o Clun_runDrap -R1 ClunT_1.fq.gz -R2 ClunT_2.fq.gz -s FR --dbg trinity --dbg-mem 400 --no-trim --norm-mem 400 --no-rate --write
vi 02-trinity.sh # change "--CPU 6" to "--CPU 22"
nohup `docker_drap_cmd`runDrap -o Clun_runDrap \
          -1 ClunT_1.fq.gz \
          -2 ClunT_2.fq.gz \
          -s FR --dbg trinity \
          --dbg-mem 400 --no-trim --norm-mem 400 --no-rate --run > Clun_runDrap.process 2>&1 &
# [1] 7957

# Daru: DaruB (16 inds); DaruG (16 inds)
# DaruB10;DaruB11;DaruB12;DaruB18;DaruB19;DaruB1;DaruB20;DaruB25;DaruB26;DaruB27;DaruB28;DaruB3;DaruB5;DaruB6;DaruB8;DaruB9;
# DaruG10;DaruG11;DaruG12;DaruG18;DaruG19;DaruG1;DaruG20;DaruG25;DaruG26;DaruG27;DaruG28;DaruG3;DaruG5;DaruG6;DaruG8;DaruG9;

# (base) kang1234@celia-PowerEdge-T640 Mon Sep 18 10:52:50 ~/New_caledonia/kraken/Daru_runDrap
cat Daru*_1.fq.gz > DaruT_1.fq.gz
cat Daru*_2.fq.gz > DaruT_2.fq.gz

export SHARED_DIR=$PWD
function docker_drap_cmd { echo "docker run --rm -e LOCAL_USER_ID=`id -u $USER` -u 1003:1003 -v $SHARED_DIR:$SHARED_DIR -w `pwd` sigenae/drap "; }
`docker_drap_cmd`runDrap -o Daru_runDrap -R1 DaruT_1.fq.gz -R2 DaruT_2.fq.gz -s FR --dbg trinity --dbg-mem 400 --no-trim --norm-mem 400 --no-rate --write
# (base) kang1234@celia-PowerEdge-T640 Mon Sep 18 10:53:29 ~/New_caledonia/kraken/Daru_runDrap
vi 02-trinity.sh # change "--CPU 6" to "--CPU 24" && 07-rmbt_editing.sh && 08-rmbt_filtering.sh
nohup `docker_drap_cmd`runDrap -o Daru_runDrap \
          -1 DaruT_1.fq.gz \
          -2 DaruT_2.fq.gz \
          -s FR --dbg trinity \
          --dbg-mem 400 --no-trim --norm-mem 400 --no-rate --run > Daru_runDrap.process 2>&1 &
# [1] 9749

# Zlep: ZlepB (10 inds); ZlepG (17 inds)
# ZlepB12;ZlepB14;ZlepB18;ZlepB19;ZlepB20;ZlepB3;ZlepB5;ZlepB6;ZlepB7;ZlepB8;
# ZlepG10;ZlepG12;ZlepG14;ZlepG15;ZlepG16;ZlepG17;ZlepG18;ZlepG19;ZlepG21;ZlepG22;ZlepG3;ZlepG4;ZlepG5;ZlepG6;ZlepG7;ZlepG8;ZlepG9
# (base) kang1234@celia-PowerEdge-T640 Mon Sep 25 13:13:22 ~/New_caledonia/kraken
cat Zlep*_1.fq.gz > ZlepT_1.fq.gz
cat Zlep*_2.fq.gz > ZlepT_2.fq.gz
export SHARED_DIR=$PWD
function docker_drap_cmd { echo "docker run --rm -e LOCAL_USER_ID=`id -u $USER` -u 1003:1003 -v $SHARED_DIR:$SHARED_DIR -w `pwd` sigenae/drap "; }
`docker_drap_cmd`runDrap -o Zlep_runDrap -R1 ZlepT_1.fq.gz -R2 ZlepT_2.fq.gz -s FR --dbg trinity --dbg-mem 400 --no-trim --norm-mem 400 --no-rate --write
nohup `docker_drap_cmd`runDrap -o Zlep_runDrap \
          -1 ZlepT_1.fq.gz \
          -2 ZlepT_2.fq.gz \
          -s FR --dbg trinity \
          --dbg-mem 400 --no-trim --norm-mem 400 --no-rate --run > Zlep_runDrap.process 2>&1 &
# [1] 26292
```
