# Mining articles for sequence data

Starting with a .pdf file, make sure the file name contains no spaces and special characters. Let's assume your article .pdf file is called GreatScience.pdf for this walkthrough.

**Step 0**: Activate environment, if you haven't done so to access the notebook. Throughout the notebook, we need to let your operating system know where conda put the perl modules. 
Note: We've put as many tools as possible in the pdf2seq.yaml for a quick conda installation (if your unfamiliar with conda/creating environments from files, see [here](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html)). *On the first use, you also need to install GROBID and teitocsv, as described [here](https://grobid.readthedocs.io/en/latest/Install-Grobid) and [here](https://github.com/komax/teitocsv).*

**Step 1**: Check .pdf format. We use `pdfinfo` from [poppler tools](https://poppler.freedesktop.org/):


```bash
%%bash
pdfinfo GreatScience.pdf
```

    Title:          Whole metagenome profiles of particulates collected from the International Space Station
    Subject:        Microbiome, 2017, doi:10.1186/s40168-017-0292-4
    Keywords:       International Space Station,Microbiome,Functional metagenomics,Built environment,Cleanroom,Propidium monoazide
    Author:         Nicholas A. Be
    Creator:        Arbortext Advanced Print Publisher 9.1.440/W Unicode
    Producer:       Acrobat Distiller 10.1.5 (Windows); modified using iText® 5.3.5 ©2000-2012 1T3XT BVBA (AGPL-version)
    CreationDate:   Sat Jul 15 05:26:10 2017 CEST
    ModDate:        Mon Jul 17 18:20:29 2017 CEST
    Tagged:         no
    UserProperties: no
    Suspects:       no
    Form:           none
    JavaScript:     no
    Pages:          19
    Encrypted:      no
    Page size:      595.276 x 790.866 pts
    Page rot:       0
    File size:      2119517 bytes
    Optimized:      no
    PDF version:    1.4


**Step 2**: Convert .pdf file to text. We will use this file later to search for accession numbers. We use `pdftotext` from [poppler tools](https://poppler.freedesktop.org/):


```bash
%%bash
mkdir GreatScience_text_mining
pdftotext GreatScience.pdf GreatScience_text_mining/GreatScience.txt
head GreatScience_text_mining/GreatScience.txt
```

    Be et al. Microbiome (2017) 5:81
    DOI 10.1186/s40168-017-0292-4
    
    RESEARCH
    
    Open Access
    
    Whole metagenome profiles of particulates
    collected from the International Space
    Station


**Step 3**: Convert .pdf article file to TEI XML. The .xml will contain structured information, including DOI, Journal, year of publication amongst others. We use [GROBID](https://github.com/kermitt2/grobid)'s command `processFulltext` for this task:


```bash
%%bash
java -jar grobid-0.5.6/grobid-core/build/libs/grobid-core-0.5.6-onejar.jar -gH grobid-0.5.6/grobid-home -dIn . -dOut GreatScience_text_mining -exe processFullText 2> GreatScience_text_mining/grobid.log
```

    Processing: ./GreatScience.pdf
    
    ************************************************************************************
    COUNTER: org.grobid.core.engines.counters.ReferenceMarkerMatcherCounters
    ************************************************************************************
    ------------------------------------------------------------------------------------
      STYLE_NUMBERED:        101
      INPUT_REF_STRINGS_CNT: 101
      MATCHED_REF_MARKERS:   122
    ====================================================================================
    
    ************************************************************************************
    COUNTER: org.grobid.core.engines.label.TaggingLabelImpl
    ************************************************************************************
    ------------------------------------------------------------------------------------
      FIGURE_LABEL:             5
      CITATION_TITLE:           85
      NAME-HEADER_MIDDLENAME:   6
      CITATION_DATE:            86
      CITATION_AUTHOR:          85
      FULLTEXT_FIGURE:          10
      NAME-HEADER_SURNAME:      14
      NAME-HEADER_OTHER:        16
      NAME-CITATION_OTHER:      389
      FIGURE_FIGDESC:           4
      CITATION_NOTE:            4
      CITATION_VOLUME:          81
      FULLTEXT_CITATION_MARKER: 200
      CITATION_LOCATION:        2
      FULLTEXT_TABLE_MARKER:    2
      CITATION_WEB:             3
      CITATION_INSTITUTION:     1
      FULLTEXT_SECTION:         74
      NAME-HEADER_FORENAME:     14
      CITATION_PAGES:           84
      CITATION_ISSUE:           65
      NAME-HEADER_MARKER:       14
      CITATION_JOURNAL:         81
      NAME-CITATION_FORENAME:   468
      NAME-CITATION_SURNAME:    458
      CITATION_PUBLISHER:       1
      CITATION_OTHER:           475
      FULLTEXT_FIGURE_MARKER:   114
      NAME-CITATION_MIDDLENAME: 2
      CITATION_TECH:            1
      CITATION_PUBNUM:          1
      FULLTEXT_PARAGRAPH:       456
      FIGURE_FIGURE_HEAD:       5
    ====================================================================================
    
    ************************************************************************************
    COUNTER: FigureCounters
    ************************************************************************************
    ------------------------------------------------------------------------------------
      STANDALONE_FIGURES:           2
      ASSIGNED_GRAPHICS_TO_FIGURES: 5
    ====================================================================================
    ====================================================================================
    



```bash
%%bash
head GreatScience_text_mining/GreatScience.tei.xml | grep -v "schemaLocation"
```

    <?xml version="1.0" encoding="UTF-8"?>
    <TEI xml:space="preserve" xmlns="http://www.tei-c.org/ns/1.0" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
     xmlns:xlink="http://www.w3.org/1999/xlink">
    	<teiHeader xml:lang="en">
    		<fileDesc>
    			<titleStmt>
    				<title level="a" type="main">Whole metagenome profiles of particulates collected from the International Space Station</title>
    			</titleStmt>


**Step 4**: Mine TEI XML and text for article metadata, in addition to accession number and keywords. This is accomplished by the command `XXX` from teitocsv. This will return a comma separated text file with all the interesting information. In particular, each line will contain one accession number.


```bash
%%bash
teitocsv/teitocsv/bacteriacsv.py GreatScience_text_mining/ GreatScience_text_mining/GreatScience.txt GreatScience_text_mining/GreatScience.csv
cat GreatScience_text_mining/GreatScience.csv
```

    Handled GreatScience_text_mining/GreatScience.tei.xml
    Done with parsing
    Done with appending
    Done with csv
    ID,Title,DOI,16ness,accession,515f,806r,seq_method,gene_region1,gene_region2,gene_region3,gene_region4,gene_region5,gene_region6,gene_region7,gene_region8,gene_region_9
    GreatScience,Whole metagenome profiles of particulates collected from the International Space Station,10.1186/s40168-017-0292-4,True,PRJNA363053,,,illumina,,,,,,,,,


**Step 5**: Access metadata via esearch and efetch NCBI's edirect suite. 


```bash
%%bash
export PERL5LIB=$CONDA_PREFIX/lib/site_perl/5.26.2
mkdir GreatScience_SRA_data
for acc in `cut -f 5 -d "," GreatScience_text_mining/GreatScience.csv | tail -n +2 | uniq `; do
  echo $acc
  esearch -db sra -q $acc | efetch -format runinfo >> GreatScience_SRA_data/runInfo.$acc.tsv
  head -n 3 GreatScience_SRA_data/runInfo.$acc.tsv
done
```

    PRJNA363053
    Run,ReleaseDate,LoadDate,spots,bases,spots_with_mates,avgLength,size_MB,AssemblyName,download_path,Experiment,LibraryName,LibraryStrategy,LibrarySelection,LibrarySource,LibraryLayout,InsertSize,InsertDev,Platform,Model,SRAStudy,BioProject,Study_Pubmed_id,ProjectID,Sample,BioSample,SampleType,TaxID,ScientificName,SampleName,g1k_pop_code,source,g1k_analysis_group,Subject_ID,Sex,Disease,Tumor,Affection_Status,Analyte_Type,Histological_Type,Body_Site,CenterName,Submission,dbgap_study_accession,Consent,RunHash,ReadHash
    SRR5197511,2017-01-31 16:39:51,2017-01-31 15:51:26,18648924,5317902837,18648924,285,2171,,https://sra-downloadb.be-md.ncbi.nlm.nih.gov/sos1/sra-pub-run-1/SRR5197511/SRR5197511.1,SRX2512994,ISS_DN,WGA,MDA,METAGENOMIC,PAIRED,0,0,ILLUMINA,NextSeq 500,SRP097683,PRJNA363053,,363053,SRS1013545,SAMN03863798,simple,1256227,indoor metagenome,ISS_Debris,,,,,,,no,,,,,LAWRENCE LIVERMORE NATIONAL LABORATORY,SRA531055,,public,B00A8A6761169CB4EBFA76E3EE52EC5A,6582882B4318A23E8AA8C2AA65D3B434
    SRR5197512,2017-01-31 16:39:51,2017-01-31 15:59:12,18031154,4738329368,18031154,262,1817,,https://sra-downloadb.be-md.ncbi.nlm.nih.gov/sos2/sra-pub-run-9/SRR5197512/SRR5197512.1,SRX2512995,ISS_DP,WGA,MDA,METAGENOMIC,PAIRED,0,0,ILLUMINA,NextSeq 500,SRP097683,PRJNA363053,,363053,SRS1013547,SAMN03863799,simple,1256227,indoor metagenome,ISS_Debris_PMA,,,,,,,no,,,,,LAWRENCE LIVERMORE NATIONAL LABORATORY,SRA531055,,public,1A85C07B7ADA1C21200CFAE9EB505491,CE907CFFF9D8A46EB1027E771A16DC00


**alternative to step 5**: instead of searching sra, we could search the database containing the sort of accession we have (e.g. bioproject, run, biosample). An example for a bioproject is this:
```
acc="PRJNA356464"
esearch -db bioproject -query $acc | elink -target sra | efetch -format runinfo >> GreatScience_SRA_data/runInfo.$acc.csv
```
And a biosample would be entered like so:
```
acc="SAMN06114300"
esearch -db biosample -query $acc | elink -target sra | efetch -format runinfo >> GreatScience_SRA_data/runInfo.$acc.csv
```
While a run accession could be directly used like this:
```
acc="SRR2939657"
efetch -db sra -format runinfo -id $acc >> GreatScience_SRA_data/runInfo.$acc.csv
```

**Step 6**: For each biosample, access metadata on NCBI using edirect. *At this point, we could filter the list and include only runs with a specific sequencing technology, layout etc.*


```bash
%%bash
export PERL5LIB=$CONDA_PREFIX/lib/site_perl/5.26.2
for acc in `cut -f 5 -d "," GreatScience_text_mining/GreatScience.csv | tail -n +2 | uniq`; do
  for sam in `awk -F ',' 'NR==1 {for (i=1; i<=NF; i++) {f[$i] = i}}{if (NR!=1) print $(f["BioSample"]) }' GreatScience_SRA_data/runInfo.$acc.tsv | uniq`; do
    esearch -db sra -query $sam | elink -target biosample | efetch -format docsum | xtract -pattern DocumentSummary -element Attribute@attribute_name,Attribute >> GreatScience_SRA_data/biosampleInfo.$acc.$sam.tsv
  done
done
```


```bash
%%bash
ls GreatScience_SRA_data/biosampleInfo.*.tsv
```

    GreatScience_SRA_data/biosampleInfo.PRJNA363053.SAMN03863796.tsv
    GreatScience_SRA_data/biosampleInfo.PRJNA363053.SAMN03863797.tsv
    GreatScience_SRA_data/biosampleInfo.PRJNA363053.SAMN03863798.tsv
    GreatScience_SRA_data/biosampleInfo.PRJNA363053.SAMN03863799.tsv
    GreatScience_SRA_data/biosampleInfo.PRJNA363053.SAMN03863800.tsv
    GreatScience_SRA_data/biosampleInfo.PRJNA363053.SAMN03863801.tsv


**Step 7**: Get first few reads of one run per accession using edirect. *Here, again, we can choose to use only some samples, based on what we know from the biosample metadata*.


```bash
%%bash
mkdir GreatScience_tmp
cd GreatScience_tmp
for acc in `cut -f 5 -d "," ../GreatScience_text_mining/GreatScience.csv | tail -n +2 | uniq`; do
  for run_acc in `awk -F ',' 'NR==1 {for (i=1; i<=NF; i++) {f[$i] = i}}{if (NR==2) print $(f["Run"]) }' ../GreatScience_SRA_data/runInfo.$acc.tsv | uniq`; do
    prefetch $run_acc && vdb-validate $run_acc && fastq-dump -X 4 --split-3 $run_acc
    cat $run_acc*.fastq
  done
done
cd ..
rm -r GreatScience_tmp
```

    
    2020-03-19T10:50:06 prefetch.2.9.6: 1) 'SRR5197511' is found locally
    2020-03-19T10:50:06 prefetch.2.9.6: 'SRR5197511' has 0 unresolved dependencies
    Read 4 spots for SRR5197511
    Written 4 spots for SRR5197511
    @SRR5197511.1 1 length=151
    CGTATNGCCTGCGATACATGAGCGAGGANGAGCGGGAGCGCCAGCGNGAGCGTGGACGGCGCGGCGGACTNGTGAACAGCCAANAGCAGCAGCAAGCAAGGGCGCAAGGCCCGCAGGTCNCTGCCATGATCCGTTCGGNCNAAGCCGTAGG
    +SRR5197511.1 1 length=151
    AAAAA#EEEEEEEEEEEEEEEEEEEEEE#EEEEEEEEEEEEEEEEE#EEEEEEEEEEEEEEEEEEEEEEE#EEEAEE/EEEEE#EEEEEEEEEEEEEE/E/EEEEEAEEEEEEEEEEEE#EEAEEEEE/EEEAEEEAA#A#/AAEEAE<<A
    @SRR5197511.2 2 length=151
    CTAGTNAGTTCCAACCACTGTGCGTCCANAACCCGCCCACCTCATTTGCTATTTCTCGGTTGGGGTCTAANGGTGTCCCAGCCNCCGGCGCGGGCTCGTGCCACGTTGTGGGCCTTTGGNCCCGGTTCGTGTTGAACCNGNACTAAAGGGG
    +SRR5197511.2 2 length=151
    AAAAA#EEEEEE/AEEEEEEEEEEEEEE#EAEEEEEEAEEEEEEEEEEEEEAEEEAEEEEEEEEEEEEE/#EEEEEEEEEEEE#EEA<EEEEEEEEEEEEEEAEEEEEEEE/EEEEEEE#<EE<EEEE/EEEEA/<EA#/#EEEAAE<EE<
    @SRR5197511.3 3 length=151
    AAAGGNGGGGACCTTTAGTGCCCACCCTTTAGTGCCGTCTCCAGAACCGGCACTAAAGGGCCTTACGAACCGGGACTATACCCCAAAACAGCACAAATGACTTCCAACCACTGAGCGACNAGAACACGACCACCACATNTNGGATTTCGCG
    +SRR5197511.3 3 length=151
    AAAAA#EEEEEEAEEEEEEEEE/AAEEEE<EEEEEAEAE/EEE/AEAA//EEEEEEEEEEEEEE/A/<//////<<///</EA<A//////<///6//E////<//6E<E/A///E///#///////////<<///</#E#/////////6
    @SRR5197511.4 4 length=151
    CCTTTAGTGCCCACCCTTTAGTGCCGTCTCCAGAACCGGCACTAAAGGGCCTTACGAACCGGGACTATAGCCCAATTCTGCACTAGTGAGTTCCAACCACTGTGCGTCCAGAACCCGCCNACCTCATTTGCTATTTCTNGNTTGGGGTCTA
    +SRR5197511.4 4 length=151
    AAAAAEEEEEEAEEEEEEEEEEEAEEEEEEEEEEEEEEEEEEEAEAEEEEEEEEEEEAEEEAE/AEEE/A/</EEEEAEEE/<E/E</EEEE/E/E</<EE/EE<AE//E/EAEE/<A/#E/<E/EEEE//EEEEA/<#A#EEEAE<EAE/
    @SRR5197511.1 1 length=151
    CCCCGAGGCNTNNCCGNNNNNNNNNNNNNNNNNNCAANNNNNNNNNNCCNNNNNNCNNNNNNNNNNNNNNNNNNNNNNNNNNNNTCGNGCCTGNCNNNNNNNNNNNNNNANGGANNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
    +SRR5197511.1 1 length=151
    AAAAAEEEE#E##EEE##################EEE##########EE######E############################EEE#EEEEE#E##############E#EEE#####################################
    @SRR5197511.2 2 length=151
    CTAAAGGTCCCNNCCTNNNNNNNNNNNNNNNNNCGAANNNNNNNNNNAGGNCNACANNNNNNNNNGNNNCNGNNNNNNNNGNTNGGACACCGTNAGACNNNNNNNNNNNAATAGNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
    +SRR5197511.2 2 length=151
    AAAAAEAEEEE##EEE#################A/AE##########EEA#E#EEE#########/###E#/########A#E#/EAEEEAAE#AEAE###########/E</6#####################################
    @SRR5197511.3 3 length=151
    GCCCATTAGTGNCGGTTCNNNNNNNNNCNCTNNAGGGTGGNCNNTANAGGNCCCCCCNNNNANTNCTCTTNANNNNNNNNCNCGCACCAAACGNACACANNNNNNNNNAACCCCGNNNANNNNNNNNNNNNNNNNNNNNNNNNANNNNNNN
    +SRR5197511.3 3 length=151
    AAAA/EEEEEE#EEEAEE#########E#EE##/E/AEEE#E##EE#E/E#6EEEAE####A#/#6E6/6#E########/#/6///E/<</6#/EE//#########/6//AA/###/########################/#######
    @SRR5197511.4 4 length=151
    GGTTCGTAAGGNCCTTTANNNNNNNNTCTGGNNACGGCACNANNGGNTGGNCACTAAAGNNANCNCCCTTNANNNNNGGTTCAACACGAACCGNGACCANNGNNNNNCAACGTGGNNNANCNNNNNNNNNNNNNNNNNNNAANCNNNNNNN
    +SRR5197511.4 4 length=151
    AAAAAEEEEEE#EEEEE/########E<EEE##EEEEEEE#/##EE#EEE#EEEE<EEE##/#/#/E/EE#/#####/<EEEAE/AE</AE</#/</AA##/#####/<E//AE/###/#/###################/6#/#######


    2020-03-19T10:50:06 vdb-validate.2.9.6 info: Validating '/work/ncbi/sra/SRR5197511.sra'...
    2020-03-19T10:50:06 vdb-validate.2.9.6 info: Database 'SRR5197511.sra' metadata: md5 ok
    2020-03-19T10:50:06 vdb-validate.2.9.6 info: Table 'SEQUENCE' metadata: md5 ok
    2020-03-19T10:50:07 vdb-validate.2.9.6 info: Column 'ALTREAD': checksums ok
    2020-03-19T10:50:09 vdb-validate.2.9.6 info: Column 'QUALITY': checksums ok
    2020-03-19T10:50:11 vdb-validate.2.9.6 info: Column 'READ': checksums ok
    2020-03-19T10:50:12 vdb-validate.2.9.6 info: Column 'READ_LEN': checksums ok
    2020-03-19T10:50:13 vdb-validate.2.9.6 info: Column 'READ_START': checksums ok
    2020-03-19T10:50:14 vdb-validate.2.9.6 info: Column 'SPOT_GROUP': checksums ok
    2020-03-19T10:50:14 vdb-validate.2.9.6 info: Database '/work/ncbi/sra/SRR5197511.sra' contains only unaligned reads
    2020-03-19T10:50:14 vdb-validate.2.9.6 info: Database 'SRR5197511.sra' is consistent


**Step 7b**: If the files look alright, and the validation is okay, get the whole set (avoiding duplicates). This will take a while, if there are many files.


```bash
%%bash
mkdir GreatScience_seqs
for acc in `cut -f 5 -d "," GreatScience_text_mining/GreatScience.csv | tail -n +2 | uniq`; do
  awk -F ',' 'NR==1 {for (i=1; i<=NF; i++) {f[$i] = i}}{if (NR!=1) print $(f["Run"]) }' GreatScience_SRA_data/runInfo.$acc.tsv >> GreatScience_SRA_data/GreatScience.runIDs.txt
done
cd GreatScience_seqs
for run_acc in `cat ../GreatScience_SRA_data/GreatScience.runIDs.txt | sort | uniq`; do
    prefetch $run_acc &>> GreatScience.getFastq.log && vdb-validate $run_acc &>> GreatScience.getFastq.log && fastq-dump --split-3 $run_acc &>> GreatScience.getFastq.log
done
```

    Process is terminated.


We can review the validation results in the file `GreatScience_seqs/GreatScience.getFastq.log` .
