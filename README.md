
# NCBITaxCollector

NCBI TaxCollector is a tool for using the GI field of the FASTA format to
search taxonomy information in NCBI bases. It is designed to maximize
performance and allow many searches per second. NCBITaxCollector runs as
a command line tool and, for this reason, it is especially suitable for
use in scripts.


# Examples of usage


Searching the tax tree for a given gi:

	$ time ./ncbitc -s 2
	9913 | 9903 | species | BT | 2 | 1 | 1 | 1 | 2 | 1 | 1 | 0 |  |
	9903 | 27592 | genus |  | 2 | 1 | 1 | 1 | 2 | 1 | 0 | 0 |  |
	27592 | 9895 | subfamily |  | 2 | 1 | 1 | 1 | 2 | 1 | 0 | 0 |  |
	9895 | 35500 | family |  | 2 | 1 | 1 | 1 | 2 | 1 | 0 | 0 |  |
	35500 | 9845 | infraorder |  | 2 | 1 | 1 | 1 | 2 | 1 | 0 | 0 |  |
	9845 | 91561 | suborder |  | 2 | 1 | 1 | 1 | 2 | 1 | 0 | 0 |  |
	91561 | 314145 | no rank |  | 2 | 1 | 1 | 1 | 2 | 1 | 0 | 0 |  |
	314145 | 9347 | superorder |  | 2 | 1 | 1 | 1 | 2 | 1 | 0 | 0 |  |
	9347 | 32525 | no rank |  | 2 | 1 | 1 | 1 | 2 | 1 | 0 | 0 |  |
	32525 | 40674 | no rank |  | 2 | 1 | 1 | 1 | 2 | 1 | 1 | 0 |  |
	40674 | 32524 | class |  | 2 | 0 | 1 | 1 | 2 | 1 | 0 | 0 |  |
	32524 | 32523 | no rank |  | 10 | 1 | 1 | 1 | 2 | 1 | 1 | 0 |  |
	32523 | 8287 | no rank |  | 10 | 1 | 1 | 1 | 2 | 1 | 1 | 0 |  |
	8287 | 117571 | no rank |  | 10 | 1 | 1 | 1 | 2 | 1 | 1 | 0 |  |
	117571 | 117570 | no rank |  | 10 | 1 | 1 | 1 | 2 | 1 | 0 | 0 |  |
	117570 | 7776 | no rank |  | 10 | 1 | 1 | 1 | 2 | 1 | 1 | 0 |  |
	7776 | 7742 | superclass |  | 10 | 1 | 1 | 1 | 2 | 1 | 1 | 0 |  |
	7742 | 89593 | no rank |  | 10 | 0 | 1 | 1 | 2 | 1 | 0 | 0 |  |
	89593 | 7711 | subphylum |  | 10 | 0 | 1 | 1 | 2 | 0 | 0 | 0 |  |
	7711 | 33511 | phylum |  | 1 | 1 | 1 | 1 | 5 | 1 | 0 | 0 |  |
	33511 | 33316 | no rank |  | 1 | 1 | 1 | 1 | 5 | 1 | 1 | 0 |  |
	33316 | 33213 | no rank |  | 1 | 1 | 1 | 1 | 5 | 1 | 1 | 0 |  |
	33213 | 6072 | no rank |  | 1 | 1 | 1 | 1 | 5 | 1 | 1 | 0 |  |
	6072 | 33208 | no rank |  | 1 | 1 | 1 | 1 | 5 | 0 | 1 | 0 |  |
	33208 | 33154 | kingdom |  | 1 | 0 | 1 | 1 | 1 | 1 | 0 | 0 |  |
	33154 | 2759 | no rank |  | 4 | 0 | 1 | 1 | 1 | 1 | 1 | 0 |  |
	2759 | 131567 | superkingdom |  | 1 | 0 | 1 | 0 | 1 | 0 | 0 | 0 |  |

	real	0m0.002s
	user	0m0.000s
	sys		0m0.000s


