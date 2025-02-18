# SPLUNK

## Version Matrix
| version | artifactory | xray   | distribution | mission_control | pipelines | splunk                    |
|---------|-------------|--------|--------------|-----------------|-----------|---------------------------|
| 0.9.1   | 7.12.6      | 3.15.3 | 2.6.0        | 4.6.2           | 1.10.0    | 8.0.5 Build: a1a6394cc5ae |
| 0.9.0   | 7.11.5      | 3.15.1 | 2.6.0        | 4.6.2           | 1.10.0    | 8.0.5 Build: a1a6394cc5ae |
| 0.8.0   | 7.10.2      | 3.10.3 | 2.4.2        | 4.5.0           | 1.8.0     | 8.0.5 Build: a1a6394cc5ae |
| 0.7.0   | 7.9.1       | 3.9.1  | 2.4.2        | 4.5.0           | 1.8.0     | 8.0.5 Build: a1a6394cc5ae |
| 0.6.0   | 7.7.8       | 3.8.6  | 2.4.2        | 4.5.0           | 1.7.2     | 8.0.5 Build: a1a6394cc5ae |
| 0.5.0   | 7.7.3       | 3.8.0  | 2.4.2        | 4.5.0           | 1.7.2     | 8.0.5 Build: a1a6394cc5ae |
| 0.4.0   | 7.7.3       | 3.8.0  | 2.4.2        | 4.5.0           | N/A       | 8.0.5 Build: a1a6394cc5ae |
| 0.3.0   | 7.7.3       | 3.8.0  | 2.4.2        | N/A             | N/A       | 8.0.5 Build: a1a6394cc5ae |
| 0.2.0   | 7.7.3       | 3.8.0  | N/A          | N/A             | N/A       | 8.0.5 Build: a1a6394cc5ae |
| 0.1.1   | 7.6.3       | 3.6.2  | N/A          | N/A             | N/A       | 8.0.5 Build: a1a6394cc5ae |

## Table of Contents
`Note! You must follow the order of the steps throughout Splunk Configuration`

