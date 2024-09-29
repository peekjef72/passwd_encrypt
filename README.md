<meta name="google-site-verification" content="W1W5S9kNhNL2ZcTEZJ6lMZpAGeNnka6I3iIVFhiUO-I" />

# passwd_encrypt

This tool can encrypt clear password that you don't want to expose in config file.

It can be used in conjonction with [httpapi_exporter](https://github.com/peekjef72/httpapi_exporter) and [sql_exporter](https://github.com/peekjef72/sql_exporter).

# Build

## use promu tool

```shell
go install github.com/prometheus/promu@latest

$GOBIN/promu build

```
This will build the passwd_encrypt tool to crypt/decrypt ciphertext with a shared passphrase.

# Usage

## arguments

```shell
$ ./passwd_encrypt -h
usage: passwd_crypt [<flags>]

encrypt password wyth a shared key.


Flags:
  -h, --[no-]help     Show context-sensitive help (also try --help-long and --help-man).
  -d, --[no-]decrypt  Decrypt the provided password with key.
  -x, --[no-]hexa     Encode password in hexastring.(default base64).
  -q, --[no-]quiet    only ouput the result (no explaination text).
  -V, --[no-]version  Show application version.

```

## Usage password encryption

If you don't want to write the users' password in clear text in config file (targets files on the exporter), you can encrypt them with a shared password.

How it works:
- choose a shared password (passphrase) of 16 24 or 32 bytes length and store it your in your favorite password keeper (keepass for me).
- use passwd_encrypt tool:

    ```bash
    ./passwd_encrypt 
    give the key: must be 16 24 or 32 bytes long
    enter key: 0123456789abcdef 
    enter password: mypassword
    Encrypting...
    Encrypted message hex: CsG1r/o52tjX6zZH+uHHbQx97BaHTnayaGNP0tcTHLGpt5lMesw=
    $
    ```

- set the user password in the target file or in auth_configs part:

    ```yaml
    name: hp3parhost
    scheme: https
    host: "1.2.3.4"
    port: 8080
    baseUrl: /api/v1
    auth_config:
      # mode: basic(default)|token|[anything else:=> user defined login script]
      user: <user>
      # password: "/encrypted/base64_encrypted_password_by_passwd_crypt_cmd"
      password: /encrypted/CsG1r/o52tjX6zZH+uHHbQx97BaHTnayaGNP0tcTHLGpt5lMesw=
    ```

    or

    ```yaml
    auth_configs:
      <auth_name>:
        # mode: basic|token|[anything else:=> user defined login script]
        mode: <mode>
        user: <user>
        # password: "/encrypted/base64_encrypted_password_by_passwd_crypt_cmd"
        password: /encrypted/CsG1r/o52tjX6zZH+uHHbQx97BaHTnayaGNP0tcTHLGpt5lMesw=

    ```

- set the shared passphrase in prometheus config (either job or node file)

  * prometheus jobs with target files:
    ```yaml
    #--------- Start prometheus hp3par exporter  ---------#
    - job_name: "hp3par"
        metrics_path: /metrics
        file_sd_configs:
        - files: [ "/etc/prometheus/hp3par_nodes/*.yml" ]
        relabel_configs:
        - source_labels: [__address__]
            target_label: __param_target
        - source_labels: [__tmp_source_host]
            target_label: __address__

    #--------- End prometheus hp3par exporter ---------#
    ```

    ```yaml
    - targets: [ "hp3par_node_1" ]
    labels:
        __tmp_source_host: "hp3par_exporter_host.domain.name:9321"
    # if you have activated password encrypted passphrass
        __param_auth_key: 0123456789abcdef
        host: "hp3par_node_1_fullqualified.domain.name"
        # custom labels…
        environment: "DEV"
    ```

## usage in quiet mode

if you want to encrypt/decrypt passwords on the fly without user prompt it is possible to use environment variables :
  * CIPHER_KEY : to set the shared secret
  * PASSWORD : to set the password to encrypt/decrypt

  e.g.: to crypt

  ```shell
  $ CIPHER_KEY=0123456789ABCDEF PASSWORD=my_secret_password ./passwd_encrypt -q
  /encrypted/Egj8Jfmqn/DKQTSiS251xMpQmEroLOw5HRbQUcPBHHD51RxsWCVRrcASvJGFbw==
  $
  ```

  e.g.: to decrypt

  ```shell
  $ CIPHER_KEY=0123456789ABCDEF PASSWORD="/encrypted/Egj8Jfmqn/DKQTSiS251xMpQmEroLOw5HRbQUcPBHHD51RxsWCVRrcASvJGFbw==" ./passwd_encrypt -qd
  my_secret_password
  $
  ```

