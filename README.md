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
