---
title: "RED Format: Protecting Credentials"
permalink: /docs/red-format-protecting-credentials
---

A typical RED file contains information to connect, various data management systems, often using secret user credentials for authorization. In the example below a `username` and a `password` are given in the `access` information of an SSH SFTP connector.

```yaml
outputs:
  file_one:
    class: File
    connector:
      pyModule: "red_connector_ssh.sftp"
      pyClass: "Sftp"
      access:
        host: "example.com"
        port: 22
        username: "myusername"
        password: "mypassword"
        fileDir: "/home/username/files"
        fileName: "file_one.txt"
  file_two:
    class: File
    connector:
      pyModule: "red_connector_ssh.sftp"
      pyClass: "Sftp"
      access:
        host: "example.com"
        port: 22
        username: "myusername"
        password: "mypassword"
        fileDir: "/home/username/files"
        fileName: "file_two.txt"
```

RED supports two complementary concepts, *variables* and *protected key* to ensure the safety of credentials. This allows RED files to be easily **stored**, **published** or **shared**, without compromising any functionality.


## Variables

In the example above, two files are sent to the same SSH server using the same user credentials. Using *variables*, this information can be replaced as follows.

{% raw %}
```yaml
outputs:
  file_one:
    class: File
    connector:
      pyModule: "red_connector_ssh.sftp"
      pyClass: "Sftp"
      access:
        host: "example.com"
        port: 22
        username: "{{ssh_username}}"
        password: "{{ssh_password}}"
        fileDir: "/home/username/files"
        fileName: "file_one.txt"
  file_two:
    class: File
    connector:
      pyModule: "red_connector_ssh.sftp"
      pyClass: "Sftp"
      access:
        host: "example.com"
        port: 22
        username: "{{ssh_username}}"
        password: "{{ssh_password}}"
        fileDir: "/home/username/files"
        fileName: "file_two.txt"
```
{% endraw %}

In this case we replaced both occurrences of `myusername` with the variable `ssh_username` and both occurrences of `mypassword` with `ssh_password`. Of course the variable names can be chosen arbitrarily.

If you are now using CC-FAICE CLI tools like `faice agent red` or `faice exec` it will interactively ask you once for `ssh_username` and once for `ssh_password` to insert the values.

{% raw %}
Variables can only be used with string values, which must be located somewhere under an `access` or `auth` key. The string must start with `{{` and end with `}}`, everything in between is the variable's name.
{% endraw %}


### Fill File

You can store values to be filled into your variables in a separate *fill-file*. A fill-file can be JSON or YAML and should only contain key-value pairs, where the key is the variable name. The following YAML content could be stored in as `secrets.yml`.

```yaml
ssh_username: "myusername"
ssh_password: "mypassword"
```

You can then use `faice agent red` or `faice exec` commands with the `--file-file` option.

```
faice agent red red.yml --file-file=secrets.yml
faice exec red.yml --file-file=secrets.yml
```



## Protected Keys

Protected keys are an additional concept and can be used in RED files. A protected key is a hint for CC-FAICE and CC-Agency, that the corresponding value must be handled with care. This means that these values should not appear in log files and that CC-Agency must delete this information from its database after the processing is done.

Protected keys can only be used with string values, which must be located somewhere under an `access` or `auth` key. Write an underscore in front of the key to mark it as protected.

```yaml
outputs:
  file_one:
    class: File
    connector:
      pyModule: "red_connector_ssh.sftp"
      pyClass: "Sftp"
      access:
        host: "example.com"
        port: 22
        _username: "myusername"
        password: "mypassword"
        fileDir: "/home/username/files"
        fileName: "file_one.txt"
  file_two:
    class: File
    connector:
      pyModule: "red_connector_ssh.sftp"
      pyClass: "Sftp"
      access:
        host: "example.com"
        port: 22
        _username: "myusername"
        password: "mypassword"
        fileDir: "/home/username/files"
        fileName: "file_two.txt"
```

Here both occurences of `username` have been changed to `_username`, such that both occurrences of `myusername` will be treated as secrets. Please note, that marking one occurrence of `username` as `_username` does not automatically protect the other occurence of `username`.

The key `password` is a special case and therefore **always** considered to be a **protected** key. You could write `_password`, but it would be redundant.

{% raw %}
Of course protected keys can and should be used in combination with variables (e.g. `_username: "{{ssh_username}}"`).
{% endraw %}
