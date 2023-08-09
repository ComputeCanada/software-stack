# Testing software

**Home:** [Software management](INDEX.md)

There are many ways that installed software can be tested before making it
available to the users.

### Contents

- [Installing in your home folder](#installing-in-your-home-folder)
- [Testing on the build nodes](#testing-on-the-build-nodes)
- [Testing with the test nodes](#testing-with-the-test-nodes)
- [Testing with proot on the clusters](#testing-with-proot-on-the-clusters)

## Installing in your home folder

EasyBuild allows you to build and install software in a different
directory than our default. You simply have to **omit** the `sudo -i -u ebuser` 
part of the commands and it will install it only for you. 
EasyBuild will install the software and modules in the folder `$HOME/.local`.

## Testing on the build nodes

Once you have installed new software on the build nodes it should be 
automatically usable on this node. Until software is
actually pushed to CVMFS, none of it is visible to any users. It is therefore a
good time to test the software before it is pushed to production. You can run
small tests on this node, since it is dedicated to this purpose.

## Testing with proot on the clusters (prefered method)

On the clusters (Cedar, Graham and Béluga only), one can test the content of the
`dev` repository by using `proot`. This would be done with the following
command:

```
export PROOT_NO_SECCOMP=1
/cvmfs/soft.computecanada.ca/gentoo/2020/usr/bin/proot -b /cvmfs/soft-dev.computecanada.ca:/cvmfs/soft.computecanada.ca env -i CC_CLUSTER=$CC_CLUSTER TERM=$TERM HOME=$HOME bash -l
```

For the duration of this bash session, this will intercept each call to a path
in `/cvmfs/soft.computecanada.ca`  and reroute it to the same path in
`/cvmfs/soft-dev.computecanada.ca`.

## Testing with the test nodes

You can push your newly installed software to the development CVMFS repository.
This will make it visible on all nodes that mount it. There is one such node:

- `cvmfs-client-dev.alliancecan.ca`

However, the production clusters do not see these software packages repository.

