# How to connect to slurm compute nodes (in 2 minutes)

## Why?
In my research internship, I found out that dealing with slurm login nodes is sometimes a bit painful, especially when it comes to running a debugger etc. 

Thus, after some research and digging, here is a solution that I found which works reliably.
After following the steps below, you'll be able to connect your vscode directly to a compute node which will help you debug and play around in the node. It basically gives you a closed environment connected to your vscode.

## Setup

1. Create SSH public key <br>
    You can follow the tutorial [here](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server) for this step.

2. Setup `sshd.job` script <br>
    Copy the `sshd.job` script into your root directory `~`. <br>
    Make sure to change the path to your key in the last part of the script `${HOME}/.ssh/<your_key_name>`.

3. Configure known hosts <br>
    Start by opening `~/.ssh/config` on your local machine. <br>
    You may have a defined host for your login nodes. If you do, skip to the next part; if not, paste the following into your config file. <br>
    ```
    Host <slurm_name>
        HostName <slurm_hostname>
        User <username>
        IdentityFile ~/.ssh/<your_key_name>
        IdentitiesOnly yes
    ```
    Then, we will paste the following configuration for going into the compute node. For `<the compute_node_name>` variable, see the next section.
    ```
    Host <compute_node_name> !<slurm_hostname>  # notice the "!" sign. Do not remove it, it's not a typo.
        User <username>
        ProxyJump <slurm_hostname>
        PreferredAuthentications publickey
    ```
    
## Retrieving Compute Nodes

1. Go into one of your cluster's login nodes. Simply type `ssh <slurm_hostname>` into your local terminal. Make sure that you are in a directory where `sshd.job` is visible. If you followed the steps above, it should be in your home directory.

2. Once inside, use a command that is appropriate for your needs to retrieve a compute node. For example:
    ```
    sbatch --gpus=rtx_4090:4 --mem-per-cpu=16G --time=48:00:00 --ntasks=8 ./sshd.job
    -- or
    sbatch --gpus=1 --gres=gpumem:80g --mem-per-cpu=32G --time=120:00:00 --ntasks=8 ./sshd.job
    ```

3. This will submit a job with `sshd.job` script as the target. You can now simply retrieve the compute node's name by typing `squeue` in your terminal inside the login node. It should give an output as follows. Simply grab the compute node name from this list and use in your configuration file.
    ```
    JOBID PARTITION     NAME     USER    ST       TIME         NODES NODELIST(REASON)
    <id>  gpuhe.120     sshd     <user>  R        0:00:01      1     <compute_node_name>
    ```

## Working with compute nodes
One way is to directly use the shell by typing `ssh <compute_node_name>`. Though it's pointless at this point. <br>
You can also use VSCode's remote development tool to connect to your compute node. The name will appear on the known hosts list. Simply click on it and you'll be good to go.

## Warning
I'm aware this is not the way slurm is intended to be used. Though it's easier to debug or do some small tests, especially with vscode. Hence, do not forget to cancel your job with `scancel <job_id>` after you are done and use proper `sbatch` commands for actual trainings so that everyone can benefit from the compute nodes and you'll not be holding the nodes for no reason.