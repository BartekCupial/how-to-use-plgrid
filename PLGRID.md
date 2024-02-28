### To use PLGrid clusters, follow these steps:

1. **Create an Account**: Visit the PLGrid Portal at https://portal.plgrid.pl/ and sign up as a new user, indicating your affiliation with a Polish research unit.

2. **Add Affiliation**: Navigate to Portal PLGrid -> Affiliation -> Add, and enter the details of your supervisor (e.g., Piotr Miłoś if you're working with him). The affiliation must be confirmed, so you'll need to request confirmation from your supervisor.

3. **Join a Team**: To access resources, you must be added to a team. Request the team leader to add you or create your own team and apply for a grant.

4. **Apply for Cluster Access**: Go to Portal PLGrid -> Services -> Manage services, and apply for access to the Prometheus / Athena cluster or other clusters depending on your grant.

5. **Log in to the Cluster**: Once the above steps are completed, you can log in to prometheus.cyfronet.pl, ares.cyfronet.pl, or athena.cyfronet.pl (depending on your grant) using your PLGrid portal login and password.

6. **Set Up SSH Keys**: To log in without a password, generate an SSH key pair and add the public key to the servers.

7. **Use mrunner**: mrunner is a tool for submitting jobs to Slurm clusters. Install mrunner and try running one of the examples from https://gitlab.com/awarelab/mrunner/-/tree/master/examples. If the job is submitted correctly, it should be visible when you run `sacct` on the server.

### Useful Slurm Commands

- `sacct`: Shows your recent jobs.
- `sinfo`: Displays information about cluster nodes.
- `squeue`: Lists jobs in the queue.
- `hpc-fs`: Shows disk resources and quota usage.
- `hpc-grants`: Displays PLGrid grants you have access to.

### mrunner Configuration

To efficiently use mrunner, you need a configuration file with different contexts you'll be using. A context roughly corresponds to a combination of cluster, Python environment, and default resources. Here's an example context from the mrunner configuration file:

This configuration specifies the resources and environment for a job running on the Athena cluster with access to a GPU.

```plaintext
athena_sf2_nethack_1gpu:
    account: plgfoundationrl-gpu-a100
    backend_type: slurm
    cmd_type: sbatch
    cpu: 16
    mem: 20G
    gpu: 1
    nodes: 1
    ntasks: 1
    partition: plgrid-gpu-a100
    singularity_container: --nv -H $PWD:/homeplaceholder --env WANDB_API_KEY=<WANDB_KEY> --env WANDBPWD=$PWD -B $TMPDIR:/tmp /net/pr2/projects/plgrid/plgggmum_crl/bcupial/sf_cuda.sif
    slurm_url: plgbartekcupial@athena.cyfronet.pl
    storage_dir: /net/tscratch/people/plgbartekcupial/
    time: 720
```





