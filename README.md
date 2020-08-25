# Problem statement

Running usable graphics terminal within a slurm cluster.

1. create an image that contains minimum viable systems
2. containing chrome, x, x11vnc
3. add python, openbox, tint2, tmux, r
4. tunnel through to the container

# set up kerberos
[advanced connection](https://www.sherlock.stanford.edu/docs/advanced-topics/connection/#avoiding-multiple-duo-prompts)

```bash
sudo apt update

# install kinit use <Tab> and <Enter> use default choices
sudo apt-get install krb5-user

# download kerberos environment
sudo curl -o /etc/krb5.conf https://web.stanford.edu/dept/its/support/kerberos/dist/krb5.conf

kinit frankliu@stanford.edu
klist
```

# change ~/.ssh/config

```
Host login.sherlock.stanford.edu
   GSSAPIDelegateCredentials yes
   GSSAPIAuthentication yes

Host login.sherlock.stanford.edu
   ControlMaster auto
   ControlPersist yes
   ControlPath ~/.ssh/%l%r@%h:%p
```

In bash, change permissions on that file

```bash
# change permissions
chmod o-rw config
chmod g-rw config
ls -la
```

# add ssh keys

[ssh keys](https://www.sherlock.stanford.edu/docs/getting-started/connecting/)

Login once to ~login.sherlock.stanford.edu~ so that an entry to ~/.ssh/known_hosts
```bash
ssh frankliu@login.sherlock.stanford.edu # make it a known host
ssh-keygen -R sherlock.stanford.edu
```

# (local) Login into sherlock

Start logging into sherlock via ssh:

```bash
ssh -Y frankliu@login.sherlock.stanford.edu
# don't if using tigervnc
ssh frankliu@login.sherlock.stanford.edu
```

# Download from sherlock

```bash
sftp frankliu@login.sherlock.stanford.edu
sftp> cd /home/groups/robertj2/frankliu/Image
sftp> cd Wurzetatlas\ 2002\ vol\ 6/
sftp> cd Root
sftp> cd ..      # change to the parent directory
sftp> lcd <local directory>
sftp> ls         # list directory
sftp> ls -l      # more info
sftp> lls        # local list
sftp> pwd        # shows present working directory
sftp> lpwd       # local pwd
sftp> get <filename>
sftp> mget <filename>*
sftp> put <filename>
sftp> mput <filename>*
sftp> get -r <directory name>
```

# getting to images

```bash
cd /home/groups/robertj2/frankliu/Images
cd Wurzetatlas\ 2002\ vol\ 6/
cd Root
cd W2002.Abb.92.right.b73.1
```

# useful

`ls` allows you to list all files in a directory
`-ltr` reverse order

```bash
ls -ltr
```

# display image

```bash
ml system imagemagick graphicsmagick x11
```

```bash
gm display <image_name>
```

[gm keyboard commands](http://www.graphicsmagick.org/display.html)

# (sherlock) Start interactive job

Begin an interactive job within sherlock.

```bash
# . ~/bin/interactive
ml system x11
srun -c 4 -t 8:00:00 --x11 --pty bash
```

```bash
# without x through tigervnc for window users
ml system
srun -c 4 -t 8:00:00 --pty bash  # choose # cpus and # hours
```

# tigervnc
https://github.com/TigerVNC/tigervnc/releases

https://bintray.com/tigervnc/beta/tigervnc/1.11beta

For windows download .exe:

[tigervnc64-1.10.90.exe](https://bintray.com/tigervnc/beta/download_file?file_path=tigervnc64-1.10.90.exe)

# (sherlock) Run singularity

Start running an image for graphics/web/r.  This is my docker, which can be
obtained from hub.docker: (`docker push
frankieliu/chrome-vnc-openbox-r:latest`) You don't need to construct a `.sif`
file, you can just use the one already built in
`/home/groups/robertj2/frankliu/Singularity` (`singularity pull
docker://frankieliu/chrome-vnc-openbox-r:latest`)

```bash
# . ~/bin/sing-vnc
singularity shell /home/groups/robertj2/frankliu/Singularity/chrome-vnc-openbox-r_latest.sif
```

# (sherlock/singularity) Start vncserver

Once you enter singularity  you 
```bash
# . ~/bin/start-vnc.sh
export DISPLAY=:99

# 16 bit color is not good enough for idea
# Xvfb :99 -screen 0 1366x768x16 &
Xvfb :99 -screen 0 1366x768x24 &

x11vnc -passwd 1234 -display :99 -advertise_truecolor -N -repeat -forever &
sleep 2
xrdb /home/users/frankliu/bin/Xdefaults
uxterm &
# /home/users/frankliu/bin/twm &
openbox &
```

# (sherlock/$HOME/bin/Xdefaults) Xdefaults

```bash
# file: ~/bin/Xdefaults
!XTerm*background: #2D2D2D
!XTerm*background: #FFFFFF
!XTerm*foreground: #D2D2D2
!Xterm*foreground: #300a24
!XTerm*background: #300a24
XTerm*background: #2d0922
!XTerm*background: #151517161818
XTerm*foreground: #FEFEFEFEFEFE
XTerm*cursorColor: #FEFEFEFEFEFE
XTerm*scrollBar: True
XTerm*rightScrollBar: True
XTerm*saveLines: 10000 
XTerm*vt100*geometry:   88x24
!XTerm*faceName: DejaVu Sans Mono Bold:size=11
XTerm*faceName: Ubuntu Mono:size=15
XTerm*renderFont: true

UXTerm*background:         black
UXTerm*foreground:         white
UXTerm*cursorColor:        white
UXTerm*VT100.geometry:     88x24

UXTerm*faceName:           Terminus:style=Regular:size=15
! UXTerm*font:              -*-dina-medium-r-*-*-16-*-*-*-*-*-*-*

!UXTerm*font:                terminus-10
UXTerm*dynamicColors:      true
UXTerm*utf8:               2
UXTerm*eightBitInput:      true
UXTerm*saveLines:          512
UXTerm*scrollKey:          true
UXTerm*scrollTtyOutput:    false
UXTerm*scrollBar:          true
UXTerm*rightScrollBar:     true
UXTerm*jumpScroll:         true
UXTerm*multiScroll:        true
UXTerm*toolBar:            false
```

# (local) Setting ssh tunnel for vnc

## Finding running node
Determine where one is running the interactive session.  We need to know the
host machine in order to tunnel to it.  The script below will save the `squeue`
information so that we can later use it to tunnel a vnc port to outside of
sherlock.

```bash
# . ~/bin/ssh-sherlock-node
ssh frankliu@login.sherlock.stanford.edu squeue -u frankliu > ~/sherlock-queue 
#             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON) 
#           4309617    normal     bash frankliu  R       1:56      1 sh02-01n13 
```

## Tunneling to the running node

This will set up a ssh tunnel from local port 5999 to remote port on the
container.

```bash
sherlock_file=~/sherlock-queue
node=$(cat $sherlock_file | sed -n '2{p;q}' | awk '{print $8}')
cmd="ssh frankliu@login.sherlock.stanford.edu -L5999:$node:5999"
echo $cmd
$cmd
```

# (local) vncviewer

You can choose any of the vncviewer's available.  I use tigervnc.
Currently you cannot resize the Xvfb, but I will change it so that you
may be able to resize with xrandr

(tigerserver)
[https://www.cyberciti.biz/faq/install-and-configure-tigervnc-server-on-ubuntu-18-04/]
- this may be a better option if we want resizing.

```bash
vncviewer localhost:5999
```

# (sherlock) Configurations

Make keys be able to repeat.

```bash
xset r on
```

# In general

After you finished the one time setup:

1. All in one local window
    1. local> ssh frankliu@login.sherlock.stanford.edu
    1. sherlock> ml system
    1. sherlock> srun --pty bash      # wait for a machine
    1. sherlock> squeue -u frankliu   # jot down machine name
    1. sworker> singularity shell /home/groups/robertj2/frankliu/Singularity/chrome-vnc-openbox-r_latest.sif 
    1. Singularity> . bin/start-vnc.sh
     
1. On another local window
    1. ssh frankliu@login.sherlock.stanford.edu -L5999:(machine name):5999
    1. you can change the portnumber from -L5999 to -L5998, etc if you get an error

1. Open tigervnc, connect to localhost:99 (password: 1234)  (or localhost:98 if you used the fix above)
    1. vnc> tint2 &
    1. vnc> /home/groups/robertj2/frankliu/Apps/rstudio/bin/rstudio &
