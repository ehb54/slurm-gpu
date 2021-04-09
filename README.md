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
 - regular job, one core
```
srun -n1 hostname
```
 - gpu job, one gpu one core
```
srun --gres=gpu:1 -n1 nvidia-smi
```

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
