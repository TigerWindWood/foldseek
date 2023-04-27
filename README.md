# Foldseek 
Foldseek enables fast and sensitive comparisons of large structure sets.

<p align="center"><img src="https://github.com/steineggerlab/foldseek/blob/master/.github/foldseek.png" height="250"/></p>

## Publications

[van Kempen M, Kim S, Tumescheit C, Mirdita M, Söding J, and Steinegger M. Foldseek:  fast and accurate protein structure search. bioRxiv, doi:10.1101/2022.02.07.479398  (2022)](https://www.biorxiv.org/content/10.1101/2022.02.07.479398)

## Tutorial Video
We presented a Foldseek tutorial at the SBGrid where we demonstrate the webserver and command line interface of foldseek. 
Check it out [here](https://www.youtube.com/watch?v=k5Rbi22TtOA).

<a href="https://www.youtube.com/watch?v=k5Rbi22TtOA"><img src="https://img.shields.io/youtube/views/k5Rbi22TtOA?style=social"></a>.

## Webserver 
Search your protein structures against the [AlphaFoldDB](https://alphafold.ebi.ac.uk/) and [PDB](https://www.rcsb.org/) in seconds using our Foldseek webserver: [search.foldseek.com](https://search.foldseek.com) 🚀

## Installation
```
# Linux AVX2 build (check using: cat /proc/cpuinfo | grep avx2)
wget https://mmseqs.com/foldseek/foldseek-linux-avx2.tar.gz; tar xvzf foldseek-linux-avx2.tar.gz; export PATH=$(pwd)/foldseek/bin/:$PATH

# Linux SSE4.1 build (check using: cat /proc/cpuinfo | grep sse4_1)
wget https://mmseqs.com/foldseek/foldseek-linux-sse41.tar.gz; tar xvzf foldseek-linux-sse41.tar.gz; export PATH=$(pwd)/foldseek/bin/:$PATH

# MacOS
wget https://mmseqs.com/foldseek/foldseek-osx-universal.tar.gz; tar xvzf foldseek-osx-universal.tar.gz; export PATH=$(pwd)/foldseek/bin/:$PATH

# Conda installer (Linux and macOS)
conda install -c conda-forge -c bioconda foldseek
```
Other precompiled binaries for ARM64 amd SSE2 are available at [https://mmseqs.com/foldseek](https://mmseqs.com/foldseek).

## Documentation
Many of Foldseek's modules (subprograms) rely on MMseqs2. For more information about these modules, refer to the [MMseqs2 wiki](https://github.com/soedinglab/MMseqs2/wiki). For documentation specific to Foldseek, checkout the Foldseek wiki [here](https://github.com/steineggerlab/foldseek/wiki).

## Quick start

### Search
The `easy-search` module allows to search single or multiple query structures, formatted in PDB/mmCIF format (flat or gzipped), against a target database, folder or single protein structures. In default it outputs the alignment information as a [tab-separated file](#tab-separated) but we support also [Superposed Cα PDBs](#superpositioned-cα-only-pdb-files) or a [HTML](#interactive-html) output.

    foldseek easy-search example/d1asha_ example/ aln tmpFolder
    
#### Output
##### Tab-separated
  
The default fields are containing the following fields: `query,target,fident,alnlen,mismatch,gapopen,qstart,qend,tstart,tend,evalue,bits` but they can be customized with the `--format-output` option e.g. `--format-output "query,target,qaln,taln"` returns the query and target accession and the pairwise alignments in tab separated format. You can choose many different output columns.

| Code | Description |
| --- | --- |
|query | Query sequence identifier |
|target | Target sequence identifier |
|qca        | Calpha corrdinates of the query |
|tca        | Calpha corrdinates of the target |
|alntmscore | TM-score of the alignment | 
|qtmscore   | TM-score normalized by the query length |
|ttmscore   | TM-score normalized by the target length |
|u          | Rotation matrix (computed to by TM-score) |
|t          | Translation vector (computed to by TM-score) |
|lddt       | Average LDDT of the alignment |
|lddtfull   | LDDT per aligned position |
|prob       | Estimated probability for query and target to be homologous (e.g. being within the same SCOPe superfamily) |

Check out the [MMseqs2 documentation for more format output codes](https://github.com/soedinglab/MMseqs2/wiki#custom-alignment-format-with-convertalis).

##### Superpositioned Cα only PDB files
Foldseek's `--format-mode 5` generates PDB files with all Cα atoms superimposed based on the aligned coordinates on to the query structure. 
For each pairwise alignment it will write a single PDB files, so be carefull when using this options for large searches. 

##### Interactive HTML
Foldseek can locally generate a search result HTML similiar to the [webserver](https://search.foldseek.com) by specifying the format mode `--format-mode 3`

```
foldseek easy-search example/d1asha_ example/ result.html tmp --format-mode 3
```

<p align="center"><img src="./.github/results.png" height="400"/></p>

#### Important search parameters

| Option            | Category        | Description                                                                                               |
|-------------------|-----------------|-----------------------------------------------------------------------------------------------------------|
| -s              | Sensitivity     | Adjust sensitivity to speed trade-off; lower is faster, higher more sensitive (fast: 7.5, default: 9.5)   |
| --exhaustive-search | Sensitivity | Skips prefilter and performs an all-vs-all alignment (more sensitive but much slower)                     |
| --max-seqs      | Sensitivity     | Adjust the amount of prefilter handed to alignment; increasing it can lead to more hits (default: 1000)   |
| -e              | Sensitivity     | List matches below this E-value (range 0.0-inf, default: 0.001); increasing it reports more distant structures |
| --alignment-type| Alignment       | 0: 3Di Gotoh-Smith-Waterman (local, not recommended), 1: TMalign (global, slow), 2: 3Di+AA Gotoh-Smith-Waterman (local, default) |
| -c              | Alignment  | List matches above this fraction of aligned (covered) residues (see --cov-mode) (default: 0.0); higher coverage = more global alignment |
| --cov-mode      | Alignment  | 0: coverage of query and target, 1: coverage of target, 2: coverage of query                               |

#### Using TMalign for the alignment
In default Foldseek uses its local 3Di+AA strutural alignment but it also supports to realign hits using the global TMalign as well as rescoring alignments using TMscore. 
```
foldseek easy-search example/d1asha_ example/ aln tmp --alignment-type 1
```
In case of the alignment type (`--alignment-type 1`) tmalign, we sort the results by the TMscore normalized by query length. We write the TMscore into the e-value=(qTMscore+tTMscore)/2 as well as into the score(=qTMscore*100) field. All output fields (like pident, fident, and alnlen) are calculated from the TMalign alignment.

### Databases 
The `databases` command downloads pre-generated databases like PDB or AlphaFoldDB.
    
    # pdb  
    foldseek databases PDB100 pdb tmp 
    # alphafold db
    foldseek databases Alphafold/Proteome afdb tmp 

We currently support the following databases: 
```
  Name                   	Type     	Taxonomy	Url
- Alphafold/UniProt   	Aminoacid	     yes	https://alphafold.ebi.ac.uk/
- Alphafold/UniProt50 	Aminoacid	     yes	https://alphafold.ebi.ac.uk/
- Alphafold/Proteome  	Aminoacid	     yes	https://alphafold.ebi.ac.uk/
- Alphafold/Swiss-Prot	Aminoacid	     yes	https://alphafold.ebi.ac.uk/
- ESMAtlas30          	Aminoacid	       -	https://esmatlas.com
- PDB                 	Aminoacid	     yes	https://www.rcsb.org
```

#### Create custom databases and indexes
The target database can be pre-processed by `createdb`. This make sense if searched multiple times. 
 
    foldseek createdb example/ targetDB
    foldseek createindex targetDB tmp  #OPTIONAL generates and stores the index on disk
    foldseek easy-search example/d1asha_ targetDB aln.m8 tmpFolder

## Main Modules
- `easy-search`       fast protein structure search  
- `easy-cluster`      fast protein structure clustering  
- `createdb`          create a database from protein structures (PDB,mmCIF, mmJSON)
- `databases`         download pre-assembled databases

## Examples
### Rescore aligments using TMscore
Easiest way to get the alignment TMscore normalized by min(alnLen,qLen,targetLen) as well as a rotation matrix is through the following command:
```
foldseek easy-search example/ example/ aln tmp --format-output query,target,alntmscore,u,t
```

Alternative, it is possible to compute TMscores for the kind of alignment output (e.g. 3Di/AA) using the following commands: 
```
foldseek createdb example/ targetDB
foldseek createdb example/ queryDB
foldseek search queryDB targetDB aln tmpFolder -a
foldseek aln2tmscore queryDB targetDB aln aln_tmscore
foldseek createtsv queryDB targetDB aln_tmscore aln_tmscore.tsv
```

Output format `aln_tmscore.tsv`: query and target identifier, TMscore, translation(3) and rotation vector=(3x3)


### Cluster search results 
The following command aligns the input structures all-against-all and keeps only alignments with 80% of the sequence covered by the alignment (-c 0.8) (read more about alignment coverage [here](https://github.com/soedinglab/MMseqs2/wiki#how-to-set-the-right-alignment-coverage-to-cluster)). It then clusters the results using greedy set cover algorithm. The clustering mode can be adjusted using --cluster-mode, read more [here](https://github.com/soedinglab/MMseqs2/wiki#clustering-modes). The clustering output format is described [here](https://github.com/soedinglab/MMseqs2/wiki#cluster-tsv-format).

```
foldseek createdb example/ db
foldseek search db db aln tmpFolder -c 0.8 
foldseek clust db aln clu
foldseek createtsv db db clu clu.tsv
```

### Query centered multiple sequence alignment 
Foldseek can generate a3m based multiple sequence alignments using the following commands. 
a3m can be converted to fasta format using [reformat.pl](https://raw.githubusercontent.com/soedinglab/hh-suite/master/scripts/reformat.pl) (`reformat.pl in.a3m out.fas`).
```
foldseek createdb example/ targetDB
foldseek createdb example/ queryDB
foldseek search queryDB targetDB aln tmpFolder -a
foldseek result2msa queryDB targetDB aln msa --msa-format-mode 6
foldseek unpackdb msa msa_output --unpack-suffix a3m --unpack-name-mode 0
```
