#How to setup NYU HPC cluster software

##PROGRAMS YOU WILL NEED TO DOWNLOAD ONTO THE CLUSTER 

	#Anaconda 
	curl -O https://3230d63b5fc54e62148e-c95ac804525aac4b6dba79b00b39d1d3.ssl.cf1.rackcdn.com/Anaconda-2.2.0-Linux-x86_64.sh

	#Smapp Tool Kit
	~/anaconda/bin/pip install smapp-toolkit

	#Mongo DB (use linux 64 bit legacy distribution to avoid problems)
	curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.0.4.tgz
	tar xvf mongodb-linux-x86_64-3.0.4.tgz 
 

##PROGRAMS YOU WILL NEED TO DOWNLOD ON YOUR MACHINE
	#GitHub Client(enables you to edit/push code to github) 

##LOG INTO CLUSTER WITHOUT TUNNELING 
	#Open new terminal window
	`ssh YOUR_NET_ID_HERE@hpc.nyu.edu` 
    #enter NYU net ID password when prompted
    `ssh mercer`
    #enter NYU net ID password when prompted 

##ACCESS SMAPP COLLECTIONS/DATA THROUGH THE HPC
	#ask Shenglong Wang <hpc@nyu.edu> to add your net ID to the smapp group
	`ls /scratch/smapp` 
	#lets you see what smapp data is already on the cluster eg: ls /scratch/smapp/arab_events_all_6_19 

##COPYING FILES FROM THE CLUSTER TO YOUR COMPUTER 
	#Set up ssh tunneling (to move files from the cluster to your computer)
	#follow instructions at: https://wikis.nyu.edu/display/NYUHPC/SSH+tunneling+overview 
	#nano can be used to edit text in ssh/config file if you don't have a prefered editor  
	`nano .ssh/config` 
	#Open terminal window
	`ssh hpctunnel`
	#enter NYU net ID password when prompted
	#Open second terminal window and type scp mercer:[FILE PATH ON CLUSTER] [FILE PATH ON MACHINE]
	`scp mercer:~/history-jul1.txt /Users/YOUR_NET_ID_HERE/Dropbox/`
	#enter NYU net ID password when prompted and file will download to the location specified 

##START INTERACTIVE SESSION (can't do long-running jobs on login node so anytime you're dumping data it should be done on the interactive node) 
	`qsub -I interactive.pbs`
	#(output should then say compute when you type "hostname")

##DUMPING TWEETS ONTO CLUSTER
	#go into sandbox
    `cd Sandbox`
    #now update the Sandbox... if you've made new changes before gitpull type git reset --hard
    `git pull` 
    #ignore error message and write in github username and password when prompted
    #edit file to change DB name, password, job time etc. 
	`nano Sandbox/cluster/simple_mongodump.pbs` 

	#Control X to quit then type Y to save 
	#run job on cluster 
    `qsub -V Sandbox/cluster/simple_mongodump.pbs` 


##CHECK TO SEE IF JOB IS RUNNING WITHOUT ERRORS
	#insert your netID where I have YOUR_NET_ID_HERE
    `qstat -u YOUR_NET_ID_HERE`
	#Q means job is qued, C means job is complete and R means job is running
	#Once job is running: 
	#change directory to home
	`cd` 
	#create directory to store jobs
	`mkdir jobs` 
	`cd jobs`
	#see job names
	`ls -la`
	#shows how job is running... shift f will continue to show you changes over time, q will quit out of "less"
	`less JOBNAME` 
	`cd /scratch`
	`cd smapp`
	#shows new dumped data
	`ls -la` 


##CODE TO RUN MULTIPLE JOBS AT ONCE (for large tweet collections divided into multiple parts)
	`cd Sandbox`
	`cd cluster`
	#lets you see the file for running multiple jobs at once 
	#go to home directory
	`cd` 
	`nano simple_launch_n_jobs.py`
	# clones github file to run on the cluster (type in URL of github repository)
	`git clone https://github.com/SMAPPNYU/Projects.git`
	#Code to launch multiple jobs at once: 
	`~/anaconda/bin/python ~/Sandbox/cluster/simple_launch_n_jobs.py -i /scratch/smapp/arab_events_all_6_19/*.bson -c '~/anaconda/bin/python ~/Projects/data-reports/2015_6_19-ArabEvents-ISIS/get_data_for_split.py -o /scratch/smapp/arabevents-isis-data-report'`

 	#To just launch job for one collection: 
    `~/anaconda/bin/python ~/Sandbox/cluster/simple_launch_n_jobs.py -i /scratch/smapp/arab_events_all_6_19/ArabEvents___tweets_18.bson -c '~/anaconda/bin/python ~/Projects/data-reports/2015_6_19-ArabEvents-ISIS/get_data_for_split.py -o /scratch/smapp/arabevents-isis-data-report'`

    # -i gives input file, -c is the command to be executed and -o shows where to put the output file 

##SIMPLIFIED VERSION OF DUMPING WHEN EVERYTHING IS SET UP
	`ssh YOUR_NET_ID_HERE@hpc.nyu.edu` 
	    #enter NYU net ID password when prompted
	`ssh mercer`
	    #enter NYU net ID password when prompted 

	`nano ~/Sandbox/cluster/simple_mongodump.pbs`
	#edit the tweet collection you want (so change to 19, 20, etc)
	#runs job on cluster
	`qsub -V ~/Sandbox/cluster/simple_mongodump.pbs` 
	#check that a job is running
	`qstat -u YOUR_NET_ID_HERE` 
	`cd jobs`
	#copy most recent job name
	`ls -la` 
	`less MOSTRECENTJOB`

##Then once all files are dumped....
	`~/anaconda/bin/python ~/Sandbox/cluster/simple_launch_n_jobs.py -i /scratch/smapp/arab_events_all_6_19/*.bson -c '~/anaconda/bin/python ~/Projects/data-reports/2015_6_19-ArabEvents-ISIS/get_data_for_split.py -o /scratch/smapp/arabevents-isis-data-report'`

##ONCE DATA HAS RUN ON CLUSTER...
	#downloads all data to your computer
	`rsync --progress mercer:/scratch/smapp/results/arabevents-isis-data-report/*.csv ~/data/`
	#merges data files 
	`python merge_results.py -i ~/data/ArabEvents*.csv` 
	Point the notebook file visualize.ipynb at the correct folder where the files above are found, and run it to produce the graphs.

##EDITING GITHUB CODE (on your computer, not the cluster)
###1- Open github program on your computer
###2- clone project repository
###3- click plus
###4- find document to edit
###5- after editing create summary of your changes, click "commit to master", click "sync" button 

##USEFUL COMMANDS & JARGON
	#this followed by a web address allows you to download programs
	`curl -O`
	#change from host we're on to a different host
	`ssh` 
	#get the name of host you're on
	`hostname` 
	#lists files with info 
	`ls -la` 
	`mkdir` #makes new directory
	`cd` #changes directory to home or a given directory if you add the directory name eg: cd jobs 
	#prints current working directory 
	`pwd`
	`grep` #grabs regular expression---used to search text for lines containing a match to the given strings or words
	#the command used for job submission to the cluster
	qsub 
	-i #input
	-c #command to be executed
	-o #output
	#stops job running on cluster 
	`qdel JOBNAME` 
	#tab key completes file paths etc. press twice to see multiple options 
	#pbs stands for portable batch system files which are scripts to set up and launch jobs on any cluster

###authors

written by by <a href="https://github.com/YOUR_NET_ID_HERE">Alex Siegel</a>
edited by <a href="https://github.com/yvan">Yvan Scher</a>
