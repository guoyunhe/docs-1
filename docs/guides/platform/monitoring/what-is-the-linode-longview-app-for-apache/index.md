---
slug: what-is-the-linode-longview-app-for-apache
author:
  name: Linode
  email: docs@linode.com
description: Using the Linode Longview App for Apache
keywords: ["Longview", " Apache", " statistics", " mod\\_status"]
tags: ["apache","cloud manager","statistics","monitoring","linode platform"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
aliases: ['/longview/longview-for-apache/','/platform/longview/what-is-the-linode-longview-app-for-apache/','/platform/longview/longview-app-for-apache/']
modified: 2018-12-26
modified_by:
  name: Linode
published: 2013-11-04
title: What is the Linode Longview App for Apache
h1_title: Using the Linode Longview App for Apache
enable_h1: true
classic_manager_link: platform/longview/longview-app-for-apache-classic
external_resources:
 - '[cPanel Products News](https://news.cpanel.com/category/products/)'
 - '[cPanel Security News](https://news.cpanel.com/category/security/)'
 - '[Unix Toolbox](http://cb.vu/unixtoolbox.xhtml)'
relations:
    platform:
        key: what-is-longview
        keywords:
            - distribution: Apache
---

Longview for Apache is a Longview App. The Longview Apache tab appears in the Linode Cloud Manager when Longview detects that you have Apache installed on your Linode. With the Longview Apache App, you'll be able to view statistics for Apache on your Linode. It can help you keep track of Apache's settings, workers and requests, system resource consumption, and other information.

## In this Guide:

This guide discusses Linode Longview for Apache. The guide covers the following topics:

- [Configuring Linode Longview for Apache](#debian-and-ubuntu-automatic-configuration).
- [Interacting with the Apache data provided by Longview in the Linode Cloud Manager](#viewing-statistics).
- [Troubleshooting Linode Longview for Apache](#troubleshooting).

## Before you Begin

In order to use the Longview App for Apache, ensure you have completed the following things:

-  A Linode with [Apache installed and running](/docs/web-servers/apache/).
-  Create a [Longview client](/docs/platform/longview/what-is-longview/#install-linode-longview) instance using the Cloud Manager.
-  Install the [Longview Agent](/docs/platform/longview/what-is-longview/#install-the-longview-agent) on your Linode.

### Debian and Ubuntu Automatic Configuration

If Apache is installed and running when you install the Longview client, the Apache App should enable and configure itself automatically.

If you already have Longview installed, and later want to install Apache and enable the Longview App for it, you may find that Apache is not automatically detected on initial setup. When this happens, you can run Longview through its automatic configuration sequence again. In most cases, this will result in Longview finding everything it needs to get the Apache App started. And don't worry - your old Longview data will stay safe. To run the automatic Longview configuration:

1. [SSH into your Linode](/docs/getting-started/#connect-to-your-linode-via-ssh) whose system you are monitoring with Longview.

1.  Ensure that Apache is running using your distribution's initialization system.

    > **CentOS, Debian, and Ubuntu**
    >
    >     sudo systemctl status apache2   # For distributions with systemd.

    >  **Other Distributions**
    >
    >     sudo service apache2 status     # For distributions without systemd.

1.  Reconfigure the Apache Longview App:

        dpkg-reconfigure -phigh linode-longview

    For most people, Longview should be able to configure itself automatically, and you will receive output something like this:

    {{< output >}}
[ ok ] Stopping Longview Agent: longview.
Checking Apache configuration...
Found Apache status page at http://127.0.0.1/server-status?auto (default URL)
[ ok ] Starting Longview Agent: longview.
update-rc.d: using dependency based boot sequencing
    {{</ output >}}

    Once you see this successful message, the Longview Apache App should automatically start collecting Apache data. Refresh Longview in the Linode Cloud Manager to start viewing your Apache stats for your Longview Client instance.

    If you receive a failure message or the popup shown below, you should visit the [Troubleshooting](#troubleshooting) section at the end of this article.

[![Longview has detected Apache running on this server but was unable to access the server status page. Would you like to attempt to automatically configure mod\_status? This will require reloading Apache to enable. Autoconfigure Mod\_Status: \<Yes\> \<No\>](1451-longview_apache_popup_crop.png)](1451-longview_apache_popup_crop.png)

### Manual Configuration (All Distributions)

To enable the Apache Longview app manually, follow these steps on your Linode via SSH:

1.  [SSH into your Linode](/docs/getting-started/#connect-to-your-linode-via-ssh) whose system you are monitoring with Longview.

1.  Make sure **mod\_status** is enabled for Apache (it should be by default). You can follow the instructions list in the [apache.org](https://httpd.apache.org/docs/2.4/mod/mod_status.html) site. Or, on Debian and Ubuntu systems, run this command:

        sudo a2enmod status

1.  Update your Apache configuration file to include the block in the example file below. Depending on your Linux distribution and version, your Apache configuration file may be stored in one of the following locations:

    - /etc/apache2/httpd.conf
    - /etc/apache2/apache2.conf
    - /etc/httpd/httpd.conf
    - /etc/httpd/conf/httpd.conf

    {{< file "httpd.conf or apache2.conf" apache >}}
<IfModule mod_status.c>
    ExtendedStatus On
    <Location /server-status>
        SetHandler server-status
        Require local
    </Location>
</IfModule>

{{< /file >}}

1.  Edit `/etc/linode/longview.d/Apache.conf` to look like the following:

    {{< file "/etc/linode/longview.d/Apache.conf" apache >}}
location http://127.0.0.1/server-status?auto

{{< /file >}}

1.  Restart Apache:

    >**Debian and Ubuntu**:
    >
    >     sudo systemctl restart apache2

    >**Other Distributions**:
    >
    >     sudo apachectl restart

1.  Restart Longview:

    > **CentOS, Debian, and Ubuntu**
    >
    >     sudo systemctl restart longview

    >  **Other Distributions**
    >
    >     sudo service longview restart

1.  Refresh the Longview in your Linode Cloud Manager.

You should now be able to see Longview data for Apache. If that's not the case, proceed to the [Troubleshooting](#troubleshooting) section at the end of this article.

## Viewing Statistics

To see the output for the Longview Apache App:

1.  Log in to the [Linode Cloud Manager](https://cloud.linode.com/).
2.  Select the **Longview** link in the sidebar.
3.  Select the **Apache** tab.

    ![The Longview Apache App.](longview_apache_stats.png)

    You'll see the current version of Apache listed on the upper left hand side of the page.

    Mouse over a data point to see the exact numbers for that time. With [Longview Pro](/docs/platform/longview/pricing/), you can view older time periods for your data. The next sections cover the Longview Apache App in detail.

### Requests

The **Requests** graph shows the total number of requests Apache handled at the selected time. This is every HTTP and HTTPS request to your Linode.

### Throughput

The **Throughput** graph shows the amount of data that Apache sent and received via web requests at the time selected.

### Workers

The **Workers** graph shows all of the Apache workers at the selected time. The workers are broken down by state:

-   Waiting
-   Starting
-   Reading
-   Sending
-   Keepalive
-   DNS Lookup
-   Closing
-   Logging
-   Finishing
-   Cleanup

### CPU

The **CPU** graph shows the percentage of your Linode's CPU being used by Apache at the selected time. If you want to see the total CPU used instead, check the [Overview tab](/docs/platform/longview/what-is-longview/#overview).

### Memory

The **Memory** graph shows the amount of RAM being used by Apache at the selected time. If you want to see your Linode's total memory use instead, check the [Overview tab](/docs/platform/longview/what-is-longview/#overview).

### Disk IO

The **Disk IO** graph shows the amount of input to and output from the disk caused by Apache at the selected time. To see the total IO instead, visit the [Disks tab](/docs/platform/longview/what-is-longview/#disks).

### Process Count

The **Process Count** graph shows the total number of processes on your Linode spawned by Apache at the selected time. If you want to see more details, and how this stacks up against the total number of processes on your Linode, see the [Processes tab](/docs/platform/longview/what-is-longview/#processes).

## Troubleshooting

If you don't see Longview data for Apache, you'll instead get an error on the page and instructions on how to fix it. As a general tip, you can check the `/var/log/linode/longview.log` file for errors as well. You should also compare your mod\_status configuration file to the example shown in Step 2 of the [Manual Configuration (All Distributions)](#manual-configuration-all-distributions) section of this article.

By default Longview uses port 80 for its automatic configuration. In the event you are experiencing problems you may need to edit the `/etc/apache2/ports.conf` file to use port 8080 or another non-standard port.

### Autoconfigure Mod\_Status Popup

If you run the [automatic Longview configuration tool](#debian-and-ubuntu-automatic-configuration), and get the popup message shown below:

[![Longview has detected Apache running on this server but was unable to access the server status page. Would you like to attempt to automatically configure mod\_status? This will require reloading Apache to enable. Autoconfigure Mod\_Status: \<Yes\> \<No\>](1451-longview_apache_popup_crop.png)](1451-longview_apache_popup_crop.png)

This indicates that Longview can't locate the Apache status page. In turn, this could indicate that either:

1.  The status page is in an unusual and unspecified location.

2.  `mod_status` isn't enabled.

3.  An Apache virtual host setting is interfering with requests to the status page.

4.  Apache itself is misconfigured.

If you choose:

-   **\<No\>**: the Longview tool will quit, and you can do a [manual configuration](#manual-configuration-all-distributions). This is the safer option.
-   **\<Yes\>**: the Longview tool will attempt to enable `mod\_status`, set the status page location, and restart Apache. This option is easier, but has the potential to disrupt your current Apache configuration. If you choose yes, and the configuration is successful, you should see output like the following:

    {{< output >}}
[ ok ] Stopping Longview Agent: longview.
Checking Apache configuration...
Enabling module status.
To activate the new configuration, you need to run:
  service apache2 restart
[....] Reloading web server config: . ok
Apache mod_status enabled
[ ok ] Starting Longview Agent: longview.
update-rc.d: using dependency based boot sequencing
    {{</ output >}}

    Refresh the Longview Apache tab in the Linode Manager to verify that it's working now.

    If instead you receive a failure message, such as:

    {{< output >}}
[FAIL] Reloading web server config: apache2 failed!
    {{</ output >}}

    You will need to double-check your Apache installation, and then do a [manual configuration](#manual-configuration-all-distributions).

### Unable to Access Local Server Status for Apache

This error will state `Unable to access local server status for Apache at <http://example.com/example?auto>: <error>:`. This error occurs when either:

1.  Apache's `mod_status` setting is disabled or has been changed from the default location.

2.  An Apache virtual host configuration is interfering with web requests to the `mod_status` location.

    {{< note >}}
This error occurs when Longview attempts to check the status page `location` listed in `/etc/linode/longview.d/Apache.conf`, or the default page at `127.0.0.1/server-status?auto`, but receives a non-200 HTTP response code. Basically, it means that the status page Longview is expecting is not being returned by the server.
    {{< /note >}}

To fix this, follow these steps:

1.  Make sure Apache is running:

    >**Debian and Ubuntu**:
    >
    >     sudo systemctl restart apache2

    >**Other Distributions**:
    >
    >     sudo apachectl restart

2.  Make sure `mod_status` is enabled. See the [Apache website](https://httpd.apache.org/docs/2.4/mod/mod_status.html#page-header) for details. You can also check the output of the following command:

        apachectl -M

3.  Check the location for `mod_status`. The default location on Debian and Ubuntu systems is `http://127.0.0.1/server-status?auto` on localhost. In the Apache configuration file, this is designated with the lines:

    {{< file "httpd.conf or apache2.conf" apache >}}
<Location /server-status>
    SetHandler server-status

{{< /file >}}

    The `SetHandler server-status` line indicates that this is the location block for mod\_status. The location line itself sets the location.

    ##### On cPanel/WHM

    To direct Longview to the cPanel customized status page, edit the `location` line in `/etc/linode/longview.d/Apache.conf` to match the following:

    {{< file "/etc/linode/longview.d/Apache.conf" apache >}}
location http://localhost/whm-server-status?auto

{{< /file >}}

4.  Longview is designed to check the default location automatically. If you use the default location shown above, you should be done. Refresh the Longview in the Linode Cloud Manager to verify that it's working now.

5.  If you're not using the default location, you need to create a new file, `/etc/linode/longview.d/Apache.conf`, and set the `location` variable to match what you set in the Apache configuration file:

    {{< file "/etc/linode/longview.d/Apache.conf" apache >}}
location http://127.0.0.1/custom/location/path

{{< /file >}}

6.  Determine if an Apache virtual host configuration is interfering with requests to the `mod_status location. Use a tool like `curl` or `wget` to request the server status location:

        curl http://127.0.0.1/server-status?auto

    Observe the output. If the output looks like something other than a simple status page, then you'll have to fix your [Apache virtual host](/docs/web-servers/apache-tips-and-tricks/) configuration.


7.  Restart Longview:

        sudo service longview restart

8.  Refresh the Longview in the Linode Cloud Manager to verify that it's working now.

### The Apache Status Page Doesn't Look Right

This error will state `The Apache status page doesn't look right. Check <http://example.com/example?auto> and investigate any redirects for misconfiguration.` This error occurs when Longview is able to reach the `mod_status` page, but doesn't receive the expected content.

 {{< note >}}
This error occurs when Longview attempts to check the status page, and receives a 200 HTTP response code, but can't scrape the expected status content from the page. That is, the page exists on your Linode, but it doesn't have the right content. If, for example, Longview was to check your website's home page, you would get this error.
{{< /note >}}

To resolve this issue, follow these steps:

1.  Visit the URL shown in the error. See if it directs or redirects you to a page that isn't the Apache status page.
2.  Update your Apache and Longview settings so that they specify the same location:

    -   The **\<Location \>** line in your Apache configuration file
    -   The **location** line in `/etc/linode/longview.d/Apache.conf`

    If neither of these is set, the default location of `http://127.0.0.1/server-status?auto` on localhost will be used.

3.  Make sure there aren't any Apache redirects or other settings that are affecting this page.
4.  Restart Longview:

    > **CentOS, Debian, and Ubuntu**
    >
    >     sudo systemctl restart longview

    >  **Other Distributions**
    >
    >     sudo service longview restart

5.  Refresh the Longview in the Linode Cloud Manager to verify that it's working now.

### Missing Graphs: Enable ExtendedStatus

If some of your Apache graphs are missing, you may see the error `Enable ExtendedStatus in your Apache configuration for throughput and request graphs.`

This indicates that you need to add the following line to your Apache configuration file in the `<IfModule mod_status.c>` section:

{{< file "httpd.conf" apache >}}
ExtendedStatus On

{{< /file >}}


When you've finished modifying the configuration file, restart Apache:

  >**Debian and Ubuntu**:
  >
  >     sudo systemctl restart apache2

  >**Other Distributions**:
  >
  >     sudo apachectl restart

### Apache Tab is Missing

If the Longview Apache tab is missing entirely, this indicates that Apache is either not installed, or has stopped. If you restart Apache, you will be able to see the tab again and view all of your old data.
