# isaac-sim-singularity-container
This repo contains the instructions on creating a singularity container on the SCC that supports Nvidia Isaac Sim functionality.


Created on April 18, 2025

Research Computing Services

rcs.bu.edu

## Step 1. Convert the Docker container to Singularity

Get an OnDemand Desktop or ssh to a login node.

```
# connect to scc-i01. This is a compute node that is
# not part of the job queue, it is used for container
# building.
ssh scc-i01

# go to the /scratch directory

cd /scratch
mkdir $USER
cd $USER

# Pull the Docker container and convert it to
# Singularity format

singularity pull docker://nvcr.io/nvidia/isaac-sim:4.5.0
# Move to your /projectnb space


mv isaac-sim_4.5.0.sif /projectnb/your/directory
cd
rm -rf /scratch/$USER
# Log off from scc-i01
exit
```


## Step 2. Request a GPU OnDemand session

```
# In the OnDemand Desktop form select 1 GPU and a compute
# capability of 8.0. Request at least 8 cores (see notes below).
# In the "additional arguments to qsub" field put (exactly as shown):
# -v COMPUTE_MODE=Default

# You can try multiple GPUs but asking for more than 1 will
# greatly increase queue waiting times.

# When your GPU desktop is ready connect to it.

# Make a directory for cached files in your /projectnb
# space.
mkdir /projectnb/your/cache/path
# make a symlink
ln -s /projectnb/your/cache/path ~/.cache/ov

# Change to the directory where the container is installed
cd /projectnb/your/container/dir

# set env variables for convenience in the container.
export SINGULARITYENV_ISAAC_CACHE=/projectnb/your/cache/path
export SINGULARITYENV_ACCEPT_EULA=Y

# Launch singularity
scc-singularity --nolocal shell isaac-sim_4.5.0.sif

# In the container...
cd /isaac-sim/

# Run the GUI.
# --portable-root --> lets it write files someplace not
# in the read-only container
# --/renderer/activeGpu --> limits its access to your
# assigned GPU
# This will take 3-5 minutes before it's ready the 1st time
# as it downloads files.
./isaac-sim.sh --portable-root $ISAAC_CACHE \
--/renderer/activeGpu=$CUDA_VISIBLE_DEVICES

# Run the command line (this is a headless mode)
python.sh \
standalone_examples/api/isaacsim.simulation_app/constant_fps.py \
--portable-root $TMPDIR \
--/renderer/activeGpu=$CUDA_VISIBLE_DEVICES
```


## **NOTE**

```
# We are unable to determine how to limit the number of CPU
# threads, to stay within the number of requested cores. You
# will have to investigate the documentation:
# https://docs.isaacsim.omniverse.nvidia.com/latest/index.html
# and/or consult the forum:
# https://forums.developer.nvidia.com/c/omniverse/300
#
# * Workaround: request a GPU node with all cores. Note that
this will result in long queue wait times.
```
