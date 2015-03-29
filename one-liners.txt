# Extract FASTA seq (one line) based on keyword in title
gawk '/^>/{if(($0~/16[89]/)||($0~/17[012]/)){h=$0;print}}!/[^ACGTN]/{if(h){print;h=""}}' input.fas > output.fas

# Extract multiline FASTA seq by title pattern
$ gawk '/^>/{p=0}/^>.+[^L]$/{print;p=1}!/[^ACGTN]/&&p{print}' input.fas > output.fas

# Multiline FASTA to one line
gawk 'BEGIN{ORS=""}/^>/{print"\n";print;print"\n"}!/[^ACGTN-]/{print}' input.fas > output.fas

# Trim after space in FASTA title
sed "/^>/s/ .*//" input.fas > output.fas

# Delete string between || and trim after space in FASTA title
sed "/^>/s/| .*//" | sed "/^>/s/\([a-z0-9]\+|\)\{3\}//" input.fas > output.fas

# Correct coordinates in BED file
gawk 'BEGIN{OFS="\t";val=2649520}/^chrY/{$2+=val;$3+=val;$7+=val;$8+=val}{print}' input.bed > output.bed

# Add something to the end of FASTA sequence title
gawk '/^>/{print $0 "\174" "S3";next}{print}'

# CHAR \174 | (vertical line)
# CHAR \76  > (greater then sign)

# FASTQ to FASTA
gawk 'BEGIN{n=1}n==1{sub(/^@/,">");print}n==2{print}n==4{n=0}{n++}' input.fastq > output.fas

# Trim FASTQ to 75
gawk 'BEGIN{n=1;l=75}n==1||n==3{print}n==2{print substr($0,1,l)}n==4{print substr($0,1,l);n=0}{n++}'

# Trim FASTQ with len >= 75 to 100
gawk 'BEGIN{n=1;l=75;c=100}n==1||n==3{a[n]=$0}(n==2||n==4)&&length>=l{printf("%s\n%s\n",a[n-1],substr($0,1,c))}n==4{n=0}{n++}'

# Trim all FASTQ files with len >= 75 to 100
find -type f -name "*.fastq" -exec sh -c "gawk 'BEGIN{n=1;l=75;c=100}n==1||n==3{a[n]=\$0}(n==2||n==4)&&length>=l{printf(\"%s\n%s\n\",a[n-1],substr(\$0,1,c))}n==4{n=0}{n++}' {} > $(dirname {})/$(basename {}.fastq)-len_gt75-trim100.fastq" \;

# From PERCON output get AS (summa2) and time
gawk '/summa2=/{printf("%s %d",FILENAME,$2)}/calculation time=/{printf(" %d\n",$4)}' prcn > out

# From PERCON output calc AI SF1: (summa ? number) / (summa summarum)
gawk '/? number=/{sqn=$3}/summa summarum=/{ss=$3;printf("%s %.2f\n",FILENAME,sqn/ss);nextfile}' $(ls *.prcn|xargs)

# Extract seq from zipped FASTQ
find . -type f -name "*.fastq.gz" -exec sh -c 'zless {} | gawk "BEGIN{n=1}n==1{sub(/^@/,\">\");print}n==2{print}n==4{n=0}{n++}" > $(basename {} .fastq.gz).fas' \;

# Slice long FASTA sequence into pieces
gawk 'BEGIN{FS="";n=1}/>/{h=$0;print h "_" n}!/[^ACGTN]/{for(i=1;i<=NF;i++){printf("%s",$i);j++;if(j%100==0){print "\n" h "_" ++n;j=0}}}' input.fas > out-by100bp.fas

# Sort FASTA sequences by coordinate in title
sed "$!N;s/d/ d/;s/_/_ /;s/\n/ $/" XPA1D.fas | sort -n -k 2 | sed "s/ //g;s/\$/\n/" > XPA1D-s.fas

# Sort BED by contig and first coordinate (w/o header)
sort -k 1.4,1V -k 2,2n input.bed > output.bed

# Sort tab delimited file with header row
(read -r; printf "%s\n" "$REPLY"; sort -t $'\t' -k1V) < input > output.sorted

# Translate lines UNIX -> DOS (winxp)
sed.exe -e "s/\r/\n/" input.gb > output.gb
# Translate lines UNIX -> DOS (win7, linux)
sed.exe -e "s/$/\r/" input.gb > output.gb

# MIIP XLS to FASTA
gawk "/^[0-9]+/{print \"\76\"$2\"\174\"$3\"_\"$4\"\n\"toupper($6)}" out-miip.xls > seq.fas

# Convert *my* GenBank to FASTA
gawk "/LOCUS/{print $2}!/[^ACGTN[:blank:]]/{sub(/^[[:blank:]]+/,\"\",$0);print}" input.gb > output.fas

# Convert FASTA files to GenBank
find -type f -name "*.fas" -exec sh -c '/home/al/fas2gb/fas2gb {} > $(dirname {})/$(basename {} .fastq.gz.fas).gb' \;

# Print FASTA files (with one seq in each) to one line
find -type f -name "*.fas" -exec sh -c "gawk 'NR==1{print}NR>1{ORS=\"\";print}' {} > $(dirname {})/$(basename {} .fas)-oneline.fas" \;

