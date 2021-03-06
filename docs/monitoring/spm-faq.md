title: Sematext Monitoring FAQ
description: FAQ about Sematext server, application, and container monitoring platform with integrated alerts, dashboards, reports, and 3rd party notifications, and available in cloud as SaaS or private on-premises installation

### General
**What should I do if I can't find the answer to my question in this FAQ?**

Check the [general FAQ](/faq) for questions that are not strictly
about Sematext Monitoring.  If you can't find the answer to your
question please email <support@sematext.com> or use our live chat.

**Is there a limit to how many servers/nodes/containers I can monitor with Sematext?**

There are no limits on the number of servers, nodes, or
  containers. Each [Sematext Monitoring
  plan](https://sematext.com/spm/pricing) covers monitoring of a
  certain number of containers at no extra charge.

**What types of applications and infrastructure can Sematext monitor?**

See [integrations](/integration). You can also send [custom metrics](custom-metrics).

**What are the various configuration files present in spm-client package?**

The spm-client package contains Infra and App Agent. The configuration files of these agents are:

1. `/opt/spm/properties/st-agent.yml` - Contains Infra Agent related configurations.
2. `/opt/spm/spm-monitor/conf/spm-monitor-config-<YOUR_TOKEN>-<JVM_NAME>.properties` - One file for each of the App Agent setup on this host.
3. `/opt/spm/properties/agent.properties` - Configurations that are common to both Infra and App Agents.


**Can I reduce the amount of logs generated by SPM monitor?**

Yes. You can make SPM monitor reduce the number of messages it logs every minute
(otherwise it logs detailed info about its monitoring lifecycle).

For App Agent, you can find the property files for your monitoring apps with:

``` bash
ls /opt/spm/spm-monitor/conf/spm-monitor-config-*.properties
```

You can adjust one or more of them, depending on which application's
monitor log output you want to reduce. At the bottom of those files add
the following line:

``` properties
SPM_MONITOR_LOGGING_LEVEL=reduced
```

For Infra Agent, you can configure the following properties in `/opt/spm/properties/st-agent.yml`

```yml
logging:
    level: error
    write-events: false
```

After you have made the changes, restart your SPM monitor.

```sudo service spm-monitor restart```

If you are running in-process agents, restart your application/java process.

**How frequently does SPM monitor collect metrics and can I adjust that interval?**

SPM monitor by default collects metrics every 10 seconds. As example, to reduce this
frequency to 30 seconds, for App Agent simply add the following line to
your SPM monitor properties files located in
`/opt/spm/spm-monitor/conf`:

``` properties
SPM_MONITOR_COLLECT_INTERVAL=30000
```

The value is expressed in milliseconds. If you are adjusting it, we
recommend setting it to 30000. With bigger values it is possible some
1-minute intervals would be displayed without the data in the UI.

For Infra Agent, set `interval` property `/opt/spm/properties/st-agent.yml`.
To set it to 30 seconds, set the value to `30s`.

After you have made the changes, restart your SPM monitor.

```sudo service spm-monitor restart```

If you are running in-process agents, restart your application/java process.

**How much memory is standalone App Agent using and can it be adjusted?**

By default, each standalone SPM monitor process is started with
"-Xmx192m" setting. This means that its JVM heap will use 192 MB at
most. In many cases SPM monitor doesn't actually need or use that much
memory. If you want to be absolutely sure about it, simply lower this
number in `/opt/spm/spm-monitor/bin/spm-monitor-starter.sh`, with the
following variable (around line 63):

``` properties
JAVA_OPTIONS="$JAVA_OPTIONS -Xmx384m -Xms128m -Xss256k"
```

You can change "-Xmx" part to "-Xmx192m" for example and see how it goes
(check if SPM monitor is still working over the next few hours/days and
is still not using much CPU).

We don't recommend changing this setting if you are monitoring
Elasticsearch or Solr clusters with a high number of indices or
shards.  In those cases SPM monitor has to parse and gather large
amounts of metrics data returned by Elasticsearch or Solr. Contact us
in chat or via <support@sematext.com> if you'd like to get more
info and help making this adjustment.

**Can I use Sematext to monitor multiple applications running on the same server / VM?**

Yes. There are really 2 different scenarios here:

1.  If each of those applications should be monitored under a different
Monitoring App (e.g., you could have Solr running on your server along
with some Java app and you want to monitor both - Solr would be
monitored with Monitoring App of Solr or SolrCloud type, while the Java
app would be monitored with Monitoring App of JVM type), just complete
all installation steps (which are accessible
from <https://apps.sematext.com/ui/monitoring>, click Actions \> Install
Monitor for app you are installing) for each of them separately.

2.  If you want them monitored under the same Monitoring App (e.g., you
have 3 Solr instances running on a server), you must use different JVM
name for each of them. To do this, "1. Package installation" step should
be run only once on this machine, while "2. Client configuration setup"
step should be run once for each of the 3 Solr instances (installation
instructions are accessible
from <https://apps.sematext.com/ui/monitoring>, click Actions \> Install
Monitor for app you are installing). When running
script `/opt/spm/bin/setup-sematext` in step "2. Client
configuration setup", you should add `jvm-name` parameter (and value)
at the end of parameter list, like this:

```
sudo bash /opt/spm/bin/setup-sematext --monitoring-token 11111111-1111-1111-1111-111111111111 --app-type solr --agent-type javaagent --jvm-name solr1
```

In this example, we are setting up things for 3 separate Solr processes,
monitored under JVM names: solr1, solr2 and solr3 (you choose any
names), so this is adjusted command for solr1 instance.

In the remaining sub-steps of "2. Client configuration setup", replace
word "**default**" with the jvm-name value you just used.

"2. Client configuration setup" step will have to be repeated N times,
once for each monitored application (in our example 3 times with 3
different jvm-name values).

**Note**: By using this kind of setup, you will be able to see JVM stats
of all 3 processes separately (JVM name filter is used to do the
filtering). When it comes to other, non-JVM stats, they will be
aggregated into single value (for instance, request rate chart will show
sum of request rate value of these 3 Solr instances). If you want to see
even non-JVM stats separately, you will have to create 3 separate SPM
Solr applications (one for each Solr instance running on this
machine).

**Can I use Sematext to monitor my service which runs on Windows or Mac?**

SPM client installers currently exist only for Linux, however
there is still a way to monitor your service. If you are OK with
installing SPM client on separate linux box, and pointing it (more about
that further below) to your service (which should be monitored) running
on Windows or Mac, you can use Sematext to monitor all non-OS related
metrics. You can stop and disable OS metric collector using:

```bash
sudo service spm-monitor-sta stop
sudo systemd disable spm-monitor-sta # systemd supported OS
echo manual | sudo tee /etc/init/spm-monitor-sta.override  # init.d supported OS
```

Metrics specific to your process will be displayed (for
example, in case of Solr, you will see all search, index, cache metrics
along with all JVM metrics like pool memory, GC stats, JVM threads etc;
as another example, in case of MySQL you will see all metrics related to
connections, users, queries, handlers, commands, MyISAM, InnoDB, MySQL
traffic, etc) .

When monitoring Solr, Elasticsearch, HBase, Hadoop and other Java-based
services, you will have an option to choose between using
[In-process](spm-monitor-javaagent) (javaagent) or
[Standalone](spm-monitor-standalone) monitor. The
workaround described here requires the use of standalone monitor
variant. Here's what you'd need to do to see your metrics in SPM:

1.  Install spm-client on any Linux box (you can use this box for
    anything else, it is needed here just to run a process which
    collects metrics and sends them to Sematext). Installation
    instructions are accessible
    from <https://apps.sematext.com/ui/monitoring>, just click
    Actions \> Install Monitor for app are installing. In step 1 choose
    the "Other" tab to create a minimal installation.  This will not
    install modules needed for monitoring OS metrics - if those modules
    were installed, they would collect OS metrics of your Linux box,
    which is probably something you don't need. If you decide to follow
    instructions from some other tab, keep in mind OS metrics displayed
    in Sematext UI will not be OS metrics of your Windows/Mac machine.
2.  In step 2, if you are given a choice between
    [In-process](spm-monitor-javaagent) and
    [Standalone](spm-monitor-standalone) monitor,
    **choose Standalone monitor**. It will use remote JMX to collect
    metrics from your Windows/Mac machine. Just follow instructions
    given on Standalone tab.  The only difference you will want to make
    is in `-Dspm.remote.jmx.url` parameter used in monitor
    properties.  You will know where to adjust it if you follow standard
    instructions for Standalone monitor. While this parameter will
    usually have a value like `-Dspm.remote.jmx.url=localhost:3000`,
    in this case you will have to replace `localhost` with the
    address of your Windows/Mac machine(s) by which it can be reached
    from your helper Linux box.
3.  If you are monitoring something that doesn't offer a choice between
    In-process and Standalone monitor, installation instructions will
    explain where you can define your own address. Instructions
    typically use `localhost`, but you can instead use something
    like 10.1.2.3 or my-win-solr-server.mycompany.com to point to the
    machine that hosts the service you intend to monitor.

In case you want to monitor multiple machines belonging to the same
cluster in this way, you can still use SPM monitor installed on a
single Linux helper box. Just do this:

  - Create a separate Monitoring App
    (<https://apps.sematext.com/ui/integrations>) for each of the
    machines which should be monitored
  - For each of those Monitoring Apps (and your monitored machines), go
    through installation process presented after the app was created
    (installation instructions are accessible
    from <https://apps.sematext.com/ui/monitoring>, just click
    Actions \> Install Monitor for app your are installing). Note: Since
    each Monitoring App uses its own token, they all have slightly
    different installation commands. Besides different token, you will
    also use different addresses of Windows/Mac machines that host the
    monitored service.

**Can Sematext monitor metrics via http-remoting-jmx protocol (e.g. WildFly, JBoss, etc.)?**

Yes. The following steps are needed:

- In `/opt/spm/spm-monitor/conf/spm-monitor-config-YOUR_TOKEN-default.properties` add the following property.

``` properties
SPM_MONITOR_JAR="/opt/spm/spm-monitor/lib/spm-monitor-generic.jar:/path/to/your/wildfly-client-all.jar"
```

-  Change the value of `SPM_MONITOR_JMX_PARAMS` in
    `/opt/spm/spm-monitor/conf/spm-monitor-config-YOUR_TOKEN-default.properties.`
    Of course, you can append to that additional JMX parameters, for
    example with password file location etc:

``` properties
SPM_MONITOR_JMX_PARAMS="-Dspm.remote.jmx.url=service:jmx:remote+http://localhost:9990"
```

-  Restart spm-monitor with: ```sudo service spm-monitor restart```

At this point the metrics will start appearing in charts. If they don't,
run the diagnostics script to get fresh output of errors:
    ```sudo bash /opt/spm/bin/spm-client-diagnostics.sh``` 

If you see errors similar to:

``` java
Caused by: javax.security.sasl.SaslException: Authentication failed: all available authentication mechanisms failed:

java.io.FileNotFoundException: /opt/wildfly/standalone/tmp/auth/local8363680782928117445.challenge (Permission denied)

javax.security.sasl.SaslException: DIGEST-MD5: Cannot perform callback to acquire realm, authentication ID or password [Caused by javax.security.auth.callback.UnsupportedCallbackException
```

it means there is permissions issue on WildFly dirs. There are two ways
to get around this:

1.  Run SPM monitor with the same user that is running WildFly. To do
    that, add an entry like this to the end of
    `/opt/spm/spm-monitor/conf/spm-monitor-config-YOUR_TOKEN-default.properties`:

``` properties
SPM_MONITOR_USER="wildfly"
```

After that restart spm-monitor with: `sudo service spm-monitor
    restart`

2.  Change permissions for the problematic directory, adjusting the path
    to match your environment:

``` bash
chmod 777 /opt/wildfly/standalone/tmp/auth
```

This approach is not encouraged because of the obvious security
    problem, so use such approach only while testing, or if other
    options are not possible. As usual, restart spm-monitor after this
    change.

**When should I run Standalone and when Embedded SPM monitor?**

[Standalone SPM monitor](spm-monitor-standalone)
runs as a separate process, while [Embedded monitor](spm-monitor-javaagent) runs embedded in the
Java/JVM process. Thus, if you are monitoring a non-Java application,
Standalone monitor is the only option. Standalone monitor is a bit more
complex to set up when one uses it to monitor Java applications because
it typically requires one to enable out-of-process JMX access, as
described on [Standalone SPM monitor page](spm-monitor-standalone). With Embedded monitor
this is not needed, but one needs to add the SPM agent to the Java
command-line and restart the process of the monitored application. When
running Standalone monitor one can update the SPM monitor without
restarting the Java process being monitored, while a restart is needed
when Embedded SPM monitor is being used.  To be able to [trace transactions](/tracing) 
or [database operations](/tracing/database-operations) you need to use the
Embedded SPM monitor.

**Can I use Sematext for (business) transaction tracing?**

Yes, see [Transaction Tracing](/tracing).

**Can I move SPM client to a different directory?**

Yes. Soft move script moves all SPM files/directories to a new
location, but symlinks `/opt/spm` to the new location. Use this script if
you are OK with having `/opt/spm` symlinked. This script is recommended
for most situations since it keeps your SPM client installation
completely in line with standard setup (all standard SPM client commands
and arguments are still valid).

It accepts 1 parameter: new directory where SPM client should be moved
to (if such directory doesn't exist, it will be created)

``` bash
sudo bash /opt/spm/bin/move-spm-home-dir-soft.sh /mnt/some_dir
```

And that is it.

**Is there an HTTP API?**

Yes, see [API Reference](/api).

**I have multiple Monitoring Apps installed on my machine, can I uninstall just one of them?**

Yes, you can use the following command for that (it accepts only
one parameter, the token of the Monitoring App you want to uninstall):

```
sudo bash /opt/spm/bin/spm-remove-application.sh 11111111-1111-1111-1111-111111111111
```

**Can I disable SPM agent without uninstalling the SPM client?**

Yes, just find its .properties file in
`/opt/spm/spm-monitor/conf` and add to it:

``` properties
SPM_MONITOR_ENABLED=false
```

After that restart the monitor to apply the change. In case of
standalone agent, run:

``` bash
sudo service spm-monitor restart
```

And in case of in-process agent, just restart the service that has this
monitor's `javaagent` parameter.

### Sharing

**How can I share my Sematext Apps with other users?**

See [sharing FAQ](/faq/#sharing).

**What is the difference between OWNER, ADMIN, BILLING_ADMIN, and USER roles?**

See info about user roles in [sharing FAQ](/faq/#sharing).

### Agent Automation

**Is there an Ansible Playbook for the SPM client?**

Yes, see the
[Install](https://galaxy.ansible.com/sematext/spm-monitor-install/)
and
[Configure](https://galaxy.ansible.com/sematext/spm-monitor-config/)
playbooks, with examples.

**Is there a Puppet Module for the SPM client?**

Yes, see the Install and
Configure [module](https://forge.puppet.com/sematext/spm_monitor),
with examples.

**Is there a Chef Recipe for the SPM client?**

Yes, see [SPM client Chef Recipe](chef-recipe) example.

### Agent Updating

**How do I upgrade the SPM client?**

If you have previously installed the SPM client package (RPM,
Deb, etc.), simply upgrade via apt-get (Debian, Ubuntu, etc.), or yum
(RedHat, CentOS, etc.), or zypper (SuSE).

Debian/Ubuntu:

``` bash
# NOTE: this will update the sematext gpg key
wget -O - https://pub-repo.sematext.com/ubuntu/sematext.gpg.key | sudo apt-key add -
# NOTE: this does not update the whole server, just spm-client
sudo apt-get update && sudo apt-get install spm-client
```

RedHat/CentOS/...

``` bash
# NOTE: this will update sematext repo file
sudo wget https://pub-repo.sematext.com/centos/sematext.repo -O /etc/yum.repos.d/sematext.repo
# NOTE: this does not update the whole server, just spm-client
sudo yum clean all && sudo yum update spm-client
```

SuSE:

``` bash
sudo zypper up spm-client
```

After that is done, also do:

  - if you are using SPM monitor in in-process/javaagent mode - restart
    monitored server (restart your Solr, Elasticsearch, Hadoop node,
    HBase node... **Exceptions**: In case of Memcached, Apache and plain
    Nginx - no need to restart anything; in case of Redis only
    standalone SPM monitor exists so check below how to restart it)

**OR**

  - if you are using standalone SPM monitor, restart it with:  

``` bash
sudo service spm-monitor restart
```

**Note**: In case of **Memcached**, **Apache** and plain **Nginx** -
after completing upgrade steps described above, you must also run
commands described in Step 2 - Client Configuration Setup (which is
accessible from <https://apps.sematext.com/ui/monitoring>, click
Actions \> Install Monitor for app you have installed)

### Agent Uninstalling

**How do I uninstall the SPM client?**

On servers where you want to uninstall the client do the
following:

1.  remove spm-client, for instance: `sudo apt-get purge spm-client`  
    OR   `sudo yum remove spm-client`
2.  after that, ensure there are no old logs, configs, etc. by running
    the following command: `sudo rm -R /opt/spm`
3.  if you used in-process (javaagent) version of monitor, remove
    "-javaagent" definition from startup parameters of process which was
    monitored

**Note**: in case you used installer described on "Other" tab (found on
<https://apps.sematext.com/ui/monitoring>, click Actions \> Install
Monitor for app your are installing), instead of commands from step 1
run: `sudo bash /opt/spm/bin/spm-client-uninstall.sh` . After that
proceed with steps 2 and 3 described above.

### Alerts

**Can I send alerts to HipChat, Slack, Nagios, or other WebHooks?**

See [alerts FAQ](/faq/#alerts).

**What are Threshold-based Alerts?**

See [alerts FAQ](/faq/#alerts).

**What is Anomaly Detection?**

See [alerts FAQ](/faq/#alerts).

**What are Heartbeat Alerts?**

See [alerts FAQ](/faq/#alerts).

### Troubleshooting

**Can I enable debugging in the SPM agent?**

Yes. For App Agent, simply add or edit the `SPM_MONITOR_LOGGING_LEVEL` property in
any of the `/opt/spm/spm-monitor/conf/spm-monitor/*.properties`
files and restart the agent (or the process the agent is attached to).
Available levels are: FATAL, ERROR, WARN, INFO, DEBUG, TRACE

For Infra Agent, edit `logging.level` property in `/opt/spm/properties/st-agent.yml` file.
Available levels are: panic, fatal, error, warn, info, debug

**Can I install SPM client on servers that are behind a proxy?**

Yes. If you are installing the RPM, add this to `/etc/yum.conf`:

``` properties
proxy=http or https://proxy-host-here:port
proxy_username=optional_proxy_username
proxy_password=optional_proxy_password
```

If you are using apt-get, set the http_proxy environmental variable:

``` bash
export http_proxy=http://username:password@yourproxyaddress:proxyport
```

**Can SPM client send data out from servers that are behind a proxy?**

Yes. You can update the proxy settings using the following command:

```bash
sudo bash /opt/spm/bin/setup-env --proxy-host "HOST" --proxy-port "PORT" --proxy-user "USER" --proxy-password "PASSWORD"
```

**Can I change the region settings for the SPM agent installation?**

Yes. By default the region is set to US. You can change it to EU using:

```bash
sudo bash /opt/spm/bin/setup-env --region eu
```

**When installing SPM client, I see "The certificate of pub-repo.sematext.com is not trusted" or similar error.  How can I avoid it?**

There can be multiple reasons for this, most likely:

  - system time on your machine is not correct and adjusting it should
    fix the problem
  - you are installing SPM client as root user - in some cases that can
    cause this error and installing SPM client as a different user may
    help
  - you are missing the ca-certificates package.  Use one of the
    following commands to install it:

``` bash
sudo apt-get install ca-certificates
sudo yum install ca-certificates
```

If none of the above eliminates the problem for you try adding a flag to
avoid certificate checking. In case the command with wget is failing in
your case, add `--no-check-certificate` as wget argument. If command
with curl is failing, add `-k` flag.

**How do I create the diagnostics package?**

If you are having issues with Sematext Monitoring, you can
create diagnostics package on affected machines where SPM client was
installed by running:

``` bash
sudo bash /opt/spm/bin/spm-client-diagnostics.sh
```

The resulting package will contain all relevant info needed for our
investigation. You can send it, along with short description of your
problem, to <support@sematext.com> or contact us in chat.

**I see only my system metrics (e.g. CPU, Memory, Network, Disk...), but where is the rest of my data?**

Make sure you have followed all steps listed on [installation instructions page](https://apps.sematext.com/ui/monitoring).
**Package installation** steps should be done first, followed by
**Client configuration setup**. If you have done that and you still
don't see application metrics, run ```sudo
bash /opt/spm/bin/spm-client-diagnostics.sh``` to generate diagnostics
package and send it to <support@sematext.com> with description of
your problem.

**I do not see any system metrics (e.g. CPU, Memory, Network, Disk), what could be the problem?**

Make sure you have followed all steps listed on [installation instructions page](https://apps.sematext.com/ui/monitoring). It
is possible you missed **Client configuration setup** step. If you have
done that and you still don't see application metrics, run ```sudo
bash /opt/spm/bin/spm-client-diagnostics.sh``` to generate diagnostics
package and send it to <support@sematext.com> with description of
your problem.

**I am using trying to monitor Solr / Elasticsearch. My Request Rate and Latency charts are empty?**

If other Solr/ES charts are showing the data, it is most likely
that there were no requests sent to your Solr/ES in the time range you
are looking at. Try sending some queries and see if request rate/latency
charts will show them. If they don't, please send us an email to
<support@sematext.com> or contact
us in chat.

**I am not seeing any data in Monitoring charts. How do I check if network connectivity is OK?**

SPM agents send the data to Sematext via HTTP(S) so it is important
that servers where you install SPM agent can access the
internet. Things to check to ensure network connectivity is ok:

1\. Try connecting to spm-receiver.sematext.com / spm-receiver.eu.sematext.com (if using Sematext Cloud Europe) with the following
command:

``` bash
nc -zv -w 20 spm-receiver.sematext.com 443
```

The output should show something like: Connection to
spm-receiver.sematext.com 443 port
\[tcp/https\] succeeded\!

In case you see some other result:

  - if your server requires proxy to access the internet, you can define
    its settings using `/opt/spm/bin/setup-env` command. After
    that restart SPM agent.
  - if firewall is used to protect your server, it may be blocking
    outbound traffic from it. SPM agent sends the data over port 443, so
    please ensure with your network admins that port 443 is open for
    outbound traffic
  - check your DNS (see below)

2\. Check if your DNS has correct entries for SPM Receiver:

``` bash
nslookup spm-receiver.sematext.com
nslookup spm-receiver.eu.sematext.com
```

The output of this command should look like this, although the IP
addresses and names may be somewhat different, as they change
periodically:

```
Server:        127.0.1.1
Address:    127.0.1.1#53

Non-authoritative answer:
spm-receiver.sematext.com    canonical name = SPM-Prod-Receiver-LB-402293491.us-east-1.elb.amazonaws.com.
Name:    SPM-Prod-Receiver-LB-402293491.us-east-1.elb.amazonaws.com
Address: 50.16.206.179
Name:    SPM-Prod-Receiver-LB-402293491.us-east-1.elb.amazonaws.com
Address: 107.20.222.136
```

If you see different output, you may want to check with your network
admins if everything with your DNS is ok or may need adjustment to be
able to reach spm-receiver.sematext.com / spm-receiver.eu.sematext.com.

**I don't see any of my metrics / I am getting errors when starting SPM monitor or when starting my service which uses javaagent version of SPM monitor.  What should I check?**

Here are a few things to check and do:

1.  Log into your monitored servers and make sure there are running SPM
    monitor processes (there should be more than one of them)

2.  Check if system time is correct. If not, you should adjust the time,
    restart the SPM monitor with:  
    
        sudo service spm-monitor restart
    
    and restart any other javaagent (in-process) based SPM monitors by
    restarting your server which is being monitored.

3.  Check network connectivity as described elsewhere in the FAQ

4.  Make sure disks are not full

5.  Make sure user spmmon can have more than 1024 files open:  

```
sudo vim /etc/security/limits.conf
spmmon     -    nofile    32000
```
and
```    
sudo vim /etc/pam.d/su
session    required   pam_limits.so
```
    
Restart SPM monitor after the above changes.

6.  Check if hostname of your server is defined in `/etc/hosts`

7.  If you are starting your Jetty (or some other server) with command
    like `java ... -jar start.jar ...` and using inprocess (javaagent)
    version of monitor, make sure -D and -javaagent definitions occur
    before "-jar start.jar" part in your command

8.  If none of the suggestions helped, run ```sudo bash
    /opt/spm/bin/spm-client-diagnostics.sh``` to generate diagnostics
    package and send it to <support@sematext.com>

**My server stopped sending metrics, so why do I still see it under Hosts Filter?**

Filters have 2 hour granularity, which means that a server will be
listed under Hosts filter until 2 hours since it last sent data have
passed. For example, if a server stopped sending data at 1 PM and if at
2:30 PM you are looking at the last 1 hour of data (for a period from 1:30 PM
until 2:30 PM) you will not see data from this server on the graph, but you
will still see this server listed under the Hosts filter until 3 PM.
After 3 PM this server should disappear from the Hosts filter.

**I rebooted my server and now I don't see any data in my graphs. What should I check?**

Here are a few things to check and do:  

1.  Make sure SPM monitor is running: sudo service spm-monitor restart
2.  Make sure disk is not full: `df -h`
3.  Make sure maximal open files limit was not reached: see **"I
    registered for SPM more than 5 minutes ago and I don't see any of my
    data, what should I check?"**

**How come Disk Space Usage report shows more free disk space than df command?**

Sematext Monitoring reports both free and reserved disk space
as free, while df does not include reserved disk space by default.

**I changed my server's hostname and now I don't see new data in my graphs. What should I do?**

Simply restart SPM monitor:

``` bash
sudo service spm-monitor restart
```

and restart any process which is using javaagent/in-process version of
SPM monitor.

**Can I specify which Java runtime the SPM monitor should use?**

Yes, you can edit the `/opt/spm/properties/java.properties` file
where you can specify the location of Java you want the SPM monitor to
use.

**Can SPM client use HTTP instead of HTTPS to send metrics from my servers?**

Yes, although we recommend using HTTPS.

Sematext agents by default uses HTTPS to send metrics data to Sematext. If you
prefer to use HTTP instead (for example, if you are running Sematext
Enterprise on premises or if you don't need metric data to be encrypted
when being sent to Sematext over the Internet), you can adjust that in
`/opt/spm/properties/agent.properties` by changing protocol to
**http** in property:

``` properties
server_base_url=https://spm-receiver.sematext.com
```

### Security

**What information are SPM agents sending?**

SPM agents are sending metrics and data used for filtering those
metrics, such as hostnames (which can be obfuscated).  To see what
exactly is being sent you can use tcpdump or a similar tool to sniff the
traffic.  SPM agents ship data via HTTPS, but can also ship it via HTTP.
 Java-based agents let you change the protocol in appropriate files
under `/opt/spm/properties/` directory, while Node.js-based agents
let you change it via the `SPM_RECEIVER_URL` environmental variable.

**Can hostnames in Sematext Monitoring be obfuscated or customized?**

Yes, you can obfuscate or alias hostnames. This lets you:

  - never send your real hostnames over the internet
  - use custom hostname in Sematext UI in case original hostname is too
    cryptic (e.g. have Sematext show "my-solr-host1" instead of
    "ip-12-123-321-123")

To achieve this, after SPM client is installed, open
`/opt/spm/properties/agent.properties` file and just add desired
hostname to the value of property `hostname_alias`, e.g.

``` properties
hostname_alias=web1.production.
```

After that, restart SPM monitor with:

```sudo service spm-monitor restart```

and restart any Java process which was using javaagent/in-process
version of SPM monitor.

Note:

  - old data will still be seen in Sematext under the old hostname, while new
    data (after hostname change) will be displayed under the new
    hostname
  - if you are installing SPM client for the first time
    and you want to be 100% sure its original hostname never leaves your
    network, define your hostname alias in `agent.properties`
    file immediately after you complete "1. Package installation" step
    and before you begin with "2. Client configuration setup" step
    (installation instructions can be accessed from
    <https://apps.sematext.com/ui/monitoring>, click Actions \> Install
    Monitor on app you are installing)

### Billing

**How do you bill for infrastructure and server monitoring?**

Usage is metered hourly on a per-agent basis. For example:

If you send metrics from a server A to Monitoring App Foo between 01:00 and
02:00 that's $0.035 for the Standard plan.

If another agent is monitoring something else, even if that is running on the same
server A, and sending metrics to a different Monitoring App Bar, that's another $0.035.

If you are not sending metrics from a server A for a Monitoring App Foo between
02:00 and 03:00 then you pay $0 for that hour.

A single agent monitoring 24/7 will end up being ~ $25/month.  If you
run another agent on another server it will be 2 x ~ $25/mo.

**How do you bill for Docker container monitoring?**

Docker monitoring is based on the base price and per-container
price.  The base price includes monitoring of a Docker host and free
monitoring of up to N containers. Per-container price is applied only if
you run more than N containers per host.  The number of containers per
host is averaged for the whole account.  The base price and the number
of containers included in it depends on the plan.  Note that monitoring
of Docker host and containers is independent of monitoring of
applications you run in those containers.  Containerized applications
monitored by Sematext are metered as separate hosts. In other words, whether
the monitored application is running in a container or in a VM or
directly on a server or in a public cloud instance is the same as far as
metering and billing is concerned. For plans and price details
see <https://sematext.com/spm/pricing>.

**Which credit cards are accepted?**

See [billing FAQ](/faq/#billing).

**Can I be invoiced instead of paying with a credit card?**

See [billing FAQ](/faq/#billing).

**How often will I get billed?**

See [billing FAQ](/faq/#billing).

**Can the billing email be sent to our Accounts Payable/Accounting instead of me?**

See [billing FAQ](/faq/#billing).

**Do I have to commit or can I stop using Sematext at any time?**

See [billing FAQ](/faq/#billing).

**Can I get invoices?**

See [billing FAQ](/faq/#billing).