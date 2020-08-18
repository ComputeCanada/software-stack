# Testing software

**Home:** [Software management](INDEX.md)

There are many ways that installed software can be tested before making it
available to the users.

### Contents

- [Installing in your home folder](#installing-in-your-home-folder)
- [Testing on the build node](#testing-on-the-build-node)
- [Testing with the test nodes](#testing-with-the-test-nodes)
- [Testing with proot on the clusters](#testing-with-proot-on-the-clusters)

## Installing in your home folder

Both Nix and EasyBuild allow you to build and install software in a different
directory than our default. You simply have to **omit** the `sudo -i -u nixuser`
or `sudo -i -u ebuser` parts of the commands and it will install it only for
you. Nix run this way will install something in your own profile, while
EasyBuild will install the software and modules in the folder `$HOME/.local`.

## Testing on the build node

Once you have installed new software on the build node, either using Nix or
EasyBuild, it should be automatically usable on this node. Until software is
actually pushed to CVMFS, none of it is visible to any users. It is therefore a
good time to test the software before it is pushed to production. You can run
small tests on this node, since it is dedicated to this purpose.

## Testing with the test nodes

You can push your newly installed software to the development CVMFS repository.
This will make it visible on all nodes that mount it. There are two such nodes:

- `cvmfs-client-dev.computecanada.ca`
- `cvmfs-gpu-test.computecanada.ca`

However, the production clusters do not see these software packages repository.

## Testing with proot on the clusters

On the clusters (Cedar, Graham and Béluga only), one can test the content of the
`dev` repository by using `proot`. This would be done with the following
command:

```
/cvmfs/soft.computecanada.ca/nix/var/nix/profiles/16.09/bin/proot -b /cvmfs/soft-dev.computecanada.ca:/cvmfs/soft.computecanada.ca env -i CC_CLUSTER=$CC_CLUSTER TERM=$TERM HOME=$HOME bash -l
```

For the duration of this bash session, this will intercept each call to a path
in `/cvmfs/soft.computecanada.ca`  and reroute it to the same path in
`/cvmfs/soft-dev.computecanada.ca`.
