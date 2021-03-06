---
title: Manifest Files Specification
permalink: /extend/common-interface/manifest-files/
---

* TOC
{:toc}

A manifest file contains additional information about tables and files injected to the
[`/data/in` folders](/extend/common-interface/folders/).
It also provides a way to specify options for tables and files transferred back to Storage from `/data/out`
folders. Manifest files have the `.manifest` suffix to the original file.

All files in `/data/in` have the manifest file generated by us. For files generated by your code
in `/data/out`, the manifest file **is optional**. Also, keep in mind that all manifests have a lower priority
than input and output mapping.

## Format

The format of the manifest file is always *JSON*. The manifest
file always has the `.manifest` extension. This applies to files with multiple extensions as well, so the following
filenames are expected:

| Data File Name | Manifest File Name       |
|----------------|--------------------------|
| myfile         | myfile.manifest          |
| myfile.csv     | myfile.csv.manifest      |
| myfile.csv.gz  | myfile.csv.gz.manifest   |

## Examples

### Tables

#### `/data/in/tables` manifests
An input table manifest stores metadata about a downloaded table. For example, a table
with the ID `in.c-docker-demo.data` will be downloaded into
`/in/tables/in.c-docker-demo.data.csv` (unless stated otherwise in the
[input mapping](/extend/common-interface/config-file/) and a manifest file
'/in/tables/in.c-docker-demo.data.csv.manifest' will be created with the following
contents:

{% highlight json %}
{
    "id": "in.c-docker-demo.data",
    "uri": "https://connection.keboola.com//v2/storage/tables/in.c-docker-demo.data",
    "name": "data",
    "primary_key": [],
    "created": "2015-01-25T01:35:14+0100",
    "last_change_date": "2015-01-25T01:35:14+0100",
    "last_import_date": "2015-01-25T01:35:14+0100",
    "columns": [
        "id",
        "name",
        "text"
    ],
    "metadata": [
        {
            "id": "228956",
            "key": "KBC.createdBy.component.id",
            "value": "keboola.python-transformation",
            "provider": "system",
            "timestamp": "2017-05-26 00:39:07"
        }
    ],
    "column_metadata": {
        "id": [],
        "name": [],
        "text": []
    }
}
{% endhighlight %}

The `name` node refers to the name of the component configuration.
The `metadata` and `column_metadata` fields contains
[Metadata](https://keboola.docs.apiary.io/#reference/metadata) for the table and its columns.

#### `/data/out/tables` manifests

An output table manifest sets options for transferring a table to Storage. The following examples list available
manifest fields; **all of them are optional**. The `destination` field overrides the table name generated
from the file name; it can (and commonly is) overridden by the end-user configuration. The `columns` option defines
the columns of the imported table. If the `columns` option is provided, then the CSV files are **assumed to be headless**.
If you the component is producing [Sliced tables](/extend/common-interface/folders/#sliced-tables), then they are always
assumed to be headless and you *have to* use the `columns` option.

{% highlight json %}
{
    "destination": "out.c-main.Leads",
    "columns": ["column1", "column2", "column3"],
    "incremental": true,
    "primary_key": ["column1", "column2"],
    "delimiter": "\t",
    "enclosure": "\"",
    "metadata": ...,
    "column_metadata": ...
}
{% endhighlight %}

Additionally, the following options can be specified:

{% highlight json %}
{
    ...
    "delete_where_column": "column name",
    "delete_where_values": ["value1", "value2"],
    "delete_where_operator": "eq"
}
{% endhighlight %}

The options will cause the specified rows to be deleted from the source table before the new
table is imported. See an [example](/extend/common-interface/config-file/#output-mapping---delete-rows).
Using this option makes sense only with [incremental loads](/extend/generic-extractor/incremental/).

The `metadata` and `column_metadata` fields allow you to set
[Metadata](https://keboola.docs.apiary.io/#reference/metadata) for the table and its columns.
The `metadata` field corresponds to the [Table Metadata API call](https://keboola.docs.apiary.io/#reference/metadata/table-metadata/create-or-update).
The `column_metadata` field corresponds to the [Column Metadata API call](https://keboola.docs.apiary.io/#reference/metadata/column-metadata/create-or-update).
In both cases, the `key` and `value` are passed directly to the API; the `provider` value is
filled by the Id of the running component (e.g., `keboola.ex-db-snowflake`).

{% highlight json %}
{
    ...,
    "metadata": [
        {
            "key": "an.arbitrary.key",
            "value": "Some value"
        },
        {
            "key": "another.arbitrary.key",
            "value": "A different value"
        }
    ],
    "column_metadata": {
        "column1": [
            {
                "key": "yet.another.key",
                "value": "Some other value"
            }
        ]
    }
}
{% endhighlight %}

### Files

#### `/data/in/files` manifests

An input file manifest stores metadata about a downloaded file.

{% highlight json %}
{
    "id": 75807657,
    "created": "2015-01-14T00:47:00+0100",
    "is_public": false,
    "is_sliced": false,
    "is_encrypted": true,
    "name": "fooBar.jpg",
    "size_bytes": 563416,
    "tags": [
        "tag1",
        "tag2"
    ],
    "max_age_days": 15
}
{% endhighlight %}

#### `/data/out/files` manifests

An output file manifest sets options for transferring a file to Storage. The following example lists available
manifest fields; all of them are optional.

{% highlight json %}
{
    "is_public": true,
    "is_permanent": true,
    "is_encrypted": true,
    "notify": false,
    "tags": [
        "image",
        "pie-chart"
    ]
}
{% endhighlight %}

These parameters can be used (taken from [Storage API File Import](https://keboola.docs.apiary.io/#reference/files/upload-file/create-file-resource)):

- If `is_permanent` is false, the file will be automatically deleted after 15 days.
- When `notify` is true, the members of the project will be notified that a file has been uploaded to the project.

### S3 Staging
When using [AWS S3 for direct data exchange](/extend/common-interface/folders/#exchanging-data-via-s3),
the manifest files will contain an additional `s3` section with
credentials for downloading the actual file data.

{% highlight json %}
{
    "id": "in.c-docker-demo.data",
    ...
    "s3": {
        "isSliced": true,
        "region": "us-east-1",
        "bucket": "kbc-sapi-files",
        "key": "exp-2/1581/table-exports/in/c-docker-test/test/243100072.csv.gzmanifest",
        "credentials": {
            "access_key_id": "ASI...CDQ",
            "secret_access_key": "tCE..I+T",
            "session_token": "Ago...POP"
        }
    }
}
{% endhighlight %}

If the file is sliced and you need to merge it into a single file, read through the guide to
[working with sliced files](/integrate/storage/api/import-export/#working-with-sliced-files).
In that case, the `key` points to another manifest, which contains a list of sliced files.

Note: Exchanging data via AWS S3 is currently available only for input mapping.

### ABS Staging
When using [Azure Blob Storage for direct data exchange](/extend/common-interface/folders/#exchanging-data-via-abs),
the manifest files will contain an additional `abs` section with
credentials for downloading the actual file data.

{% highlight json %}
{
    "id": "in.c-docker-demo.data",
    ...
    "abs": {
        "is_sliced": true,
        "region": "us-east-1",
        "container": "exp-2-export-7647-627703071-in-c-docker-test-test",
        "name": "627703071.csv.gzmanifest",
        "credentials": {
            "sas_connection_string": "BlobEndpoint=https://kbcfsdxcgtsezztoqc.blob.core.windows.net;SharedAccessSignature=sv=2017-11-09&sr=c&st=2020-08-27T08:42:08Z&se=2020-08-27T20:42:08Z&sp=rl&sig=UJW4DPh%2Baaaaaaaaaa",
            "expiration": "2020-08-27T22:42:08+0200"
        }
    }
}
{% endhighlight %}

If the file is sliced and you need to merge it into a single file, read through the guide to
[working with sliced files](/integrate/storage/api/import-export/#working-with-sliced-files).
In that case, the `name` points to another manifest, which contains a list of sliced files.

Note: Exchanging data via Azure ABS is currently available only for input mapping.
