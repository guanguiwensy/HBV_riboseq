# HBV_riboseq
A pipeline to process HBV riboseq

cat sample |while read id; do bowtie2 --seedlen=23 --threads 40 --un-gz=$id.fq.filtered.gz -U $id.ribo_trimmed_trimmed.fq -x /home/guanguiwen/data3/reference/human/hg19/riboseq/rst_genome/human_rRNA_MTtRNA_snoRNA.fasta -S $id.rts.sam; done

cat sample |while read id; do STAR --runThreadN $thread --sjdbGTFfile $gtf --outSAMtype  BAM Unsorted  --outFileNamePrefix $id --genomeDir $index  --readFilesIn $id.fq.filtered.gz --readFilesCommand zcat; done


cat sample1|while read id
do

bed=/home/guanguiwen/data3/reference/human/hg19/riboseq/start_codon.bed

script=/home/guanguiwen/data1/GSE135860/clean1/Ribowave-master/scripts/

gtf=/home/guanguiwen/data3/reference/human/hg19/riboseq/exons.gtf

genome_size=/home/guanguiwen/data3/reference/human/hg19/riboseq/STATindex/hg19_hbv.fa.fai

samtools view -H ${id}Aligned.out.bam > header.sam

samtools view -@ 20 ${id}Aligned.out.bam | rg -j 20 -w NH:i:1 | cat header.sam - >${id}.uniq.sam

samtools sort -@ 40 ${id}.uniq.sam -o ${id}.uniq.sort.bam

samtools index ${id}.uniq.sort.bam

read_distribution.py -r hg19_RefSeq.bed12 -i ${id}.uniq.sort.bam >  $id.read_distribution.log

./Ribowave-master/scripts/P-site_determination.sh -i ${id}.uniq.sort.bam -S $bed -o ${id} -n $id -s $script
done

$script/create_track_Ribo.sh -i $id.uniq.sort.bam -G $gtf -g $genome_size -P $id/P-site/$id.psite1nt.txt -o $id -n $id -s $script
