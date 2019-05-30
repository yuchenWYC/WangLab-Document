# UHNClusterGuide 
**Guide to using UHN H4H Cluster**

#### If you're on an open network: 
- Email `helpdesk@uhnresearch.ca` asking for One Time Passwords
- Say you will pick them up in person, otherwise they will try to fax them
1. Enter username and OTP at `https://www.uhnresearch.ca/remote`.
2. - `ssh -p 5500 \<your username\>@192.75.165.28`.
   - For Windows users, host is `192.75.165.28`, port is `5500`.
                    
#### If you're on the UHN corporate network:  
- Use `ssh -p 10022 \<your username\>@172.27.23.163`.
- For Windows users, host is `172.27.23.163`, port is `10022`. 

## Usage
The cluster uses Slurm for cluster management and job scheduling, feel free to visit slurm.schedmd.com for complete documentation on its usage. But we will go over the following in the next section.

### Basic Commands
   - `sinfo` : check the status of the cluster/partitions.
      - `sinfo -IN` :  shows per-node status as well.
      
   - `squeue` : shows the status of jobs.
      - `squeue -u $USER` : shows status of jobs for `$USER`.
      - `squeue -l` : shows status of jobs in long format.
      - `squeue -t $STATE` : shows status of jobs in state `$STATE` where `$STATE` is `PENDING`, `RUNNING`, or `SUSPENDED`. 
    
   - `scancel` : command to kill jobs.
      - `scancel -i $JOBID` : kill job with `$JOBID`.
      - `scancel -u $USER` : kill all jobs for user `$USER`.
      - `scancel -t $STATE` : kill all jobs in state `$STATE`.
      
## Running Jobs
   To run jobs, you will first want to scp your project from your local machine to your remote directory of either `/cluster/home/$USER/projects` or `/cluster/home/$USER/scratch` folder.
   
   `scp` works the following: <p> 
   
        scp -r <local folder directory> -p $PORT user@$HOST:<remote directory>
       
   - `-r` is to recursively copy all files in the folder onto the remote directory. **Remember to use the tag when copying over folders.**
   - The `~/scratch` folder is used to store files short-term, they are usually erased after 3 months. 
   - Store important files that you do not want erased in the `~/projects` folder.
   
   **Note: If you're working on group projects, your group directory is under `/cluster/projects` instead. There is also a 50GB quota per user so it may not be the best place to store datasets.**
   
### Installing Python and libraries
   We must first install Python and all other required libraries for your project.
   ```
   # Log into the remote server.
   ~ $ ssh -p $PORT user@$HOST
   
   # Installing Python 3.7 virtual environment(or any other version, just replace 3.7 with desired) 
   # <ENV> is folder directory to store the virtual env. I recommend storing it in /cluster/home/$USER
   ~ $ virtualenv --no-download --python=python3.7 <ENV>
   
   # Load Python 3.7 or which ever version accordingly.
   ~ $ module load python/3.7
   
   # Activate the virtual environment.
   ~ $ source <ENV>/bin/activate
   
   # Check if pip is up to date.
   (virtualenv) ~ $ pip install --upgrade pip
   
   # Install any other packages. We will install torch and torchvision as an example.
   (virtualenv) ~ $ pip install torch torchvision
   ```
   
### Writing a Batch Script and Submitting
   Now that we have all our libraries installed, to submit a job, we should write a batch script as the following -
   Create a new file using `vim train.sh` (you can name the file anything else but it must be a `.sh` file). Then, write more or less the following depending on the resources you're requesting:
   ```
   #!/bin/bash
   # The following three commands allow us to take advantage of whole-node
   # scheduling
   #SBATCH --nodes=1
   #SBATCH --gre=gpus:1 #Using one GPU
   #SBATCH --cpus-per-task=6
   #SBATCH --mem=0
   # Wall time
   #SBATCH --time=12:00:00
   #SBATCH --job-name=example
   #SBATCH --output=$SCRATCH/output/example_jobid_%j.txt
   # Emails me when job starts, ends or fails
   #SBATCH --mail-user=example@gmail.com
   #SBATCH --mail-type=ALL

   # load any required modules
   module load python/3.7.0
   # activate the virtual environment
   source ~/<ENV>/bin/activate

   # run a training session
   srun python example.py
   ```
     
   To exit out of edit mode, hit your `ESC` button and type `:w` followed by `:q`. `:w` is to write to your file and `:q` is to leave the file view in vim.
 
Now, all that is left is to submit the job. To submit the job, type the follow, `sbatch <file_name>` or `sbatch train.sh` in our example.
 
 - You can track the jobs progress via `squeue -u $USER` and cancel jobs via `scancel -i $JOBID` as shown above.
 - Further, to get data about memory usage **after** a job is completed, you can type `sacct`. 
 - To get data about memory usage **during** a job, you can type:
    `srun --jobid $JOBID --pty tmux new-session -d 'htop -u $USER' \; split-window -h 'watch nvidia-smi' \; attach`
 
 <p> 
   And that is all!
