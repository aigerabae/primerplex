<img width="158" height="17" alt="image" src="https://github.com/user-attachments/assets/7331b734-ba4b-4d9c-bb6e-c8ad01af5419" />At prom@PCA100416:/data/rkalendar/primerplex:

Downloading fasta files and twobittofa software and pulling multiplex image:
```bash
docker pull aakechin/ngs-primerplex:1.3.4
wget http://hgdownload.soe.ucsc.edu/goldenPath/hg19/bigZips/hg19.2bit
wget http://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/hg38.2bit
wget http://hgdownload.cse.ucsc.edu/admin/exe/linux.x86_64/twoBitToFa
```

Downloading Vcf files:
```bash
wget ftp://ftp.ncbi.nih.gov/snp/organisms/human_9606_b151_GRCh37p13/VCF/common_all_20180423.vcf.gz
wget ftp://ftp.ncbi.nih.gov/snp/organisms/human_9606_b151_GRCh37p13/VCF/common_all_20180423.vcf.gz.tbi
# patches add minor additions so having regular hg38 for fasta and p7 for vcf should be no deal
wget ftp://ftp.ncbi.nih.gov/snp/organisms/human_9606_b151_GRCh38p7/VCF/common_all_20180418.vcf.gz
wget ftp://ftp.ncbi.nih.gov/snp/organisms/human_9606_b151_GRCh38p7/VCF/common_all_20180418.vcf.gz.tbi
```

Making new image with conda, python
```bash
docker run -it --entrypoint 'bash' -v '/data/rkalendar:/home' my-docker-image
cd home
cd primerplex
apt install python3-pip
apt install wget
wget https://www.anaconda.com/download/success/
chmod +x Anaconda3-2025.06-0-Linux-x86_64.sh
./Anaconda3-2025.06-0-Linux-x86_64.sh
~/anaconda3/bin/conda init bash
```

Saving it and installing conda and bwa (for some reason doesn't save, have to reinstall each time):
In a new window:
```bash
docker commit -m "Installed conda" d96c41999cd7 aygerim_general
# note that the sequence afetr message will be different each time!
docker run -it --entrypoint 'bash' -v '/data/rkalendar:/home' aygerim_general
conda create --name bwa
conda activate bwa
conda install bioconda::bwa
```

2bit to fa, fa indexing:
```bash
./twoBitToFa hg19.2bit ucsc.hg19.fa
./twoBitToFa hg38.2bit ucsc.hg38.fa
bwa index ucsc.hg19.fa
bwa index ucsc.hg38.fa
```

Saving conda again:
In a new window:
```bash
docker commit -m "Installed conda" a488183cbc28 aygerim_general
# note that the sequence afetr message will be different each time!
```

Dowloading genbank files (only for hg38 for now):
```bash
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/405/GCF_000001405.40_GRCh38.p14/GCF_000001405.40_GRCh38.p14_genomic.gbff.gz
```

After it's downloaded I need to split them by chromosome and rename to match names in fasta files:
```bash
gunzip GCF_000001405.40_GRCh38.p14_genomic.gbff.gz
docker run -it --entrypoint 'bash' -v '/data/rkalendar:/home' aygerim_general
cd home/primerplex
conda install -c bioconda seqkit
mkdir split_chr
cd split_chr
mv ../GCF_000001405.40_GRCh38.p14_genomic.gbff ./
python3 -m venv venv
source venv/bin/activate
pip install biopython
pip install pandas
```

Spltting:
```python3
from Bio import SeqIO
import pandas as pd

# Parse GenBank with multiple records
stream = SeqIO.parse("GCF_000001405.40_GRCh38.p14_genomic.gbff", format="genbank")
names = pd.read_csv('sequence_report.tsv',sep="\t")
mapping = dict(zip(names["RefSeq seq accession"], names["UCSC style name"]))
# I downloaded genbank sequence report from here (https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000001405.40/) as a tsv file and used it for renaming
# Used column "GenBank seq accession" to rename files from there to "UCSC style name" in sequence_report.tsv

# Print the id of each record and save as new file with 
for rec in stream:
    acc = rec.id  # e.g., "NC_000001.11"

    if acc in mapping:
        outname = f"{mapping[acc]}.gb"
        print(f"Saving {acc} → {outname}")
        SeqIO.write(rec, outname, "genbank")
    else:
        print(f"⚠️ No UCSC name found for {acc}, skipping")
```

I moved everything to a respective folder:
- vcf files into vcf folders for hg19 and hg38,   
- gb files into their respective folkders (renamed split_chr),   
- each annotated genome got its own folder too  

Getting back to testing the software:
```bash
# entering the docker image
docker run -it --entrypoint 'bash' -v '/data/rkalendar:/home' aakechin/ngs-primerplex:1.3.4

# I edited out because 1 line had = instead of == and saved as test1.py (because i diodn't save that image again you need to run this every time)
sed -i 's/if withVcf=True:/if withVcf==True:/g' /NGS-PrimerPlex/test.py

python3 /NGS-PrimerPlex/test.py -wgref /home/primerplex/indexed_ref_hg38/ucsc.hg38.fa -ref /home/primerplex/combined_ref_hg38/
# doesn' work because it doesn't like the unlocalized contigs in my fasta files.
```
