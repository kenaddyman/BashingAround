# BashingAround
Bash one liners and functions I use.

#### filefinder ####

There are many one liners for finding large files on linux systems this is one I put together a while back and use all the time.

```bash
function filefinder(){
  (find $(pwd) -type f -size +25M -print0 2> /dev/null | xargs -0 ls -lhsS) | column -t | cut -d ' ' -f 2- | sed -e 's/^[ \t]*//'
}
```

**Example:**
<pre>
[root@cj8dcl102 var]# (find $(pwd) -type f -size +25M -print0 2> /dev/null | xargs -0 ls -lhsS) | column -t | cut -d ' ' -f 2- | sed -e 's/^[ \t]*//'
-rw-------   1  root  root  401M  Sep  18  03:41  /var/log/messages-20170918
-rw-------   1  root  root  331M  Sep  24  03:27  /var/log/messages-20170924
-rw-------   1  root  root  293M  Sep  29  11:39  /var/log/messages
-rw-------   1  root  root  280M  Mar  7   2017   /var/lib/docker/tmp/GetImageBlob233346684
-rw-------   1  root  root  212M  Jun  15  2016   /var/lib/docker/tmp/GetImageBlob060374704
-rw-------   1  root  root  212M  Jun  15  2016   /var/lib/docker/tmp/GetImageBlob783178594
-rw-r--r--   1  root  root  203M  Sep  27  08:30  /var/cache/yum/x86_64/7OUL/base/gen/primary.xml.sqlite
-rw-r--r--   1  root  root  169M  Sep  27  07:10  /var/cache/yum/x86_64/7OUL/base/gen/primary.xml
-rw-r--r--   1  root  root  135M  Sep  27  08:30  /var/cache/yum/x86_64/7OUL/base_UEK4/gen/primary.xml.sqlite
-rw-r--r--   1  root  root  119M  Sep  27  07:15  /var/cache/yum/x86_64/7OUL/base_UEK4/gen/primary.xml
-rw-r--r--.  1  root  root  73M   Sep  1   13:26  /var/lib/rpm/Packages
-rw-------   1  root  root  60M   Jun  15  2016   /var/lib/docker/tmp/GetImageBlob061386980
-rw-------   1  root  root  41M   Apr  14  2016   /var/chef/cache/cookbooks/cars-automic/files/default/webhelpe.tar.gz
-rw-------   1  root  root  31M   Aug  3   10:30  /var/lib/docker/tmp/GetImageBlob025163980
</pre>

----------

#### fsuse ####

Used to fix display of df results and order filesystems from most used to least.

```bash
function fsuse(){
df -hP | column -t | grep -v ^none | ( read header ; echo "$header" | sed 's/Mounted.*on/Mounted_on/g' ; sort -rn -k 5)
}
```


**Exmaple:**
<pre>
[kaddyman@cj8dcl102 ~]$ df -hP | column -t | grep -v ^none | ( read header ; echo "$header" | sed 's/Mounted.*on/Mounted_on/g' ; sort -rn -k 5)
Filesystem                      Size  Used  Avail  Use%  Mounted_on
/dev/mapper/rootvg-varlv        3.9G  2.4G  1.3G   65%   /var
/dev/mapper/appvg-splunklv      477M  282M  166M   63%   /apps/Splunk
/dev/mapper/rootvg-rootlv       2.0G  870M  964M   48%   /
/dev/sda1                       477M  186M  262M   42%   /boot
/dev/mapper/rootvg-usrlv        9.8G  3.9G  5.4G   42%   /usr
/dev/mapper/appvg-dockerlogslv  20G   4.0G  15G    22%   /apps/docker/logs
/dev/mapper/rootvg-optlv        3.9G  423M  3.2G   12%   /opt
tmpfs                           16G   1.7G  14G    11%   /run
/dev/mapper/rootvg-homelv       3.9G  84M   3.6G   3%    /home
/dev/mapper/rootvg-tmplv        3.9G  25M   3.6G   1%    /tmp
/dev/mapper/appvg-powertrainlv  4.8G  22M   4.6G   1%    /apps/powertrain
/dev/mapper/appvg-dockerlv      30G   45M   28G    1%    /apps/docker
tmpfs                           3.2G  0     3.2G   0%    /run/user/10056
tmpfs                           16G   0     16G    0%    /sys/fs/cgroup
tmpfs                           16G   0     16G    0%    /dev/shm
devtmpfs                        16G   0     16G    0%    /dev
</pre>

