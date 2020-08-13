# ANSYS software installation procedure

**Home:** [Software management](../INDEX.md)

This is a complete installation procedure of ANSYS 18.1 with
[EasyBuild](../easybuild.md).

### Contents

- [Source files](#source-files)
- [Easybuild recipe](#easybuild-recipe)
- [Testing the local installation](#testing-the-local-installation)
- [Main installation and publication on dev](#main-installation-and-publication-on-dev)
- [Testing the dev publication](#testing-the-dev-publication)
- [Publication on prod](#publication-on-prod)

## Source files

On the build node (see [Initial setup](../setup.md)), one needs to download all
three (3) ISO files in the restricted space:

```
cd /cvmfs/restricted.computecanada.ca/easybuild/sources/a/ANSYS/
ls -1 ANSYS_181*
  ANSYS_181_LINX64_DISK1.iso
  ANSYS_181_LINX64_DISK2.iso
  ANSYS_181_LINX64_DISK3.iso
```

Mount (or ask someone to mount as root) all three ISO files. Create folder
`ANSYS-18.1` and merge the content of the three ISO files in that new folder.
Then, archive the `ANSYS-18.1` folder in `ANSYS-18.1.tar`.

```
mkdir disk1 disk2 disk3
mount -o loop ANSYS_181_LINX64_DISK1.iso disk1
mount -o loop ANSYS_181_LINX64_DISK2.iso disk2
mount -o loop ANSYS_181_LINX64_DISK3.iso disk3
mkdir ANSYS-18.1
rsync -a disk1/ ANSYS-18.1/
rsync -a disk2/ ANSYS-18.1/
rsync -a disk3/ ANSYS-18.1/
umount disk1 disk2 disk3 && rmdir disk1 disk2 disk3
tar -cf ANSYS-18.1.tar ANSYS-18.1

cd /cvmfs/restricted.computecanada.ca/easybuild/sources/a/ANSYS/
ls -1 ANSYS-18.1.tar
  ANSYS-18.1.tar
```

## Easybuild recipe

On the build-node:

```
# Clone repository if not done already
cd
git clone https://github.com/ComputeCanada/easybuild-easyconfigs.git

# Update repository and checkout the computecanada-master branch
cd easybuild-easyconfigs/
git pull
git checkout computecanada-master
```

From
[`ANSYS-18.2.eb`](https://github.com/ComputeCanada/easybuild-easyconfigs/blob/computecanada-master/easybuild/easyconfigs/a/ANSYS/ANSYS-18.2.eb),
create `ANSYS-18.1.eb`. Update the version number and the path to the `tmi.conf`
file:

```
cd easybuild/easyconfigs/a/ANSYS
cp ANSYS-18.2.eb ANSYS-18.1.eb

# Edit ANSYS-18.1.eb: the version number and the tmi.conf path
diff ANSYS-18.1.eb ANSYS-18.2.eb
  2c2
  < version = '18.1'
  ---
  > version = '18.2'
  40c40
  <     setenv("TMI_CONFIG",pathJoin(root,"v181/commonfiles/MPI/Intel/5.1.3.223/linx64/etc/tmi.conf"))
  ---
  >     setenv("TMI_CONFIG",pathJoin(root,"v182/commonfiles/MPI/Intel/5.1.3.223/linx64/etc/tmi.conf"))
```

## Testing the local installation

Proceed with a test installation in your home directory (`./local/easybuild`) on
the build-node:

```
screen -S ansys181
eb ANSYS-18.1.eb --sourcepath=/cvmfs/restricted.computecanada.ca/easybuild/sources
# To detach from screen: Ctrl+A, D
```

Once installed, reconnect to the build-node with X11 forwarding. Then:

```
module use $HOME/.local/easybuild/modules/2017/Core
module load ansys/18.1

# You may have to put the build-node's IP address in the license server firewall
export ANSYSLMD_LICENSE_FILE=1055@<license_server>
export ANSYSLI_SERVERS=2325@<license_server>

fluent &
```

If the test succeeds, publish the Easybuild file and do some cleanup:

```
cd $HOME/easybuild-easyconfigs/easybuild/easyconfigs/a/ANSYS
git add ANSYS-18.1.eb
git pull origin computecanada-master
git commit -m "New ANSYS version 18.1"
git push origin computecanada-master

cd
chmod -R u+w .local/easybuild
rm -rf .local/easybuild
```

## Main installation and publication on dev

Installation on the restricted space:

```
screen -d -r ansys181

sudo -iu ebuser eb-pull-cc
sudo -i -g rsnt_soft -u ebuser eb ANSYS-18.1.eb
```

Publication on the dev repository:

```
sudo su - libuser

sudo /etc/rsnt/start_transaction dev
sudo /etc/rsnt/start_transaction restricted
/etc/rsnt/rsnt-sync --what easybuild --software ansys --version 18.1
ls /stratum0/cvmfs/restricted.computecanada.ca/easybuild/software/2017/Core/ansys/18.1
sudo /etc/rsnt/publish_transaction restricted
sudo /etc/rsnt/publish_transaction dev && exit
```

## Testing the dev publication

Connect to `cvmfs-client.computecanada.ca` with X11-forwarding. Then:

```
proot -b /cvmfs/soft-dev.computecanada.ca:/cvmfs/soft.computecanada.ca bash
touch .licenses/ansys.lic
```

The content of the `ansys.lic` is described [on that
page](https://docs.computecanada.ca/wiki/ANSYS#Configuring_your_own_license_file).
Then:

```
module load ansys/18.1
fluent &
```

If the test succeed, then it is possible to publish to the prod repository.

## Publication on prod

Back to `build-node`:

```
screen -d -r ansys181
```

```
sudo su - libuser

sudo /etc/rsnt/start_transaction prod
sudo /etc/rsnt/start_transaction restricted
/etc/rsnt/rsnt-sync --what easybuild --software ansys --version 18.1
sudo /etc/rsnt/publish_transaction restricted
sudo /etc/rsnt/publish_transaction prod && exit

exit # Screen
```
