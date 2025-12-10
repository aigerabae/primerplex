At prom@PCA100416:/data/rkalendar/primerplex:

```bash
docker pull aakechin/ngs-primerplex:1.3.4
wget http://hgdownload.soe.ucsc.edu/goldenPath/hg19/bigZips/hg19.2bit
wget http://hgdownload.cse.ucsc.edu/admin/exe/linux.x86_64/twoBitToFa
```

Didn't install yet:
```bash
wget ftp://ftp.ncbi.nih.gov/snp/organisms/human_9606_b151_GRCh37p13/VCF/common_all_20180423.vcf.gz
wget ftp://ftp.ncbi.nih.gov/snp/organisms/human_9606_b151_GRCh37p13/VCF/common_all_20180423.vcf.gz.tbi
wget ftp://ftp.ncbi.nih.gov/snp/organisms/human_9606_b151_GRCh38p14/VCF/common_all_20180418.vcf.gz
wget ftp://ftp.ncbi.nih.gov/snp/organisms/human_9606_b151_GRCh38p14/VCF/common_all_20180418.vcf.gz.tbi
wget http://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/hg38.2bit
```

```bash
docker run -it --entrypoint 'bash' -v '/data/rkalendar:/home' my-docker-image
cd home
cd primerplex
./twoBitToFa hg19.2bit ucsc.hg19.fa
apt install python3-pip
apt install wget
wget https://www.anaconda.com/download/success/
chmod +x Anaconda3-2025.06-0-Linux-x86_64.sh
./Anaconda3-2025.06-0-Linux-x86_64.sh
~/anaconda3/bin/conda init bash
```

In a new window:
```bash
docker commit -m "Installed new software" d96c41999cd7 aygerim_general
docker run -it --entrypoint 'bash' -v '/data/rkalendar:/home' aygerim_general
conda create --name bwa
conda activate bwa
conda install bioconda::bwa
bwa index ucsc.hg19.fa
```

Getting back to testing the software:
```bash
docker run -it --entrypoint 'bash' --name ngs_primerplex_ref -v '/data/rkalendar:/home' aakechin/ngs-primerplex:1.3.4
# I edited out because 1 line had = instead of == and saved as test1.py
python3 /NGS-PrimerPlex/test1.py --wgref ucsc.hg19.fa.bwt --ref ucsc.hg19.fa.bwt
doesnt work! need to download and do the genbank sequence stuff