# Make BAT for PERCON
find -type f -name "*.gb" -printf "c:\\deep720\\perconTask.exe -db %h\\%f -rs 0.20 -fasMon 1\n" > percon.bat

# Extract FASTA sequences from PERCON output
gawk 'BEGIN{ORS=""}/^seq ID/{f=1;getline}f&&/^\76/{gsub(/\r/,"");if(n++)printf"\n";print;printf"\n"}!/[^ACGTN\r]/{gsub(/\r/,"");print}END{printf"\n"}' $(ls -v *.prcn | xargs) > seq.fasta

# Count seq in some FASTA files
find -type f -name "*.fas" -exec sh -c 'gawk "/^>/{n++}END{print FILENAME,n}" {}' \;

# Count seq by names in some FASTA files (linux)
gawk '/^>/{a[substr($1,2,24)]++}END{for(i in a){print i,a[i];n+=a[i]}print n}' $(ls *.fas | xargs)

# Print each FASTA seq to separate file
gawk '/^>/{n=substr($1,2)}{print > "contigs/" n ".fas"}' input.fas

# Replace string in all files
find -type f -name "*.fas" -exec sed -i'' -e 's/_A_D0B5GACXX//' {} \;

# Combine files based name list (prepare index)
cat $(cat ../list.txt | xargs -i echo {}.fasta) > output

# Add new names from list to FASTA seq title (or rename)
gawk -v f=list.txt 'BEGIN{while((getline<f)>0){a[$2]=$1}close(f)}/^>/{s=substr($1,2);if(s in a){print $1"|"a[s]}else{print}}!/[^ACGTN]/{print}' input.fas > output.fas

# Print accesion list by interval
gawk 'BEGIN{s="JQ685";n=186;m=223;for(i=n;i<=m;i++){if(i>n)printf(",");printf("%s%d", s, i)}}'

# Sort by array value without losing indices
gawk 'BEGIN{a["GR"]=1;a["T2R"]=3;a["ZR3"]=1;a["YY"]=2;for(i in a)printf("%s\t%d\n",i,a[i]) | "sort -k2 -n"}'

# Count and print names of uniq sequences in PERCON output
gawk "/^seq ID/{f=1;getline}/^invalid/{exit}f&&/^[^ ]/{if($1~/\./){a[substr($1,1,index($1,\".\")-1)]++}else{a[$1]++}}END{for(i in a)print i;print length(a)}" input.prcn > output

# Extract only masked sequences from Repeat Masker FASTA output
gawk 'BEGIN{ORS=""}/^>/{if((s)&&(s~/[acgtn]/)){print h s}h="\n"$1"\n";s=""}!/[^AaCcGgTtNn]/{s=s$1}END{if((s)&&(s~/[acgtn]/)){print h s}}' RM.masked.txt > RM.masked.fas

# Extract only unmasked sequences from Repeat Masker FASTA output
gawk 'BEGIN{ORS=""}/^>/{if((s)&&(s!~/[acgtn]/)){print h s}h="\n"$1"\n";s=""}!/[^AaCcGgTtNn]/{s=s$1}END{if((s)&&(s!~/[acgtn]/)){print h s}}' RM.masked.txt > RM.unmasked.fas

# Wrap lines in all FASTA files at 60 characters
$ find . -type f -name "*.fasta" -exec sh -c 'fold -w 60 {} > $(basename {} .fasta).fas' \;

# View some fields in SAM
gawk '{print $1,$4,$5,$6,$(NF-2),$(NF-1)}' input.sam | less -SN

# Sort SAM by MD:Z field
gawk '{print $(NF-1),$0}' input.sam | sort -k1Vr | gawk '{sub(/^.+? /,"",$0);print}' > ouput.sam

# Sort SAM by NM:i field
gawk '{print $(NF-2),$0}' input.sam | sort -k1V | gawk '{sub(/^.+? /,"",$0);print}' > output.sam

# Print FASTA seq from SAM with MD:Z field
gawk '{print "\76"$1"\174"$(NF-1);print$10}' input.sam > output.fas

# Convert all SAM files to BAM (sort & index)
ls *.sam | parallel "samtools view -bS {} | samtools sort - {.}-sorted; samtools index {.}-sorted.bam"

# Convert all BAM files to SAM
find -type f -name "*.bam" -exec sh -c '~/samtools/samtools view -h {} -o $(dirname {})/$(basename {} .bam).sam' \;

# Convert all SAM files to FASTQ
find -type f -name "*.sam" -exec sh -c 'gawk "!/^@/{print \"@\"\$1\"\n\"\$10\"\n+\n\"\$11}" {} > $(dirname {})/$(basename {} .sam).fastq' \;

# Calculate GC-content
gawk '!/[^ACGTN]/{split($0,a,"");for(i in a)b[a[i]]++}END{printf("%.2f%%\n",(b["G"]+b["C"])/(b["A"]+b["T"]+b["G"]+b["C"])*100)}' input.fasta

# Delete txt files except those which name has a "-sort".
find -maxdepth 1 -type f \( -iname "*.txt" ! -iname "*-sort*.txt" \) -exec rm {} \;

