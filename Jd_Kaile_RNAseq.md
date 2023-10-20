# JD_Kaile: RNA-seq
## Data transfer
```bash
# account: u3007075@hpc2021.hku.hk
# passwd : hku371424mnbv
# u3007075@hpc2021 Thu Oct 05 11:46:45 /lustre1/g/sbs_ibeer/kailedata/3_1fastpresult_f
# (base) kang1234@celia-PowerEdge-T640 Thu Oct 05 11:52:44 ~/New_caledonia/Kaile
nohup scp -r u3007075@hpc2021-io1.hku.hk:/lustre1/g/sbs_ibeer/kailedata/3_1fastpresult_f/reads.ALL.*.fq.gz ./ > nohup.out 2>&1
# control + z
bg

# based on the de novo fasta by Trinity
# (base) kang1234@celia-PowerEdge-T640 Thu Oct 05 13:33:46 ~/New_caledonia/Kaile
mv reads.ALL.left.fq.gz All_1.fq.gz
mv reads.ALL.right.fq.gz All_2.fq.gz
export SHARED_DIR=$PWD
function docker_drap_cmd { echo "docker run --rm -e LOCAL_USER_ID=`id -u $USER` -u 1003:1003 -v $SHARED_DIR:$SHARED_DIR -w `pwd` sigenae/drap "; }
`docker_drap_cmd`runDrap -o Sw_runDrap -R1 All_1.fq.gz -R2 All_2.fq.gz -s FR --dbg trinity --dbg-mem 400 --no-trim --norm-mem 400 --no-rate --write
rm 02-trinity.sh
# (base) kang1234@celia-PowerEdge-T640 Thu Oct 05 13:43:05 ~/New_caledonia/Kaile/Sw_runDrap/a-trinity
scp u3007075@hpc2021-io1.hku.hk:/lustre1/g/sbs_ibeer/kailedata/3_1fastpresult_f/trinity_out_dir.Trinity.fasta ./Trinity.fasta
nohup `docker_drap_cmd`runDrap -o Sw_runDrap -1 All_1.fq.gz -2 All_2.fq.gz -s FR --dbg trinity \
          --dbg-mem 400 --no-trim --norm-mem 400 --no-rate --run > Sw_runDrap.process 2>&1 &
# [1] 2102

# (base) kang1234@celia-PowerEdge-T640 Mon Oct 16 11:47:22 ~/New_caledonia/Kaile/Sw_runDrap/a-trinity
scp u3007075@hpc2021-io1.hku.hk:/lustre1/g/sbs_ibeer/kailedata/3_1fastpresult_f/1raw.Trinity.fasta ./
mv 1raw.Trinity.fasta Trinity.fasta
# (base) kang1234@celia-PowerEdge-T640 Mon Oct 16 11:55:22 ~/New_caledonia/Kaile
nohup `docker_drap_cmd`runDrap -o Sw_runDrap -1 All_1.fq.gz -2 All_2.fq.gz -s FR --dbg trinity \
           --dbg-mem 400 --no-trim --norm-mem 400 --no-rate --run > Sw_runDrap.process 2>&1 &
# [1] 8250
```
