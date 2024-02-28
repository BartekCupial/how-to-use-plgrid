# Running on Compute Nodes

* Users submit work to the scheduler (typically as a batch job) having specified the resources (compute,
time, memory) needed for the work
* The scheduler chooses which resources to allocate to the job and runs the job
* The scheduler tracks resource usage and will kill jobs that exceed resource requirements

# Viewing Available Partitions

NODES(A/I/O/T)
* A = Allocated
* I = Idle
* O = Other state (failed, drained...)
* T = Total

```bash
[ares][plgbartekcupial@login01 ~]$ sinfo -s
PARTITION       AVAIL  TIMELIMIT   NODES(A/I/O/T) NODELIST
all                up 3-00:00:00      787/4/6/797 ac[0001-0788],ag[0001-0009]
cpu                up 3-00:00:00      779/3/6/788 ac[0001-0788]
cpu-lowprio        up 3-00:00:00      779/3/6/788 ac[0001-0788]
cpu-bigmem         up 3-00:00:00      255/1/0/256 ac[0129-0384]
cpu-lowmem         up 3-00:00:00      524/2/6/532 ac[0001-0128,0385-0788]
gpu                up 3-00:00:00          8/1/0/9 ag[0001-0009]
plgrid*            up 3-00:00:00      779/3/6/788 ac[0001-0788]
plgrid-bigmem      up 3-00:00:00      255/1/0/256 ac[0129-0384]
plgrid-gpu-v100    up 2-00:00:00          8/1/0/9 ag[0001-0009]
plgrid-long        up 7-00:00:00      524/2/6/532 ac[0001-0128,0385-0788]
plgrid-now         up   12:00:00      524/2/6/532 ac[0001-0128,0385-0788]
plgrid-testing     up    1:00:00      524/2/6/532 ac[0001-0128,0385-0788]
8k3                up 3-00:00:00      779/3/6/788 ac[0001-0788]

```

```bash
[ares][plgbartekcupial@login01 ~]$ sinfo -o "%10R %10c %10m %25f %20G %20F"
PARTITION  CPUS       MEMORY     AVAIL_FEATURES            GRES                 NODES(A/I/O/T)      
all        32         368000     memfs,localfs             gpu:8(S:0-1)         8/1/0/9             
all        48         184800+    memfs,localfs             (null)               779/3/6/788         
cpu        48         184800+    memfs,localfs             (null)               779/3/6/788         
cpu-lowpri 48         184800+    memfs,localfs             (null)               779/3/6/788         
cpu-bigmem 48         369600     memfs,localfs             (null)               255/1/0/256         
cpu-lowmem 48         184800     memfs,localfs             (null)               524/2/6/532         
gpu        32         368000     memfs,localfs             gpu:8(S:0-1)         8/1/0/9             
plgrid     48         184800+    memfs,localfs             (null)               779/3/6/788         
plgrid-big 48         369600     memfs,localfs             (null)               255/1/0/256         
plgrid-gpu 32         368000     memfs,localfs             gpu:8(S:0-1)         8/1/0/9             
plgrid-lon 48         184800     memfs,localfs             (null)               524/2/6/532         
plgrid-now 48         184800     memfs,localfs             (null)               524/2/6/532         
plgrid-tes 48         184800     memfs,localfs             (null)               524/2/6/532         
8k3        48         184800+    memfs,localfs             (null)               779/3/6/788 
```

# Showing Node Information

```bash
sinfo -p plgrid -N -l | head -6
Thu Nov 23 12:22:33 2023
NODELIST   NODES PARTITION       STATE CPUS    S:C:T MEMORY TMP_DISK WEIGHT AVAIL_FE REASON              
ac0001         1   plgrid*   allocated 48     4:12:1 184800        0      1 memfs,lo none                
ac0002         1   plgrid*   allocated 48     4:12:1 184800        0      1 memfs,lo none                
ac0003         1   plgrid*       mixed 48     4:12:1 184800        0      1 memfs,lo none                
ac0004         1   plgrid*   allocated 48     4:12:1 184800        0      1 memfs,lo none  

```

