# BashingAround
Bash one liners and functions I use.

#### Prompt ####

My PS1 `export PS1='\e\[\033[0;36m\]\w\e[0m \e\[\033[0;32m\]$(date -u "+%Y-%m-%d %H:%M:%S")\e[0m \e\[\033[0;33m\]$(git branch 2>/dev/null | grep '^*' | colrm 1 2)\e[0m \n $ '`

add to `bash_profile` for mac `if [ -f "${HOME}/.bashrc" ]; then source "${HOME}/.bashrc"; fi`.

---

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
function myip(){
  dig +short myip.opendns.com @resolver1.opendns.com
}
```

# Docker container times drift when sleeping your mac, this will fix that.
```bash
function fixdockertime() {
  docker run -it --rm --privileged --pid=host debian nsenter -t 1 -m -u -n -i date -u $(date -u +%m%d%H%M%Y)
}
```

# git functions
```bash
function fgit(){
  # fetch all remote branches
  git fetch --all
  git branch -a
}

function rgit() {
  # nuke option, will reset your repo like you just rm'd it and cloned it down again
  git fetch origin
  git reset --hard origin/master
  git clean -fdx
}

function dgit() {
  # remove all local branches expect for master
  git branch | grep -v "master" | xargs git branch -D
}

function sgit() {
  # show files staged for push
  git diff --stat --cached origin/master
}

function mgit() {
  # used when you want to update your branch with master
  git merge origin/master
}

function bgit() {
  # create a new branch, change to it and push it up
  # ex. bgit <YOUR_BRANCH_NAME>
  git branch "${1:-}"
  git checkout "${1:-}"
  git push -u origin "${1:-}"
}

function prgit() {
  # just run prgit in your branch to open a PR for it
  if [[ -d "/Applications/Google Chrome.app" ]]; then
    INSTALLED="Google Chrome"
  elif [[ -d "/Applications/Firefox.app" ]]; then
    INSTALLED="firefox"
  elif [[ -d "/Applications/Safari.app" ]]; then
    INSTALLED="safari"
  else
    INSTALLED="none"
    printf "You don't have a supported browser (Chrome/Firefox/Safari) installed."
    exit 1
  fi

  repo=$(basename $(git rev-parse --show-toplevel))
  branch=$(git branch | grep \* | cut -d ' ' -f2)

  open -a "${INSTALLED}" "https://github.com/sweetride/${repo}/pull/new/${branch}"
}
```

# last grep you will ever need

```bash
function gf() {
  grep -iR ''"${1:-}"'' '.'
}
```

# set temp aws keys using assumed role

```bash
function aws-auth(){
    unset AWS_ACCESS_KEY_ID
    unset AWS_SECRET_ACCESS_KEY
    unset AWS_SESSION_TOKEN

    if [[ -z "${AWS_PROFILE:-}" ]]; then
      echo 'Please set $AWS_PROFILE or AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY.' >&2
      return 1
    fi

    if aws configure get source_profile &>/dev/null; then
      local creds source_profile arn

      source_profile="$(aws configure get source_profile)"
      arn="$(aws configure get role_arn)"
      creds="$(AWS_PROFILE="$source_profile" aws sts assume-role --role-arn "$arn" --role-session-name "${USER}--terraform" --duration-seconds 900)"
      echo "${creds}"

      export AWS_ACCESS_KEY_ID="$(echo "$creds" | jq -r .Credentials.AccessKeyId)"
      export AWS_SECRET_ACCESS_KEY="$(echo "$creds" | jq -r .Credentials.SecretAccessKey)"
      export AWS_SESSION_TOKEN="$(echo "$creds" | jq -r .Credentials.SessionToken)"
    fi
}
```

# temp file

```bash
function blah() {
  junkfile=$(mktemp)
  vim ${junkfile}
  rm -rf ${junkfile}
}
```

----------

#### For Fun ####

**file contents:**

The quick brown fox jumps over the lazy dog.

**Letter Count: (use sed "$ d" to remove last line if needed)**
```bash
(for i in $(grep -o . file); do echo ${i}; done;) | tr '[[:upper:]]' '[[:lower:]]' | sed 's/[^a-z]*//g' | sort -f | uniq -ic | sort -rk 1,1
```

**Word Count:**
```bash
(for i in $(cat file); do echo ${i}; done;) | tr '[[:upper:]]' '[[:lower:]]' | sed 's/[^a-z]*//g' | sort -f | uniq -ic | sort -rk 1,1
```

# solving coding challenges in bash

```bash
#!/usr/bin/env bash

# Suppose you are given an array of size 1 to n-length. Write a function that
# will log to console which values between 1 and n that are NOT in the array.
# For example - given an array of size 5 containing values [5, 2, 5, 1, 1] your
# function would print to console "3,4". On the other hand, given an array of
# size 6 that contained values [6, 3, 4, 1, 2, 5], your function would print nothing ("").

set -oue pipefail

function ohgod_what_i_have_done() {
  array=$1
  declare -a dedupsorted
  declare -a newlist

  IFS=$'\n' sorted=($(echo "${array[@]}" | tr ' ' '\n' | sort -nu))

  for x in "${sorted[@]}"; do
    dedupsorted+=("${x}")
  done
  unset IFS

  last=$(echo "${dedupsorted[-1]}")
  first=$(echo "${dedupsorted[0]}")

  IFS=$'\n'
  for i in $(seq ${first} ${last}); do
    newlist+=("${i}")
  done
  unset IFS

  echo ${newlist[@]} ${dedupsorted[@]} | tr ' ' '\n' | sort -n | uniq -u
}

function main() {
  if [[ -z "${1:-}" ]]; then echo "example: $0 \"1,12,3,6,17\""; exit 1; fi
  IFS=', ' read -r -a array <<< "$1"
  ohgod_what_i_have_done "${array[@]}"
}

main "$@"
```
