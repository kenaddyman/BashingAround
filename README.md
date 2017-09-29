# BashingAround
Bash one liners and functions I use.

#### filefinder ####

There are many one liners for finding large files on linux systems this is one I put together a while back and use all the time.

<pre>
<code>
function filefinder(){
  (find $(pwd) -type f -size +25M -print0 2> /dev/null | xargs -0 ls -lhsS) | column -t | cut -d ' ' -f 2- | sed -e 's/^[ \t]*//'
}
</code>
</pre>


<pre>
Example: 
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

#### fsuse ####

Used to fix display of df results and order filesystems from most used to least.

<pre><code>
function fsuse(){
df -hP | column -t | grep -v ^none | ( read header ; echo "$header" | sed 's/Mounted.*on/Mounted_on/g' ; sort -rn -k 5)
}
</code></pre>



Exmaple:
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
