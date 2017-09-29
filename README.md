# BashingAround
Bash one liners and functions I use.

There are many one liners for finding large files on linux systems this is one I put together a while back and use all the time.

<code>
function filefinder(){
  (find $(pwd) -type f -size +25M -print0 2> /dev/null | xargs -0 ls -lhsS) | column -t | cut -d ' ' -f 2- | sed -e 's/^[ \t]*//'
}
</code>

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
