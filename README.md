# potatometabiome_endophytes
# Impact of growing conditions and potato cultivars on the diversity of endophytic microbial communities
#Sequences for this analysis are deposited in (NCBI) Sequence Read Archive (SRA) under the accession number PRJNA1140627
#For the analysis of the ITS region (Fungal data set), we used PIPITS. # this script was build by Xiu Jia followed the script from Stefanie Vink. For more details please go to https://github.com/hsgweon/pipits
#Create the job file with the script. FastQ_ITS_20220408 is a folder containing the original sequences. Create a output file named ITS for storing tthe files#########################
# Submit the job to the cluster
#!/bin/sh
#SBATCH --time=0-48:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --job-name=ITS2-pipits
#SBATCH -o pipits_%j.out
#SBATCH -e pipits_%j.error
#SBATCH --partition=regular
#SBATCH --mem 150GB
#SBATCH --mail-user=j.nepal@rug.nl
#SBATCH --mail-type=ALL
module load PIPITS/3.0 
cd /scratch/p312531/ITS/
pispino_createreadpairslist -i /scratch/p312531/ITS/FastQ_ITS_20220408/ -o /scratch/p312531/ITS/readpairslist.txt #quick, can be done interactively. 
pispino_seqprep -i /scratch/p312531/ITS/FastQ-ITS_20220408/ -o /scratch/p312531/ITS/out_seqprep -l /scratch/p312531/ITS/readpairslist.txt #--FASTX-n ##quick-ish, can be done interactively# #other options: #--forwardreadsonly #--joiner_method PEAR (use Pear to join), --FASTX-n (remove seqs with an N)!
pipits_funits -i /scratch/p312531/ITS/out_seqprep/prepped.fasta -o /scratch/p312531/ITS/out_funits -x ITS2 -v -t 8 # options -r (retain intm files) ## this step takes a long time: >  hours for 1M input seqs when running interactively (i.e. single thread)
pipits_process -i /scratch/p312531/ITS/out_funits/ITS.fasta -o /scratch/p312531/ITS/out_process -v -t 8 -l readpairslist.txt
pipits_funguild.py -i /scratch/p312531/ITS/out_process/otu_table.txt -o /scratch/p312531/ITS/out_process/otu_table_funguild.txt # this converts the otu table into a format that can be used by funguild

# ##############Qiime command prepared by Jyotsna Nepal for the analysis of bacterial community data/16S of potatoMETAbiome project###########
###Load bioconda and qiime2 before running the commands below########
#/zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/RAWDATA/16SROOTS1STPRIORITY11072022/ is a folder containing the raw sequences./zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/ is a directory for the folder to store output files
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/RAWDATA/16SROOTS1STPRIORITY11072022/ \
  --input-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/demux-paired-end.qza  

qiime demux summarize \
  --i-data /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/demux-paired-end.qza \
  --o-visualization /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/demux-paired-end.qzv

# Remove primer seq
  qiime cutadapt trim-paired \
    --i-demultiplexed-sequences /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/demux-paired-end.qza \
    --p-cores 16 \
    --p-front-f GTGYCAGCMGCCGCGGTAA \
    --p-front-r GGACTACNVGGGTWTCTAAT \
    --o-trimmed-sequences /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/demux-PE-trim-$SOURCE.qza

# Visualize trimmed sequences
 qiime demux summarize \
    --i-data /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/demux-PE-trim-$SOURCE.qza \
    --o-visualization /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/demux-PE-trim-$SOURCE.qzv


