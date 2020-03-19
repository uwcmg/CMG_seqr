# CMG_seqr
A place for CMG seqr users and tutorials, scripts to load data, issues

# QuickStart
## Our CMG server location is 
rainier.gs.washington.edu:8000

## The Macarthur lab Github site is here:

https://github.com/macarthur-lab/seqr/

## To access seqr remotely (at home) from a shell without VOL6 access (must copy files locally or to cloud to use Data)

This is very slow and limits to nmber of users to the number of display ids - not ideal

open a terminal and link into the server

ssh -L 127.0.0.1:8000:rainier.gs.washington.edu:8000 ${user}@nexus.gs.washington.edu

open your browser to 127.0.0.1:8000 and you should see the index page 
* the server is working on chrome/chromium firefox has had issues depending on version - if this does not work try switching your browser.

you will not have vol6 access

## On windows using UW's Husky OnNet service (re-branded F5 Big-IP Edge VPN client).

https://itconnect.uw.edu/connect/uw-networks/about-husky-onnet/

Same access restrictions as above ... working with GSIT to see if we can get vol6 access...

post an issue or email if anyone can get this working for vol6 access!

## To access seqr remotely from a shell AND VOL6 access (can use data in situ)

This will allow CMG analysts to access to vol6 

1) ssh -Y nexus ...
2) ssh -Y krakatoa ...
3) vncviewer -via ${user}@krakatoa localhost:${DisplayId}

${user} is your grc username
${DisplayId} is your assigned display (Daniel's is :1 - use a unique display id or contact gsit)
see https://docs.google.com/presentation/d/1uP8xknepxDb-MLFQegD3Xujv2vW2B2RCrv9tEnxr1U4/edit?usp=sharing

# create a project for your CMG project
use the ${projectName} in the directory of the project of the form .../mendelian_projects/${projectName}
to name your new project in seqr

find and click "Create Project +"

enter the Project Name = ${pi}_${uwcmg}_${phenotype}_${n}
enter the Description
select the genome version

click the project and note the "project index" in the navigation bar e.g.
from 
http://127.0.0.1:8000/project/R0004_urban_uwcmg_cl_1/project_page

this is:
"R0004_urban_uwcmg_cl_1"

# Pedigree Data:
Click "Edit Families and Individuals" and use tabs

"Bulk Edit families"
give your file 

essentially "cut -f1 ${your pedigree}" to seqr

and individuals according to the instructions 
You will need to make a family file and use the pedigree file.

# DATASETS: Upload BAM and multi-vcf data 

see step 5 of the seqr github instructions here: 

https://github.com/macarthur-lab/seqr/blob/master/deploy/LOCAL_INSTALL.md

you will need access to the seqr server (rainier) from a separate shell to run the script located under ${SEQR_DIR}/hail_elasticsearch_pipelines/ for now. A user Script is in progress to remedy this ... more to come.

The call is
python2.7 gcloud_dataproc/submit.py --run-locally hail_scripts/v01/load_dataset_to_es.py  --spark-home $SPARK_HOME --genome-version $GENOME_VERSION --project-guid $PROJECT_GUID --sample-type $SAMPLE_TYPE --dataset-type $DATASET_TYPE --skip-validation  --exclude-hgmd --vep-block-size 100 --es-block-size 10 --num-shards 1 --hail-version 0.1 --use-nested-objects-for-vep --use-nested-objects-for-genotypes $INPUT_VCF

1) you may encounter an error due to a conflict in the $SPARK_CLASSPATH if so copy the value of the SPARK_CLASSPATH to SPARK_CLASSPATH_ORIG and execute

"unset SPARK_CLASSPATH" and try again

2) add two more parameters --driver-memory [] and --executor-memory [] if you have a large multivcf!  Values that match your project e.g. from 5G for a trio to 30G+ for a larger cohort (again if the size of the multivcf is very large). Currently I am using trial and error but a ratio memG/vcfG is coming...the server has 40cpu's and large memory capacity

seqr will then convert the dataset to Hail and load it into elasticsearch...

If all goes well, go back to the web interface in your browser and use your uploaded data within seqr.
