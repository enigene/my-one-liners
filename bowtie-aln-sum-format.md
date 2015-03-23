# Format Bowtie alignment summary

``` awk
gawk 'BEGIN { ORS = "" } \
	NR > 2 && !/^\./ { \
		sub(/[^[:digit:]\.%]+/, ""); \
		sub(/ \(/,"\t");sub(/\) .+/, ""); \
		sub(/ .+/, ""); \
		if ($0) { printf "\t"; print } \
		} \
	/^\./ { \
		if (n++) printf "\n"; \
		print } \
' summary
```

## For datasets consisting of unpaired reads:

Source (Bowtie 1):

```
./Z3BC2026-356E-D897-12F0-3158C29975EB.sam
# reads processed: 1391147
# reads with at least one reported alignment: 616264 (44.30%)
# reads that failed to align: 699734 (50.30%)
# reads with alignments suppressed due to -m: 75149 (5.40%)
Reported 616264 alignments to 1 output stream(s)
```

Formatted:

<table><tbody><tr>
<td>./Z3BC2026-356E-D897-12F0-3158C29975EB.sam</td><td>1391147</td><td>616264</td><td>44.30%</td><td>699734</td><td>50.30%</td><td>75149</td><td>5.40%</td><td>616264</td>
</tr></tbody></table>

Source (Bowtie 2):

```
./ERX137399/ERR161549/ERR161549
5000000 reads; of these:
  5000000 (100.00%) were unpaired; of these:
    16813 (0.34%) aligned 0 times
    2568427 (51.37%) aligned exactly 1 time
    2414760 (48.30%) aligned >1 times
99.66% overall alignment rate
```

Formatted:

<table><tbody><tr>
<td>./ERX137399/ERR161549/ERR161549</td><td>5000000</td><td>5000000</td><td>100.00%</td><td>16813</td><td>0.34%</td><td>2568427</td><td>51.37%</td><td>2414760</td><td>48.30%</td><td>99.66%</td>
</tr></tbody></table>

## For datasets consisting of pairs:

Source (Bowtie 2):

```
./ERR251248
59363650 reads; of these:
  59363650 (100.00%) were paired; of these:
    1573706 (2.65%) aligned concordantly 0 times
    41775075 (70.37%) aligned concordantly exactly 1 time
    16014869 (26.98%) aligned concordantly >1 times
    ----
    1573706 pairs aligned concordantly 0 times; of these:
      477403 (30.34%) aligned discordantly 1 time
    ----
    1096303 pairs aligned 0 times concordantly or discordantly; of these:
      2192606 mates make up the pairs; of these:
        411715 (18.78%) aligned 0 times
        640737 (29.22%) aligned exactly 1 time
        1140154 (52.00%) aligned >1 times
99.65% overall alignment rate
```

Formatted:

<table><tbody><tr>
<td>./ERR251248</td><td>59363650</td><td>59363650</td><td>100.00%</td><td>1573706</td><td>2.65%</td><td>41775075</td><td>70.37%</td><td>16014869</td><td>26.98%</td><td>1573706</td><td>477403</td><td>30.34%</td><td>1096303</td><td>2192606</td><td>411715</td><td>18.78%</td><td>640737</td><td>29.22%</td><td>1140154</td><td>52.00%</td><td>99.65%</td>
</tr></tbody></table>

In one line:

`gawk 'BEGIN{ORS=""}NR>2&&!/^\./{sub(/[^[:digit:]\.%]+/,"");sub(/ \(/,"\t");sub(/\) .+|\)$/,"");sub(/ .+/,"");if($0){printf"\t";print}}/^\./{if(n++)printf"\n";print}' summary`