Searching all name entries for a given tax id:


	$ ./ncbitc -n 9913
	9913 |  Bos Tauurus |  |  misspelling |
	9913 |  Bos bovis |  |  synonym |
	9913 |  Bos primigenius taurus |  |  synonym |
	9913 |  Bos taurus |  |  scientific name |
	9913 |  Bovidae sp. Adi Nefas |  |  includes |
	9913 |  bovine |  |  common name |
	9913 |  cattle |  |  genbank common name |
	9913 |  cow |  |  common name |
	9913 |  domestic cattle |  |  common name |
	9913 |  domestic cow |  |  common name |
	
	real	0m0.001s
	user	0m0.000s
	sys		0m0.000s



# Installing


Download and build the program:

	git clone git://github.com/mvneves/ncbitaxcollector
	cd ncbitaxcollector
	make


Download gi_taxid_nucl.dmp, names.dmp and nodes.dmp from NCBI's FTP site:

	wget ftp://ftp.ncbi.nih.gov/pub/taxonomy/taxdump.tar.gz
	tar xzf taxdump.tar.gz


# Binary Format Description

One of main characteristics of our tool is its ability to create optimized binary versions of the NCBI database files.
Here is the a detailed description of the binary records used in our code. 
Please refer to the [NCBI Handbook](http://www.ncbi.nlm.nih.gov/books/NBK21100/) for more information about the fields.

## Nodes ##

This struct represents a binary entry/record in the taxonomy nodes file.

	struct nodes_dmp {
		int tax_id;							// node id in GenBank taxonomy database
		int parent_tax_id;					// parent node id in GenBank taxonomy database
		char rank;							// rank of this node (superkingdom, kingdom, ...) 
		char embl_code[3];					// locus-name prefix; not unique
		short division_id;					// see division.dmp file
		char inherited_div_flag;			// 1 if node inherits division from parent
		short genetic_code_id;				// see gencode.dmp file
		char inherited_GC_flag;				// 1 if node inherits genetic code from parent
		int mitochondrial_genetic_code_id;	// see gencode.dmp file
		char inherited_MGC_flag;			// 1 if node inherits mitochondrial gencode from parent
		char GenBank_hidden_flag;			// 1 if name is suppressed in GenBank entry lineage
		char hidden_subtree_root_flag;		// 1 if this subtree has no sequence data yet
	#ifdef NCBITC_WITH_COMMENTS
		char comments[255];					// free-text comments and citations
	#endif
	};

Total of 28 bytes per record.

## Names ##

This struct represents a binary entry/record in the taxonomy names file.

	struct names_dmp {
		int tax_id;								// the id of node associated with this name
		char name_txt[NCBITC_MAX_LONG_NAME];	// name itself
		char unique_name[NCBITC_MAX_LONG_NAME];	// the unique variant of this name if name not unique
		char name_class[NCBITC_MAX_LONG_NAME];	// (synonym, common name, ...)
	};

Total of 196 bytes per record.

## GI to NCBI ID Mapping ##

This struct represents a binary entry/record in the mapping file.

	struct gi_taxid_nucl_dmp {
	#ifdef NCBITC_WITH_GIINDEX
		int gi;					// GI field of the FASTA format
	#endif
		int tax_id;				// NCBI taxonomy id
	};

Total of 4 bytes per record.

# References

- NCBI Taxonomy tree in SQL: [http://linnaeus.zoology.gla.ac.uk/~rpage/tbmap/downloads/ncbi/](http://linnaeus.zoology.gla.ac.uk/~rpage/tbmap/downloads/ncbi/) 
- A TaxCollector in python: [https://github.com/audy/taxcollector](https://github.com/audy/taxcollector)
- The Taxonomy Project - The NCBI Handbook: [http://www.ncbi.nlm.nih.gov/books/NBK21100/](http://www.ncbi.nlm.nih.gov/books/NBK21100/)
- Taxdump readme: [ftp://ftp.ncbi.nih.gov/pub/taxonomy/taxdump_readme.txt](ftp://ftp.ncbi.nih.gov/pub/taxonomy/taxdump_readme.txt)
