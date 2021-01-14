# CMG_seqr
A place for CMG seqr testing for potential users and tutorials, scripts to load data, issues...

# QuickStart
## Our INTRANET CMG server location is 
rainier.gs.washington.edu :8000

## The Macarthur lab Github site is here:

https://github.com/macarthur-lab/seqr/

## ACCESS REMOTELY - f5 Big IP edge client : 
To access in windows running the f5 client simply navigate to 

http://rainier.gs.washington.edu/

this is the easiest way to access seqr

## ACCESS REMOTELY - SSH -L
###To access seqr remotely (at home) from a shell without VOL6 access (must copy files locally or to cloud to use Data)

This is very slow and limits to nmber of users to the number of display ids - not ideal

open a terminal and link into the server

ssh -L 127.0.0.1:8000:rainier.gs.washington.edu:8000 ${user}@nexus.gs.washington.edu

open your browser to 127.0.0.1:8000 and you should see the index page 
* the server is working on chrome/chromium firefox has had issues depending on version - if this does not work try switching your browser.

you will not have vol6 access

## ACCESS REMOTELY: UW's Husky OnNet service on Windows (re-branded F5 Big-IP Edge VPN client).

https://itconnect.uw.edu/connect/uw-networks/about-husky-onnet/

Same access restrictions as above ... 

## ACCESS REMOTELY - TIGERVNC: 
### To access seqr remotely from a shell AND HAVE VOL6 access (can use data in situ; slower web interface)

Again - This will allow CMG analysts to access to vol6 

1) ssh -Y nexus ...
2) ssh -Y krakatoa ...
3) vncviewer -via ${user}@krakatoa localhost:${DisplayId}

${user} is your grc username
${DisplayId} is your assigned display (Daniel's is :1 - use a unique display id or contact gsit)
see https://docs.google.com/presentation/d/1uP8xknepxDb-MLFQegD3Xujv2vW2B2RCrv9tEnxr1U4/edit?usp=sharing

# 1) Create a project for your CMG project
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

# 2) ADD Pedigree Data using a local file and the seqr web interface:
Click "Edit Families and Individuals" and use tabs

"Bulk Edit families"

example families tsv:
FamilyID	Display	Name	Description	Coded Phenotype
K49179	K49179	Congenital Diaphramatic Hernia	CDH

drag in your families file as  instructed

Also make an individuals file according to the instructions 

example individuals tsv:
Family ID	Individual ID	Paternal ID	Maternal ID	Sex	Affected Status	Notes	Proband Relation
K49179	415425		415426	Male	Affected	K49179-36874	Self
K49179	415426			Female	Unaffected	K49179-36875	Mother 
drag in your individuals file as  instructed


# 3) ADD DATASETS: Upload on-prem multi-vcf data 

Seqr converts a vcf to a hail dataset, annotates it and then loads it into elasticsearch.
This is all done from command line on the rainier seqr server.

The procedure to load seqr with project data is to 

ssh to rainier.gs.washington.edu with your login and work from command line.

navigate to /data/docker-shares

you should see something like this:
drwxrwxr-x 3 docker nick-mendelian        4096 Sep 30 09:40 config
drwxrwxr-x 3 docker nick-mendelian        4096 Sep 11 12:11 data
-r--r--r-- 1 docker nick-mendelian        2413 Dec 15 12:34 docker-compose.yml
drwxrwxr-x 3 101000 nick-mendelian        4096 Dec 16 13:01 elasticsearch
drwxrwxr-x 5 docker nick-mendelian        4096 Jan 12 11:48 input_vcfs
-rw-r--r-- 1 docker nick-mendelian 10441015296 Dec  4 12:06 original_data.tgz
drwxrwxr-x 2 docker nick-mendelian        4096 Sep 29 10:46 output_mt
drwxrwxr-x 3 docker nick-mendelian        4096 Sep 11 11:43 postgres
drwxrwsr-x 3 docker nick-mendelian        4096 Dec  9 13:24 seqr-reference-data
drwxrwsr-x 4 docker nick-mendelian        4096 Dec  9 10:24 vep_data

confirm there is a docker-compose.yml in this location. This is the file that will direct all containers to use this 
directory hierarchy, permisssons and mounted volumes for us.

## create a project data input directory on the seqr server under 

/data/docker-shares/input_vcf/${project}

Example:
[<user>@rainier bamshad_uwcmg_cdh_5]$ pwd
/data/docker-shares/input_vcfs/bamshad_uwcmg_cdh_5

cp (or link?) the project's vcf.gz and .tbi index from vol6 to the project directory and set permisssions (775) and owner (docker:nick-mendelian)
[<user>@rainier bamshad_uwcmg_cdh_5]$ ls -l

-rwxrwxr-x 1 docker nick-mendelian 291635632 Jan  7 12:14 bamshad_uwcmg_cdh_5.HF.final.vcf.gz.VT.vcf.gz
-rwxrwxr-x 1 docker nick-mendelian   1671159 Jan  7 12:14 bamshad_uwcmg_cdh_5.HF.final.vcf.gz.VT.vcf.gz.tbi

ne sure to set permisssions and user:group as above

## On-prem Step 1 - Convert vcf to hail

All steps from here on will use a docker container instance of "pipeline runner"
refer to:
https://github.com/broadinstitute/seqr/blob/master/deploy/LOCAL_INSTALL.md

###Confirm you are in the /data/docker-shares directory and ensure the docker-compose.yml is in this location!

### launch the pipeline runner container on rainier:
docker-compose up -d pipeline-runner            # start the pipeline-runner container using the docker-compose.yml build instructions
docker-compose exec pipeline-runner /bin/bash   # open a shell inside the pipeline-runner container (analogous to ssh'ing into a remote machine)

you will see a new instance of the container running in a shell

example terminal:

[mcgold@rainier docker-shares]$ docker-compose up -d pipeline-runner

dockershares_elasticsearch_1 is up-to-date
dockershares_pipeline-runner_1 is up-to-date

[mcgold@rainier docker-shares]$ docker-compose exec pipeline-runner /bin/bash

This shell is in the PIPELINE-RUNNER container.

d240401caef1:/]$

