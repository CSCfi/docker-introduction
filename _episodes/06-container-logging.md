---
title: "Checking your container logs"
teaching: 15
exercises: 5
questions:
- "How can I check my container logs?"
objectives:
- "Explain the difference between a container STDOUT output and the container's own logs."
- "Demonstrate how to start a container running in the background writing simple output to STDOUT."
- "Demonstrate how to examine the logs being written by the container to STDOUT."
- "Demonstrate how to check the container's own logs"
keypoints:
- "The `docker exec` command enables the user to interact with a running container."
- "The `docker attach` command is used to a local standard input, output, and error streams to a running container"
- "The `docker start`, `docker stop`, `docker kill` commands enable the user to perfom basic container management."
---

### Logging and debugging containers

So far, we have merely run containers that run once and exit, and send little or no output to stdout. But what if we like to monitor the logs of a container running as a service?

Let's run a container from the `centos` image in the background ("-d" option ), that echoes a simple message to stdout:  

{: .language-bash}
~~~
$ docker run -d --name loop-guru centos sh -c "while true; do $(echo date); sleep 1; done"
~~~

We named this container "loop-guru" (the `--name` option), and that makes it easier to follow its logs. Check that it's running with `docker ps` and follow its logs:


{: .language-bash}
~~~
$ docker logs -f loop-guru
~~~

Lets test pausing the loop-guru without exiting or stopping. In another terminal run `docker pause loop-guru`. Notice how the logs output has paused in the first terminal. To unpause do `docker unpause loop-guru`.

Keep the logs open and attach to the container on the second terminal using 'attach':

{: .language-bash}
~~~
$ docker attach loop-guru
Mon Jan 15 19:26:54 UTC 2018
Mon Jan 15 19:26:55 UTC 2018
~~~

Now you have logs (STDOUT) running in two terminals. Now in the attach window press control+c. The container is stopped because the process is no longer running.

If we want to attach to a container and make sure we don't close it we can disable signal proxying. Lets start the stopped container with `docker start loop-guru` and attach to it with `--sig-proxy=false`.

Then try control+c.  

{: .language-bash}
~~~
$ docker start loop-guru

$ docker attach --sig-proxy=false loop-guru
Mon Jan 15 19:27:54 UTC 2018
Mon Jan 15 19:27:55 UTC 2018
^C
~~~

The container will stays running, just disconnecting you from the STDOUT.

To enter a container, we can start a new process in it.

{: .language-bash}
~~~
$ docker exec -it loop-guru bash
root@2a49df3ba735:/# ps aux

USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   4496  1716 ?        Ss   10:31   0:00 sh -c while true; do date; sleep 1; done
root       271  0.0  0.0   4496   704 ?        Ss   10:33   0:00 sh
root       300  0.0  0.0  18380  3364 pts/0    Ss   10:33   0:00 bash
root       386  0.0  0.0   4368   672 ?        S    10:33   0:00 sleep 1
root       387  0.0  0.0  36836  2900 pts/0    R+   10:34   0:00 ps aux
~~~

In our command `-it` is short for `-i`  and `-t` where `-i` is "interactive, connect STDIN" and `-t` "allocate a pseudo-TTY". From `ps aux` listing we can see that our `bash` process got pid 300. Or to put it more simply, `-it` allows you to interact with the container by using the command line.

Now that we're inside the container it behaves like you'd expect from Centos and we can terminate the container by killing the process with `kill 1` or exit the container with `exit` and either kill or stop the container.

Our loop-guru won't stop for SIGTERM that the stop sends but stop sends SIGKILL command after grace period. In this case it's simply faster to use kill.

{: .language-bash}
~~~
$ docker kill loop-guru
$ docker rm loop-guru
~~~

The previous two commands would be basically the same as `docker rm --force loop-guru`

Let's start another process with `-it` and also with `--rm` to remove it automatically after it has exited. This means that there is no garbage containers left behind, but also that `docker start` can not be used to start the container after it has exited.