#!/bin/bash
#
#### JOBSEQUENTIEL ####
# NOMS DU JOB ET DU FICHIER DE SORTIE. Decommenter si besoin.
#SBATCH --job-name=DADA2   
#SBATCH --output=DADA2SMP.out             
#
# NOTIFICATIONS PAR MAIL (debut, fin, echec d'un job). Decommenter si besoin.
#SBATCH --mail-type=ALL
#SBATCH --mail-user=jyotsna.nepal@univ-pau.fr
#
# EXECUTION DANS LE REPERTOIRE COURANT
#SBATCH --workdir=.
#
# DUREE MAXIMALE DU JOB. Format : jours-heures:minutes:secondes
#SBATCH --time=1-05:0:0
#
# NOMBRE DE COEURS
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#
# MEMOIRE DEMANDEE PAR COEUR (Mo)
#SBATCH --mem-per-cpu=4700
# Pour connaitre la memoire et l'etat des noeuds ("PARTITION,HOSTNAMES,SOCKETS,CORES,MEMORY,STATE") :"
#   sinfo -a --format=%P,%n,%X,%Y,%m,%t
#
# ACCOUNT
# Pour connaitre les accounts auxquels vous avez acces, taper la commande :
#   sacctmgr list user withassoc name=votre_user format=user,account,defaultaccount
#SBATCH --account=smp
#
# PARTITION
# Pour connaitre la liste des partitions disponibles, taper la commande :
#   sinfo
#SBATCH --partition=smp
#
# CHARGEMENT DES MODULES
. /etc/profile.d/modules.sh
module purge
module load bioconda-tools/3
source activate qiime2-2021.
#
# EXECUTION
mkdir tmp
export TMPDIR="/zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/" 
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/demux-PE-trim-$SOURCE.qza  \
  --o-table /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-dna.qza \
  --o-representative-sequences /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rep-seqs-dna.qza \
  --p-trim-left-f 0 \
  --p-trim-left-r 0 \
  --p-trunc-len-f 160 \
  --p-trunc-len-r 200 \
  --p-chimera-method consensus \
  --o-denoising-stats /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/stats-dada2-dna.qza


qiime feature-table tabulate-seqs \
  --i-data /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rep-seqs-dna.qza \
  --o-visualization /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rep-seqs.qzv

qiime metadata tabulate \
  --m-input-file /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/stats-dada2-dna.qza \
  --o-visualization /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/stats-dada2.qzv

qiime feature-table summarize \
  --i-table /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-dna.qza \
  --o-visualization /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table.qzv 


### Taxonomic analysis ###

# Assign taxonomy by silva database
qiime feature-classifier classify-sklearn \
  --i-classifier /zfs/home/user/j/jnepal/qiime2/16SINVITRO_11GENGRONINGEN_WORKDIR/silva-138-99-515-806-nb-classifier.qza \
  --i-reads /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rep-seqs-dna.qza \
  --o-classification /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/taxonomy.qza

# Summerize taxonomy info
qiime metadata tabulate \
  --m-input-file /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/taxonomy.qza \
  --o-visualization /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/.qzv

# Filter feature table
qiime taxa filter-table \
--i-table  /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-dna.qza \
--i-taxonomy /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/taxonomy.qza \
--p-include p__ \
--p-exclude mitochondria,chloroplast \
--o-filtered-table /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-with-phyla-no-mitochondria-chloroplast.qza

Remove Archaea
qiime taxa filter-table \
--i-table /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-with-phyla-no-mitochondria-chloroplast.qza \
--i-taxonomy /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/taxonomy.qza \
--p-exclude "k__Archaea" \
--o-filtered-table /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-with-phyla-no-mitochondria-chloroplasts-archaea.qza


Filter Eukaryota
qiime taxa filter-table \
--i-table /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-with-phyla-no-mitochondria-chloroplasts-archaea.qza \
--i-taxonomy /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/taxonomy.qza \
--p-exclude "k__Eukaryota" \
--o-filtered-table /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-with-phyla-no-mitochondria-chloroplasts-archaea-eukaryota.qza

qiime feature-table filter-features \
  --i-table /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-with-phyla-no-mitochondria-chloroplasts-archaea-eukaryota.qza \
  --p-min-frequency 2 \
  --o-filtered-table /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-filtered.qza

