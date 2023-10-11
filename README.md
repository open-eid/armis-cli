# ARMIS CLI

A command line tool for testing ARMIS Applet Ecosystem components on smart cards.

## Table of Contents

- [Related repositories](#related-repositories)
  - [Pre-built artifacts](#prebuilt-artifacts)
- [Prerequisites](#prerequisites)
  - [Global Platform keys](#gp-keys)
- [Usage](#usage)
  - [CAP file signing](#action-sign)
  - [Listing card contents](#action-list)
  - [ARMIS Manager Applet deployment](#action-deploy-manager)
  - [ARMIS Client Applet deployment](#action-deploy-client)
  - [Additional card contents management commands](#action-manage)
    - [Loading a package](#action-manage-load)
    - [Installing an applet](#action-manage-install)
    - [Uninstalling an applet](#action-manage-uninstall)
    - [Unloading a package](#action-manage-unload)
    - [Unloading all packages](#action-manage-unload-all)
    - [Listing card contents](#action-manage-list)
    - [Command aggregation](#action-manage-aggregation)
- [Known issues](#known-issues)

<a name="related-repositories"></a>
## Related repositories

- [ARMIS Applet Ecosystem](https://github.com/open-eid/armis-applet-ecosystem)
- [Client Issuer Service](https://github.com/open-eid/armis-test-client-issuer-service)

<a name="prebuilt-artifacts"></a>
### Pre-built artifacts

For testing purposes, the following artifacts have been pre-built and can be found in the [`prebuilt/`](prebuilt)
directory:

- `prebuilt/applets/` - pre-built artifacts of the ARMIS Applet Ecosystem
  - `armis-ecosystem-lib.cap` - pre-built CAP file of the ARMIS Applet Ecosystem Library
  - `armis-ecosystem-manager.cap` - pre-built CAP file of the ARMIS Manager Applet
  - `armis-test-client.cap` - pre-built CAP file of the example client applet for testing

You will additionally need these pre-built artifacts:
- `armis-cli.jar` - ARMIS CLI; a command line tool for testing ARMIS Applet Ecosystem components on smart cards; can be
  downloaded from the [Releases page of this repository](https://github.com/open-eid/armis-cli/releases)
- `test-client-issuer-service-*-exec.jar` - Client applet issuer service; can be downloaded from the
  [Releases page of Client Issuer Service repository](https://github.com/open-eid/armis-test-client-issuer-service/releases).

The client applet issuer service can be run using the following command:
```shell
java -jar prebuilt/test-client-issuer-service/test-client-issuer-service-*-exec.jar
```

<a name="prerequisites"></a>
## Prerequisites

- Java 11 JDK or JRE (optionally at least Java 17 JDK or JRE is required for running the pre-built client applet issuer
  service for testing)
- A Java Card v3.0.4 Classic Edition and Global Platform v2.2.1 compliant smart card
- Access to card's SSD (Supplementary Security Domain) and DM (Delegated Management) keys

<a name="gp-keys"></a>
### Global Platform keys

- **SSD key** - **Security Domain key for Supplementary Security Domain**
  - This key is used for establishing a secure connection to the on-card *Supplementary Security Domain*
  - This must be a symmetric key (e.g. AES256) in HEX-encoded format
- **DM key** - **Domain key for Delegated Management**
  - This key is used for providing authenticity for DAP (*Data Authentication Pattern*) verification
  - This must be an RSA private key in PEM-encoded format

<a name="usage"></a>
## Usage

<a name="action-sign"></a>
### CAP file signing

Sign a specific CAP file and either overwrite the old one or save it to a specified location.

**NB:** Signing a CAP file requires access to a valid **Domain key for Delegated Management**.

| Parameter       | Required | Description | Example |
| --------------- | -------- | ----------- | ------- |
| `armis.action`  | YES      | The action to perform. **Must** be `sign` to perform CAP signing. | `sign` |
| `armis.dm-key`  | NO       | Domain key for Delegated Management in PEM format. Must be an RSA key. Defaults to `classpath:sensitive/dm.key.pem`, if not provided. | - `file:path/to/dm.key.pem`<br>- `classpath:path/to/dm.key.pem` |
| `armis.sd-key`  | NO       | Key of the security domain to open a GP session to. Defaults to Supplementary Security Domain key `classpath:sensitive/ssd.key.hex`, if not provided. | - `file:path/to/ssd.key.hex`<br>- `classpath:path/to/ssd.key.hex`<br>- `hex:0102030405060708090A0B0C0D0E0F` |
| `armis.cap-in`  | YES      | CAP file to sign. | `file:path/to/file.cap` |
| `armis.cap-out` | NO       | Signed CAP file output path. If not provided, defaults to the path specified by `armis.cap-in` if the path is a path to a file. | `path/to/file.cap` |

Example usage:
```shell
java -jar armis-cli.jar --armis.action=sign --armis.dm-key=file:dm.key.pem --armis.sd-key=file:ssd.key.hex --armis.cap-in=file:armis-ecosystem-lib.cap
```

<a name="action-list"></a>
### Listing card contents

List applets and packages currently present on a card.

**NB:** Communication with a card's security domain requires access to valid **Security Domain key** and **Domain key
for Delegated Management**.

| Parameter                      | Required | Description | Example |
| ------------------------------ | -------- | ----------- | ------- |
| `armis.action`                 | YES      | The action to perform. **Must** be `list` to list card contents. | `list` |
| `armis.sd-aid`                 | NO       | AID of the security domain to open a GP session to. Defaults to Supplementary Security Domain AID `D233000000444F4D`, if not provided. | `D233000000444F4D` |
| `armis.sd-key`                 | NO       | Key of the security domain to open a GP session to. Defaults to Supplementary Security Domain key `classpath:sensitive/ssd.key.hex`, if not provided. | - `file:path/to/ssd.key.hex`<br>- `classpath:path/to/ssd.key.hex`<br>- `hex:0102030405060708090A0B0C0D0E0F` |
| `armis.sd-key-diversification` | NO       | Security domain key diversification algorithm. Must be one of `NONE`, `VISA2`, `EMV` or `KDF3`. Defaults to `KDF3`, if not provided. | `KDF3` |
| `armis.dm-key`                 | NO       | Domain key for Delegated Management in PEM format. Must be an RSA key. Defaults to `classpath:sensitive/dm.key.pem`, if not provided. | - `file:path/to/dm.key.pem`<br>- `classpath:path/to/dm.key.pem` |

Example usage:
```shell
java -jar armis-cli.jar --armis.action=list --armis.sd-key=file:ssd.key.hex --armis.dm-key=file:dm.key.pem
```

<a name="action-deploy-manager"></a>
### ARMIS Manager Applet deployment

Deploy the ARMIS Manager Applet onto a card and personalize it:
* (optional) loads the library from the specified library CAP file (if provided) onto a card
* loads and installs the applet from the specified manager CAP file onto a card
* instructs the applet to generate a new EC key-pair based on the specified curve name, and retrieves the public key
* requests a new certificate to be issued for the newly generated and retrieved public key
* sends a newly issued certificate to the applet to finalize personalization

**NB:** Communication with a card's security domain requires access to valid **Security Domain key** and **Domain key
for Delegated Management**.

| Parameter                      | Required | Description | Example |
| ------------------------------ | -------- | ----------- | ------- |
| `armis.action`                 | YES      | The action to perform. **Must** be `deploy-manager` to deploy ARMIS Manager Applet. | `deploy-manager` |
| `armis.sd-aid`                 | NO       | AID of the security domain to open a GP session to. Defaults to Supplementary Security Domain AID `D233000000444F4D`, if not provided. | `D233000000444F4D` |
| `armis.sd-key`                 | NO       | Key of the security domain to open a GP session to. Defaults to Supplementary Security Domain key `classpath:sensitive/ssd.key.hex`, if not provided. | - `file:path/to/ssd.key.hex`<br>- `classpath:path/to/ssd.key.hex`<br>- `hex:0102030405060708090A0B0C0D0E0F` |
| `armis.sd-key-diversification` | NO       | Security domain key diversification algorithm. Must be one of `NONE`, `VISA2`, `EMV` or `KDF3`. Defaults to `KDF3`, if not provided. | `KDF3` |
| `armis.dm-key`                 | NO       | Domain key for Delegated Management in PEM format. Must be an RSA key. Defaults to `classpath:sensitive/dm.key.pem`, if not provided. | - `file:path/to/dm.key.pem`<br>- `classpath:path/to/dm.key.pem` |
| `armis.cap-file-hash-function` | NO       | Hash function for hashing CAP files for INSTALL FOR LOAD. Must be one of `SHA1`, `SHA256`, `SHA384` or `SHA512`. Defaults to `SHA256`, if not provided. | `SHA256` |
| `armis.manager-cap-file`       | YES      | Path to ARMIS Manager Applet CAP file. | `file:path/to/applet.cap` |
| `armis.library-cap-file`       | NO       | Path to ARMIS Utilities Library CAP file. Only manager applet is deployed if library path is not provided. | `file:path/to/library.cap` |
| `armis.curve-name`             | NO       | Name of the EC curve to configure ARMIS Manager Applet to use for its key-pair. Defaults to `secp384r1`, if not provided. | `secp384r1` |
| `armis.issuer-dn`              | NO       | ARMIS CA issuerDN. Defaults to `CN=DEV of ARMIS-CA,organizationIdentifier=NTREE-12345678,O=ACME Corporation,C=EE`, if not provided. | `CN=DEV of ARMIS-CA,organizationIdentifier=NTREE-12345678,O=ACME Corporation,C=EE` |
| `armis.card-holder-cert`       | NO       | Card holder certificate<sup>CRT</sup> in PEM format. A temporary certificate with default values is generated, if not provided. | -`file:path/to/card-holder-cert.pem`<br>- `classpath:path/to/card-holder-cert.pem` |

<sup>CRT</sup> The information extracted from the card holder certificate is used to issue a certificate for ARMIS
Manager Applet. **NB:** The certificate profile of the card holder certificate must conform to the
[Certificate Profile for ID-1 Format Identity Documents Issued by the Republic of Estonia](https://www.skidsolutions.eu/upload/files/SK-CPR-ESTEID2018-EN-v1_2_20200630.pdf)!

Example usage:
```shell
java -jar armis-cli.jar --armis.action=deploy-manager --armis.sd-key=file:ssd.key.hex --armis.dm-key=file:dm.key.pem --armis.manager-cap-file=file:path/to/applet.cap --armis.library-cap-file=file:path/to/library.cap
```

<a name="action-deploy-client"></a>
### ARMIS Client Applet deployment

Some cards may require CAP file to be signed. In such case use `sign` command before using `deploy-client` command.

Deploy the ARMIS Client Applet onto a card and personalize it:
* loads and installs the applet from the specified client CAP file onto a card
* obtains the STORE DATA command containing the first personalization command from the issuer service
* calls INSTALL FOR PERSONALIZATION for the specified applet in order to make the subsequent STORE DATA commands to be
  routed to that applet and sends the first STORE DATA command to the applet
* obtains subsequent STORE DATA commands to be routed to that applet until the issuer returns no more personalization
  data

**NB:** Communication with a card's security domain requires access to valid **Security Domain key** and **Domain key
for Delegated Management**.

| Parameter                      | Required | Description | Example |
| ------------------------------ | -------- | ----------- | ------- |
| `armis.action`                 | YES      | The action to perform. **Must** be `deploy-client` to deploy a client applet. | `deploy-client` |
| `armis.sd-aid`                 | NO       | AID of the security domain to open a GP session to. Defaults to Supplementary Security Domain AID `D233000000444F4D`, if not provided. | `D233000000444F4D` |
| `armis.sd-key`                 | NO       | Key of the security domain to open a GP session to. Defaults to Supplementary Security Domain key `classpath:sensitive/ssd.key.hex`, if not provided. | - `file:path/to/ssd.key.hex`<br>- `classpath:path/to/ssd.key.hex`<br>- `hex:0102030405060708090A0B0C0D0E0F` |
| `armis.sd-key-diversification` | NO       | Security domain key diversification algorithm. Must be one of `NONE`, `VISA2`, `EMV` or `KDF3`. Defaults to `KDF3`, if not provided. | `KDF3` |
| `armis.dm-key`                 | NO       | Domain key for Delegated Management in PEM format. Must be an RSA key. Defaults to `classpath:sensitive/dm.key.pem`, if not provided. | - `file:path/to/dm.key.pem`<br>- `classpath:path/to/dm.key.pem` |
| `armis.cap-file-hash-function` | NO       | Hash function for hashing CAP files for INSTALL FOR LOAD. Must be one of `SHA1`, `SHA256`, `SHA384` or `SHA512`. Defaults to `SHA256`, if not provided. | `SHA256` |
| `armis.client-cap-file`        | YES      | Path to the Client Applet CAP file. | `file:path/to/applet.cap` |
| `armis.client-aid`             | NO       | AID of the client applet. Defaults to `4D616E61676572417071`, if not provided. | `4D616E61676572417071` |
| `armis.client-instance-aid`    | NO       | AID of the client applet instance. Defaults to `4D616E61676572417071`, if not provided. | `4D616E61676572417071` |
| `armis.manager-aid`            | NO       | AID of the manager applet. Defaults to `4D616E61676572417070`, if not provided. | `4D616E61676572417070` |
| `armis.issuer-url`             | NO       | Client issuer service URL. Defaults to `http://localhost:8080/v1`, if not provided. | `http://localhost:8080/v1` |

Example usage:
```shell
java -jar armis-cli.jar --armis.action=deploy-client --armis.sd-key=file:ssd.key.hex --armis.dm-key=file:dm.key.pem --armis.client-cap-file=file:path/to/applet.cap --armis.issuer-url=http://localhost:8080/v1
```

<a name="action-manage"></a>
### Additional card contents management commands

Additional commands are available under the *action* `manage`.

<a name="action-manage-load"></a>
#### Loading a package

Load a package from a specific CAP file onto a card.

**NB:** Communication with a card's security domain requires access to valid **Security Domain key** and **Domain key
for Delegated Management**.

| Parameter                      | Required | Description | Example |
| ------------------------------ | -------- | ----------- | ------- |
| `armis.action`                 | YES      | The action to perform. **Must** be `manage` to manage card contents. | `manage` |
| `armis.steps`                  | YES      | The management step(s) to perform. **Must** be `load` to load a package onto a card. | `load` |
| `armis.sd-aid`                 | NO       | AID of the security domain to open a GP session to. Defaults to Supplementary Security Domain AID `D233000000444F4D`, if not provided. | `D233000000444F4D` |
| `armis.sd-key`                 | NO       | Key of the security domain to open a GP session to. Defaults to Supplementary Security Domain key `classpath:sensitive/ssd.key.hex`, if not provided. | - `file:path/to/ssd.key.hex`<br>- `classpath:path/to/ssd.key.hex`<br>- `hex:0102030405060708090A0B0C0D0E0F` |
| `armis.sd-key-diversification` | NO       | Security domain key diversification algorithm. Must be one of `NONE`, `VISA2`, `EMV` or `KDF3`. Defaults to `KDF3`, if not provided. | `KDF3` |
| `armis.dm-key`                 | NO       | Domain key for Delegated Management in PEM format. Must be an RSA key. Defaults to `classpath:sensitive/dm.key.pem`, if not provided. | - `file:path/to/dm.key.pem`<br>- `classpath:path/to/dm.key.pem` |
| `armis.cap-file-hash-function` | NO       | Hash function for hashing CAP files for INSTALL FOR LOAD. Must be one of `SHA1`, `SHA256`, `SHA384` or `SHA512`. Defaults to `SHA256`, if not provided. | `SHA256` |
| `armis.steps=load:{cap-file}`  | YES      | Path to the CAP file to load. | `'file:path/to/file.cap'` |

Example usage:
```shell
java -jar armis-cli.jar --armis.action=manage --armis.sd-key=file:ssd.key.hex --armis.dm-key=file:dm.key.pem --armis.steps[0]="load:{cap-file:'file:path/to/file.cap'}"
```

<a name="action-manage-install"></a>
#### Installing an applet

Install a specific applet onto a card from a package that prior to installation is either already present on a card or
is loaded onto a card from a specific CAP file. Some cards may require CAP file to be signed. In such case use `sign`
command before using `install` command.

**NB:** Communication with a card's security domain requires access to valid **Security Domain key** and **Domain key
for Delegated Management**.

| Parameter                                   | Required        | Description | Example |
| ------------------------------------------- | --------------- | ----------- | ------- |
| `armis.action`                              | YES             | The action to perform. **Must** be `manage` to manage card contents. | `manage` |
| `armis.steps`                               | YES             | The management step(s) to perform. **Must** be `install` to install an applet onto a card. | `install` |
| `armis.sd-aid`                              | NO              | AID of the security domain to open a GP session to. Defaults to Supplementary Security Domain AID `D233000000444F4D`, if not provided. | `D233000000444F4D` |
| `armis.sd-key`                              | NO              | Key of the security domain to open a GP session to. Defaults to Supplementary Security Domain key `classpath:sensitive/ssd.key.hex`, if not provided. | - `file:path/to/ssd.key.hex`<br>- `classpath:path/to/ssd.key.hex`<br>- `hex:0102030405060708090A0B0C0D0E0F` |
| `armis.sd-key-diversification`              | NO              | Security domain key diversification algorithm. Must be one of `NONE`, `VISA2`, `EMV` or `KDF3`. Defaults to `KDF3`, if not provided. | `KDF3` |
| `armis.dm-key`                              | NO              | Domain key for Delegated Management in PEM format. Must be an RSA key. Defaults to `classpath:sensitive/dm.key.pem`, if not provided. | - `file:path/to/dm.key.pem`<br>- `classpath:path/to/dm.key.pem` |
| `armis.cap-file-hash-function`              | NO              | Hash function for hashing CAP files for INSTALL FOR LOAD. Must be one of `SHA1`, `SHA256`, `SHA384` or `SHA512`. Defaults to `SHA256`, if not provided. | `SHA256` |
| `armis.steps=install:{cap-file}`            | YES<sup>1</sup> | Path to the CAP file containing the applet to install. | `'file:path/to/file.cap'` |
| `armis.steps=install:{package-aid}`         | YES<sup>2</sup> | AID of the package to install the applet from. | `'0102030405'` |
| `armis.steps=install:{applet-aid}`          | YES<sup>2</sup> | AID of the applet to install. | `'010203040506'` |
| `armis.steps=install:{instance-aid}`        | NO              | Instance AID of the applet to install. Defaults to applet AID if not provided. | `'01020304050607'` |
| `armis.steps=install:{payload}`             | NO              | Optional applet installation payload in HEX format. | - `'file:path/to/installation-payload.hex'`<br>- `'hex:0102030405060708090A0B0C0D0E0F'` |
| `armis.steps=install:{load-before-install}` | NO              | Whether to load the containing package onto a card before installing the applet. Defaults to `'false'`, if not provided. **NB:** applicable only if `cap-file` is provided! | `'true'` |

<sup>1</sup> CAP file is mandatory only if `package-aid` and `applet-aid` are not provided. If CAP file is provided,
package AID and applet AID are extracted from the CAP file.

<sup>2</sup> Package AID and applet AID are mandatory only if `cap-file` is not provided.

Example usages:
```shell
java -jar armis-cli.jar --armis.action=manage --armis.sd-key=file:ssd.key.hex --armis.dm-key=file:dm.key.pem --armis.steps[0]="install:{cap-file:'file:path/to/file.cap',payload:'hex:A07F',load-before-install:'true'}"
java -jar armis-cli.jar --armis.action=manage --armis.sd-key=file:ssd.key.hex --armis.dm-key=file:dm.key.pem --armis.steps[0]="install:{package-aid:'0102030405',applet-aid:'010203040506',payload:'hex:A07F'}"
```

<a name="action-manage-uninstall"></a>
#### Uninstalling an applet

Uninstall a specific instance of an applet from a card.

**NB:** Communication with a card's security domain requires access to valid **Security Domain key** and **Domain key
for Delegated Management**.

| Parameter                              | Required        | Description | Example |
| -------------------------------------- | --------------- | ----------- | ------- |
| `armis.action`                         | YES             | The action to perform. **Must** be `manage` to manage card contents. | `manage` |
| `armis.steps`                          | YES             | The management step(s) to perform. **Must** be `uninstall` to uninstall an applet from a card. | `uninstall` |
| `armis.sd-aid`                         | NO              | AID of the security domain to open a GP session to. Defaults to Supplementary Security Domain AID `D233000000444F4D`, if not provided. | `D233000000444F4D` |
| `armis.sd-key`                         | NO              | Key of the security domain to open a GP session to. Defaults to Supplementary Security Domain key `classpath:sensitive/ssd.key.hex`, if not provided. | - `file:path/to/ssd.key.hex`<br>- `classpath:path/to/ssd.key.hex`<br>- `hex:0102030405060708090A0B0C0D0E0F` |
| `armis.sd-key-diversification`         | NO              | Security domain key diversification algorithm. Must be one of `NONE`, `VISA2`, `EMV` or `KDF3`. Defaults to `KDF3`, if not provided. | `KDF3` |
| `armis.dm-key`                         | NO              | Domain key for Delegated Management in PEM format. Must be an RSA key. Defaults to `classpath:sensitive/dm.key.pem`, if not provided. | - `file:path/to/dm.key.pem`<br>- `classpath:path/to/dm.key.pem` |
| `armis.cap-file-hash-function`         | NO              | Hash function for hashing CAP files for INSTALL FOR LOAD. Must be one of `SHA1`, `SHA256`, `SHA384` or `SHA512`. Defaults to `SHA256`, if not provided. | `SHA256` |
| `armis.steps=uninstall:{cap-file}`     | YES<sup>1</sup> | Path to the CAP file containing the applet to uninstall. | `'file:path/to/file.cap'` |
| `armis.steps=uninstall:{instance-aid}` | YES<sup>2</sup> | Instance AID of the applet to uninstall. | `'01020304050607'` |

<sup>1</sup> CAP file is mandatory only if `instance-aid` is not provided. If CAP file is provided, applet AID is
extracted from the CAP file and used as an instance AID.

<sup>2</sup> Instance AID is mandatory only if `cap-file` is not provided.

Example usages:
```shell
java -jar armis-cli.jar --armis.action=manage --armis.sd-key=file:ssd.key.hex --armis.dm-key=file:dm.key.pem --armis.steps[0]="uninstall:{cap-file:'file:path/to/file.cap'}"
java -jar armis-cli.jar --armis.action=manage --armis.sd-key=file:ssd.key.hex --armis.dm-key=file:dm.key.pem --armis.steps[0]="uninstall:{instance-aid:'01020304050607'}"
```

<a name="action-manage-unload"></a>
#### Unloading a package

Remove a specific package from a card (and uninstall any applets from that package if present).

**NB:** Communication with a card's security domain requires access to valid **Security Domain key** and **Domain key
for Delegated Management**.

| Parameter                          | Required        | Description | Example |
| ---------------------------------- | --------------- | ----------- | ------- |
| `armis.action`                     | YES             | The action to perform. **Must** be `manage` to manage card contents. | `manage` |
| `armis.steps`                      | YES             | The management step(s) to perform. **Must** be `unload` to remove a package from a card. | `unload` |
| `armis.sd-aid`                     | NO              | AID of the security domain to open a GP session to. Defaults to Supplementary Security Domain AID `D233000000444F4D`, if not provided. | `D233000000444F4D` |
| `armis.sd-key`                     | NO              | Key of the security domain to open a GP session to. Defaults to Supplementary Security Domain key `classpath:sensitive/ssd.key.hex`, if not provided. | - `file:path/to/ssd.key.hex`<br>- `classpath:path/to/ssd.key.hex`<br>- `hex:0102030405060708090A0B0C0D0E0F` |
| `armis.sd-key-diversification`     | NO              | Security domain key diversification algorithm. Must be one of `NONE`, `VISA2`, `EMV` or `KDF3`. Defaults to `KDF3`, if not provided. | `KDF3` |
| `armis.dm-key`                     | NO              | Domain key for Delegated Management in PEM format. Must be an RSA key. Defaults to `classpath:sensitive/dm.key.pem`, if not provided. | - `file:path/to/dm.key.pem`<br>- `classpath:path/to/dm.key.pem` |
| `armis.cap-file-hash-function`     | NO              | Hash function for hashing CAP files for INSTALL FOR LOAD. Must be one of `SHA1`, `SHA256`, `SHA384` or `SHA512`. Defaults to `SHA256`, if not provided. | `SHA256` |
| `armis.steps=unload:{cap-file}`    | YES<sup>1</sup> | Path to the CAP file containing the package to remove. | `'file:path/to/file.cap'` |
| `armis.steps=unload:{package-aid}` | YES<sup>2</sup> | AID of the package to remove. | `'0102030405'` |

<sup>1</sup> CAP file is mandatory only if `package-aid` is not provided. If CAP file is provided, package AID is
extracted from the CAP file.

<sup>2</sup> Package AID is mandatory only if `cap-file` is not provided.

Example usages:
```shell
java -jar armis-cli.jar --armis.action=manage --armis.sd-key=file:ssd.key.hex --armis.dm-key=file:dm.key.pem --armis.steps[0]="unload:{cap-file:'file:path/to/file.cap'}"
java -jar armis-cli.jar --armis.action=manage --armis.sd-key=file:ssd.key.hex --armis.dm-key=file:dm.key.pem --armis.steps[0]="unload:{package-aid:'0102030405'}"
```

<a name="action-manage-unload-all"></a>
#### Unloading all packages

Try to remove all packages from a card in the reverse order they are presented in the GP registry.

**NB:** Communication with a card's security domain requires access to valid **Security Domain key** and **Domain key
for Delegated Management**.

| Parameter                          | Required | Description | Example |
| ---------------------------------- | -------- | ----------- | ------- |
| `armis.action`                     | YES      | The action to perform. **Must** be `manage` to manage card contents. | `manage` |
| `armis.steps`                      | YES      | The management step(s) to perform. **Must** be `unload-all` to try to remove all packages from a card. | `unload-all` |
| `armis.sd-aid`                     | NO       | AID of the security domain to open a GP session to. Defaults to Supplementary Security Domain AID `D233000000444F4D`, if not provided. | `D233000000444F4D` |
| `armis.sd-key`                     | NO       | Key of the security domain to open a GP session to. Defaults to Supplementary Security Domain key `classpath:sensitive/ssd.key.hex`, if not provided. | - `file:path/to/ssd.key.hex`<br>- `classpath:path/to/ssd.key.hex`<br>- `hex:0102030405060708090A0B0C0D0E0F` |
| `armis.sd-key-diversification`     | NO       | Security domain key diversification algorithm. Must be one of `NONE`, `VISA2`, `EMV` or `KDF3`. Defaults to `KDF3`, if not provided. | `KDF3` |
| `armis.dm-key`                     | NO       | Domain key for Delegated Management in PEM format. Must be an RSA key. Defaults to `classpath:sensitive/dm.key.pem`, if not provided. | - `file:path/to/dm.key.pem`<br>- `classpath:path/to/dm.key.pem` |

Example usage:
```shell
java -jar armis-cli.jar --armis.action=manage --armis.sd-key=file:ssd.key.hex --armis.dm-key=file:dm.key.pem --armis.steps[0]=unload-all
```

<a name="action-manage-list"></a>
#### Listing card contents

List applets and packages present on a card.

**NB:** Communication with a card's security domain requires access to valid **Security Domain key** and **Domain key
for Delegated Management**.

| Parameter                          | Required | Description | Example |
| ---------------------------------- | -------- | ----------- | ------- |
| `armis.action`                     | YES      | The action to perform. **Must** be `manage` to manage card contents. | `manage` |
| `armis.steps`                      | YES      | The management step(s) to perform. **Must** be `list` to list card contents. | `list` |
| `armis.sd-aid`                     | NO       | AID of the security domain to open a GP session to. Defaults to Supplementary Security Domain AID `D233000000444F4D`, if not provided. | `D233000000444F4D` |
| `armis.sd-key`                     | NO       | Key of the security domain to open a GP session to. Defaults to Supplementary Security Domain key `classpath:sensitive/ssd.key.hex`, if not provided. | - `file:path/to/ssd.key.hex`<br>- `classpath:path/to/ssd.key.hex`<br>- `hex:0102030405060708090A0B0C0D0E0F` |
| `armis.sd-key-diversification`     | NO       | Security domain key diversification algorithm. Must be one of `NONE`, `VISA2`, `EMV` or `KDF3`. Defaults to `KDF3`, if not provided. | `KDF3` |
| `armis.dm-key`                     | NO       | Domain key for Delegated Management in PEM format. Must be an RSA key. Defaults to `classpath:sensitive/dm.key.pem`, if not provided. | - `file:path/to/dm.key.pem`<br>- `classpath:path/to/dm.key.pem` |

Example usage:
```shell
java -jar armis-cli.jar --armis.action=manage --armis.sd-key=file:ssd.key.hex --armis.dm-key=file:dm.key.pem --armis.steps[0]=list
```

<a name="action-manage-aggregation"></a>
#### Command aggregation

Additional card contents management commands can be aggregated in order to perform multiple steps in sequence.

Example usages:
```shell
java -jar armis-cli.jar --armis.action=manage --armis.sd-key=file:ssd.key.hex --armis.dm-key=file:dm.key.pem --armis.steps=list,unload-all,list
java -jar armis-cli.jar --armis.action=manage --armis.sd-key=file:ssd.key.hex --armis.dm-key=file:dm.key.pem --armis.steps[0]="load:{cap-file:'file:path/to/file.cap'}" --armis.steps[1]=list --armis.steps[2]="install:{package-aid:'0102030405',applet-aid:'010203040506'}" --armis.steps[3]=list
```

<a name="known-issues"></a>
## Known issues

- False positive "Invalid LV" ERROR messages in logs
- Exception 'module java.smartcardio does not "opens sun.security.smartcardio" to unnamed module' when running on Java
  17 and up; as a work-around, add the following runtime option to `java` commands:
  ```
  --add-opens java.smartcardio/sun.security.smartcardio=ALL-UNNAMED
  ```
