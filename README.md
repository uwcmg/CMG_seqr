# CMG_seqr
A place for CMG seqr users and tutorials, scripts to load data, issues

# QuickStart
## Our CMG server location is 
rainier.gs.washington.edu:8000

## To access seqr remotely from a shell 

open a terminal and link into the server

ssh -L 127.0.0.1:8000:rainier.gs.washington.edu:8000 ${user}@nexus.gs.washington.edu

## open your browser to 127.0.0.1:8000 and you should see the index page
* the server is working on chrome/chromium [not firefox - daniel check this again] if this does not work try switching browser

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
Click edit Families and Individuals and Bulk edit families and individuals according to the instructions 
You will need to make a family file and use the pedigree file.

# Datasets:
To load BAM:

# Upload BAM and multi-vcf data 

more to follow ...
