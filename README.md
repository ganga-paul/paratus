# Gene Loss Detection Pipeline

## Introduction
This pipeline is designed to identify orthologous genes and detect gene loss in a given set of genomes, starting from soft-masked FASTA files. It consists of three modular parts, each handling a distinct stage of the workflow:

### 1. Prepare Genomes
- Build repeat libraries.
- Soft-mask the genome.

### 2. Make LASTZ Chains
- Align masked genomes to the human reference genome (**hg38**).
- Generate alignment chains.

### 3. TOGA
- Use the alignment chains and human annotation to identify orthologous genes.
- Classify the gene status.

## Pipeline Breakdown

### Step 0: Genome Download and Cleaning
- Download raw genome assemblies.
- Clean and preprocess genomes for downstream analysis.

---

### Part 1: Prepare Genomes
**Step 1:** `BuildDatabase` and `RepeatModeler`  
**Step 2:** `RepeatClassifier`  
**Step 3:** `RepeatMasker`  

---

### Part 2: Make LASTZ Chains
**Step 4:** `Make_lastz_chains`  
- Align masked genomes to the human reference genome (**hg38**).  
- Generate alignment chains.

---

### Part 3: TOGA
**Step 5:** `TOGA`  
- Use the alignment chains and human annotation to identify orthologous genes.  
- Classify gene status.

---

## Repository Links

- **Genome Cleaning Scripts:**  
  [https://github.com/paratusbio/Positive-selection/tree/el-scripts/GENE_LOSS/Genome_Cleaning](https://github.com/paratusbio/Positive-selection/tree/el-scripts/GENE_LOSS/Genome_Cleaning)  

- **Batch Generation Scripts for Each Step:**  
  [https://github.com/paratusbio/Positive-selection/tree/el-scripts/GENE_LOSS](https://github.com/paratusbio/Positive-selection/tree/el-scripts/GENE_LOSS)  

- **Sanity Check Scripts for Each Step:**  
  [https://github.com/paratusbio/Positive-selection/tree/el-scripts/GENE_LOSS/Sanity_checks](https://github.com/paratusbio/Positive-selection/tree/el-scripts/GENE_LOSS/Sanity_checks)  

---


## Step 0: Genome Download and Cleaning
🔄 **Manually Downloading and Scoping Multiple Genomes for Unwanted Chromosomes**  
- Genomes are downloaded from the **S3 path** provided by Paratus using the `aws s3 cp` command.  
- Downloaded genomes are arranged in a **predetermined directory structure** (explained in detail in Step 1).  
- Unwanted chromosomes are filtered out during this step.  
- Genomes are **unzipped** before the next step begins.
```
sudo mkdir -p /shared/input_genomes/paratus-bat/mAetAle1.hap1.cur.20250328/
sudo chmod 777 /shared/input_genomes/paratus-bat/mAetAle1.hap1.cur.20250328/
aws s3 cp s3://paratus-ven-elucidata-collab/positive_selection/input_genomes/paratus-bat/mAetAle1.hap1.cur.20250328.fasta.gz /shared/input_genomes/paratus-bat/mAetAle1.hap1.cur.20250328/mAetAle1.hap1.cur.20250328.fa.gz
sudo chmod 777 /shared/input_genomes/paratus-bat/mAetAle1.hap1.cur.20250328/mAetAle1.hap1.cur.20250328.fa.gz
gunzip -c /shared/input_genomes/paratus-bat/mAetAle1.hap1.cur.20250328/mAetAle1.hap1.cur.20250328.fa.gz > /shared/input_genomes/paratus-bat/mAetAle1.hap1.cur.20250328/mAetAle1.hap1.cur.20250328.fa
```
Once the genomes are downloaded and unzipped they are screened for presence of unwanted chromosomes (defined by Paratus ie. X, Y, MT, Z chromosomes ). 
```
echo "/shared/input_genomes/paratus-bat/mAetAle1.hap1.cur.20250328/mAetAle1.hap1.cur.20250328.fa" >> chromname_checking_0305.txt
grep ">" /shared/input_genomes/paratus-bat/mAetAle1.hap1.cur.20250328/mAetAle1.hap1.cur.20250328.fa | grep -i -e "M" -e "X" -e "Y" -e "Z" >> chromname_checking_0305.txt
```
Once screened the output (chromname_checking_0305.txt here) is manually checked for presence of any chromosome present in the genomes.

🔄 **Automating Unwanted chromosomes removal for Multiple Genomes**
To streamline the process of Unwanted chromosomes removal for multiple genomes, we use a loop that:

## Genome Cleaning Process

- **Reads chromosome names** from a text file:  
  `chromosomes_to_removev3.txt`

- **Reads genome names** (absolute file paths of FASTA files) from:  
  `genomev3.txt`

- **Removes the specifi
