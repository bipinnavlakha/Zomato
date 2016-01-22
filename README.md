# Reads Simulation wrapper
This is a small wrapper script which will call upon dwgsim to create simulated data of specified mixture ratios. The script will output a .fastq file containing merged reads for all the specified contributors. It can also optionally call upon a shell script for mapping the .fastq file.
The output .bam file from mapping can be used with the microhaplotyper.jar file.

It simulates mixtures by varying the coverage for mixture contributors. So if a 1:3 mixture is specified the wrapper will take a base coverage value say 100 and then generate reads seperately for the two contributors at 1(100x) and 3(300x) coverage. After generating the reads for individual samples it will mix them to give a merged .fastq file which can be further used for mapping.

The user can specify the microhaplotype alleles for the contributors is a .csv file. These mutations will be incorporated in the generated reads.

## Installing Prerequisites
### Installing dwgsim
The simulation tool [dwgsim](https://github.com/nh13/DWGSIM) can be installed on Ubuntu and Mac in the following steps
#### Mac
```bash
git clone https://github.com/nh13/DWGSIM.git
cd DWGSIM
git submodule init
git submodule update
make
sudo cp dwgsim /usr/bin
```
#### Ubuntu 14.04 LTS
```bash
git clone https://github.com/nh13/DWGSIM.git
cd DWGSIM
git submodule init
git submodule update
sudo apt-get install libncurses5-dev
make
sudo cp dwgsim /usr/bin
```
The commands are also included in the installation scripts for dwgsim at scripts/dwgsim_install_script_mac.sh and scripts/dwgsim_install_script_ubuntu.sh
If you have installed dwgsim at any other path than /usr/bin/dwgsim please update the path in the config.ini file
It would also be helpful to go through the dwgsim wiki at https://github.com/nh13/DWGSIM/wiki/Simulating-Reads-with-DWGSIM

### Installing python modules
The script will require one python module i.e. pyvcf as a prerequisite. If you have pip installed on you system then installation should be straightforward as
```bash
pip install pyvcf
```

If you dont have pip on you system you can install it using the following commands
#### Mac
```bash
wget https://bootstrap.pypa.io/get-pip.py
sudo get-pip.py
```

#### Ubuntu
```bash
sudo apt-get install python-pip
```

## Installing the wrapper
Once the prerequisites are installed you can use the wrapper script. If you will be using the wrapper script within the folder where it is present then no more configuration should be required. If you want to use it from a different directory that where the script is present do the following change in config file
```bash
# change
scriptDir=.
# to [Don't add / at the end of the path]
scriptDir=<path where the script is located e.g. /home/user1/readsSimuationWrapper>
```

## Using the Wrapper
Once done with the installation you can try running the wrapper. To get help type the following command
```bash
python readSimulationWrapper -h
```
To generate a small mixture dataset with default set of options simply run
```bash
python readSimulationWrapper
```
You should have a .fastq file with the name sim_reads.fastq. This is the merged reads fastq file. Intermediate files can be found in sim_reads folder.

To change the mixture ratio to a different one try
```bash
python readSimulationWrapper -r 1:2:1
```

To map the reads and generate a bam file try(you can use the tmap_align or bwa_align scripts here. You should have tmap/bwa,samtools installed to run these scripts)
```bash
python readSimulationWrapper -r 1:2:1 -a ./scripts/tmap_align.sh
```

Open the config.ini file and change some parameter say first_read_length from 198 to 100 and rerun the simulator as
```bash
python readSimulationWrapper
```
Open the fastq file to see the new read length.

The wrapper will give the output of dwgsim and the mapping tools if any onto stdout

## Wrapper Command line options
* **-f REFERENCEFILE, --fasta REFERENCEFILE** - [FILEPATH, OPTIONAL, Default:From Config file] - The reference genome to generate reads against in .fasta/.fa format. For a sample reference file refer to */defaults/Default_Human_Mitochondira.fasta*.
* **-b BEDFILE, --bed BEDFILE** - [FILEPATH, OPTIONAL, Default:From Config file] - A bed file specifying the regions to restrict the generated reads to. The bed file should have three columns CHR, START POS, END POS. Please remove the header line from the bed file if any as this would generate an error. For a sample bed file please refer to */defaults/Default_Regions.bed*.
* **-m MUTATIONSFILE, --mutation MUTATIONSFILE** - [FILEPATH, OPTIONAL, Default:From Config file] - A csv file specifying details of the mutations to be incorporated in the generated reads. For details of this mutation csv file please refer to the Mutations file section below
* **-r MIXTURERATIO, --ratio MIXTURERATIO** - [STRING, OPTIONAL, Default:1:3] - The ratio in which the mixture is to be simulated. e.g 1:3:2 says that there are 3 contributors and in the proportions of 1x, 3x and 2x
* **-c CONFIGFILE, --config CONFIGFILE** - [FILEPATH, OPTIONAL, Default:'./config.ini'] - A config file specifying as the configurabe options. Th config file enable you to control the dwgsim command line and base coverage for mixture
* **-a MAPPINGSCRIPT, --mappingscript MAPPINGSCRIPT** [FILEPATH, OPTIONAL, Default:None] - A shell script specifying the mapping commands. You can call on any mapping software you want in the shell script. You script should expect 3 inputs from the wrapper - Reference file, Generated fastq file and the jobname. You can find examples of mapping scripts at /scripts/bwa_align.sh and /scripts/tmap_align.sh
* **-s SEED, --seed SEED** - [INT, OPTIONAL, Default:None] - The seed for random data generation. You can include any integer to ensure the generated data is replicated
* **-j JOBNAME, --jobname JOBNAME** - [STRING, OPTIONAL, Default:'sim_reads'] - A name for the current simulated dataset

## Extended configuration with config.ini
The wrapper also uses a config.ini file to enable the user to change all the configurable options. However if you specify values on the commandline they will be considered by the wrapper over the config.ini values.

Lets look at the values configurable in the config.ini
- **[defaultFiles]**
  - **scriptDir** = The directory where the readSimulatorWrapper.py is located
  - **defaultReference** = The reference .fasta/fa file
  - **defaultBed** = The bed file
  - **defaultMutations** = The mutations csv file
  - **vcfTemplate** = This is the template used for vcf header. Unless required dont change this path

- **[simulator]**
This section is for dwgsim options. You can include any other options for dwgsim that you want or change the values of current options.
The options are used the generate the dwgsim option commandline given by the parameter sim_options

If you want to add a parameter lets say rate of mutations you could add it as
```bash
rateOfMutations = 0.1
sim_options = -e %(base_error_rate)s -1 %(first_read_length)s ... -r %(rateOfMutations)s
```
Thus the general syntax is %(paramName)s

Some options like the coverage, reference file, bed file, vcf file are added in by the wrapper later.

- **[global_options]**
  - **base_coverage** = The base coverage for mixtures. If a contributor is 1x it will have this coverage. If 2x the 2*(base_coverage) value
  - **simulator_path** = Path for the dwgsim executable. Change it if dwgsim is not at /usr/bin/dwgsim
  - **default_mixture_ratio** = The default mixture ratio


## Mutations csv file
This is a csv file specifying the mutations to be incorporated in the generated reads.
The csv file has the following 5 columns
1. Chromosome - The chromosome where the mutation should be placed
2. Position - The position of the specified chromosome
3. Reference Allele - The allele in the reference file at specified position
4. Alternate Allele - The alternate alleles to be included in place of the reference. For multiple contributors seperate the alternate allele by '/'. For keeping the allele same as reference keep it as '-'. For e.g. T/-/C indicates you want contributor 1 to have T, contributor 2 to have allele same as reference and contributor3 to have allele.
5. Allele frequency - The frequency of the alternate alleles. The possible values are [1,0.5,-]. 1 will indicate a Homozygous mutation, 0.5 will indicate heterozygous mutation and - will be ignored as alternate allele will be same as reference.

You can specify values for multiple contributors in the csv file and use only part of them. i.e. you can specify alternate alleles as T/A/C but have the mixture ratio as 1:2 in which case values for one first two contributors will be considered.

For a sample mutations csv file refer /defaults/Default_Mutations.csv

## Sample usage scenarios
1. Run with all the default options
```bash
python readSimulationWrapper
```
2. Run with custom bed and mutations file
```bash
python readSimulationWrapper -b newbed.bed -m newmutationsfile.csv
```
3. Run with a different config file
```bash
python readSimulationWrapper -c newconfig.ini
```
4. Run with mapping script option
```bash
python readSimulationWrapper -a /path/to/tmap.sh
```
5. Run with a seed for reproducibility
```bash
python readSimulationWrapper -s 1000
```
6. Run with a specified jobname
```bash
python readSimulationWrapper -j trialjob
```

