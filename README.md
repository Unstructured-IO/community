## Community

Welcome to the Unstructured Community!

We are building an ecosystem of preprepocessing pipeline tools for Data Scientists
and Data Engineers so they may quickly work through the challenge of extracting
structured data from unstructured raw documents.

## Contributions

We welcome contributions! This repo's [Issues](https://github.com/Unstructured-IO/community-tasks/issues)
tracks bugs, features, and enhancement requests for [Unstructured's open source repositories](https://github.com/Unstructured-IO/).

## Key Concepts

### Bricks

Bricks are the "blocks" or Python functions from which preprocessing pipelines are made, and are organized
in the [Unstructured](https://github.com/Unstructured-IO/unstructured) library. These collectively form
the Swiss Army knife that Python developers can use to extract structured data from raw documents into
the format that they want. They may be used independently of any other Unstructured repos  under the
terms of its license. `pip install unstructured` and you are good to go.

### Preprocessing pipeline APIs

A preprocessing pipeline API (or just "pipeline API") is a notebook that includes a Python function
capble of transforming a raw document to structured data. By following the documented conventions,
FastAPI APIs may be auto-generated from a pipeline notebook.

See [pipeline-sec-filings](https://github.com/Unstructured-IO/pipeline-sec-filings/) for an example repo
that includes a preprocessing pipeline API and auto-generated FastAPI.

### Developer tools for generating FastAPIs

The [unstructured-api-tools](https://github.com/Unstructured-IO/unstructured-api-tools) library includes the
tooling required to create FastAPIs from pipeline notebooks.
