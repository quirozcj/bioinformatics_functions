# Functions to manipulate genome sequences

```py
#!/usr/bin/env python3
"""
Extract multiple regions from a FASTA file.

Usage:
    python extract_regions.py <fasta_file> <regions_file> <output_file>

The regions_file should contain one region per line, with three columns:
    chromosome start end
for example:
    chr1    1000    2000
    chr2    35000   35500
"""

import sys
from pyfaidx import Fasta

def extract_regions(fasta_file, regions_file, output_file):
    # Open the FASTA file using pyfaidx. This builds an index if one does not exist.
    fasta = Fasta(fasta_file, rebuild=False)
    
    with open(regions_file, 'r') as rf, open(output_file, 'w') as outf:
        for line in rf:
            line = line.strip()
            if not line or line.startswith("#"):
                continue  # Skip empty or commented lines
            
            # Parse the region line. Expecting "chromosome start end"
            parts = line.split()
            if len(parts) < 3:
                sys.stderr.write(f"Skipping line (not enough columns): {line}\n")
                continue
            
            chrom = parts[0]
            try:
                start = int(parts[1])
                end = int(parts[2])
            except ValueError:
                sys.stderr.write(f"Skipping line (start or end not integer): {line}\n")
                continue

            # pyfaidx uses 0-indexed slicing but the positions are 1-indexed.
            # To extract from a 1-indexed region [start, end] we use [start-1:end]
            try:
                seq = fasta[chrom][start-1:end].seq
            except KeyError:
                sys.stderr.write(f"Chromosome {chrom} not found in FASTA file.\n")
                continue

            # Write the output in FASTA format with a header indicating the region.
            outf.write(f">{chrom}:{start}-{end}\n")
            # Optionally wrap the sequence to 60 characters per line.
            for i in range(0, len(seq), 60):
                outf.write(seq[i:i+60] + "\n")

if __name__ == '__main__':
    if len(sys.argv) != 4:
        sys.stderr.write("Usage: python extract_regions.py <fasta_file> <regions_file> <output_file>\n")
        sys.exit(1)
    fasta_file = sys.argv[1]
    regions_file = sys.argv[2]
    output_file = sys.argv[3]
    extract_regions(fasta_file, regions_file, output_file)
```
#### Usage:
```sh
python extract_regions.py genome.fasta regions.txt output.fasta
```