qiime feature-table summarize \
  --i-table /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-with-phyla-no-mitochondria-chloroplast.qza \
  --o-visualization /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-with-phyla-no-mitochondria-chloroplast.qzv \
  --m-sample-metadata-file /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/METADATA3countries.txt


qiime feature-table summarize \
  --i-table /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-with-phyla-no-mitochondria-chloroplasts-archaea-eukaryota.qza \
  --o-visualization /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-with-phyla-no-mitochondria-chloroplasts-archaea-eukaryota.qzv \
  --m-sample-metadata-file /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/METADATA3countries.txt

qiime feature-table summarize \
  --i-table /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-filtered.qza \
  --o-visualization /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-filtered.qzv \
  --m-sample-metadata-file /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/METADATA3countries.txt

Filter from sequences
Remove features that contain mitochondria or chloroplast
qiime taxa filter-seqs \
--i-sequences /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rep-seqs-dna.qza \
--i-taxonomy /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/taxonomy.qza \
--p-include p__ \
--p-exclude mitochondria,chloroplast \
--o-filtered-sequences /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rep-seqs-with-phyla-no-mitochondria-chloroplast.qza

Remove Archaea
qiime taxa filter-seqs \
--i-sequences /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rep-seqs-with-phyla-no-mitochondria-chloroplast.qza \
--i-taxonomy /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/taxonomy.qza \
--p-exclude "k__Archaea" \
--o-filtered-sequences /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rep-seqs-with-phyla-no-mitochondria-chloroplasts-archaea.qza


Remove Eukaryota
qiime taxa filter-seqs \
--i-sequences /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rep-seqs-with-phyla-no-mitochondria-chloroplasts-archaea.qza \
--i-taxonomy /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/taxonomy.qza \
--p-exclude "k__Eukaryota" \
--o-filtered-sequences /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rep-seqs-with-phyla-no-mitochondria-chloroplasts-archaea-eukaryota.qza



qiime feature-table tabulate-seqs \
  --i-data /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rep-seqs-with-phyla-no-mitochondria-chloroplasts-archaea-eukaryota.qza \
  --o-visualization /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rep-seqs-filtered.qzv

qiime tools export \
  --input-path /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/table-filtered.qza \
  --output-path /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/exported-table-filtered



qiime feature-classifier classify-sklearn \
  --i-classifier /zfs/home/user/j/jnepal/qiime2/16SINVITRO_11GENGRONINGEN_WORKDIR/silva-138-99-515-806-nb-classifier.qza \
  --i-reads /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rep-seqs-with-phyla-no-mitochondria-chloroplasts-archaea-eukaryota.qza \
  --o-classification /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/filtered-taxonomy.qza


# Summerize taxonomy info
qiime metadata tabulate \
  --m-input-file /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/filtered-taxonomy.qza \
  --o-visualization /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/filtered-taxonomy.qzv

# Export taxonomy
qiime tools export \
  --input-path /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/filtered-taxonomy.qza \
  --output-path /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/EXPORTED_TAXONOMYfiltered


biom convert -i /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/exported-table-filtered/feature-table.biom -o /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/exported-table-filtered/feature-table-filtered.tsv --to-tsv


### Generate a phylogenetic tree using the filtered represented sequences
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rep-seqs-with-phyla-no-mitochondria-chloroplasts-archaea-eukaryota.qza  \
  --o-alignment /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/aligned-rep-seqs.qza \
  --o-masked-alignment /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/masked-aligned-rep-seqs.qza \
  --o-tree /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/unrooted-tree.qza \
  --o-rooted-tree /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rooted-tree.qza

qiime tools export \
  --input-path /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/rooted-tree.qza \
  --output-path /zfs/home/user/j/jnepal/qiime2/16S1STPRIORITY/1stpaper16SPL_NL_GER_WORKDIR/exported-tree