# Requesting Resources in Slurm
* Users interact with Slurm by executing commands on the User Access Nodes (UANs)
* Slurm provides multiple mechanisms for users to access compute node resources
<br>
<br>
* Interactive use
    * srun command
* Interactive use within a predefined allocation
    * salloc to allocate resources
    * srun command(s) to start an application
* Batch usage
    * sbatch submits a batch script
    * srun command(s) within the batch script
* Resource requests are made by
    * Structured comments in a batch job
    * Environment variables
    * Command-line arguments to sbatch, salloc or srun

# Example srun command 

```bash
srun -A plgtimeseriescpu-cpu -c 1 --nodes 1 --mem=4G -t 120 -p plgrid --pty bash
```

# Lifecycle of a Batch Script

job.slurm
```bash
#!/bin/bash
#SBATCH -p <your_partition>
#SBATCH -A <your_project>
#SBATCH --time=00:02:00
#SBATCH --nodes=2
#SBATCH --gres=gpu:8

cd <some_working_directory>
srun –n 256 ./simulation.exe
```

# Useful Resource-Related Options (srun/sbatch/salloc)

| Description                       | Option                   |
|-----------------------------------|--------------------------|
| Total Number of tasks              | -n, --ntasks             |
| Number of tasks per compute node   | --ntasks-per-node        |
| Number of threads per task         | -c, --cpus-per-task      |
| Number of nodes                    | -N, --nodes              |
| Walltime                           | -t, --time               |
| Request N GPUs                     | --gres=gpu:N             |

# Some Examples

To run each of those scripts 
```bash
sbatch script.sh
```

### MOST BASIC

```bash
#!/bin/bash

#SBATCH -o logger-%j.txt
#SBATCH --nodes 1
#SBATCH --mem=4G
#SBATCH -c 1
#SBATCH -t 60
#SBATCH -A plgtimeseriescpu-cpu
#SBATCH -p plgrid

singularity exec /net/ascratch/people/plgbartekcupial/transfer.sif python3 -c "print(4+4)"

```

### ARES

```bash
#!/usr/bin/env bash

#SBATCH -o logger-%j.txt
#SBATCH --gres gpu:1  
#SBATCH --nodes 1
#SBATCH --mem=20G
#SBATCH -c 20
#SBATCH -t 2880
#SBATCH -A plgmodernclgpu-gpu
#SBATCH -p plgrid-gpu-v100


set -e


module load cuda/11.6.0
export PYTHONPATH=$PYTHONPATH:.

singularity exec --nv \
    -H /net/people/plgrid/plgbartekcupial/dungeonsdata-neurips2022/experiment_code \
    -B /net/ascratch/people/plgbartekcupial/nle:/nle \
    -B $TMPDIR:/tmp \
    /net/ascratch/people/plgbartekcupial/nle/dungeons.sif \
    ./train.sh
```

