**// This file is for PDL internal developers**

# carp-umbrella

Download, build, and install carp (and its dependencies) a single highly-automated step.

All pdlfs repositories are mirrored at both github.com and on the pdlfs-internal git server running at dev.pdl.cmu.edu.

We may also used a service known as git-lfs to efficiently track large binary files
to help us reduce the overhead of maintaining those big things within our repository.
These large binary files are typically compressed tar files of many external dependencies.

**The purpose of this guild to show fellow pdlfs internal developers the steps of
properly setting up the repository and syncing the mirrors at both github.com and dev.pdl.cmu.edu.**

## Install git-lfs client

If git-lfs is not installed, you may be able to install it with the package manager (e.g. "apt-get install git-lfs").   If not, it can be installed as follows:

```bash
# First, get latest git-lfs from https://git-lfs.github.com/
# The latest version may be higher than 1.5.3.#
# For example, on 64-bit Ubuntu:
wget https://github.com/git-lfs/git-lfs/releases/download/v1.5.3/git-lfs-linux-amd64-1.5.3.tar.gz
tar xzf git-lfs-linux-amd64-1.5.3.tar.gz -C .

cd git-lfs-1.5.3

sudo ./install.sh

# Secondly, add a convenient git-lfs config
git config --global --add lfs.skipdownloaderrors true
```

## Local repository setup guide

We will use dev.pdl.cmu.edu as our primary git repository, and github.com as our secondary repository.
For git-lfs, however, both will use the service provided by github.com.
This ensures our external collaborators can `git-lfs pull` files without knowing the existence of dev.pdl.cmu.edu.

Here, it is assumed that git-lfs has been installed.

```bash
# First, git-clone from dev.pdl.cmu.edu but avoid fetching
# any git-lfs files because this will fail anyway
env GIT_LFS_SKIP_SMUDGE=1 git clone git@dev.pdl.cmu.edu:pdlfs/carp-umbrella.git

cd carp-umbrella

# Install git lfs for the repository (if you haven't globally enabled it)
git lfs install --local

# Secondly, config git-lfs to use github.com as the service provider,
# which by default will be dev.pdl.cmu.edu
git config --add lfs.url git@github.com:pdlfs/carp-umbrella.git

# Next, fetch all git-lfs files
git lfs pull

# Finally, add a push alias "all" that pushes to both dev.pdl and github
git remote add all git@github.com:pdlfs/carp-umbrella.git
git remote set-url --add --push all git@dev.pdl.cmu.edu:pdlfs/carp-umbrella.git
git remote set-url --add --push all git@github.com:pdlfs/carp-umbrella.git

# Verify settings
git remote -v
```
