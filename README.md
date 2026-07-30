# ![nfcore/test-datasets](docs/images/test-datasets_logo.png)
Test data to be used for automated testing with the nf-core pipelines

> ⚠️ **Do not merge your test data to `master`! Each pipeline has a dedicated branch (and a special one for modules)**

## Introduction

nf-core is a collection of high quality Nextflow pipelines. This repository contains various files for CI and unit testing of nf-core pipelines and infrastructure.

The principle for nf-core test data is as small as possible, as large as necessary. Please see the [guidelines](https://nf-co.re/docs/contributing/test_data_guidelines) for more detailed information. Always ask for guidance on the [nf-core slack](https://nf-co.re/join) before adding new test data.

## Documentation

nf-core/test-datasets comes with documentation in the `docs/` directory:

01. [Add a new  test dataset](https://github.com/nf-core/test-datasets/blob/master/docs/ADD_NEW_DATA.md)
02. [Use an existing test dataset](https://github.com/nf-core/test-datasets/blob/master/docs/USE_EXISTING_DATA.md)

## Downloading test data

Due the large number of large files in this repository for each pipeline, we highly recommend cloning only the branches you would use.

```bash
git clone <url> --single-branch --branch <pipeline/modules/branch_name>
```

To subsequently clone other branches[^1]

```bash
git remote set-branches --add origin [remote-branch]
git fetch
```

## Support

For further information or help, don't hesitate to get in touch on our [Slack organisation](https://nf-co.re/join/slack) (a tool for instant messaging).

[^1]: From [stackoverflow](https://stackoverflow.com/a/60846265/11502856)


## Funcprofiler  CI test specific information

### humann

We include two sets of MetaPHlan/HUMAnN databases: one compatible with the HUMAnN 3.6 and one with HUMAnN 4.0a. These are derived from the toy datasets bundled with these HUMAnN releases, with additional subsetting of the utility databases to further shrink the sizes of the files. The file names themselves are changed as well, as the tools detect the usage of the demo files and issue warnings accordingly.

#### A note about versions

In order to understand the database compatibility network, the following can be run:

```bash
for tag in 3.6.1 3.7 v3.8 v3.9 4.0.0.alpha.1 4.0.0.alpha.1-final ; do git checkout $tag ; cat humann/config.py | grep -e "metaphlan_v4_db_version\|metaphlan_v3_db_version\|metaphlan_v3_db_matching_uniref\|matching_uniref" | sed "s|^|$tag\t|g" ; done  > version_table

#### Generating the test datasets

The data for HUMAnN v3 is generated as follows:

```bash
mkdir -p ./data/database/humann/v3
mkdir -p ./data/database/humann/v4


# make a local copy of humann, checkout an old release,
# git clone https://github.com/biobakery/humann
# git checkout 3.6.1

# copy over the test and utility databases

cp -r <path to clone of humann repo>/humann/humann/data/ ./data/database/humann/v3
rm -r data/database/humann/v3/pathways
# rename so it doesn't warn about us using demo databases
mv ./data/database/humann/v3/chocophlan_DEMO/ ./data/database/humann/v3/chocophlan_nfDEMO/
mv ./data/database/humann/v3/uniref_DEMO/ ./data/database/humann/v3/uniref_nfDEMO/

# retain only strictly necessary files (ie any file that is in the full db at https://g-227ca.190ebd.75bc.data.globus.org/humann_data/full_mapping_v201901b.tar.gz)
for f in map_EC_to_triplet_AC_U50_U90_Swissprot_and_Trembl.txt.gz KeggOrgId2OrgNameTable.txt map_infogo1000_uniref50.txt.gz map_infogo1000_uniref90.txt.gz  map_kegg-pwy_name.txt.gz   map_metacyc-pwy_name.txt.gz map_metacyc-rxn_name.txt.gz map_transporter_uniref50.txt.gz mpa_vJan21_CHOCOPhlAnSGB_202103.tsv uniref50-tol-lca.dat.gz uniref90-tol-lca.dat.gz ; do
rm ./data/database/humann/v3/misc/$f
done

# shrink the remaining files
find ./data/database/humann/v3/misc/ -name "*.gz" | while read f ; do tmpfile="tmp_$(basename $f)"; mv $f $tmpfile ; zcat < $tmpfile | head -n 100 | gzip --best > $f ; rm $tmpfile ; done
find ./data/database/humann/v3/misc/ -name "*.bz2" | while read f ; do tmpfile="tmp_$(basename $f)"; mv $f $tmpfile ; bzcat < $tmpfile | head -n 100 | bzip2 --best > $f ; rm $tmpfile ; done



tar czf data/database/humann/v3/chocophlan_nfDEMO.tar.gz -C data/database/humann/v3/chocophlan_nfDEMO .
tar czf data/database/humann/v3/uniref_nfDEMO.tar.gz -C data/database/humann/v3/uniref_nfDEMO .
tar czf data/database/humann/v3/utility_nfDEMO.tar.gz -C data/database/humann/v3/misc .
```

The data for HUMAnN v4 is generated as follows:

```bash
# in the humann repo cloned above
# git checkout 4.0.0.alpha.1-final

cp -r <path to clone of humann repo>/humann/data/ ./data/database/humann/v4/
mv  ./data/database/humann/v4/chocophlan_DEMO/ ./data/database/humann/v4/chocophlan_nfDEMO/
mv ./data/database/humann/v4/uniref_DEMO/ ./data/database/humann/v4/uniref_nfDEMO/

mkdir tmp && cd tmp
wget http://huttenhower.sph.harvard.edu/humann_data/full_mapping_v4_alpha.tar.gz
tar xzf full_mapping_v4_alpha.tar.gz
mkdir tmp
ls *bz2 | while read f ; do tmpfile="tmp_$(basename $f)"; bzcat < $f | head -n 25 > tmpbz ;  bzip2 -c --best tmpbz > tmp/$f  ; done
find . -name "*txt.gz" | while read f ; do tmpfile="tmp_$(basename $f)"; zcat < $f | head -n 25 | gzip --best > tmp/$f  ; done
for f in metacyc_pathways_structured_filtered_v24_subreactions mpa_vJan21_CHOCOPhlAnSGB_202103.tsv unipathway_pathways vOct22_SGB_mapping.tsv ; do head $f > tmp/$f ; done
cd ../

tar czf data/database/humann/v4/utility_nfDEMO.tar.gz -C  tmp/tmp/ .
tar czf data/database/humann/v4/chocophlan_nfDEMO.tar.gz -C data/database/humann/v4/chocophlan_nfDEMO/ .
tar czf data/database/humann/v4/uniref_nfDEMO.tar.gz -C data/database/humann/v4/uniref_nfDEMO/ .

```

## Metaphlan database for HUMANn4

```bash
mkdir data/database/metaphlan
cd data/database/metaphlan
wget https://github.com/nf-core/test-datasets/raw/409834b927c3a4e9314691b1125acee1434f7dd8/data/delete_me/metaphlan4_database.tar.gz
for i in mpa_vJan21_TOY_CHOCOPhlAnSGB_20210* ; do newid=$(echo $i | sed "s|mpa_vJan21_TOY_CHOCOPhlAnSGB_202103|mpa_vOct22_CHOCOPhlAnSGB_202403|g") ; mv $i $newid ; done
rm metaphlan4_database.tar.gz
tar czf metaphlan_demo_for_humann4.tar.gz mpa*
```

## fmhfunprofiler

```bash
mkdir data/database/fmhfunprofiler/

cd data/database/fmhfunprofiler/
wget https://zenodo.org/records/10045253/files/KOs_sketched_scaled_1000.sig.zip

unzip KOs_sketched_scaled_1000.sig.zip

# find  ribosomal protein KOs; many start with K028 for a nice essential gene to test with
# added in K00147, found when running minigut R1 using full database
# see https://github.com/KoslickiLab/fmh-funprofiler/tree/dbb016b8c3a6d2d55e565d39b179dc27a96a62ec#start-from-scratch
mkdir -p tmp/signatures

cat SOURMASH-MANIFEST.csv | grep "K028\|K00147\|moltype\|SOURMASH" > tmp/SOURMASH-MANIFEST.csv
cat tmp/SOURMASH-MANIFEST.csv | tail -n+3 | cut -f1 -d, | while read x ; do cp $x tmp/$x ; done

cd tmp

zip ../KOs_sketched_scaled_1000_demo.sig.zip SOURMASH-MANIFEST.csv signatures/*
cd ../../../../
rm -r  data/database/fmhfunprofiler/signatures/ data/database/fmhfunprofiler/tmp/ data/database/fmhfunprofiler/SOURMASH-MANIFEST.csv
```


## Mifaser

```
mkdir data/database/mifaser
git clone https://bitbucket.org/bromberglab/mifaser/src/3e41beae9746189e116d6ec6b15c73af68e9a99c/ tmpmifaser
tar czf data/database/mifaser/GS-24-all.tar.gz  -C tmpmifaser/mifaser/database/ GS-24-all
rm -r tmpmifaser
```

## Eggnog-mapper

```
wget https://raw.githubusercontent.com/nf-core/test-datasets/modules/data/genomics/sarscov2/genome/proteome.fasta
podman run -v $PWD:$PWD quay.io/biocontainers/eggnog-mapper:2.1.13--pyhdfd78af_2 diamond makedb --in $PWD/proteome.fasta -d $PWD/proteome
mkdir data/database/eggnog-mapper
mv proteome.dmnd data/database/eggnog-mapper/

```

## Long reads: `data/reads/test_minigut_hifi.fastq.gz`

A simulated PacBio HiFi version of the same `minigut` community as the short-read test data, used by
`samplesheets/test_longread.csv` and `database_longread.csv`.

Why simulated rather than a real HiFi metagenome: the demo databases on this branch are tuned to the
minigut community — the FMH FunProfiler sketch above keeps only `K028*` plus `K00147`, chosen by running
minigut R1 against the full database. A HiFi fixture from any other community returns empty profiles
against them, so it cannot show that long reads are profiled correctly. Keeping the community identical
also makes short-read and long-read output directly comparable.

Community, taken from `test-datasets@mag:test_data/test_minigut_abundance.txt.gz`:

| Accession | Organism | Abundance |
|---|---|---|
| `NC_000913.3` | *Escherichia coli* K-12 MG1655 | 0.42 |
| `NC_006347.1` | *Bacteroides fragilis* YCH46 | 0.58 |

Abundances are applied as a fraction of total simulated bases, so per-genome depth is
`fraction * 10 Mbp / genome_length` — 0.9049x for E. coli, 1.0991x for B. fragilis.

Resulting file: 643 reads, 9,665,815 bp, mean read 15,032 bp (min 7,814, max 25,915), N50 15,594,
Q30 97.5%, 8.6 MB gzipped.

```bash
conda create -n pbsim3 -c conda-forge -c bioconda pbsim3 pbccs samtools seqkit
conda activate pbsim3

mkdir -p ref data/reads && cd ref
for acc in NC_000913.3 NC_006347.1 ; do
  curl -sSL "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=${acc}&rettype=fasta&retmode=text" -o ${acc}.fa
done
cd ..

# MODELS is the data/ directory of the pbsim3 installation ($CONDA_PREFIX/data).
# --pass-num 10 makes pbsim3 emit a multi-pass subread BAM, which is what ccs consumes to
# produce HiFi reads; difference-ratio 22:45:33 is pbsim3's recommended PacBio Sequel setting.
simulate () {  # tag accession depth seed
  pbsim --strategy wgs \
        --method qshmm --qshmm $MODELS/QSHMM-RSII.model \
        --genome ref/$2.fa --depth $3 --seed $4 \
        --length-mean 15000 --length-sd 3000 --length-min 5000 --length-max 30000 \
        --accuracy-mean 0.85 --difference-ratio 22:45:33 --pass-num 10 \
        --prefix $1 --id-prefix $1
  ccs ${1}_0001.bam ${1}.hifi.bam --num-threads 8
  samtools fastq ${1}.hifi.bam > ${1}.hifi.fastq
}

simulate ecoli NC_000913.3 0.9049 42
simulate bfrag NC_006347.1 1.0991 43

cat ecoli.hifi.fastq bfrag.hifi.fastq \
  | seqkit shuffle -s 42 \
  | gzip -9 > data/reads/test_minigut_hifi.fastq.gz
```

Read IDs keep their `ecoli`/`bfrag` prefix, so the fixture doubles as its own ground truth.

`samplesheets/test_longread.csv` points at this branch on the `vinisalazar` fork; repoint it at
`nf-core/test-datasets` once the branch is merged upstream. `database_longread.csv` carries the
`db_type` column (`long` on every row) and passes `--long-reads` to DIAMOND as `db_params`, following
`nf-core/taxprofiler`'s `database_v3.0.csv`.
