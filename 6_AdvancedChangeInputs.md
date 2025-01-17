Changing inputs in MAgPIE model
================
Miško Stevanović (<stevanovic@pik-potsdam.de>)
09 September, 2019

1 Introduction
==============

The input data for MAgPIE is prepared by a set of pre-processing routines that take the data from original sources (e.g. FAO, LPJmL...), execute additional calculations and convert it to the required MAgPIE parameter format. These pre-processing routines are not accessible as open source at the moment. A user is instead provided with a ready-made inputs that are necessary for the model execution.

The input files are setup in the config file `config/default.cfg`, usually at the beginning of the settings. Currently, the input data is set as:

``` r
cfg$input <- c("magpie4.1_default_apr19.tgz")
```

Once specified in the configuration as the input data, the data is automatically downloaded (if needed) when the model run is started.

The prepared input data is a compressed tar archive file "`.tgz`", which can be opened with software such as [7-Zip](https://www.7-zip.org/), or in terminal by `tar` and `untar` commands. The data archive file contains the following types of data:

-   climate depended bio-physical data from a crop model (region and cell specific, crop yields, water requirements, terrestrial carbon content...)
-   processed data from other data sources used for particularization of MAgPIE (FAO, SSP, GTAP...)
-   processed data from other data sources used for validation of MAgPIE output
-   additional data prepared for model particularization (national policies...)
-   calibration factors

### 1.1 Learning objectives

There is a specific procedure on how to handle the changing of the input data. It will be demonstrated by the example of changing the USA NDC policy on afforestation target at 2030 with the following learning objectives:

1.  Create local input data repository.
2.  Package the patch file.
3.  Update the configuration to automatically apply designed changes.

2 Demonstration of change in input data for national land-based NDC policies
----------------------------------------------------------------------------

Once the input data is downloaded to the local MAgPIE repository in forms of different input files in designated input folders (in the core, scripts, modules and module realizations), a user can update or change these input data files. This can be done directly by manipulating the files, but this approach carries a risk that such changes are not documented and that in certain cases the made changes can be overwritten by the repeated download of the date from the declared repositories. In order to avoid this risk, it is recommended to create a local folder that serves as a repository for the patch files that will apply changes to the data by overwriting the original data.

### 2.1 Create folder for local input data repository

The folder for local input data repository can be created anywhere and it's path must be provided to the settings in `config/default.cfg` file.

Let us assume that the patch folder (`patch_inputdata`) is created in the main MAgPIE repository. One can do it in R:

``` r
dir.create("./patch_inputdata")
```

or in the command line:

``` cmd
mkdir patch_inputdata
```

Once the directory is created, provide its location to the configuration file. It's important to keep the list structure of the repository information:

``` r
cfg$repositories <- append(list("https://rse.pik-potsdam.de/data/magpie/public"=NULL,
                                "./patch_inputdata"=NULL),
                           getOption("magpie_repos"))
```

### 2.2 Create a patched file policy\_definition.csv and package it with lucode::tardir()

Create a sub-directory in the `./patch_inputdata` which is going to be used for packaging of the patched files.

``` r
dir.create("./patch_inputdata/patch_ndc_usa_190909")
```

Copy the original file the `policy_definition.csv` in the patch folder. In R:

``` r
file.copy(from="./scripts/npi_ndc/policies/policy_definitions.csv",
          to="./patch_inputdata/patch_ndc_usa_190909/.")
```

or in the command line:

``` cmd
cp scripts/npi_ndc/policies/policy_definitions.csv patch_inputdata/patch_ndc_usa_190909/.
```

Edit the content, in this case update the USA afforestation NDC policy (`affore`) with a more ambitious target of 15 MHa of afforested area starting in 2020 and reaching the target at 2030:

``` txt
USA,affore,ndc,1,2020,2030,15
```

After saving the file, package it with the tardir function in R environment and delete the file patch folder:

``` r
lucode::tardir(dir="patch_inputdata/patch_ndc_usa_190909",
               tarfile="patch_inputdata/patch_ndc_usa_190909.tgz")

unlink("patch_inputdata/patch_ndc_usa_190909", recursive=TRUE)
```

### 2.3 Add the .tgz packed patch file in the configuration file

Finally, the configuration file should be informed about the change in the input data and the existing patch file that replaces the existing input data. For this, edit the `config/default.cfg` file from:

``` r
cfg$input <- c("magpie4.1_default_apr19.tgz")
```

to:

``` r
cfg$input <- c("magpie4.1_default_apr19.tgz",
               "patch_ndc_usa_190909.tgz")
```

It is very important to add the patch file at the end of the listings in the `cfg$input` listings, because every next `.tgz` archive will overwrite the files previously imported by the files that are contained in it.

At the next start of the model by `Rscript`, the new patch will place the file with change inputs according to the changes in the settings.

3 Additional information
========================

There are other pre-processed input data that have different regional resolution available for download from the PIK public repositories. They include special regional definitions focused on singly countries as regions: China, India, Ethiopia and USA:

``` text
"magpie4.1_ind_apr19.tgz"
"magpie4.1_cha_apr19.tgz"
"magpie4.1_aus13_jul19.tgz"
```

These prepared inputs can be used instead of the default regional definition inputs in `magpie4.1_default_apr19.tgz`

4. Excercise:
=============

Write your own starting script that will test the scenario with changed NDC policy for the USA described above. None of the changes should actually occur in the default.cfg, but the starting scrip should introduce them to the loaded cfg object instead.