----------

**showDeletedFiles**

Used to find deleted files that are still associated with a running WebSphere Java process and list them from largest to smallest. If you are low on file system space but the file system is not showing any large files it might because of a java process holding on to it. Once you restart that java processes it will release that space back to the OS.

```bash
function showDeletedFiles {
  echo
  (
  echo "PID JVM FILESIZE FILENAME"
  (
  DELETEDFILES=$(/usr/sbin/lsof | grep \(del[e]ted\) | grep j[a]va| awk '{print $2","$7","$9}')
  JAVAPIDS=$(ps -ef |grep jav[a] | awk '{print $1","$2","$5","$NF}')

  for i in $(echo "${DELETEDFILES}"); do

    PID=$(echo "${i}" | awk -F',' '{print $1}')
    FILESIZE=$(echo "${i}" | awk -F',' '{print $2}')
    FILENAME=$(echo "${i}" | awk -F',' '{print $3}')

    if [[ ! "${FILESIZE}" == '0' ]]; then
      #echo "PID: ${PID} | FILESIZE: ${FILESIZE} | FILENAME: ${FILENAME}"

      for x in $(echo "${JAVAPIDS}"); do
        USER=$(echo "${x}" | awk -F',' '{print $1}')
        PSPID=$(echo "${x}" | awk -F',' '{print $2}')
        RUNAS=$(echo "${x}" | awk -F',' '{print $3}')
        JVMNAME=$(echo "${x}" | awk -F',' '{print $4}')

        #echo "User: ${USER} | PSPID: ${PSPID} | RUNAS: ${RUNAS} | JVMName: ${JVMNAME}"

        if [[ "${PID}" -eq "${PSPID}" ]]; then
          echo "${PSPID} ${JVMNAME} ${FILESIZE} ${FILENAME}"
        fi

      done

    fi

  done ) | sort -k 3 -n -r
  ) | column -t
  echo
}
```

**Example:**	
<pre>
[cars@xxxxxxx ~]$ showDeletedFiles

PID    JVM                   FILESIZE   FILENAME
14563  Composite_Type1xxx01  789190585  /tmp/spring.log.7
27746  Composite_Type1xxx01  478772908  /tmp/spring.log.7
27746  Composite_Type2xxx01  478772908  /tmp/spring.log.7
19361  Composite_Type1xxx01  130243699  /tmp/spring.log.7
19361  Composite_Type2xxx01  36581879   /tmp/spring.log.1
27746  Composite_Type3xxx01  17926023   /tmp/spring.log.7
...
</pre>

----------

#### Useful bash functions for your bashrc ####

```bash
# Shows what my external IP is.
function showmyip(){
  dig +short myip.opendns.com @resolver1.opendns.com
}
```

```bash
# Drops you into the cars user in the sre-docker container
function sredocker {
  local DOCKERID=$1
  if [[ ! "${DOCKERID}" ]]; then
    DOCKERID=$(docker ps --filter "label=sre-docker" --format "{{.ID}}")
  fi
  docker exec -i -t ${DOCKERID} /home/cars/login.sh
}
```

```bash
# Docker container times drift when sleeping your mac, this will fix that.
function fixdockertime() {
  docker run -it --rm --privileged --pid=host debian nsenter -t 1 -m -u -n -i date -u $(date -u +%m%d%H%M%Y)
}
```

```bash
# show commits that are ready to be pushed up
showpush() {
  git diff --stat --cached origin/master
}
```

```bash
# reset git repo to last commit, delete any changes made
function resetgit() {
  git fetch origin
  git reset --hard origin/master
  git clean -fdx
}
```

----------

#### For Fun ####

**file contents:**

The quick brown fox jumps over the lazy dog.

**Letter Count: (use sed "$ d" to remove last line if needed)**

(for i in $(grep -o . file); do echo ${i}; done;) | tr '[[:upper:]]' '[[:lower:]]' | sed 's/[^a-z]*//g' | sort -f | uniq -ic | sort -rk 1,1

**Word Count:**

(for i in $(cat file); do echo ${i}; done;) | tr '[[:upper:]]' '[[:lower:]]' | sed 's/[^a-z]*//g' | sort -f | uniq -ic | sort -rk 1,1

