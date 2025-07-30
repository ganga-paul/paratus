# Step 0: Genome Download and Cleaning
## 🔄Manually Downloading and Scoping Multiple Genomes for Unwanted Chromosomes##  
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

## 🔄Automating Unwanted chromosomes removal for Multiple Genomes##
- To streamline the process of Unwanted chromosomes removal for multiple genomes, we use a loop that:

## Genome Cleaning Process

- **Reads chromosome names** from a text file `chromosomes_to_removev3.txt`

- **Reads genome names** absolute file paths of FASTA files from `genomev3.txt`

- **Removes the chromosomes from genomes** using the custom script `genome_cleaner.sh`


## 📁 Required Input File: `chromosomes_to_removev3.txt` 

This file contains a list of chromosomes names. They are fetched from file chromname_checking_0305.txt after manual checks.

Example contents:
```
SUPER_X
SUPER_Y
SUPER_X_unloc_1
SUPER_Y_unloc_2
SUPER_X_unloc_4
SUPER_X_unloc_3
SUPER_X_unloc_5
SUPER_Y_unloc_1
SUPER_X_unloc_2
SUPER_X
```

## 📁 Required Input File: `genomev3.txt`

This file contains a list of `genome` fasta files absolute paths. They are also fetched from file `chromname_checking_0305.txt` after manual checks.

Example contents:
```
/shared/input_genomes/paratus-bat/mAetAle1.hap1.cur.20250328/mAetAle1.hap1.cur.20250328.fa
/shared/input_genomes/paratus-bat/mArtInt1.hap1.cur.20230911/mArtInt1.hap1.cur.20230911.fa
/shared/input_genomes/paratus-bat/mArtLit.hap1.cur.20230911/mArtLit.hap1.cur.20230911.fa
/shared/input_genomes/paratus-bat/mAseTrd1.HiC.hap1.20240423/mAseTrd1.HiC.hap1.20240423.fa
/shared/input_genomes/paratus-bat/mAntDub1_hap1.cur.20250116/mAntDub1_hap1.cur.20250116.fa
/shared/input_genomes/paratus-bat/mBraNan1.HiC.hap1.decontam.20240328/mBraNan1.HiC.hap1.decontam.20240328.fa
```

## 🚀 Script to remove the chromosomes from genomes
genome_cleaner.sh

```
#!/bin/bash

# Input lists
genome_list="genomev3.txt"
chromosome_list="chromosomes_to_removev3.txt"

# Build the regex pattern from chromosomes
chrom_pattern=$(paste -sd'|' "$chromosome_list")

# Loop through all genomes
while read -r genome; do
    echo "Processing $genome..."

    output_fasta="${genome%.fa}_filtered.fa"
    log_file="${genome%.fa}_removed_chromosomes.log"

    # Reset log
    > "$log_file"

    awk -v pattern="$chrom_pattern" -v logfile="$log_file" '
    BEGIN { remove = 0 }
    /^>/ {
        if ($0 ~ "^>(" pattern ")") {
            print substr($0,2) >> logfile  # log the chromosome name without ">"
            remove = 1
        } else {
            remove = 0
        }
    }
    remove == 0
    ' "$genome" > "$output_fasta"

    echo "Saved filtered genome to $output_fasta"
    echo "Removed chromosomes logged to $log_file"
done < "$genome_list"
```

## 🔍 Explanation

- Reads a list of genome FASTA files (genomev3.txt) and chromosomes to remove (chromosomes_to_removev3.txt)

- Builds a regex pattern from chromosome names for matching headers

- For each genome file, removes sequences whose headers match the pattern

- Saves the filtered genome to *_filtered.fa and logs removed headers to *_removed_chromosomes.log

## ▶️ How to Run

- Create a directory inside `/shared/el-scripts/`

- Add your genome filenames to `genomev3.txt` (one per line)

- Add unwanted chromosome names to `chromosomes_to_removev3.txt`

- Make script executable: `chmod +x genome_cleaner.sh`

- Run: `./genome_cleaner.sh`

- Output FASTAs and logs are saved in their respective directories of those genomes
