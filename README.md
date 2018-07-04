Ansible Role to Download Files
=========

Ansible role to download files from different locations with or without authentication.

Role Variables
--------------

Variables are defined in `defaults/main.yml` as well as `vars/`. Based on the operating system family, version, and installation method, variables will be set the appropriate values.

| Name              | Default Value       | Description          |
|-------------------|---------------------|----------------------|
| `autorun` | `False`  | Boolean to define if the role "autorun". Useful when you want to have dependencies solved by galaxy (`meta/main.yml`) but don't want it to run automatically.  |
| `download` | `curl`  | Options are: `mvn` (Maven Repository), `gs` (Google Storage), `s3` (Amazon S3), `asb` (Azure StorageBlob) and `curl` (Standard URL). |
| `download_user` | `None` | Username if file access requires authentication. |
| `download_pass` | `None` | Password if file access requires authentication. |
| `download_from` | `None` | The location (`s3`, `gs`, `asb`) or url (`curl`, `mvn`) to download from. |
| `download_from_bucket` | `None` | The bucket to download from (`gs`, `s3`, `asb`) |
| `download_from_region` | `None` | The region to download from (`gs`, `s3`, `asb`) |
| `download_to` | `/tmp` | The location to save in the local filesystem. |
| `download_secure` | `True` | Bool to define if SSL certificates should be valid. |
| `download_overwrite` | `False` | Bool to define if existing files should be overwritten or not. |
| `download_mvn_extension` | `None` | Maven Artifact Extension. |
| `download_mvn_group_id` | `None` | Maven Artifact Group ID. |
| `download_mvn_artifact_id` | `None` | Maven Artifact ID. |
| `download_mvn_version` | `None` | Maven Artifact Version. |
| `download_asb_resource_group` | `None` | Azure Storage Blob: Resource Group Name. |
| `download_asb_tenant` | `None` | Azure Storage Blob: Tenant. |
| `download_asb_subscription_id` | `None` | Azure Storage Blob: Subscription ID. |
| `download_asb_ad_user` | `None` | Azure Storage Blob: Active Directory Username. |
| `download_asb_ad_pass` | `None` | Azure Storage Blob: Active Directory Password. |


Examples
------------

Follow below different examples and ways to use this role.

>Playbook: Downloading from Google Storage

```YAML
---
- name: "download: Google Storage"
  hosts: libvirt
  gather_facts: false
  become: true
  become_user: qemu

  vars:
    download: "gs"
    download_user: "{{ lookup('ENV', 'GS_ACCESS_KEY_ID') }}"
    download_pass: "{{ lookup('ENV', 'GS_SECRET_ACCESS_KEY') }}"
    download_file: "image.qcow2"
    download_from: "/my/folder/path/to/{{ download_file }}"
    download_from_bucket: "instances-image-store"
    download_from_region: "europe-west1"
    download_to: "/var/lib/libvirt/images/{{ download_file }}"

  roles:
    - role: victorock.download
      autorun: true

```

>Playbook: Downloading from Amazon S3

```YAML
---
- name: "download: Amazon S3"
  hosts: libvirt
  gather_facts: false
  become: true
  become_user: qemu

  tasks:
    - name: "Download private image to libvirt images folder"
      include_role:
        name: victorock.download
        tasks_from: s3
      vars:
        download_user: "{{ lookup('ENV', 'AWS_ACCESS_KEY_ID') }}"
        download_pass: "{{ lookup('ENV', 'AWS_SECRET_ACCESS_KEY') }}"
        download_file: "image.qcow2"
        download_from: "/my/folder/path/to/{{ download_file }}"
        download_from_bucket: "instances-image-store"
        download_from_region: "eu-west-1"
        download_to: "/var/lib/libvirt/images/{{ download_file }}"
```

>Playbook: Downloading anonymously from URL

>> Example1:

```YAML
---
- name: "download: Curl"
  hosts: libvirt
  gather_facts: false
  become: true
  become_user: qemu

  vars:
    download: "curl"
    download_file: "CentOS-7-x86_64-GenericCloud.qcow2"
    download_file: "CentOS-7-x86_64-GenericCloud.qcow2"
    download_from: "http://cloud.centos.org/centos/7/images/{{ download_file }}"
    download_to: "/var/lib/libvirt/images/{{ download_file }}"

  roles:
    - role: victorock.download
      autorun: true
```

>> Example2:

```YAML
---
- name: "download: Curl"
  hosts: libvirt
  gather_facts: false
  become: true
  become_user: qemu

  tasks:
    - name: "Download Centos7 Image to libvirt images folder"
      include_role:
        name: victorock.download
        tasks_from: curl
      vars:
        download: "curl"
        download_file: "CentOS-7-x86_64-GenericCloud.qcow2"
        download_from: "http://cloud.centos.org/centos/7/images/{{ download_file }}"
        download_to: "/var/lib/libvirt/images/{{ download_file }}"

```


>Playbook: Downloading maven artifact.

>> Example1:

```YAML
---
- name: "download: Maven Artifact"
  hosts: tomcat:&myapp
  gather_facts: false
  become: true
  become_user: tomcat

  vars:
    download: "mvn"
    download_user: "myartifactoryUser"
    download_pass: "myartifactoryPass"
    download_from: "http://artifactory.acme.com/artifactory/libs-release-local"
    download_mvn_extension: "war"
    download_mvn_group_id: "com.acme"
    download_mvn_artifact_id: "myapp"
    download_mvn_version: "1.0"
    download_to: "/usr/share/tomcat/webapps/{{download_mvn_artifact_id}}.{{download_mvn_extension}}"


  roles:
    - role: victorock.download
      autorun: true
```

>> Example2:

```YAML
---
- name: "download: Maven Artifact"
  hosts: tomcat:&myapp
  gather_facts: false
  become: true
  become_user: tomcat

  tasks:
    - name: "Download Maven Artifact to Tomcat auto-deploy"
      include_role:
        name: victorock.download
        tasks_from: mvn
      vars:
        download: "mvn"
        download_user: "myartifactoryUser"
        download_pass: "myartifactoryPass"
        download_from: "http://artifactory.acme.com/artifactory/libs-release-local"
        download_mvn_extension: "war"
        download_mvn_group_id: "com.acme"
        download_mvn_artifact_id: "myapp"
        download_mvn_version: "1.0"
        download_to: "/usr/share/tomcat/webapps/{{download_mvn_artifact_id}}.{{download_mvn_extension}}"

```

Dependencies
------------

None

Requirements
------------

None

License
------------

GPLv3

Author
------------

Victor da Costa (@victorock)