1. [Splunk Setup](#splunk-setup)
2. [Environment Configuration](#environment-configuration)
3. [Fluentd Installation](#fluentd-installation)
    * [OS / Virtual Machine](#os--virtual-machine)
    * [Docker](#docker)
    * [Kubernetes Deployment with Helm](#kubernetes-deployment-with-helm)
    * [Kubernetes Deployment without Helm](#kubernetes-deployment-without-helm)
4. [Fluentd Configuration for Splunk](#fluentd-configuration-for-splunk)
    * [Configuration steps for Artifactory](#configuration-steps-for-artifactory)
    * [Configuration steps for Xray](#configuration-steps-for-xray)
    * [Configuration steps for Mission Control](#configuration-steps-for-mission-control)
    * [Configuration steps for Distribution](#configuration-steps-for-distribution)
    * [Configuration steps for Pipelines](#configuration-steps-for-pipelines)
5. [Dashboards](#dashboards)
6. [Splunk Demo](#splunk-demo)
7. [References](#references)

## Splunk Setup

### Splunkbase App

Install the `JFrog Log Analytics Platform` app from Splunkbase [here!](https://splunkbase.splunk.com/app/5023/)

````text
1. Download file from Splunkbase
2. Open Splunk web console as adminstrator
3. From homepage click on settings wheel in top right of Apps section
4. Click on "Install app from file"
5. Select download file from Splunkbase on your computer
6. Click upgrade 
7. Click upload
````

Restart Splunk post installation of App.

````text 
1. Open Splunk web console as adminstrator
2. Click on Settings then Server Controls
3. Click on Restart 
````

Login to Splunk after the restart completes.

Confirm the version is the latest version available in Splunkbase.

### Configure Splunk

Our integration uses the [Splunk HEC](https://dev.splunk.com/enterprise/docs/dataapps/httpeventcollector/) to send data to Splunk.

Users will need to configure the HEC to accept data (enabled) and also create a new token. Steps are below.

#### Create index jfrog_splunk
````text
1. Open Splunk web console as adminstrator
2. Click on "Settings" in dropdown select "Indexes"
3. Click on "New Index"
4. Enter Index name as jfrog_splunk
5. Click "Save"
````

#### Configure new HEC token to receive Logs
````text
1. Open Splunk web console as adminstrator
2. Click on "Settings" in dropdown select "Data inputs"
3. Click on "HTTP Event Collector"
4. Click on "New Token"
5. Enter a "Name" in the textbox
6. (Optional) Enter a "Description" in the textbox
7. Click on the green "Next" button
8. Select App Context of "JFrog Platform Log Analytics" in the dropdown
9. Add "jfrog_splunk" index to store the JFrog platform log data into.
10. Click on the green "Review" button
11. If good, Click on the green "Done" button
12. Save the generated token value
````

## Environment Configuration

We rely heavily on environment variables so that the correct log files are streamed to your observalibity dashboards. Ensure that you set the JF_PRODUCT_DATA_INTERNAL environment variable to the correct path for your product

The environment variable JF_PRODUCT_DATA_INTERNAL must be defined to the correct location.

Helm based installs will already have this defined based upon the underlying docker images.

For non-k8s based installations below is a reference to the Docker image locations per product. Note these locations may be different based upon the installation location chosen.

````text
Artifactory: 
export JF_PRODUCT_DATA_INTERNAL=/var/opt/jfrog/artifactory/
````

````text
Xray:
export JF_PRODUCT_DATA_INTERNAL=/var/opt/jfrog/xray/
````

````text
Mision Control:
export JF_PRODUCT_DATA_INTERNAL=/var/opt/jfrog/mc/
````

````text
Distribution:
export JF_PRODUCT_DATA_INTERNAL=/var/opt/jfrog/distribution/
````

````text
Pipelines:
export JF_PRODUCT_DATA_INTERNAL=/opt/jfrog/pipelines/var/
````

## Fluentd Installation

### OS / Virtual Machine

Ensure you have access to the Internet from VM. Recommended install is through fluentd's native OS based package installs:

| OS            | Package Manager | Link |
|---------------|-----------------|------|
| CentOS/RHEL   | RPM (YUM)       | https://docs.fluentd.org/installation/install-by-rpm |
| Debian/Ubuntu | APT             | https://docs.fluentd.org/installation/install-by-deb |
| MacOS/Darwin  | DMG             | https://docs.fluentd.org/installation/install-by-dmg |
| Windows       | MSI             | https://docs.fluentd.org/installation/install-by-msi |

User installs can utilize the zip installer for Linux

| OS            | Package Manager | Link |
|---------------|-----------------|------|
| Linux (x86_64)| ZIP             | https://github.com/jfrog/log-analytics/raw/master/fluentd-installer/fluentd-1.11.0-linux-x86_64.tar.gz |

Download it to a directory the user has permissions to write such as the `$JF_PRODUCT_DATA_INTERNAL` locations discussed above in the [Environment Configuration](#environment-configuration) section.

````text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://github.com/jfrog/log-analytics/raw/master/fluentd-installer/fluentd-1.11.0-linux-x86_64.tar.gz
````

Untar to create the folder:
````text
tar -xvf fluentd-1.11.0-linux-x86_64.tar.gz
````
Move into the new folder:

````text
cd fluentd-1.11.0-linux-x86_64
````

Configure `fluent.conf.*` according to the instructions mentioned in [Fluentd Configuration for Splunk](#fluentd-configuration-for-splunk) section and then run the fluentd wrapper with one argument pointed to the `fluent.conf.*` file configured.

````text
./fluentd $JF_PRODUCT_DATA_INTERNAL/fluent.conf.<product_name>
````

### Docker

Recommended install for Docker is to utilize the zip installer for Linux

| OS            | Package Manager | Link |
|---------------|-----------------|------|
| Linux (x86_64)| ZIP             | https://github.com/jfrog/log-analytics/raw/master/fluentd-installer/fluentd-1.11.0-linux-x86_64.tar.gz |

Download it to a directory the user has permissions to write such as the `$JF_PRODUCT_DATA_INTERNAL` locations discussed above in the [Environment Configuration](#environment-configuration) section.

````text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://github.com/jfrog/log-analytics/raw/master/fluentd-installer/fluentd-1.11.0-linux-x86_64.tar.gz
````

Untar to create the folder:
````text
tar -xvf fluentd-1.11.0-linux-x86_64.tar.gz
````
Move into the new folder:

````text
cd fluentd-1.11.0-linux-x86_64
````

Configure `fluent.conf.*` according to the instructions mentioned in [Fluentd Configuration for Splunk](#fluentd-configuration-for-splunk) section and then run the fluentd wrapper with one argument pointed to the `fluent.conf.*` file configured.

````text
./fluentd $JF_PRODUCT_DATA_INTERNAL/fluent.conf.<product_name>
````

### Kubernetes Deployment with Helm

Recommended installation for Kubernetes is to utilize the helm chart with the associated values.yaml in this repo.

| Product | Example Values File |
|---------|-------------|
| Artifactory | helm/artifactory-values.yaml |
| Artifactory HA | helm/artifactory-ha-values.yaml |
| Xray | helm/xray-values.yaml |

Update the values.yaml associated to the product you want to deploy with your Splunk settings.

Then deploy the helm chart as described below:

Add JFrog Helm repository:

```text
helm repo add jfrog https://charts.jfrog.io
helm repo update
```

Replace placeholders with your ``masterKey`` and ``joinKey``. To generate each of them, use the command
``openssl rand -hex 32``

Artifactory ⎈:

Replace the `splunk.example.com` in `splunk.host` at the end of the yaml file with IP address or DNS of Splunk HEC

Replace the `splunk_hec_token` in `splunk.token` at the end of the yaml file with the HEC Token created in [Configure new HEC token to receive Logs](#configure-new-hec-token-to-receive-logs) section and then run the following helm command:

```text
helm upgrade --install artifactory-ha  jfrog/artifactory-ha \
       --set artifactory.masterKey=FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF \
       --set artifactory.joinKey=EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE \
       -f helm/artifactory-values.yaml
```

Artifactory-HA ⎈:

For HA installation, please create a license secret on your cluster prior to installation.

```text
kubectl create secret generic artifactory-license --from-file=<path_to_license_file>artifactory.cluster.license 
```

Note: Replace placeholders with your ``masterKey`` and ``joinKey``. To generate each of them, use the command
``openssl rand -hex 32``

Replace the `splunk.example.com` in `splunk.host` at the end of the yaml file with splunk IP address or DNS of Splunk HEC

Replace the `splunk_hec_token` in `splunk.token` at the end of the yaml file with the HEC Token created in [Configure new HEC token to receive Logs](#configure-new-hec-token-to-receive-logs) section and then run the following helm command:

```text
helm upgrade --install artifactory-ha  jfrog/artifactory-ha \
       --set artifactory.masterKey=FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF \
       --set artifactory.joinKey=EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE \
       -f helm/artifactory-ha-values.yaml
```

Xray ⎈:

Update the following fields in `/helm/xray-values.yaml`:

Replace the `splunk.example.com` in `splunk.host` at the end of the yaml file with splunk IP address or DNS of Splunk HEC

Replace the `splunk_hec_token` in `splunk.token` at the end of the yaml file with the HEC Token created in [Configure new HEC token to receive Logs](#configure-new-hec-token-to-receive-logs) section

Replace `jfrog_user` in `jfrog.siem.username` with Artifactory username 

Replace `jfrog_api_key` in `jfrog.siem.apikey` with [Artifactory API Key](https://www.jfrog.com/confluence/display/JFROG/User+Profile#UserProfile-APIKey) and then run the following helm command:

Use the same `joinKey` as you used in Artifactory installation to allow Xray node to successfully connect to Artifactory.

```text
helm upgrade --install xray jfrog/xray --set xray.jfrogUrl=http://my-artifactory-nginx-url \
       --set xray.masterKey=FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF \
       --set xray.joinKey=EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE \
       -f helm/xray-values.yaml
```

### Kubernetes Deployment without Helm

To modify existing Kubernetes based deployments without using Helm, users can use the zip installer for Linux:

| OS            | Package Manager | Link |
|---------------|-----------------|------|
| Linux (x86_64)| ZIP             | https://github.com/jfrog/log-analytics/raw/master/fluentd-installer/fluentd-1.11.0-linux-x86_64.tar.gz |

Download it to a directory the user has permissions to write such as the `$JF_PRODUCT_DATA_INTERNAL` locations discussed above in the [Environment Configuration](#environment-configuration) section.

````text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://github.com/jfrog/log-analytics/raw/master/fluentd-installer/fluentd-1.11.0-linux-x86_64.tar.gz
````

Untar to create the folder:
````text
tar -xvf fluentd-1.11.0-linux-x86_64.tar.gz
````
Move into the new folder:

````text
cd fluentd-1.11.0-linux-x86_64
````
Configure `fluent.conf.*` according to the instructions mentioned in [Fluentd Configuration for Splunk](#fluentd-configuration-for-splunk) section and then run the fluentd wrapper with one argument pointed to the `fluent.conf.*` file configured.

````text
./fluentd $JF_PRODUCT_DATA_INTERNAL/fluent.conf.<product_name>
````

## Fluentd Configuration for Splunk

Download and configure the relevant fluentd.conf files for Splunk

### Configuration steps for Artifactory

Download the artifactory fluentd configuration file to a directory the user has permissions to write, such as the $JF_PRODUCT_DATA_INTERNAL locations discussed above in the [Environment Configuration](#environment-configuration) section.

````text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://raw.githubusercontent.com/jfrog/log-analytics-splunk/master/fluent.conf.rt
````

Override the match directive(last section) of the downloaded `fluent.conf.rt` with the details given below

```
<match jfrog.**>
  @type splunk_hec
  host HEC_HOST
  port HEC_PORT
  token HEC_TOKEN
  index jfrog_splunk
  format json
  # buffered output parameter
  flush_interval 10s
  # ssl parameter
  use_ssl false
  ca_file /path/to/ca.pem
</match>
```

_**required**_: ```HEC_HOST``` is the IP address or DNS of Splunk HEC

_**required**_: ```HEC_PORT``` is the Splunk HEC port which by default is 8088

_**required**_: ```HEC_TOKEN``` is the saved generated token from [Configure new HEC token to receive Logs](#configure-new-hec-token-to-receive-logs)

If ssl is enabled, ca file will be used and must be supplied

### Configuration steps for Xray

Download the Xray fluentd configuration file to a directory the user has permissions to write, such as the $JF_PRODUCT_DATA_INTERNAL locations discussed above in the [Environment Configuration](#environment-configuration) section.

````text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://raw.githubusercontent.com/jfrog/log-analytics-splunk/master/fluent.conf.xray
````

Fill in the JPD_URL, USER, JFROG_API_KEY fields in the source directive of the downloaded `fluent.conf.xray` with the details given below

```text
<source>
  @type jfrog_siem
  tag jfrog.xray.siem.vulnerabilities
  jpd_url JPD_URL
  username USER
  apikey JFROG_API_KEY
  pos_file "#{ENV['JF_PRODUCT_DATA_INTERNAL']}/log/jfrog_siem.log.pos"
</source>
```

_**required**_: ```JPD_URL``` is the Artifactory JPD URL of the format `http://<ip_address>` with is used to pull Xray Violations

_**required**_: ```USER``` is the Artifactory username for authentication

_**required**_: ```JFROG_API_KEY``` is the [Artifactory API Key](https://www.jfrog.com/confluence/display/JFROG/User+Profile#UserProfile-APIKey) for authentication

Override the match directive (last section) of the downloaded `fluent.conf.xray` with the details given below

```
<match jfrog.**>
  @type splunk_hec
  host HEC_HOST
  port HEC_PORT
  token HEC_TOKEN
  index jfrog_splunk
  format json
  # buffered output parameter
  flush_interval 10s
  # ssl parameter
  use_ssl false
  ca_file /path/to/ca.pem
</match>
```

_**required**_: ```HEC_HOST``` is the IP address or DNS of Splunk HEC

_**required**_: ```HEC_PORT``` is the Splunk HEC port which by default is 8088

_**required**_: ```HEC_TOKEN``` is the saved generated token from [Configure new HEC token to receive Logs](#configure-new-hec-token-to-receive-logs)

If ssl is enabled, ca file will be used and must be supplied

### Configuration steps for Mission Control
``
Download the Mission Control fluentd configuration file to a directory the user has permissions to write, such as the $JF_PRODUCT_DATA_INTERNAL locations discussed above in the [Environment Configuration](#environment-configuration) section.

````text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://raw.githubusercontent.com/jfrog/log-analytics-splunk/master/fluent.conf.missioncontrol
````

Override the match directive(last section) of the downloaded `fluent.conf.missioncontrol` with the details given below

```
<match jfrog.**>
  @type splunk_hec
  host HEC_HOST
  port HEC_PORT
  token HEC_TOKEN
  index jfrog_splunk
  format json
  # buffered output parameter
  flush_interval 10s
  # ssl parameter
  use_ssl false
  ca_file /path/to/ca.pem
</match>
```

_**required**_: ```HEC_HOST``` is the IP address or DNS of Splunk HEC

_**required**_: ```HEC_PORT``` is the Splunk HEC port which by default is 8088

_**required**_: ```HEC_TOKEN``` is the saved generated token from [Configure new HEC token to receive Logs](#configure-new-hec-token-to-receive-logs)

If ssl is enabled, ca file will be used and must be supplied

### Configuration steps for Distribution

Download the distribution fluentd configuration file to a directory the user has permissions to write, such as the $JF_PRODUCT_DATA_INTERNAL locations discussed above in the [Environment Configuration](#environment-configuration) section.

````text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://raw.githubusercontent.com/jfrog/log-analytics-splunk/master/fluent.conf.distribution
````

Override the match directive(last section) of the downloaded `fluent.conf.distribution` with the details given below

```
<match jfrog.**>
  @type splunk_hec
  host HEC_HOST
  port HEC_PORT
  token HEC_TOKEN
  index jfrog_splunk
  format json
  # buffered output parameter
  flush_interval 10s
  # ssl parameter
  use_ssl false
  ca_file /path/to/ca.pem
</match>
```

_**required**_: ```HEC_HOST``` is the IP address or DNS of Splunk HEC

_**required**_: ```HEC_PORT``` is the Splunk HEC port which by default is 8088

_**required**_: ```HEC_TOKEN``` is the saved generated token from [Configure new HEC token to receive Logs](#configure-new-hec-token-to-receive-logs)

If ssl is enabled, ca file will be used and must be supplied

### Configuration steps for Pipelines

Download the pipelines fluentd configuration file to a directory the user has permissions to write, such as the $JF_PRODUCT_DATA_INTERNAL locations discussed above in the [Environment Configuration](#environment-configuration) section.

````text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://raw.githubusercontent.com/jfrog/log-analytics-splunk/master/fluent.conf.pipelines
````

Override the match directive(last section) of the downloaded `fluent.conf.pipelines` with the details given below

```
<match jfrog.**>
  @type splunk_hec
  host HEC_HOST
  port HEC_PORT
  token HEC_TOKEN
  index jfrog_splunk
  format json
  # buffered output parameter
  flush_interval 10s
  # ssl parameter
  use_ssl false
  ca_file /path/to/ca.pem
</match>
```

_**required**_: ```HEC_HOST``` is the IP address or DNS of Splunk HEC

_**required**_: ```HEC_PORT``` is the Splunk HEC port which by default is 8088

_**required**_: ```HEC_TOKEN``` is the saved generated token from [Configure new HEC token to receive Logs](#configure-new-hec-token-to-receive-logs)

If ssl is enabled, ca file will be used and must be supplied

## Dashboards

### Artifactory dashboard
JFrog Artifactory Dashboard is divided into three sections Application, Audit, Requests and Docker

* **Application** - This section tracks Log Volume(information about different log sources) and Artifactory Errors over time(bursts of application errors that may otherwise go undetected)
* **Audit** - This section tracks audit logs help you determine who is accessing your Artifactory instance and from where. These can help you track potentially malicious requests or processes (such as CI jobs) using expired credentials.
* **Requests** - This section tracks HTTP response codes, Top 10 IP addresses for uploads and downloads
* **Docker** - To monitor Dockerhub pull requests users should have a Dockerhub account either paid or free. Free accounts allow up to 200 pull requests per 6 hour window. Various widgets have been added in the new Docker tab under Artifactory to help monitor your Dockerhub pull requests. An alert is also available to enable if desired that will allow you to send emails or add outbound webhooks through configuration to be notified when you exceed the configurable threshold.

### Xray dashboard
JFrog Xray Dashboard is divided into two sections Logs and Violations

* **Logs** - This dashboard provides a summary of access, service and traffic log volumes associated with Xray. Additionally, customers are also able to track various HTTP response codes, HTTP 500 errors, and log errors for greater operational insight
* **Violations** - This dashboard provides an aggregated summary of all the license violations and security vulnerabilities found by Xray.  Information is segment by watch policies and rules.  Trending information is provided on the type and severity of violations over time, as well as, insights on most frequently occurring CVEs, top impacted artifacts and components.  

### CIM Compatibility

Log data from JFrog platform logs is translated to pre-defined Common Information Models (CIM) compatible with Splunk. This compatibility enables new advanced features where users can search and access JFrog log data that is compatible with data models. For example

```text
| datamodel Web Web search
| datamodel Change_Analysis All_Changes search
| datamodel Vulnerabilities Vulnerabilities search
```

## Splunk Demo

To run this integration for Splunk users can create a Splunk instance with the correct ports open in Kubernetes by applying the yaml file:

``` 
kubectl apply -f k8s/splunk.yaml
```

This will create a new Splunk instance you can use for a demo to send your JFrog logs over to.

Once they have a Splunk up for demo purposes they will need to configure the HEC and then update fluent config files with the relevant parameters for HEC_HOST, HEC_PORT, & HEC_TOKEN.

They can now access Splunk to view the JFrog dashboard as new data comes in.

## References

* [Fluentd](https://www.fluentd.org) - Fluentd Logging Aggregator/Agent
* [Splunk](https://www.splunk.com/) - Splunk Logging Platform
* [Splunk HEC](https://dev.splunk.com/enterprise/docs/dataapps/httpeventcollector/) - Splunk HEC used to upload data into Splunk
