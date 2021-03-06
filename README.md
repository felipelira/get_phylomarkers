# get_phylomarkers

<!--Version Dec. 29cnd, 2017.-->

This repository hosts the code for the *get_phylomarkers* pipeline. This file describes its
aim and basic usage notes. <!--See [INSTALL.md](INSTALL.md) for installation instructions.-->
The code is developed and maintained by [Pablo Vinuesa](http://www.ccg.unam.mx/~vinuesa/) 
at [CCG-UNAM, Mexico](http://www.ccg.unam.mx/) and 
[Bruno Contreras-Moreira](https://digital.csic.es/cris/rp/rp02661/) 
 at [EEAD-CSIC, Spain](http://www.eead.csic.es/). It is released to the public domain under the GNU GPLv3 [license](./LICENSE).
 
## Installation & dependencies

For detailed instructions and dependencies please check [INSTALL.md](INSTALL.md).

A [Docker image](https://hub.docker.com/r/csicunam/get_homologues) is available with GET_PHYLOMARKERS
bundled with [GET_HOMOLOGUES](https://github.com/eead-csic-compbio/get_homologues), ready to use.

## Aim
The pipeline selects markers with optimal phylogenetic attributes from the homologous gene 
clusters produced by [GET_HOMOLOGUES](https://github.com/eead-csic-compbio/get_homologues). 
This is a genome-analysis pipeline for microbial pan-genomics and comparative genomics originally 
described in the following publications: 
[Contreras-Moreira and Vinuesa, AEM 2013](https://www.ncbi.nlm.nih.gov/pubmed/24096415) and 
[Vinuesa and Contreras-Moreira, 2015](https://www.ncbi.nlm.nih.gov/pubmed/25343868). More recently
we have developed [GET_HOMOLOGUES-EST](https://github.com/eead-csic-compbio/get_homologues), 
which can be used to cluster eukaryotic genes and transcripts, as described in [Contreras-Moreira et al., 2017](http://journal.frontiersin.org/article/10.3389/fpls.2017.00184/full).

The homologous gene/protein clusters selected by the *get_phylomarkers* pipeline are optimally 
suited for genome phylogenies. The pipeline is primarily tailored towards selecting 
CDSs (gene markers) to infer DNA-level phylogenies of different species of the same genus or family. 
It can also select optimal markers for population genetics, when the source genomes belong to the same species.
For more divergent genome sequences, belonging to different taxonomic genera, families or even orders,
the pipeline can be also run using protein instead of DNA sequences.

## Usage synopsis

1. The pipeline is run by executing the main script *run_get_phylomarkers_pipeline.sh* inside a folder containing twin \*.fna and \*.faa FASTA files for orthologous **single-copy** CDSs and translation products. <!-- computed by the *get_homologues.pl -e* or *compare_clusters.pl* scripts of the **GET_HOMOLOGUES** suite.-->
2. There are two **runmodes**: -R 1 (for phylogenetics) and -R 2 (for population genetics).
3. The pipeline can be run on two **molecular types**: DNA or protein sequences (-t DNA|PROT). The latter is intended for the analysis of more divergent genome sequences, above the genus level.
4. FastTree is the tree searching algorithm used by default (-A F). This is the fastest maximum likelihood tree-searching algorithm available to date. 
5. The user can also choose to use the significantly more accurate, but somewhat slower, IQ-TREE algorithm using option -A I. IQ-TREE is the best ML tree-searching algorithm available to date for datasets with no more than 100-200 sequences, as recently demonstrated in a large benchmark study with empirical phylogenomic datasets (Zhou et al. 2017. Mol Biol Evol. Nov 21. doi: 10.1093/molbev/msx302.) [PMID:29177474](https://www.ncbi.nlm.nih.gov/pubmed/29177474).
6. As of version 1.9.10 (Jan, 1st, 2018), GET_PHYLOMARKERS uses IQ-TREE version 1.6.1 (released Dec. 23rd, 2017) which implements a fast search option, which almost matches the speed of FastTree, but rataining the accuracy of IQ-TREE 1.5.* Model-selection is performed with the -fast flag to for maximal speed.
7. The **global molecular-clock hypothesis** can be evaluated for DNA (codon) alignments (-R 1 -t DNA -K 1). It is not yet implemented for protein sequences.
8. A **toy sequence dataset is provided** with the distribution in the test_sequences/ directory for easy and fast testing (~14 seconds on a commodity GNU/Linux desktop machine with 4 cores; see [INSTALL.md](INSTALL.md)). 

### Basic usage examples

```
 # default run
 run_get_phylomarkers_pipeline.sh -R 1 -t DNA

 # add molecular-clock analysis assuming a HKY85+G substitution model                
 run_get_phylomarkers_pipeline.sh -R 1 -t DNA -K 1 -M HKY

 # population-genetics mode   
 run_get_phylomarkers_pipeline.sh -R 2 -t DNA                
 
 # protein alignments, user-defined kdetrees & mean branch support cutoff values
 run_get_phylomarkers_pipeline.sh -R 1 -t PROT -k 1.2 -m 0.7 

 # To run the pipeline on a remote server, we recommend using the nohup command upfront, as shown below:
 #   in this case, calling also IQ-TREE, which will select among the (TNe,TVM,TVMe,GTR)+RATE models and do
 #   5 independent tree searches under the best-fit model, computing ultrafast bootstrapp and aproximate Bayes
 #   branch support values 
 nohup run_get_phylomarkers_pipeline.sh -R 1 -t DNA -A I -S 'HKY,TN,TVM,TIM,SYM,GTR' -k 1.0 -m 0.7 -T high -N 5 &> /dev/null &
```

## Usage and design details

1. Start the run from **within the directory** holding core gene clusters generated by either *get_homologues.pl -e* or 
subsequent intersection (OMCL,COGS,BDBH) clusters produced with *compare_clusters.pl* from the 
[GET_HOMOLOGUES](https://github.com/eead-csic-compbio/get_homologues) package.
   
  NOTE: **both .faa and .fna files are required** to generate codon alignments from DNA fasta files. This
            means that two runs of *compare_clusters.pl* (from the **GET_HOMOLOGUES** package) are required,
	          one of them using the -n flag. 
	    
2. *run_get_phylomarkers_pipeline.sh* is intended to run on a collection of **single-copy** sequence clusters from 
different species or strains.

   NOTES: an absolute minimum of 4 distinct haplotypes per cluster are required
          for the cluster to be evaluated. Clustes with < 4 haplotypes are automatically
	  discarded. This means that at least 4 distinct genomes should be used as input.
	  However, the power of the pipeline for selecting optimal genome loci 
     	  for phylogenomics improves when a larger number of genomes are available 
     	  for analysis. Reasonable numbers lie in the range of 10 to 200 clearly
     	  distinct genomes from multiple species of a genus, family, order or phylum.
     	  The pipeline may not perform satisfactorily with too distant genome sequences,
     	  particularly when sequences with significantly distinct nucleotide or aminoacid
     	  compositions are used. This type of sequence heterogeneity is well known to 
     	  cause systematic bias in phylogenetic inference. In general, very distantly related
     	  organisms, such as those from different phyla or even domains, may not be
     	  properly handled by run_get_phylomarkers_pipeline.sh.

## On the filtering criteria. 

*run_get_phylomarkers_pipeline.sh* uses a **hierarchical filtering scheme**, as follows:

###   i) Detection of recombinant loci. 

Codon or protein alignments (depending on runmode) are first screened with **Phi-test** 
([Bruen et al. 2006](http://www.genetics.org/content/172/4/2665.long)) for the presence of potential recombinant sequences. It is a well established fact that recombinant sequences negatively impact phylogenetic inference when using algorithms that do not account for the effects of this evolutionary force. The permutation test with 1000 permutations is used to compute the *p*-values. These are considered significant if *p* < 0.05. Some loci may not contain sufficient polymorphisms for the test to
work. In that case, the main script assumes that the locus does not contain recombinant sites.
 
### ii) Detection of trees deviating from expectations of the (multispecies) coalescent.

The next filtering step is provided by the **kdetrees-test**, which checks the distribution of topologies, tree lengths and branch lengths. *kdetrees* is a non-parametric method for estimating distributions of phylogenetic trees 
([Weyenberg et al. 2014](https://academic.oup.com/bioinformatics/article-lookup/doi/10.1093/bioinformatics/btu258)), 
with the goal of identifying trees that are significantly different from the rest of the trees in the sample, based on the analysis of topology and branch length distributions. Such "outlier" trees may arise for example from horizontal gene transfers or gene duplication (and subsequent neofunctionalization) followed by differential loss of paralogues among lineages. Such processes will cause the affected genes to exhibit a history distinct from those of the majority of genes, which are expected to be generated by the  (multispecies) coalescent as species or populations diverge. Alignments producing significantly deviating trees in the kdetrees test are identified and saved in the kde_outliers/ directory. The corresponding alignments are not used in downstream analyses.

```      
      * Parameter for controlling kdetrees stingency:
      -k <real> kde stringency (0.7-1.6 are reasonable values; less is more stringent)
     			       [default: 1.5]
```

### iii) Phylogenetic signal content and congruence. 

The alignments passing the two previous filters are subjected to **maximum likelihood (ML) tree searches with FastTree** to 
infer the corresponding ML gene trees. Their **phylogenetic signal is computed from the Shimodaria-Hasegawa-like likelihood ratio test branch support values**, which vary between 0-1, as we have reported previously ([Vinuesa et al. 2008](http://aem.asm.org/content/74/22/6987.long)). The support values of each internal branch or bipartition are parsed to compute the mean support value for each tree. 
**Alignments/Trees with a mean support value below a cutoff threshold are discarded**. In addition, a consensus tree is computed from the collection of trees that passed filters *i* and *ii*, and the Robinson-Fould distance (**RF**) is computed between each gene tree and the consensus tree.

```
      * Parameters controlling filtering based on mean support values.
      -m <real> min. average support value (0.7-0.8 are reasonable values) 
     		for trees/loci to be selected as informative [default: 0.75]
```

### iv) Evaluating the global molecular clock hypothesis.

*run_get_phylomarkers_pipeline.sh* calls the auxiliary script *run_parallel_molecClock_test_with_paup.sh*
to evaluate the **global molecular clock hypothesis** on the top markers, selected according to the criteria explained in the three previous
points. The script calls [paup*](https://people.sc.fsu.edu/~dswofford/paup_test/) 
to evaluate the free-rates and clock hypothesis using likelihood ratio tests using R. Currently this is only performed on codon alignments. Future versions will implement the global clock hypothesis test also for protein alignments.

### v) On tree searching: 
1. FastTree searches
- *run_get_phylomarkers_pipeline.sh* performs **tree searches under the maximum-likelihood criterion** 
using by default (-A F) the [FastTree](http://microbesonline.org/fasttree/) program ([Price et al. 2010](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0009490)). 
- This program implements many heuristics tailored towards improving the cpu time and memory usage, making it the fastest ML tree searching program that is currently available. However, this comes at a price: in a recent comprehensive benchmark analysis of fast ML phylogenetic programs using large and diverse phylogeneomic datasets, FastTree was shown to be the worst-scoring one in regard to lnL values and topological accuracy [(Zhou et al. 2017)](https://www.ncbi.nlm.nih.gov/pubmed/29177474). 
- Given its outstanding speed, low RAM requirements, acceptance of DNA and protein sequence alignments and on-the-fly computation ofthe above-mentioned Shimodaria-Hasegawa-like likelihood ratio test of branch support, we use it as the default algorithm for fast exploration of large datasets.
- A limitation though, is that it implements only very few substitution models. However, for divergent sequences of different species within a bacterial taxonomic genus or family, our experience has shown that almost invariably the GTR+G model is selected by jmodeltest2, particularly when there is base frequency heterogeneity. The GTR+G+CAT is the substitution model used by *run_get_phylomarkers_pipeline.sh* 
calls of FastTree on codon alignments. 
- To maximize accuracy, it is important to compile FastTree with double precission enabled in order to obtain the highest *lnL* scores possible. This is particularly critical when highly similar sequences are present in the dataset.
- To further enhance accuracy, the gene trees are computed by performing a thorough tree search, as hardcoded in the following FastTree call, which performs a significantly more intense tree search than the default setting used by [Zhou et al. (2017)](https://www.ncbi.nlm.nih.gov/pubmed/29177474). 

```      
     	-nt -gtr -gamma -bionj -slownni -mlacc 3 -spr 8 -sprlength 10 
```    

For concatenated codon alignments, which may take a considerable time (up to several hours) or
for large datasets (~ 100 taxa and > 300 concatenated genes) the user can choose to run FastTree with at different **levels of tree-search thoroughness**: high|medium|low|lowest 
```      
      high:   -nt -gtr -bionj -slownni -gamma -mlacc 3 -spr 4 -sprlength 10
      medium: -nt -gtr -bionj -slownni -gamma -mlacc 2 -spr 4 -sprlength 10 
      low:    -nt -gtr -bionj -slownni -gamma -spr 4 -sprlength 10 
      lowest: -nt -gtr -gamma -mlnni 4
```      
where -s $spr and -l $spr_length can be set by the user. The lines above show their default values.
      
For protein alignments, the search parameters are the same, only the model changes to lg

```      
      high: -lg -bionj -slownni -gamma -mlacc 3 -spr 4 -sprlength 10
```

Please refer to the FastTree manual for the details.

2. ModelFinder + IQ-TREE searches
- As of version 1.9.9.0_22Dec17, GET_PHYLOMERKERS implements the **-A I** option, which calls [IQ-TREE](http://www.iqtree.org/) for ML tree searching [Nguyen et. al (2015)](https://academic.oup.com/mbe/article/32/1/268/2925592). This is the most recent fast ML software on the scene, which was developed with the aim of escaping from early local maxima encountered during "hill-climbing" by generating multiple seed trees to intiate tree searches, maintaining a pool of candidate trees during the entire run. Overall, it was the best-performing ML tree search algorithm among those evaluated by [Zhou et al. (2017)](https://www.ncbi.nlm.nih.gov/pubmed/29177474) in their above-mentioned benchmarking paper, at least for datasets < 200 taxa. For larger datasets (several hudreds to thousands of sequencs), current implementations of algorithms like RAxML and ExaML, which make hevay use of SPR-moves, will most likey outperform IQ-TREE, which makes more intensive use of local NNI-moves.
- In order to achieve the best compromise between speed and accuracy, when *run_get_phylomarkers_pipeline.sh* is called with -A I, it will run FastTree to rapidly score alignments for discordant trees with KDEtrees and for phylogenetic signal, as described above, but uses [IQ-TREE](http://www.iqtree.org/) for the tree search based on the concatenated supermatrix of top-scoring alignments. [ModelFinder](http://www.iqtree.org/ModelFinder/) [(Kalyaanamoorthy et al. 2017)](https://www.nature.com/articles/nmeth.4285) is first called to perform model selection, using by default the GTR or LG base model or matrix for DNA or protein alignments, respectively. The user can pass a strig of comma-separated modes to evalute, using the -S 'basemodel1,basemodel2...' option. After selecting the best substitution model, which includes taking care of among-site rate variation, IQ-TREE will search for the best tree, including bootstrapping with the **UFBoot2 ultrafast bootstrapping algorithm** [(Hoang et al. 2017)](https://www.ncbi.nlm.nih.gov/pubmed/29077904) and estimation of approximate Bayes support values. 
- If the **-T high** option is used, **-N 10 ** independent IQ-TREE searches with branch support estimation will be launched. 
- Note that a FastTree run will be also launched (before the IQ-TREE run), so that the user can obtain a first tree as rapidly as permitted by the current state of the art.

The following are example *run_get_phylomarkers_pipeline.sh* invocations to perform IQ-TREE searches. Note that as of version 1.9.10 (January 1st, 2018), the script calls IQ-TREE 1.6.1 with the -fasta flag enabled for maximal speed.

```    
# 1. Default IQ-TREEE run on a concatenated DNA supermatrix, using a single search and evaluating only the GTR base model
run_get_phylomarkers_pipeline.sh -R 1 -t DNA -A I

# 2. Run 10 independent IQ-TREEE runs on a concatenated DNA supermatrix, evaluating the TN,TIM,TVM,GTR base models
run_get_phylomarkers_pipeline.sh -R 1 -t DNA -A I -S 'TN,TIM,TVM,GTR' -k 0.9 -m 0.8 -T high -N 10 &> /dev/null &

# 3. Run 5 independent IQ-TREEE runs on a concatenated PROT supermatrix, evaluating the LG,WAG,JTT matrices 
run_get_phylomarkers_pipeline.sh -R 1 -t PROT -A I -S 'LG,WAG,JTT,VT' -k 1.0 -m 0.7 -T high -N 5 &> /dev/null &
 
# 4. To run the pipeline on a remote server, we recommend using the nohup command upfront, as shown below:
nohup run_get_phylomarkers_pipeline.sh -R 1 -t DNA -A I -S 'HKY,TN,TVM,TIM,SYM,GTR' -k 1.0 -m 0.7 -T high -N 5 &> /dev/null &	  

```

## Citation.

We are preparing a publication describing the implementation get_phylomarkers pipeline and its use in genomic taxonomy and population genomics of *Stenotrophomonas* bacteria, which was submitted in Jaunary 15th 2018 to the [Research Topic of Frontiers in Microbiology: Microbial Taxonomy, Phylogeny and Biodiversity](http://journal.frontiersin.org/researchtopic/5493/microbial-taxonomy-phylogeny-and-biodiversity).

Meanwhile, if you find the code useful for your academic work, please use the following citation:
Pablo Vinuesa and Bruno Contreras-Moreira 2017. Get_PhyloMarkers, a pipeline to select optimal markers for microbial phylogenomics, systematics and genomic taxomy. Available at https://github.com/vinuesa/get_phylomarkers and released under the GNU GPLv3 license.

## Acknowledgements

### Personal
We thank Alfredo J. Hernández and Víctor del Moral at CCG-UNAM for technical support.

### Funding
We gratefully acknowledge the funding provided by [DGAPA-PAPIIT/UNAM](http://dgapa.unam.mx/index.php/impulso-a-la-investigacion/papiit) (grants IN201806-2, IN211814 and IN206318) and [CONACyT-Mexico](http://www.conacyt.mx/) (grants P1-60071, 179133 and FC-2015-2-879) to [Pablo Vinuesa](http://www.ccg.unam.mx/~vinuesa/), as well as the Fundación ARAID,Consejo  Superior  de Investigaciones Científicas (grant 200720I038 and Spanish MINECO (AGL2013-48756-R) to [Bruno Contreras-Moreira](https://digital.csic.es/cris/rp/rp02661).