### MRUNNER SCRIPT
```bash
#!/usr/bin/env bash
set -e

echo $SLURM_ARRAY_TASK_ID

# Fork

mkdir /net/ascratch/people/plgbartekcupial/mrunner_scratch/sf2_nethack/06_10-13_24-ecstatic_meitner/2023-10-05-sweep-ks-t_ld63_$SLURM_ARRAY_TASK_ID
cp -r /net/ascratch/people/plgbartekcupial/mrunner_scratch/sf2_nethack/06_10-13_24-ecstatic_meitner/2023-10-05-sweep-ks-t_ld63/* /net/ascratch/people/plgbartekcupial/mrunner_scratch/sf2_nethack/06_10-13_24-ecstatic_meitner/2023-10-05-sweep-ks-t_ld63_$SLURM_ARRAY_TASK_ID

cp /net/ascratch/people/plgbartekcupial/mrunner_scratch/sf2_nethack/06_10-13_24-ecstatic_meitner/configs/config_$SLURM_ARRAY_TASK_ID /net/ascratch/people/plgbartekcupial/mrunner_scratch/sf2_nethack/06_10-13_24-ecstatic_meitner/2023-10-05-sweep-ks-t_ld63_$SLURM_ARRAY_TASK_ID/

cd /net/ascratch/people/plgbartekcupial/mrunner_scratch/sf2_nethack/06_10-13_24-ecstatic_meitner/2023-10-05-sweep-ks-t_ld63_$SLURM_ARRAY_TASK_ID
module load cuda/11.6.0
export PYTHONPATH=$PYTHONPATH:.


export SINGULARITY_PREFIX="singularity exec --nv -H $PWD:/homeplaceholder --env WANDB_API_KEY=<key> --env WANDBPWD=$PWD -B /net:/net -B /net/pr2/projects/plgrid/plgggmum_crl/bcupial/ttyrecs:/ttyrecs -B $TMPDIR:/tmp /net/pr2/projects/plgrid/plgggmum_crl/bcupial/sf2_nethack.sif  "
singularity exec --nv -H $PWD:/homeplaceholder --env WANDB_API_KEY=<key> --env WANDBPWD=$PWD -B /net:/net -B /net/pr2/projects/plgrid/plgggmum_crl/bcupial/ttyrecs:/ttyrecs -B $TMPDIR:/tmp /net/pr2/projects/plgrid/plgggmum_crl/bcupial/sf2_nethack.sif python3 mrunner_run.py --config config_$SLURM_ARRAY_TASK_ID

```


### GMUM SERVERS

```bash
#!/bin/bash

#SBATCH --job-name=video_GAN
#SBATCH --output=logger-%j.txt
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=10
#SBATCH --mem-per-cpu=8G
#SBATCH --gres=gpu:1
#SBATCH --qos=normal  # test (1 GPU, 1 hour), quick (1 GPU, 1 day), normal (2 GPU, 2 days), big (4 GPU, 7 days)
#SBATCH --partition=rtx2080  # rtx2080 (mini-servers), dgxteam (dgx1 for Team-Net), dgxmatinf (dgx2 for WMiI), dgx (dgx1 and dgx2)


datasets=/shared/results/Skopia/videos60frames
checkpoints=/shared/results/z1143165/jelito3d_batchsize8
dir_results=/shared/results/struski/videoGAN

singularity exec --nv -B $datasets:/datasets -B $checkpoints:/checkpoints -B $dir_results:/results /shared/sets/singularity/miniconda_pytorch_py39.sif python training_create_movie.py --resume_path /results/clipping_frame60_duration24_iterD2_alphaSimil0_proDISC_seed9056_220610-151204/checkpoints/frame_seed_generator.pt --num_samples 10 --batch_size 5 --n_frames 60 --name "extension_num_frames60"

```

# MRUNNER
We need to install mrunner (because we want to run some examples from it we need to clone it)
```bash
git clone https://gitlab.com/awarelab/mrunner.git
cd mrunner
pip install . 
```

Get the following values from up.neptune.ml. Click help and then quickstart.

```
export NEPTUNE_API_TOKEN = ...
export PROJECT_QUALIFIED_NAME = ...
```

Create file `~/.mrunner.yaml` where we will store contexts for the mrunner

```bash
contexts:
  ares_cpu:
    account: plgtimeseriescpu-cpu
    backend_type: slurm
    cmd_type: sbatch
    cpu: 1
    mem: 4G
    nodes: 1
    ntasks: 1
    partition: plgrid
    singularity_container: --nv -H $PWD:/homeplaceholder --env WANDB_API_KEY=<key> --env WANDBPWD=$PWD -B $TMPDIR:/tmp /net/ascratch/people/plgbartekcupial/nle/dungeons.sif
    slurm_url: plgbartekcupial@ares.cyfronet.pl
    storage_dir: /net/ascratch/people/plgbartekcupial
    time: 60
```

To test if works try
```bash
mrunner --config ~/.mrunner.yaml --context ares_cpu run examples/experiment_basic_conf.py
```