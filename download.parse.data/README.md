# Archival stability pipeline

Multiple scripts were combined to collect and evaluate the data in the "archival stability" section of our study.

## Step 0: Download code

These instructions make several assumptions: first, that your analysis is being performed within the repository directories, and second, that the commands are being performed in a Linux-like shell (either Linux, OSX, or a Bash environment in Windows). To begin, download all the code by running:

```sh
git clone https://github.com/smangul1/good.software.git
cd good.software/download.parse.data
```

## Step 1: Download data

We used data pulled directly from PubMed Central for this part of the study. The entire open access dataset can be downloaded at **[ftp://ftp.ncbi.nlm.nih.gov/pub/pmc/oa_bulk/](https://ftp.ncbi.nlm.nih.gov/pub/pmc/oa_bulk/)**, which is also accessible via your browser.

The collections are organized by the first letter of the journal name. For example, running this command would download the data for all journals starting with `A` through `B`:

```sh
wget ftp://ftp.ncbi.nlm.nih.gov/pub/pmc/oa_bulk/non_comm_use.A-B.xml.tar.gz
```

Extracting the files from this archive will reveal a set of directories, one for each journal. Within each of these directories, there is an XML file for each article. We provide an example journal directory (compressed) at [download.parse.data/Nat_Methods.tar.gz](https://github.com/smangul1/good.software/blob/master/download.parse.data/). (This can be extracted by running `tar -xf Nat_Methods.tar.gz` from within the `downloads.parse.data/` directory.)

The example directory is for the journal _Nature Methods_, but we pulled data from the FTP repository above for 10 journals:

```
BMC_Genomics
Genet_Res
Genome_Med
Nat_Methods
PLoS_Comput_Biol
BMC_Bioinformatics
BMC_Syst_Biol
Genome_Biol
Nat_Biotechnol
Nucleic_Acids_Res
```

To complete this ste,p download the data for whatever journals you want to evaluate, and place their directories directly within the `good.software/download.parse.data` directory. For example, if you only wanted to evaluate articles from _Nature Methods_, your directory structure would look like this:

```
good.software/
    download.parse.data/
        Nat_Methods/
```

## Step 2: Extract links, perform initial checks

Once the journal directories are all organized, navigate to the `good.software/download.parse.data/` directory in your terminal and run the `getLinksStatus.py` script, which takes a single parameter: the name of a single journal. This parameter should match the name of the journal's directory. For example, to process the links for _Nature Methods_:

```sh
python getLinksStatus.py Nat_Methods
```

**IMPORTANT NOTE:** Though most of the code in this repository is in Python 3, **the `getLinksStatus.py` script requires Python 2.**

Running this script for each journal you want to evaluate will put two files in the `download.parse.data/` directory: `abstractLinks.prepared.tsv` and `bodyLinks.prepared.tsv`.

**ANOTHER NOTE:** Each time you run this script, it will append results to the end of these two files. If you want to restart the analysis, you should first remove `abstractLinks.prepared.tsv` and `bodyLinks.prepared.tsv`.

The last step is to run the `clean.sh` script to combine these files into `links.unchecked.csv`. It does not require any parameters to run:

```sh
./clean.sh
```

## Step 3: Run detailed request checks

Improvements to the link screening process required re-processing some of the failed requests to ensure they were actually "failures." This script requires **Python 3** and the Python modules specified in `requirements.txt`; to install all dependencies and run the script, run the following commands from within the `download.parse.data/` directory:

```sh
python -m venv .
source bin/activate
pip install -r requirements.txt

# Run the script. First parameter is source data, second parameter is where output should be directed.
python recheck_timeouts.py links.unchecked.csv links.bulk.csv

deactivate
```

## Step 4: Collect additional data for analysis

There are two additional files that need to be generated for the analysis that is performed in the "Figure1" Jupyter notebook.

### Step 4a: Minor redirection information

If a redirection response changes only the protocol of a request (for example, `http://google.com` to `https://google.com`), then we count that as a `200` instead. Once `links.bulk.csv` is generated, you can generate the file with this redirection information in it by running the following command within the `download.parse.data/` directory:

```sh
python redirection.py
```

This will create a file called `http2https.redirected.csv`, which is used in the Jupyter notebook analysis.

### Step 4b: Altmetric data

The portion of the analysis that incorporates Altmetric data requires that data first be fetched from their API. This uses the same virtual environment as step 3:

```sh
source bin/activate
python fetching_altmetric.py
deactivate
```

## Step 5: Analysis

The final file created in step 3 (`links.bulk.csv`) is in same `links.bulk.csv` source file referred to in the Jupyter notebooks. Using this output as the source for the figures should generate results in the same way we did for our paper.
