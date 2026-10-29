GeneRx

Point-of-care metagenomic pathogen and antimicrobial resistance (AMR) diagnostic prototype.

Upload a sequencing file (FASTQ) from a patient swab, and GeneRx identifies the pathogen, detects resistance genes, and produces a Red / Green antibiotic suggestion in seconds, entirely in the browser. It also screens the patient's own DNA for a variant that makes aminoglycoside antibiotics dangerous.

Table of contents
The problem
How it works
What GeneRx detects
Quick start
Example output
Algorithm details
Project structure
Performance
Roadmap
References

For a severe bacterial infection, hospitals usually start with a broad-spectrum antibiotic while waiting for culture-based susceptibility testing, which commonly takes 48 to 96 hours. If the bacterium carries a resistance gene, the first choice may fail. Sequencing the sample directly can show which resistance genes are present within hours, so the treating team has evidence earlier.

GeneRx explores the software side of that idea: turning raw sequencing reads into a clear, actionable summary.

How it works
FASTQ upload
Parse reads
Canonical 15-mersboth strands
Match againstsignature index
Call genes, species,host variant
Gene-to-druglookup table
Dashboard + PDF report
Parse every read from the FASTQ file.
Break each read into k-mers (k = 15) and match them against an index of resistance-gene and species markers, on either strand.
Call a gene only when enough reads and enough of the marker are covered.
Look up drugs that the detected gene makes ineffective, and drugs that remain reasonable options.
Screen the host for the MT-RNR1 m.1555A>G variant and warn against gentamicin and amikacin if found.
Report on a dashboard with a composition chart, and export a PDF.

Everything runs client-side. No file is uploaded to a server.

What GeneRx detects

Resistance genes

Gene	Typical organism	Mechanism	Ineffective drugs flagged
mecA	Staphylococcus aureus (MRSA)	PBP2a, low-affinity target	Penicillins, oxacillin, cephalexin
blaNDM-1	Klebsiella pneumoniae	Metallo-beta-lactamase	Carbapenems, ceftazidime-avibactam
blaKPC-2	Klebsiella pneumoniae	Serine carbapenemase	Meropenem alone, piperacillin-tazobactam
blaCTX-M-15	Escherichia coli	ESBL	Ceftriaxone, cefotaxime, cefepime
blaVIM-2	Pseudomonas aeruginosa	Metallo-beta-lactamase	Carbapenems, cephalosporins, ceftazidime-avibactam
vanA	Enterococcus (VRE)	Altered cell-wall target	Vancomycin, teicoplanin

Also detected

Species markers: K. pneumoniae, S. aureus, E. coli, P. aeruginosa. A gene found without its species marker is reported as "species not confirmed."
Virus: influenza A (detection only; shows "antibiotics not indicated" and an antiviral option).
Host safety variant: MT-RNR1 m.1555A>G (aminoglycoside-induced hearing loss risk).

Result types: resistance gene found, species found with no panel resistance gene, influenza only, or clean sample. Mixed samples show the main result and list other detected genes.

Resistance genes are matched as gene families. Telling individual alleles apart is outside the current scope.

Quick start

No build step and no dependencies to install.

bash
git clone https://github.com/Tanayambekar/GeneRx.git
cd GeneRx

Open index.html in a modern browser (Chrome, Edge, Firefox).

Click Analyse New Patient File.
Choose a .fastq, .fq, or .txt file (for example sample_patient_mrsa.fastq).
Review the result cards, prescription panel, and composition chart.
Click Export Clinical PDF to save the report.

The page loads Tailwind CSS, Chart.js, Lucide icons, and html2pdf.js from CDNs, so it needs an internet connection the first time. Download local copies if you will present offline.

Example output

For sample_patient_mrsa.fastq (5 reads):

Field	Result
Pathogen	Staphylococcus aureus
Resistance gene	mecA (PBP2a)
Ineffective antibiotics	Penicillin, oxacillin, methicillin, amoxicillin, cephalexin
Recommended options	Vancomycin, linezolid
Host variant (MT-RNR1)	Not detected
Sample composition	60% other reads / 40% panel-matched reads

The alert text also lists the evidence, for example the number of supporting reads and the share of the marker covered.

Algorithm details
Input: FASTQ, where the sequence is every fourth line. Plain sequence lines are accepted as a fallback.
K-mers: k = 15, stored as 2-bit integers. Each k-mer is converted to its canonical form (the smaller of the forward and reverse-complement value), so a read matches no matter which strand it came from. A non-ACGT base restarts the window.
Index: a hash map from each marker k-mer to the gene and position it came from. Degenerate bases in a signature (for example Y) are expanded into all variants.
Read support: a read supports a gene if it shares at least 3 distinct marker k-mers with it.
Gene call: at least 3 supporting reads (1 for files under 50 reads) and at least 30% of the marker's k-mers covered.
Host variant: detected when the single k-mer containing the variant base is present, using the same minimum-read rule.
Composition: reads matching any panel marker versus all other reads. This is a rough proxy, not a true human-DNA measurement.
Drug logic: a hand-written lookup table from gene to ineffective and recommended drugs. If vanA is detected, vancomycin is removed from the recommendations.

The approach is in the same family as k-mer classifiers such as Kraken2.

Project structure
GeneRx/
├── index.html                  # entire app: UI, k-mer engine, drug logic, PDF export
├── sample_patient_mrsa.fastq   # small example file with a mecA read
└── README.md
Performance

In a Node.js benchmark of the scanning code on random 150-base reads, 5,000 reads took about 0.08 seconds and 50,000 reads took about 0.7 seconds. Browser timings will vary with the device.

Roadmap
Load full gene and species references from CARD / NCBI, with several signatures per gene
Map reads to the human genome for true host-DNA estimation
Allele-level calls and point-mutation resistance (for example fluoroquinolones)
Species identification from a larger reference set
Offline bundle with local copies of all libraries
Validation against labeled sequencing runs and culture results
References
WHO. WHO Bacterial Priority Pathogens List, 2024. https://iris.who.int/bitstream/handle/10665/376776/9789240093461-eng.pdf
GBD 2021 Antimicrobial Resistance Collaborators. The Lancet 2024; 404: 1199-1226. https://doi.org/10.1016/S0140-6736(24)01867-1
Comprehensive pathogen identification and antimicrobial resistance prediction from positive blood cultures using nanopore sequencing technology. Genome Medicine, 2024. https://doi.org/10.1186/s13073-024-01416-2
CARD, the Comprehensive Antibiotic Resistance Database. https://card.mcmaster.ca/
