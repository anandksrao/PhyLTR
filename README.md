# PhyLTR: de novo LTR retrotransposon annotation, classification, and phylogenetic analysis

PhyLTR is a software pipeline built from open source software. The main program is written in Python 3. Many of the routines in PhyLTR are parallelized, but it is not written for MPI so parallel components can only run on CPUs that share memory, i.e. on a single node.

As the pipeline runs, paths to intermediate results like alignments are stored in the file PhyLTR/status. If the execution is interrupted, this file is used to allow PhyLTR to resume more or less where it left off.
---
#### Output
PhyLTR populates a directory structure (default=PhyLTR.output/) keeping results for different clusterings separate. The results obtained prior to clutering are used for any post-clustering analysis. Within `PhyLTR.output` (default):
```
LTRharvest/		LTRharvest results
suffixerator/		suffix array used by LTRharvest
LTRdigest/		LTRdigest results
AnnotateORFs/		intermediate files for ORF annotation
Circos/			Circos intermediate files and plots
DfamClassification/	nhmmer search of Dfam for LTR-R homologs
RepbaseClassification/	tblastx search of Repbase for LTR-R homologs
FASTA_output/		some intermediate FASTA files
GFF_output/		various intermediate and final GFF3 files
MCL/			MCL clusterings and downstream analyses
WickerFamDir/		WickerFam clusterings and downstream analyses
```
Within each clustering directory, e.g: `MCL/I6/`
```
Alignments/		All alignments
Clusters/		Results of clustering
FASTAs/			Intermediate FASTA files
GENECONV/		Gene conversion analyses
GFFs/			Intermediate GFF3 files
LTR_divergence/		LTR divergence estimation analyis
Modeltest/		Model testing files
SoloLTRsearch/		"Solo LTR" search files
Trees/			Phylogenetic analyses
```
---
## Default settings
If phyltr is run without any flags specifying a task, all tasks are run. The following two calls are equivalent. The processes specified by the flags in the second call are explained below with additional optional flags. Some of the processes modify the GFF3 file that is used for downstream analyses.
```
phyltr --fasta <input> --procs <int>

phyltr --fasta <input> --procs <int> \
	--ltrharvest \
	--ltrdigest \
	--classify \
	--wicker \
	--mcl \
	--geneconvclusters \
	--circos \
	--sololtrsearch \
	--geneconvltrs \
	--ltrdivergence \
	--phylo \
	--DTT
```
---
## 1. Identify candidate LTR-R loci
#### LTRharvest: `--ltrharvest`
###### External dependencies
* GenomeTools
###### Options
```
--minlenltr (100)	Minimum LTR length (bp)
--maxlenltr (1000)	Maximum LTR length (bp)
--mindistltr (1000)	Minimum distance between LTRs (bp)
--maxdistltr (15000)	Maximum distance between LTRs (bp)
--similar (85.0)	Minimum % similarity between LTRs
--vic (60)		
--mintsd (4)
--maxtsd (20)
--xdrop	(5)
--mat (2)
--mis (-2)
--insi (-3)
--del (-3)
```
---
## 2. Identify putatve protein-coding domains in LTR-R internal regions.
#### A. Run LTRdigest: `--ltridgest`
###### External dependencies
* GenomeTools
* HMMER3
* pHMMs (database)
###### Options
```
--ltrdigest_hmms (/home/joshd/scripts/PhyLTR/LTRdigest_HMMs/hmms)	path to pHMMs
```
#### B. Run ORF annotation: `--findORFs`
###### External dependencies
* EMBOSS
###### Options
```
--min_orf_len (300)	The minimum length (bp) of ORF to find
```
---
## 3. Classify elements using homology to LTR-Rs in Dfam and/or Repbase
#### A. Run both Repbase and Dfam classification: `--classify`
#### B. Run Dfam classification: `--classify_dfam`
###### External dependencies
* BEDtools
* HMMER3
* Dfam (database)
###### Options
```
--keep_no_classifications 		Retain elements without homology to known LTR-Rs
--nhmmer_reporting_evalue (10)		See HMMER3 documentation: nhmmer -E
--nhmmer_inclusion_evalue (1e-2)	See HMMER3 documentation: nhmmer -incE
```
#### C. Run Repbase classification: `--classify_repbase`
###### External dependencies
* BEDtools
* NCBI BLAST+
* Repbase (database)
###### Options
```
--keep_no_classifications	Retain elements without homology to known LTR-Rs
--repbase_tblastx_evalue (1e-5)	Maximum E-value for tblastx hits
```
---
## 4. Cluster LTR-Rs
#### A. Run WickerFam clustering: `--wicker`
###### Options
```
--wicker_minLen (80)	Minimum length of blastn alignment
--wicker_pAln (80)	Minimum percent of LTR or internal region required in alignment
--wicker_pId (80)	Minimum %identity in alignment
--wicker_no_internals	Turns off use of internal region alignments for clustering
--wicker_no_ltrs	Turns off use of LTR alignments for clustering
```
###### External dependencies
* BEDtools
* NCBI Blast+
#### B. Run MCL clustering: `--mcl`
###### Options
```
--I (6)		MCL inflation paramter. Larger values result in smaller clusters.
```
###### External dependencies
* BEDtools
* NCBI Blast+
* MCL
---
## 5. Estimate LTR divergences (aka insertion age)
#### A. Run gene conversion assessment for intra-element LTR assessment: `--geneconvltrs`
###### Options
```
--geneconv_g (g1,g2,g3)	Comma-separated list, g1, g2, and/or g3. Stringency for mismatch-free gene conversion tracts: g0 > g2 > g1
```
###### External dependencies
* BEDtools
* MAFFT
* trimAl
* GENECONV
#### B. Run LTR divergence estimation: `--ltr_divergence`
###### Options
```
--remove_GC_from_modeltest_aln	Remove elements with gene conversion (--geneconvclusters)
```
###### External dependencies
* BEDtools
* MAFFT
* trimAl
* jModelTest2
* PAUP\*
---
## 6. "Solo LTR" search
#### Run "Solo LTR" search: `--soloLTRsearch`
###### Options
```
--soloLTRminPid (80.0)		Minimum %identity in blastn alignment to associate LTR with a cluster
--soloLTRminLen	(80.0)		Minimum % of length of LTR required in alignment to associate LTR with a cluster
--soloLTRmaxEvalue (1e-3)	Maximum E-value for blastn
```
###### External dependencies
* BEDtools
* NCBI BLAST+
---
## 7. Rene conversion assessment for between elements in clusters
#### Run GENECONV: `--geneconvclusters`
###### Options
```
--geneconv_g (g1,g2,g3)	Comma-separated list, g1, g2, and/or g3. Stringency for mismatch-free gene conversion tracts: g0 > g2 > g1
```
###### External dependencies
* BEDtools
* MAFFT
* trimAl
* GENECONV
#### Make Circos plots: `--circos`
###### External dependencies
* BEDtools
* Circos
---
## 8. Phylogenetics
#### Run phylogenetic inference: `--phylo`
###### Options
```
--min_clust_size (7)		Do not align clusters smaller than this.
--nosmalls			Do not analyze clusters smaller than --min_clust_size
--rmhomoflank			Exclude elements with non-unique flanking sequences from alignments.
--bpflank (500)			Length (bp) of flank searched
--flank_evalue (1e-5)		Maximum E-value for blastn for flank search
--flank_pId (70.0) 		Minimum %identity in blastn alignment for flank search
--flank_plencutoff (70.0)	Minimum % of length of flank required in alignment
--convert_to_ultrametric	Convert tree to ultrametric (can be used for LTT plots)
--auto_outgroup			Include as an outgroup a random element from the largest available other cluster in classification (e.g. gypsy)
--bootstrap_reps (100)		Number of bootstrap replicates to perform
--LTT				Turns on --rmhomoflank, --convert_to_ultrametric, and --auto_outgroup.
```
###### External dependencies
* BEDtools
* MAFFT
* trimAl
* FastTree2
* PATHd8
* PHYLIP
---
## 9. External scripts
---
#### Search genes for LTR-R ORF homologs
###### External dependencies
* Python 3
* NCBI BLAST+
#### Render graphical trees annotated with LTR-R diagrams with colored ORFs
###### External dependencies
* Python 3
* Python 3 modules: ETE3
#### Documentation
```
Usage:
	ete3_tree.py -t <newick_tree> -d <divergences> -g <gff> [options] 

Description:

	Draws trees for long terminal repeat retrotransposon phylogenies with diagrams of LTR elements' domain
	architecture.

-lflabel
	Show element IDs (integers) as leaf labels.

-classif
	Add the superfamily classification for each element (which is obtained from the LTR divergence file) 
	above each LTR RT diagram.

-geneconv
	Add the word 'Yes' or 'No' to the immediately to the right of each leaf depending on whether intra-element
	gene conversion tracts were detected between the LTRs of that element.

-reroot <int>|auto
	Two options are possible for -reroot: 'auto', or an <int> corresponding to the element name (i.e.
	LTR_retrotransposon<int>) to position as the earliest diverging lineage. Only use -reroot auto if
	the newick filename contains the outgroup in the format output by PhyLTR. 

-ultrametric
	Draw tree after applying ete3's convert_to_ultrametric() function. In my experience using this option, I
	have always seen terminal taxa drawn at different horizontal positions making the tree not look ultrametric.

-1, 2, ..., -n  <path> <str>
	Coloring ORFs. <path> is list of ORF IDs, str is ETE3 color


-transcribed <path>
	A file containing a list of element IDs (e.g. LTR_retrotransposon123) to mark with a green asterix.
	Instead of an asterix, a 'T' is shown as if -classif is used.

-round
	Draw ORFs with rounded corners.

-outfmt <str>
	One of (pdf, png, svg). Default=pdf


Colors
	protease	yellow
	gag		deepskyblue
	copia gag	mediumblue
	DUF4219		lime
	rt		red
	rnaseh		darkviolet
	int		magenta
	zf-h2c2		pink
	other		silver w/black text
	orf		dimgray
	annotated orf	custom
```
---
#### Visualize insertion ages
###### External dependencies
* R
* R packages:  hash, ggplot2
---
#### Lineage through time plots
###### External dependencies
* R
* R packages: ape, hash, ggplot2
---
#### Transposition rate analyses
###### External dependencies
* R
* R packages: ape, hash, ggplot2
---
#### Tree shape analyses
###### External dependencies
* R
* R packages: ape, apTreeshape, hash, ggplot2
---
#### Other scripts
---
## APPENDIX A. Global MAFFT options
This step has been the limiting process in my experience. It can be sped up by reducing the number of iterations performed for each alignment and by reducing the maximum number of elements for classiying elements as medium and small clusters. MAFFT exhausted 256 Gb RAM with ~2.7k seqs of length >5kb. Depending on resources available to you, you may need to cap the size of clusters to align using `--mafft_largeAln_maxclustsize`. Default is to not align clusters with >1000 elements. The MAFFT algorthim FFT-NS-2 is used for small and medium clusters and FFT_NS-1 for large clusters, which is much more inaccurate.
###### Options
```
--maxiterate_small_clusters (30)	MAFFT iterations. More will improve alignment quality.
--maxiterate_medium_clusters (3)	MAFFT iterations. More will improve alignment quality.
--mafft_smallAln_maxclustsize (50)	Max elements to consider a cluster small.
--mafft_mediumAln_maxclustsize (500)	Max elements to consider a cluster medium.
--mafft_largeAln_maxclustsize (1000)	Max elements to consider a cluster large. Clusters larger than this will not be aligned.
--min_clust_size (7)			Do not align clusters smaller than this.
--nosmalls				Do not combine and assemble all clusters smaller than --min_clust_size
```
---
## APPENDIX B. Other global options
```
--keep_files				Default=no. Removes some large intermediate files, including raw
--output_dir		<path>	Output directory. Default is "PhyLTR.output
--logfile			<path>  Path to where log file is written (default <output_dir>/log.txt)
```
---
## APPENDIX C. All options
---
## APPENDIX D. Example output
---
