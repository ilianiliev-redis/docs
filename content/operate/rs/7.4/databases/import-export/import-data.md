---
Title: Import data into a database
alwaysopen: false
categories:
- docs
- operate
- rs
description: You can import export or backup files of a specific Redis Enterprise
  Software database to restore data. You can either import from a single file or from
  multiple files, such as when you want to import from a backup of a clustered database.
linktitle: Import data
weight: 10
url: '/operate/rs/7.4/databases/import-export/import-data/'
---
You can import, [export]({{< relref "/operate/rs/7.4/databases/import-export/export-data" >}}),
or [backup]({{< relref "/operate/rs/7.4/databases/import-export/schedule-backups" >}})
files of a specific Redis Enterprise Software database to restore data.
You can either import from a single file or from multiple files,
such as when you want to import from a backup of a clustered database.

{{< warning >}}
Importing data erases all existing content in the database.
{{< /warning >}}

## Import data into a database

To import data into a database using the Cluster Manager UI:

1. On the **Databases** screen, select the database from the list, then select **Configuration**.
1. Click {{< image filename="/images/rs/buttons/button-toggle-actions-vertical.png#no-click" alt="Toggle actions button" width="22px" class="inline" >}} to open a list of additional actions.
1. Select **Import**.
1. Select the tab that corresponds to your storage location type and enter the location details.

    See [Supported storage locations](#supported-storage-locations) for more information about each storage location type.
1. Select **Import**.

## Supported storage locations {#supported-storage-services}

Data can be imported from a local mount point, transferred to [a URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) using FTP/SFTP, or stored on cloud provider storage.

When importing from a local mount point or a cloud provider, import locations need to be available to [the group and user]({{< relref "/operate/rs/7.4/installing-upgrading/install/customize-user-and-group.md" >}}) running Redis Enterprise Software, `redislabs:redislabs` by default.  

Redis Enterprise Software needs the ability to view objects in the storage location. Implementation details vary according to the provider and your configuration. To learn more, consult the provider's documentation.

The following sections provide general guidelines.  Because provider features change frequently, use your provider's documentation for the latest info.

### FTP server

Before importing data from an FTP server, make sure that:

- Your Redis Enterprise cluster can connect and authenticate to the FTP server.
- The user that you specify in the FTP server location has permission to read files from the server.

To import data from an FTP server, set **RDB file path/s** using the following syntax:

```sh
[protocol]://[username]:[password]@[host]:[port]/[path]/[filename].rdb
```

Where:

- *protocol*: the server's protocol, can be either `ftp` or `ftps`.
- *username*: your username, if needed.
- *password*: your password, if needed.
- *hostname*: the hostname or IP address of the server.
- *port*: the port number of the server, if needed.
- *path*: the file's location path.
- *filename*: the name of the file.

Example: `ftp://username:password@10.1.1.1/home/backups/<filename>.rdb`

Select **Add path** to add another import file path.

### Local mount point

Before importing data from a local mount point, make sure that:

- The node can connect to the server hosting the mount point.

- The `redislabs:redislabs` user has permission to read files on the local mount point and on the destination server.

- You must mount the storage in the same path on all cluster nodes. You can also use local storage, but you must copy the imported files manually to all nodes because the import source folders on the nodes are not synchronized.

To import from a local mount point:

1. On each node in the cluster, create the mount point:
    1. Connect to the node's terminal.
    1. Mount the remote storage to a local mount point.

        For example:

        ```sh
        sudo mount -t nfs 192.168.10.204:/DataVolume/Public /mnt/Public
        ```

1. In the path for the import location, enter the mount point.

    For example: `/mnt/Public/<filename>.rdb`

As of version 6.2.12, Redis Enterprise reads files directly from the mount point using a [symbolic link](https://en.wikipedia.org/wiki/Symbolic_link) (symlink) instead of copying them to a temporary directory on the node.

Select **Add path** to add another import file path.

### SFTP server

Before importing data from an SFTP server, make sure that:

- Your Redis Enterprise cluster can connect and authenticate to the SFTP server.
- The user that you specify in the SFTP server location has permission to read files from the server.
- The SSH private keys are specified correctly. You can use the key generated by the cluster or specify a custom key.
    
   To use the cluster auto generated key:
    
    1. Go to **Cluster > Security > Certificates**.

    1. Expand **Cluster SSH Public Key**.
    
    1. Download or copy the cluster SSH public key to the appropriate location on the SFTP server.

        Use the server documentation to determine the appropriate location for the SSH public key.

To import data from an SFTP server, enter the SFTP server location in the format:

```sh
[protocol]://[username]:[password]@[host]:[port]/[path]/[filename].rdb
```

Where:

- *protocol*: the server's protocol, can be either `ftp` or `ftps`.
- *username*: your username, if needed.
- *password*: your password, if needed.
- *hostname*: the hostname or IP address of the server.
- *port*: the port number of the server, if needed.
- *path*: the file's location path.
- *filename*: the name of the file.

Example: `sftp://username:password@10.1.1.1/home/backups/[filename].rdb`

Select **Add path** to add another import file path.

### AWS Simple Storage Service {#aws-s3}

Before you choose to import data from an [Amazon Web Services](https://aws.amazon.com/) (AWS) Simple Storage Service (S3) bucket, make sure you have:

- The path to the file in your bucket in the format: `s3://[bucketname]/[path]/[filename].rdb`
- [Access key ID and Secret access key](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey) for an [IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html#id_users_create_console) with permission to read files from the bucket. 

In the Redis Enterprise Software Cluster Manager UI, when you enter the export location details:

- Select **AWS S3**.

- In the **RDB file path/s** field, enter the path of your bucket. Select **Add path** to add another import file path.

- In the **Access key ID** field, enter the access key ID.

- In the **Secret access key** field, enter the secret access key.

You can also connect to a storage service that uses the S3 protocol but is not hosted by Amazon AWS. The storage service must have a valid SSL certificate.

To connect to an S3-compatible storage location:

1. Configure the S3 URL with [`rladmin cluster config`]({{<relref "/operate/rs/7.4/references/cli-utilities/rladmin/cluster/config">}}): 

    ```sh
    rladmin cluster config s3_url <URL>
    ```

    Replace `<URL>` with the hostname or IP address of the S3-compatible storage location.

1. Configure the S3 CA certificate:

    ```sh
    rladmin cluster config s3_ca_cert <filepath>
    ```

    Replace `<filepath>` with the location of the S3 CA certificate `ca.pem`.

### Google Cloud Storage

Before you import data from a [Google Cloud](https://developers.google.com/console/) storage bucket, make sure you have:

- Storage location path in the format: `/bucket_name/[path]/[filename].rdb`
- A [JSON service account key](https://cloud.google.com/iam/docs/creating-managing-service-account-keys#creating) for your account
- A [principal](https://cloud.google.com/storage/docs/access-control/using-iam-permissions#bucket-add) for your bucket with the `client_email` from the service account key and a [role](https://cloud.google.com/storage/docs/access-control/iam-roles) with permissions to get files from the bucket (such as the **Storage Legacy Object Reader** role, which grants `storage.objects.get` permissions)

In the Redis Enterprise Software Cluster Manager UI, when you enter the import location details:

- Select **Google Cloud Storage**.

- In the **RDB file path/s** field, enter the path of your file. Select **Add path** to add another import file path.

- In the **Client ID** field, enter the `client_id` from the service account key.

- In the **Client email** field, enter the `client_email` from the service account key.

- In the **Private key id** field, enter the `private_key_id` from the service account key.

- In the **Private key** field, enter the `private_key` from the service account key.
    Replace `\n` with new lines.

### Azure Blob Storage

Before you choose to import from Azure Blob Storage, make sure that you have:

- Storage location path in the format: `/container_name/[path/]/<filename>.rdb`
- Account name
- An authentication token, either an account key or an Azure [shared access signature](https://docs.microsoft.com/en-us/rest/api/storageservices/delegate-access-with-shared-access-signature) (SAS).

    To find the account name and account key, see [Manage storage account access keys](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-keys-manage).

    Azure SAS support requires Redis Software version 6.0.20.  To learn more about Azure SAS, see [Grant limited access to Azure Storage resources using shared access signatures](https://docs.microsoft.com/en-us/azure/storage/common/storage-sas-overview).

In the Redis Enterprise Software Cluster Manager UI, when you enter the import location details:

- Select **Azure Blob Storage**.

- In the **RDB file path/s** field, enter the path of your file. Select **Add path** to add another import file path.

- In the **Azure Account Name** field, enter your storage account name.

- In the **Azure Account Key** field, enter the storage account key.

## Importing into an Active-Active database

When importing data into an Active-Active database, there are two options:

- [Flush all data]({{< relref "/operate/rs/7.4/databases/import-export/flush#flush-data-from-an-active-active-database" >}}) from the Active-Active database, then import the data into the database.
- Import data but merge it into the existing database.

Because Active-Active databases have a numeric counter data type,
when you merge the imported data into the existing data RS increments counters by the value that is in the imported data.
The import through the Redis Enterprise Cluster Manager UI handles these data types for you.

You can import data into an Active-Active database [from the Cluster Manager UI](#import-data-into-a-database).
When you import data into an Active-Active database, there is a special prompt warning that the imported data will be merged into the existing database.
