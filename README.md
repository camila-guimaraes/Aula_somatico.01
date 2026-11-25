# Aula_somatico.01
Amostras tumor-normal:  

*WP190 (tumor) e WP191 (normal)*  
*WP017 (tumor) e WP018 (normal)*

## Referências: download do chr13 e chr19 (hg19, UCSC)

### Chr 13
bash
wget -c https://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr13.fa.gz

### Chr 19

bash
wget -c https://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr19.fa.gz


### Concatenar os dois cromossomos e remover o prefixo "chr"
bash
zcat chr13.fa.gz chr19.fa.gz | sed -e "s/chr//g" > hg19.fa


### Índice da referência
bash
samtools faidx hg19.fa


## GATK4

### Dicionário
bash
./gatk-4.2.2.0/gatk CreateSequenceDictionary -R hg19.fa -O hg19.dict


### Criar os intervalos sem N
bash
./gatk-4.2.2.0/gatk ScatterIntervalsByNs -R hg19.fa -O hg19.interval_list -OT ACGT


## Identificar o ID das amostras normais
bash
samtools view -H tumor_wp191.bam | grep '^@RG' | tr '\t' '\n' | grep '^SM:' | sed 's/SM://'

bash
samtools view -H tumor_wp018.bam | grep '^@RG' | tr '\t' '\n' | grep '^SM:' | sed 's/SM://'


## Mutect2
### WP190 e WP191
bash
./gatk-4.2.2.0/gatk Mutect2 \
	-R hg19.fa \
	-I tumor_wp190.bam \
	-I tumor_wp191.bam \
	-normal WP191 \
	--germline-resource af-only-gnomad-chr13-chr19.vcf.gz \
	-O wp190wp191.vcf.gz \
	-L hg19.interval_list


### WP017 e WP018
bash
./gatk-4.2.2.0/gatk Mutect2 \
	-R hg19.fa \
	-I tumor_wp017.bam \
	-I tumor_wp018.bam \
	-normal WP018 \
	--germline-resource af-only-gnomad-chr13-chr19.vcf.gz \
	-O wp017wp018.vcf.gz \
	-L hg19.interval_list

## GetPileupSummaries
### WP190 e WP191
bash
./gatk-4.2.2.0/gatk GetPileupSummaries \
	-I tumor_wp190.bam \
	-V af-only-gnomad-chr13-chr19.vcf.gz \
	-L hg19.interval_list \
	-O tumor_wp190.table


bash
./gatk-4.2.2.0/gatk GetPileupSummaries \
	-I tumor_wp191.bam \
	-V af-only-gnomad-chr13-chr19.vcf.gz \
	-L hg19.interval_list \
	-O tumor_wp191.table


### WP017 e WP018
bash
./gatk-4.2.2.0/gatk GetPileupSummaries \
	-I tumor_wp017.bam \
	-V af-only-gnomad-chr13-chr19.vcf.gz \
	-L hg19.interval_list \
	-O tumor_wp017.table


bash
./gatk-4.2.2.0/gatk GetPileupSummaries \
	-I tumor_wp018.bam \
	-V af-only-gnomad-chr13-chr19.vcf.gz \
	-L hg19.interval_list \
	-O tumor_wp018.table


## Contamination
### WP190 e WP191
bash
./gatk-4.2.2.0/gatk CalculateContamination \
	-I tumor_wp190.table \
	-matched tumor_wp191.table \
	-O contamination190191.table


### WP017 e WP018
bash
./gatk-4.2.2.0/gatk CalculateContamination \
	-I tumor_wp017.table \
	-matched tumor_wp018.table \
	-O contamination017018.table


## FilterMutectCalls
### WP190 e WP191
bash
./gatk-4.2.2.0/gatk FilterMutectCalls \
	-R hg19.fa \
	-V wp190wp191.vcf.gz \
	--contamination-table contamination190191.table \
	-O filtered190191.vcf.gz


### WP017 e WP018
bash
./gatk-4.2.2.0/gatk FilterMutectCalls \
	-R hg19.fa \
	-V wp017wp018.vcf.gz \
	--contamination-table contamination017018.table \
	-O filtered017018.vcf.gz



##  pós-filtragem 
### WP190 e WP191
Variantes totais:
bash
zcat filtered190191.vcf.gz | grep -v "\#" | wc -l


Variantes com PASS:
bash
zcat filtered190191.vcf.gz | grep -v "\#" | grep PASS | wc -l


### WP017 e WP018
Variantes totais:
bash
zcat filtered017018.vcf.gz | grep -v "\#" | wc -l


Variantes com PASS:
bash
zcat filtered017018.vcf.gz | grep -v "\#" | grep PASS | wc -l


## final

| Par tumor-normal | Variantes totais | Variantes PASS |
|---|---|---|
| WP190 / WP191 | 5 | 2 |
| WP017 / WP018 | 3 | 1 |
