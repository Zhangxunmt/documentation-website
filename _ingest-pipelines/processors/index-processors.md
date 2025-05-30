---
layout: default
title: Ingest processors
nav_order: 80
has_children: true
has_toc: false
redirect_from:
   - /api-reference/ingest-apis/ingest-processors/
---

# Ingest processors

Ingest processors are a core component of [ingest pipelines]({{site.url}}{{site.baseurl}}/ingest-pipelines/index/). They preprocess documents before indexing. For example, you can remove fields, extract values from text, convert data formats, or append additional information.

OpenSearch provides a standard [set of ingest processors](#supported-processors) within your OpenSearch installation. For a list of processors available in OpenSearch, use the [Nodes Info]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/nodes-info/) API operation:

```json
GET /_nodes/ingest?filter_path=nodes.*.ingest.processors
```
{% include copy-curl.html %}

To set up and deploy ingest processors, make sure you have the necessary permissions and access rights. See [Security plugin REST API]({{site.url}}{{site.baseurl}}/security/access-control/api/) to learn more.
{:.note}

## Supported processors

Processor types and their required or optional parameters vary depending on your specific use case. OpenSearch supports the following ingest processors. For tutorials on using these processors in an OpenSearch pipeline, go to each processor's respective documentation. 

Processor type | Description
:--- | :--- 
`append` | Adds one or more values to a field in a document. 
`bytes` | Converts a human-readable byte value to its value in bytes.
`community_id` | Generates a community ID flow hash algorithm for the network flow tuples.
`convert` | Changes the data type of a field in a document.
`copy` | Copies an entire object in an existing field to another field.
`csv` | Extracts CSVs and stores them as individual fields in a document. 
`date` | Parses dates from fields and then uses the date or timestamp as the timestamp for a document.
`date_index_name` | Indexes documents into time-based indexes based on a date or timestamp field in a document. 
`dissect` | Extracts structured fields from a text field using a defined pattern. 
`dot_expander` | Expands a field with dots into an object field. 
`drop` |Drops a document without indexing it or raising any errors.
`fail` | Raises an exception and stops the execution of a pipeline. 
`fingerprint` | Generates a hash value for either certain specified fields or all fields in a document. 
`foreach` | Allows for another processor to be applied to each element of an array or an object field in a document.
`geoip` | Adds information about the geographical location of an IP address.
`geojson-feature` | Indexes GeoJSON data into a geospatial field.
`grok` | Parses and structures unstructured data using pattern matching. 
`gsub` | Replaces or deletes substrings within a string field of a document. 
`html_strip` | Removes HTML tags from a text field and returns the plain text content. 
`ip2geo` | Adds information about the geographical location of an IPv4 or IPv6 address.
`join` | Concatenates each element of an array into a single string using a separator character between each element. 
`json` | Converts a JSON string into a structured JSON object. 
`kv` | Automatically parses key-value pairs in a field.
`lowercase` | Converts text in a specific field to lowercase letters.
`pipeline` | Runs an inner pipeline.
`remove` | Removes fields from a document.
`remove_by_pattern` | Removes fields from a document by field pattern.
`rename` | Renames an existing field.
`script` | Runs an inline or stored script on incoming documents. 
`set` | Sets the value of a field to a specified value.
`sort` | Sorts the elements of an array in ascending or descending order.
`sparse_encoding` | Generates a sparse vector/token and weights from text fields for neural sparse search using sparse retrieval. 
`split` | Splits a field into an array using a separator character.
`text_chunking` | Splits long documents into smaller chunks.
`text_embedding` | Generates vector embeddings from text fields for semantic search.
`text_image_embedding` | Generates combined vector embeddings from text and image fields for multimodal neural search.
`trim` | Removes leading and trailing white space from a string field.
`uppercase` | Converts text in a specific field to uppercase letters.
`urldecode` | Decodes a string from URL-encoded format.
`user_agent` | Extracts details from the user agent sent by a browser to its web requests. 

## Processor limit settings

You can limit the number of ingest processors using the cluster setting `cluster.ingest.max_number_processors`. The total number of processors includes both the number of processors and the number of [`on_failure`]({{site.url}}{{site.baseurl}}/ingest-pipelines/pipeline-failures/) processors.

The default value for `cluster.ingest.max_number_processors` is `Integer.MAX_VALUE`. Adding a higher number of processors than the value configured in `cluster.ingest.max_number_processors` will throw an `IllegalStateException`.

## Batch-enabled processors

Some processors support batch ingestion---they can process multiple documents at the same time as a batch. These batch-enabled processors usually provide better performance when using batch processing. For batch processing, use the [Bulk API]({{site.url}}{{site.baseurl}}/api-reference/document-apis/bulk/) and provide a `batch_size` parameter. All batch-enabled processors have a batch mode and a single-document mode. When you ingest documents using the `PUT` method, the processor functions in single-document mode and processes documents in series. Currently, only the `text_embedding` and `sparse_encoding` processors are batch enabled. All other processors process documents one at a time.

## Selectively enabling processors

Processors defined by the [ingest-common module](https://github.com/opensearch-project/OpenSearch/blob/2.x/modules/ingest-common/src/main/java/org/opensearch/ingest/common/IngestCommonPlugin.java) can be selectively enabled by providing the `ingest-common.processors.allowed` cluster setting. If not provided, then all processors are enabled by default. Specifying an empty list disables all processors. If the setting is changed to remove previously enabled processors, then any pipeline using a disabled processor will fail after node restart when the new setting takes effect.
