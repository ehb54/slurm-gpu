# slurm-gpu

Currently as setup on demeler9

## adding users
```
sacctmgr create user USERNAME account=compute-account adminlevel=None
```

## list users
```
sacctmgr list user
```

## verify
 - regular job, one cpu core
```
srun -n1 hostname
```
 - gpu job, one gpu one cpu core
```
srun --gres=gpu:1 -n1 nvidia-smi
```
## submit with slurm
 - gpu job, 1 gpu, 10 cores
   - create gpu-test.slurm containing
```
#!/bin/bash                                                                                                                                                                                                           
#SBATCH --job-name=gpu-test                                                                                                                                                                                            
#SBATCH --output=stdout.%j                                                                                                                                                                                            
#SBATCH --error=stderr.%j                                                                                                                                                                                             
#SBATCH --ntasks=1                                                                                                                                                                                                    
#SBATCH --cpus-per-task=10                                                                                                                                                                                            
#SBATCH --time=01:00:00                                                                                                                                                                                               
                                                                                                                                                                                                                      
echo Start = `date`                                                                                                                                                                                                   
echo JOB ID = $SLURM_JOB_ID                                                                                                                                                                                           
nvidia-smi                                                                                                                                                                                                            
echo Finish = `date`
```
  - submit
```
sbatch --gres=gpu:1 gpu-test.slurm
```

## interactive job
 - interactive session (i.e. a regular linux command line), 1 gpu, 4 cpu cores for 0 hours and 10 minutes:
```
$ env PS1="slurm $ " srun --pty --gres=gpu:1 -n 4 -t 00:10:00 bash
```
 You will get a ```slurm $``` prompt which will let you know you are running under slurm.
 - be sure to exit when you are done to free the resources.
## how it was setup

 - assumption: slurm is already installed and running for localhost submission
 
### setup mysql
```
# mysql -u root -p
MariaDB [(none)]> create database slurm_acct_db;
MariaDB [(none)]> show databases;
MariaDB [(none)]> create user 'slurm'@'localhost';
MariaDB [(none)]> set password for 'slurm'@'localhost' = password('PASSWORD');
MariaDB [(none)]> select User from mysql.user; # to show users
MariaDB [(none)]> grant usage on *.* to 'slurm'@'localhost';
MariaDB [(none)]> grant all privileges on slurm_acct_db.* to 'slurm'@'localhost';
MariaDB [(none)]> show grants for 'slurm'@localhost;
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> exit
```
### setup slurm
 - copied the *conf files from this repo to /etc/slurm/

### restart
```
sudo systemctl restart slurmd.service
sudo systemctl restart slurmdbd.service
sudo systemctl restart slurmctld.service
```

### sacctmgr controls
```
sacctmgr add cluster compute-cluster
sacctmgr add account compute-account description="Compute accounts"
sacctmgr create user usadmin account=compute-account adminlevel=Admin
sacctmgr create user us3 account=compute-account adminlevel=None
```

## possible improvements
 - we have 2 sockets in demeler9 - setting CPU Affinity as described [here](https://github.com/dholt/slurm-gpu) might provide a performance increase when running multi-GPU jobs
   - e.g. bind sockets to GPUs so that a GPU job is not using 2 different physical CPUs
 - will this interfere with regular non-GPU jobs?
   - i.e. if an assigned socket is loaded with non-GPU jobs and a new GPU job is requested assigned to the same socket, it seems likely the GPU job would have to wait in queue  

# Software List
| Module Name | Prerequisites | Description | 
|  ---  | --- | --- |
| gromacs/2020.5 | - | thread-MPI support; command : gmx|
| gromacs/2020.5-double | mpi/mpich-x86_64 | double precision; not supports GPU; command : gmx_mpi_d | 
| gromacs/2020.5-mpi | mpi/mpich-x86_64 | MPI support; command : gmx_mpi |
| gromacs/2020.5-plumed | mpi/mpich-x86_64; plumed/2.7.0 | Coupled with PLUMED for enhanced-sampling; command : gmx_mpi |
| lammps/double | mpi/mpich-x86_64 | version 2020; double precision; command : lmp|
| lammps/single | mpi/mpich-x86_64 | version 2020; single precision; command : lmp|
| namd/2.14 | mpi/mpich-x86_64 | command : namd2|
| namd/3.0 | mpi/mpich-x86_64 | command : namd3|
| openmm/7.5.0 | python/anaconda3; plumed/2.7.0 (optional) | command : python my_code.py |
| amber/20 | mpi/mpich-x86_64 | AmberTools 20; not supports GPU|