### Authenticate your instance with google cloud (not tested without this step):
d240401caef1:/]$ gcloud auth application-default login
Go to the link in your browser and copy paste the code to the command line:

    https://accounts.google.com/o/oauth2/auth?response_type=code&client_id=764086051850 ...

Enter verification code:

...

### response:

Credentials saved to file: [/root/.config/gcloud/application_default_credentials.json]

These credentials will be used by any library that requests Application Default Credentials (ADC).
WARNING:
Cannot find a quota project to add to ADC. You might receive a "quota exceeded" or "API not enabled" error. Run $ gcloud auth application-default set-quota-project to add a quota project.


### Run a script or cammand like /input_vcf/${project}/ESloadpart1.sh to convert your ${project} dataset to hail format from vcf:

Edit or add ${project} as a environment variable as in the following template run-script and save and run the script inside the project folder:

### EXAMPLE Script command follows - IF YOU ARE RUNNING a WES project ADD "--dont-validate" because our target is different than the Broads QC target and will not work without this!!!:
#### This is what works for me from inside the /data/docker-shares directory:

python3 -m seqr_loading SeqrVCFToMTTask --local-scheduler --dont-validate --source-paths input_vcfs/${project}/${project}.final.vcf.gz.VT.vcf.gz --genome-version 37 --sample-type WES  --dest-path /input_vcfs/${project}/${project}.mt --reference-ht-path seqr-reference-data/GRCh37/combined_reference_data_grch37.ht --clinvar-ht-path seqr-reference-data/GRCh37/clinvar.GRCh37.2020-06-15.ht --vep-config-json-path vep-GRCh37.json

### Note: Update the clinvar .ht update from Broad as needed:

This step will launch the Conversion of the VCF!

## and see Broad ref :
 SeqrVCFToMTTask --local-scheduler   
   # for GRCh38 callsets, run a command like the one below inside the pipeline-runner container to annotate and load your dataset into elasticsearch
   python3 -m seqr_loading SeqrVCFToMTTask --local-scheduler \
       --reference-ht-path /seqr_reference_data/combined_reference_data_grch38.ht \
       --clinvar-ht-path /seqr-reference-data/GRCh38/clinvar/clinvar.GRCh38.2020-06-15.ht \
       --vep-config-json-path /vep85-GRCh38-loftee-gcloud.json \
       --sample-type WES \
       --genome-version 38 \
       --source-paths gs://your-bucket/GRCh38/your-callset.vcf.gz \   # this can be a local path within the pipeline-runner container (eg. /input_vcfs/dir/your-callset.vcf.gz) or a gs:// path 
       --dest-path gs://your-bucket/GRCh38/your-callset.mt      # this can be a local path within the pipeline-runner container or a gs:// path where you have write access

## On-prem Step 2 - Now that the vcf is converted, upload the converted Hail file to elasticsearch for the seqr server to use.

Example command/script line:

python3 -m seqr_loading SeqrMTToESTask  --local-scheduler --genome-version 37 --dest-path /input_vcfs/${project}/${project}.mt --reference-ht-path /seqr-reference-data/GRCh37/combined_reference_data_grch37.ht --es-host elasticsearch --es-index ${project}_esi --es-index-min-num-shards 10

# Go back to the rainier seqr server page and your project should now be ready for variant search! 

Example:
http://rainier.gs.washington.edu/project/R0009_bamshad_uwcmg_cdh_5/project_page

Happy Seq ing!

...

Possible TODO:
make this process work in large batches of projects in a more automated fashion. 

