# Running grafana-agent on Synology DSM / NAS

This docker compose file can be used to run the
grafana-agent in a container on a Synology NAS.

Tested on a `DS420+` (AMD64) with `DSM 7.2.1`

Disclaimer: Your results might vary, use at own risk, no liability accepted.

## Scenario / Steps

1. Create and login to Free Grafan Cloud
2. Click `Getting Started` button
3. Click `Quickstart` button (with all the OS icons)
4. Click `Linux`

### Step 1. Run the Grafana Agent

Click the button and follow the instructions.

I recommend running the commands on a different linux system, or in WSL.

Once you are done make sure any started agent is stopped, then grab
the generated `grafana-agent.yaml` config file. This will become
the file we need in docker.

#### Modify Install Script (optional)

If you do need to run the script on your NAS directly you'll need to
manually download the `install-linux-binary.sh` script and edit it
to replace `unzip` with `7z x` as unzip is no longer installed.

```diff
--- install-linux-binary.sh     2024-01-20 13:36:08.137819400 +0100
+++ install-linux-binary_nas.sh 2024-01-20 13:35:52.324263900 +0100
@@ -78,7 +78,7 @@
   log '--- Verifying package checksum'
   check_sha

-  unzip "${ASSET_NAME}";
+  7z x "${ASSET_NAME}";
   chmod a+x "${ASSET_NAME}";
 }
```

## Step 2. Make configuration selections

Select `Other Distribution` as your OS and make sure you select
the correct architecture. (I only tested this on AMD64)

## Step 3. Prepare your agent configuration file

Follow all the instructions to update configuration but stop before
it asks you to run/start the grafana-agent.

### Configuration For Docker

Because we're planning to run the agent in docker the following
updates need to be made to configuration:

1. The paths for node_exporter to monitor need to be updated to those mapped
   into the container
2. Log location needs to be updated (if you are collecting logs)


```diff
--- agent_orig.yaml     2024-01-20 13:21:52.078747800 +0100
+++ agent.yaml  2024-01-20 13:20:25.666723500 +0100
@@ -23,6 +23,11 @@
   # For a correct indentation, paste snippets copied from Grafana Cloud at the beginning of the line.
   node_exporter:
     enabled: true
+    # For running in container
+    rootfs_path: /host/root
+    sysfs_path: /host/sys
+    procfs_path: /host/proc
+    udev_data_path: /host/root/run/udev/data
     # disable unused collectors
     disable_collectors:
       - ipvs #high cardinality on kubelet
@@ -82,7 +87,7 @@
         labels:
           instance: 'synnas'
           # For running in container
-          __path__: /var/log/{syslog,messages,*.log}
+          __path__: /var/host-log/{syslog,messages,*.log}
           job: integrations/node_exporter

 metrics:
```

## Start the Agent

Copy the `grafana-agent.yaml` to this folder (where the `compose.yaml` is)
and name it `agent.yaml`.

Review the `compose.yaml` and ensure you know what it does, as it will access
potentially sensitive data about your system.

Install/Start the docker compose in your preferred manner (I prefer [dockge](https://github.com/louislam/dockge))

Ensure you are getting log messages in the console
from the running agent.

## Step 3. (Continued)

With the container successfully started you can now click the
`Test Connection` button and it should succeed.

## Conclusion

It's fairly easy to do but does require some tweaking.

## References

- [Grafana Cloud](https://grafana.com/products/cloud/)
- [Grafana Agent Node Exporter Configuration](https://grafana.com/docs/agent/latest/static/configuration/integrations/node-exporter-config/)
