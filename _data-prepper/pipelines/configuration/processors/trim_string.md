---
layout: default
title: trim_string
parent: Processors
grand_parent: Pipelines
nav_order: 120
---

# trim_string

The `trim_string` processor removes white space from the beginning and end of a key and is a [mutate string](https://github.com/opensearch-project/data-prepper/tree/main/data-prepper-plugins/mutate-string-processors#mutate-string-processors) processor. The following table describes the option you can use to configure the `trim_string` processor.

<!--
This table is autogenerated. Do not edit it.
- name: trim_string
- pluginType: processor
- source: https://github.com/opensearch-project/data-prepper/blob/c4455a7785bc2da4358067c217be7085e0bc8d0f/data-prepper-plugins/mutate-string-processors/src/main/java/org/opensearch/dataprepper/plugins/processor/mutatestring/WithKeysConfig.java
-->

Option | Required | Type | Description
:--- | :--- | :--- | :---
with_keys | Yes | List | A list of keys to trim the white space from.

<!---## Configuration

Content will be added to this section.

## Metrics

Content will be added to this section.--->