`docker run -d --rm -it --name loop-guru-2 centos sh -c 'while true; do date; sleep 1; done'`

Now let's attach to the container and hit control+p, control+q that detaches us from the STDOUT.

{: .language-bash}
~~~
$ docker attach loop-guru-2

Mon Jan 15 19:50:42 UTC 2018
Mon Jan 15 19:50:43 UTC 2018
^P^Qread escape sequence
~~~

Note that hitting `^C` would still kill (and remove due to `--rm`) the process because the `docker attach` was done without `--sig-proxy=false`

But what if actually want to check out log files form the running container itself? Once option would be to use `docker exec` to read the logs:

{: .language-bash}
~~~
$ docker exec loop-guru-2 cat /var/log/yum.log
Mar 05 17:36:54 Erased: firewalld-0.5.3-5.el7.noarch
Mar 05 17:36:54 Erased: python-firewall-0.5.3-5.el7.noarch
Mar 05 17:36:54 Erased: python-slip-dbus-0.4.0-4.el7.noarch
Mar 05 17:36:54 Erased: python-slip-0.4.0-4.el7.noarch
Mar 05 17:36:54 Erased: python-decorator-3.4.0-3.el7.noarch
Mar 05 17:36:55 Erased: firewalld-filesystem-0.5.3-5.el7.noarch
Mar 05 17:36:55 Erased: linux-firmware-20180911-69.git85c5d90.el7.noarch
Mar 05 17:36:55 Erased: 32:bind-utils-9.9.4-73.el7_6.x86_64
Mar 05 17:36:55 Erased: initscripts-9.49.46-1.el7.x86_64
Mar 05 17:36:55 Erased: iproute-4.11.0-14.el7.x86_64
Mar 05 17:36:55 Erased: iptables-1.4.21-28.el7.x86_64
Mar 05 17:36:55 Erased: libnetfilter_conntrack-1.0.6-1.el7_3.x86_64
Mar 05 17:36:55 Erased: 32:bind-libs-9.9.4-73.el7_6.x86_64
Mar 05 17:36:55 Erased: ipset-6.38-3.el7_6.x86_64
Mar 05 17:36:55 Erased: ipset-libs-6.38-3.el7_6.x86_64
Mar 05 17:36:55 Erased: less-458-9.el7.x86_64
Mar 05 17:36:55 Erased: groff-base-1.22.2-8.el7.x86_64
Mar 05 17:36:55 Erased: libmnl-1.0.3-7centos.el7.x86_64
Mar 05 17:36:55 Erased: GeoIP-1.5.0-13.el7.x86_64
Mar 05 17:36:55 Erased: libnfnetlink-1.0.1-4.el7.x86_64
Mar 05 17:36:55 Erased: sysvinit-tools-2.88-14.dsf.el7.x86_64
Mar 05 17:36:55 Erased: libselinux-python-2.5-14.1.el7.x86_64
Mar 05 17:36:56 Erased: ebtables-2.0.10-16.el7.x86_64
Mar 05 17:36:56 Erased: grubby-8.28-25.el7.x86_64
~~~

Exercise: check the contents of loop-guru-2 /var/yum.log with an alternative method.


{% include links.md %}

{% comment %}
<!--  LocalWords:  keypoints Dockerfiles Dockerfile docker.io WORKDIR
 -->
<!--  LocalWords:  test.py 43288b101abf fc145d33ea49 csv numpy cmap
 -->
<!--  LocalWords:  csv-to-scatter-plot.py matplotlib.pyplot viridis
 -->
<!--  LocalWords:  np.genfromtxt seaborn-whitegrid the_plot.colorbar
 -->
<!--  LocalWords:  f.savefig matplotlib links.md endcomment
 -->
<!--  LocalWords:  5377596cb1c035c102396f5934237a046f80da69974026f90bee5db8b7ba
 -->
{% endcomment %}
<!--  LocalWords:  PowerShell
 -->
