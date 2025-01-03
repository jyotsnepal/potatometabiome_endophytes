# potatometabiome_endophytes
Impact of growing conditions and potato cultivars on the diversity of endophytic microbial communities
For the analysis of the ITS region (Fungal data set), we used PIPITS. # this script was build by Xiu Jia followed the script from Stefanie Vink. For more details please go to https://github.com/hsgweon/pipits
######################################################################################################################Submit the job to the cluster#########################################
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

