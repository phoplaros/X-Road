![](img/eu_regional_development_fund_horizontal_div_15.png "European Union | European Regional Development Fund | Investing in your future")

---

# X-Road: Configuration Proxy Manual

Version: 2.1  
Doc. ID: UG-CP

## Version History

|  Date      | Version |  Description                           | Author       |
|------------|---------|----------------------------------------|--------------|
| 26.11.2014 | 1.0     | Initial version                        |              |
| 19.01.2015 | 1.1     | License information added              |              |
| 07.05.2015 | 1.2     | Configuration properties table added   |              |
| 26.05.2015 | 1.3     | Token initialization description added |              |
| 30.06.2015 | 1.4     | Minor corrections done                 |              |
| 09.07.2015 | 1.5     | Repository address updated             |              |
| 20.09.2015 | 2.0     | Editorial changes made                 |              |
| 07.06.2017 | 2.1     | System parameter *signature-algorithm-id* replaced with *signature-digest-algorithm-id* | Kristo Heero |

## Table of Contents

<!-- vim-markdown-toc GFM -->

* [1 Introduction](#1-introduction)
    * [1.1 Target Audience](#11-target-audience)
    * [1.2 X-Road Configuration Proxy](#12-x-road-configuration-proxy)
* [2 Installation](#2-installation)
    * [2.1 Supported Platforms](#21-supported-platforms)
    * [2.2 Reference Data](#22-reference-data)
    * [2.3 Requirements for the Configuration Proxy](#23-requirements-for-the-configuration-proxy)
    * [2.4 Preparing OS](#24-preparing-os)
    * [2.5 Installation](#25-installation)
    * [2.6 Post-Installation Checks](#26-post-installation-checks)
    * [2.7 Installing the Support for Hardware Tokens](#27-installing-the-support-for-hardware-tokens)
* [3 Configuration](#3-configuration)
    * [3.1 Prerequisites](#31-prerequisites)
        * [3.1.1 Security Token Activation](#311-security-token-activation)
        * [3.1.2 User Access Privileges](#312-user-access-privileges)
    * [3.2 General Configuration](#32-general-configuration)
        * [3.2.1 Configuration Structure of the Instances](#321-configuration-structure-of-the-instances)
    * [3.3 Proxy Instance Reference Data](#33-proxy-instance-reference-data)
    * [3.4 Proxy Instance Configuration](#34-proxy-instance-configuration)
    * [3.5 Additional Configuration](#35-additional-configuration)
        * [3.5.1 Changing the Validity Interval](#351-changing-the-validity-interval)
        * [3.5.2 Deleting the Signing Keys](#352-deleting-the-signing-keys)
        * [3.5.3 Changing the Active Signing Key](#353-changing-the-active-signing-key)

<!-- vim-markdown-toc -->

## 1 Introduction

### 1.1 Target Audience

The intended audience of this Manual are X-Road system administrators responsible for installing and using X-Road configuration proxy software.

The document is intended for readers with a moderate knowledge of Linux server management, computer networks, and the X-Road working principles.

### 1.2 X-Road Configuration Proxy

The configuration proxy acts as an intermediary between X-Road servers in the matters of global configuration exchange.

The goal of the configuration proxy is to download an X-Road global configuration from a provided configuration source and further distribute it in a secure way. Optionally, the downloaded global configuration may be modified to suit the requirements of the configuration proxy owner.

The configuration proxy can be configured to mediate several global configurations (from multiple configuration sources).

## 2 Installation

### 2.1 Supported Platforms

The configuration proxy runs on the *Ubuntu Server 14.04 Long-Term Support (LTS)* operating system on a 64-bit platform. The configuration proxy's software is distributed as .deb packages through the official X-Road repository at [http://x-road.eu](http://x-road.eu/).

The software can be installed both on physical and virtualized hardware (of the latter, Xen and Oracle VirtualBox have been tested).

### 2.2 Reference Data

**Note:** The information in empty cells has to be determined before the server’s installation, by the person performing the installation.

**Caution:** Data necessary for the functioning of the operating system is not included.

| Ref  |                                          | Explanation                |
|------|------------------------------------------|----------------------------|
| 1.0  | Ubuntu 14.04, 64bit<br>2GB RAM, 3GB free disk space | Minimum requirements. |
| 1.1  | http://x-road.eu/packages                | X-Road package repository.  |
| 1.2  | http://x-road.eu/packages/xroad_repo.gpg | The repository’s key.       |
| 1.3  | TCP 80                                   | Global configuration distribution.<br>Ports for inbound connections (from the external network to the configuration proxy). |
| 1.4  | TCP 80                                   | Global configuration download.<br>Ports for outbound connections (from the configuration proxy to the external network). |
| 1.5  |                                          | Configuration proxy’s public IP address, NAT address. |

### 2.3 Requirements for the Configuration Proxy

Minimum recommended hardware parameters:

* the server’s hardware (motherboard, CPU, network interface cards, storage system) must be supported by Ubuntu 14.04 in general;
* a 64-bit dual-core Intel, AMD or compatible CPU; AES instruction set support is highly recommended;
* 2 GB RAM;
* a 100 Mbps network interface card;
* if necessary, interfaces for the use of hardware tokens.

Requirements to software and settings:

* an installed and configured Ubuntu 14.04 LTS x86-64 operating system;
* if the configuration proxy is separated from other networks by a firewall and/or NAT, the necessary connections to and from the security server must be allowed (reference data: 1.3; 1.4). The enabling of auxiliary services which are necessary for the functioning and management of the operating system (such as DNS, NTP, and SSH) is outside the scope of this guide;
* if the configuration proxy has a private IP address, a corresponding NAT record must be created in the firewall (reference data: 1.5).

### 2.4 Preparing OS

Set operating system locale. Add following line to file */etc/environment*:  
```bash
LC_ALL=en_US.UTF-8
```

### 2.5 Installation

To install the X-Road configuration proxy software, follow these steps.

1. Add to */etc/apt/sources.list* the address of X-Road package repository (reference data: 1.1) and the nginx repository:
```bash  
deb http://x-road.eu/packages trusty main
deb http://ppa.launchpad.net/nginx/stable/ubuntu trusty main
deb http://ppa.launchpad.net/openjdk-r/ppa/ubuntu trusty main
```
2. Add the X-Road repository’s signing key to the list of trusted keys (reference data: 1.2):
```bash
curl http://x-road.eu/packages/xroad_repo.gpg | sudo apt-key add -
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 00A6F0A3C300EE8C
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys EB9B1D8886F44E2A
```
3.  Issue the following commands to install the configuration proxy packages:
```bash
sudo apt-get update
sudo apt-get install xroad-confproxy
```

### 2.6 Post-Installation Checks

The installation is successful if the 'xroad-signer' service is started, the 'xroad-confproxy' cron job is added, and the configuration proxy management utilities are available from the command line.

* Check from the command line that the 'xroad-signer' service is in the start/running state (example output follows):

```bash
sudo initctl list | grep "^xroad-"

xroad-signer start/running, process 19393
```

* Check from the command line that the 'xroad-confproxy' cron job is present (example output follows):

```bash
sudo ls /etc/cron.d/ | grep "^xroad-"

xroad-confproxy
```

* Make sure that the following commands are available from the command line:

```bash
signer-console
confproxy-view-conf
confproxy-create-instance
confproxy-add-signing-key
confproxy-del-signing-key
confproxy-generate-anchor
```

### 2.7 Installing the Support for Hardware Tokens

Installation of hardware token vendorspecific library and configuration of tokens is not in scope of this documentation. Vendorspecific support for PKCS#11 API must be present before proceeding.

If you intend to use hardware tokens with the system (smartcard, USB token, Hardware Security Module), the hardware token support module must be installed. Use the following command:

```bash
sudo apt-get install xroad-addon-hwtokens
```

Vendorspecific library name is configured in */etc/xroad/devices.ini* file.

After installation and configuring library, the xroad-signer service must be restarted:

```bash
sudo service xroad-signer restart
```

## 3 Configuration

To start using the configuration proxy, a proxy instance configuration needs to be created. Several proxy instances can be configured to mediate multiple global configurations.

### 3.1 Prerequisites

#### 3.1.1 Security Token Activation

The configuration proxy uses a security token for storing the key that is used for signing the distributed configuration. The token can be stored either on hard disk (software token) or in hardware. Before the configuration proxy can be used, the security token must be initialized and activated.

Initialization of a software token can be done as follows:

```bash
signer-console init-software-token
```

A PIN is prompted, which will be used to log in to the software token afterwards. Initialization of hardware tokens is vendorspecific and is not in scope of this documentation.

Activation of the security token is done by logging in to the token, using the following command:

```bash
signer-console login-token <TOKEN_ID>
```

where &lt;TOKEN_ID&gt; is the identifier of the security token, which can be found with:

```bash
signer-console list-tokens
```

Note, that the identifier of a software token is always „0”.

#### 3.1.2 User Access Privileges

The administrator must execute configuration scripts as the 'xroad' user.

### 3.2 General Configuration

Modify '/etc/xroad/conf.d/local.ini' to contain the following:

```ini
[configuration-proxy]

; Address of the webserver serving the distributed configuration address=<public or NAT address>
```

The configuration of this parameter is necessary for generating a correctly formatted configuration anchor file that will need to be uploaded to central servers that should receive configurations mediated by this proxy, this process is described in detail in [3.4](#34-proxy-instance-configuration). There are several more system parameters that can be configured in '/etc/xroad/conf.d/local.ini' under the 'configuration-proxy' section, their descriptions and default values can be seen from the following table:

| Parameter              | Default value                          | Explanation |
|------------------------|----------------------------------------|-------------|
| address                | 0.0.0.0                                | The public IP or NAT address (reference data: 1.5) which can be accessed for downloading the distributed global configurations. |
| configuration-path     | /etc/xroad/confproxy/                  | Absolute path to the directory containing the configuration files of the proxy instance. The format of the configuration directory is described in [3.2.1](#321-configuration-structure-of-the-instances). |
| generated-conf-path    | /var/lib/xroad/public                  | Absolute path to the public web server directory where the global configuration files generated by this configuration proxy, should be placed for distribution. |
| signature-digest-algorithm-id | SHA-512                         | ID of the digest algorithm the configuration proxy should use when computing global configuration signatures. The possible values are: *SHA-256*, *SHA-384*, *SHA-512*. |
| hash-algorithm-uri     | http://www.w3.org/2001/04/xmlenc#sha512 | URI identifying the algorithm the configuration proxy should use to calculate hash values for the global configuration file. The possible values are:<br>http://www.w3.org/2001/04/xmlenc#sha256,<br>http://www.w3.org/2001/04/xmlenc#sha512. |
| download-script        | /usr/share/xroad/scripts/download_instance_configuration.sh | Absolute path to the location of the script that initializes the global configuration download procedure. |

The configuration proxy is periodically started by a cron job. It reads the properties described above, from the configuration file before executing each proxy instance configured in 'configuration-path', generating new global configuration directories using algorithms as defined by 'signature-digest-algorithm-id' and 'hash-algorithm-uri'. The generated directories are subsequently placed in 'generated-conf-path' for distribution.

#### 3.2.1 Configuration Structure of the Instances

Each global configuration that is to be mediated by the configuration proxy requires a proxy instance to be configured. The configuration of a proxy instance consists of a set of configuration files, including

* a trusted anchor .xml of the configuration being mediated;
* a configuration .ini file;
* verification certificates for the configured signing keys.

The following example file tree shows configured proxy instances named PROXY1 and PROXY2:

```bash
<configuration-path>/
|-PROXY1/
| |-cert_QWERTY123.pem
| |-cert_321YTREWQ.pem
| |-conf.ini
| \-anchor.xml
|-PROXY2/
| |-cert_1234567890.pem
| |-conf.ini
| \-anchor.xml
\-...
```

The configuration of proxy instances is described in [3.4](#34-proxy-instance-configuration).

### 3.3 Proxy Instance Reference Data

**ATTENTION:** The names in the angle brackets&lt;&gt; are chosen by the X-Road configuration proxy administrator.

| Ref |                            | Explanation |
|-----|----------------------------|-------------|
| 2.1 |  &lt;PROXY_NAME&gt;        | Name of the proxy instance being configured |
| 2.2 |  &lt;SECURITY_TOKEN_ID&gt; | ID of a security token (as defined by prerequisites [3.1](#31-prerequisites)) |
| 2.3 |  &lt;ANCHOR_FILENAME&gt;   | Filename of the generated anchor .xml file that the configuration proxy clients will need to use for downloading the global configuration |

### 3.4 Proxy Instance Configuration

1)  Create configuration files for the new proxy instance by invoking the 'confproxy-create-instance -p &lt;PROXY_NAME&gt;' command. Afterwards, use the 'confproxy-view-conf -p &lt;PROXY_NAME&gt;' command to verify that the operation has been successfully completed (example output follows):

```cmd
confproxy-create-instance -p PROXY

Populating 'conf.ini' with default values.
Done.

confproxy-view-conf -p PROXY

Configuration for proxy 'PROXY'
Validity interval: 600 s.
anchor.xml
================================================
'anchor.xml' could not be loaded: IOError: /etc/xroad/confproxy/PROXY/anchor.xml (No such file or directory)
Configuration URL
================================================
http://1.2.3.4/PROXY/conf
Signing keys and certificates
================================================
active-signing-key-id:
    NOT CONFIGURED (add 'active-signing-key-id' to 'conf.ini')
```

2) Generate a signing key and a self signed certificate for the newly created proxy instance using the following command:

```cmd
confproxy-add-signing-key -p <PROXY_NAME> -t <SECURITY_TOKEN_ID>
```

If no active signing key is configured for the proxy instance, then the new key should be set as the currently active key (example output follows):

```cmd
confproxy-add-signing-key -p PROXY -t 0

Generated key with ID QWERTY123
No active key configured, setting new key as active in conf.ini
Saved self-signed certificate to cert_QWERTY123.pem
confproxy-view-conf -p PROXY
...
Signing keys and certificates
================================================
active-signing-key-id:
QWERTY123 (Certificate: /etc/xroad/confproxy/PROXY/cert_QWERTY123.pem)
```
Alternatively, one may add an existing key using the '–k &lt;KEY_ID&gt;' argument:

```cmd
confproxy-add-signing-key -p PROXY -k QWERTY123

No active key configured, setting new key as active in conf.ini
Saved self-signed certificate to cert_QWERTY123.pem
```

3) To define which global configuration this proxy instance should distribute, download the source anchor from an X-Road central server and save it as '/etc/xroad/confproxy/&lt;PROXY_NAME&gt;/anchor.xml'.

4) The configuration proxy should be operational at this point. The periodic cron job (once a minute) should download the global configuration defined in '/etc/xroad/confproxy/&lt;PROXY_NAME&gt;/anchor.xml' and generate a directory for distribution. The output of 'confproxy-view-conf -p &lt;PROXY_NAME&gt;' should be similar to the following:

```cmd
confproxy-view-conf -p PROXY

Configuration for proxy 'PROXY'
Validity interval: 600 s.
anchor.xml
================================================
Instance identifier: AA
Generated at: UTC 2014-11-17 09:28:56
Hash: 3A:3D:B2:A4:D3:FC:E8:08:7E:EA:8A:92:5C:6E:92:0C:70:C8
Configuration URL
================================================
http://1.2.3.4/PROXY/conf
Signing keys and certificates
================================================
active-signing-key-id:
QWERTY123 (Certificate: /etc/xroad/confproxy/PROXY/cert_QWERTY123.pem)
```

5) To let clients download the generated global configuration an anchor file needs to be generated using the following command:

```cmd
confproxy-generate-anchor -p <PROXY_NAME> -f <ANCHOR_FILENAME>
```

If generation was successful the output should be simply:

```cmd
confproxy-generate-anchor -p PROXY -f anchor.xml

Generated anchor xml to 'anchor.xml'
```

6) To make sure that the global configuration is being distributed correctly use the '/usr/share/xroad/scripts/download_instance_configuration.sh' script, giving it &lt;ANCHOR_FILENAME&gt; and the path, which should hold the downloaded files, as arguments (example output follows):

```cmd
mkdir test_download
/usr/share/xroad/scripts/download_instance_configuration.sh anchor.xml test_download/

... - Downloading configuration from http://1.2.3.4/PROXY/conf
... - Downloading content from http://1.2.3.4/PROXY/123/AA/shared-params.xml
... - Saving SHARED-PARAMETERS to test_download/AA/shared-params.xml
... - Saving content to file test_download/AA/shared-params.xml
... - Downloading content from http://1.2.3.4/PROXY/123/AA/private-params.xml
... - Saving PRIVATE-PARAMETERS to test_download/AA/private-params.xml
... - Saving content to file test_download/AA/private-params.xml
```
If the proxy instance has been configured correctly, the 'test_download' directory should contain the downloaded global configuration files.

### 3.5 Additional Configuration

#### 3.5.1 Changing the Validity Interval

There is an additional property in the configuration file of the proxy instance (/etc/xroad/confproxy/&lt;PROXY_NAME&gt;/conf.ini) that determines the validity interval of the generated global configuration for a given instance.

The default value is 10 minutes (600 seconds). The property is set by modifying the following field in the configuration file:

```ini
[configuration-proxy]
...
validity-interval-seconds=600
```

Notice that when the configuration proxy instance is started, it deletes all the previously generated global configuration directories that are older than the currently configured validity interval for that instance.

#### 3.5.2 Deleting the Signing Keys

Should a signing key need to be deleted, the following command can be used:

```bash
confproxy-del-signing-key -p <PROXY_NAME> -k <SIGNING_KEY_ID>
```

where &lt;SIGNING_KEY_ID&gt; can be found in the output of 'confproxy-view-conf' (example output follows):

```bash
confproxy-del-signing-key -p PROXY -k QWERTY123

Deleted key from signer
Deleted key from conf.ini
Deleted self-signed certificate 'cert_QWERTY123.pem'
```

Attempts to delete the active signing key will be unsuccessful.

#### 3.5.3 Changing the Active Signing Key

Additional signing keys can be added with the following command:

```bash
confproxy-add-signing-key -p <PROXY_NAME> -t <SECURITY_TOKEN_ID>
```

Keys added in this manner will be marked as inactive keys ('signing-key-id-\*=&lt;KEY_ID&gt;') in the proxy instance configuration file (/etc/xroad/confproxy/&lt;PROXY_NAME&gt;/conf.ini). In case the current active signing key has to be replaced by one of the additional keys, the configuration file of the proxy instance will need to be modified, changing the following lines:

```ini
[configuration-proxy]
...
active-signing-key-id=QWERTY123
signing-key-id-1=QWERTY123
signing-key-id-2=321YTREWQ
```

to the following ones:

```ini
[configuration-proxy]
...
active-signing-key-id=321YTREWQ
signing-key-id-1=QWERTY123
signing-key-id-2=321YTREWQ
```

After the change the key 'QWERTY123' may be deleted if necessary.
