# Preprocessing Pipeline and API Framework

**Coming Soon!** Support for gzip'ed files and multi-file json responses. See [the spec](specs/Pipeline-API-gz-spec.md).

## Introduction

This document describes how preprocessing pipelines (or just "pipelines") and their derived APIs are supported, and the git repository conventions that support pipeline development. It is a working specification: not all conventions are yet supported by [unstructured-api-tools](https://github.com/Unstructured-IO/unstructured-api-tools).

Note that some pipelines may depend on a model inference API call, i.e. an API call from within API. The architecture for model inferences is not considered in this document.

## Terminology

**preprocessing pipeline** - AKA **pipeline** - the end-to-end flow. Given a raw document input(s) and (optional) parameters, return formatted data for a particular use case (labeling, Hugging Face API, internal ETL).

**pipeline family** - a collection of pipelines relevant to a particular document type. Pipelines in the same pipeline family live in the same repo and share the same dependencies.

**initial structured data (ISD)** - given a raw document, Unstructured.io's initial structured json of that document.

**brick** - a single step/transformation within a pipeline, e.g., remove boilerplate text or stage data for a labeling task. Many bricks for common processing steps are defined in the [unstructured](https://github.com/Unstructured-IO/unstructured) library.

## API Specification

By convention, the URL path for an HTTP API endpoint is formatted like:

   /**\<pipeline-family>**/**v\<semver-version>**/**\<pipeline-name>**

API endpoints are versioned by [semver](https://semver.org/) where the semver version applies to the pipeline family as a whole. Thus, the endpoint for all pipelines in a pipeline family all have the same version, which is parsed from `preprocessing-pipeline-family.yaml`.

### API Inputs

API endpoints expect HTTP `POST` requests with parameters and files specified in a `multipart/form-data` payload. The reason for using multipart/form-data is because it allows for posting multiple binary files and additional input parameters in one request.

For example, the following `curl` command posts a file with an additional parameter for `output_schema`:

```bash
curl -X 'POST' \
  'https://<hostname>/<api-path>' \
  -H 'Accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'files=@my-cool-file.pdf' \
  -F 'output_schema=labelstudio'
```

Or, in Python using `requests` as follows:

```python
import requests

response = requests.post(
  "https://<hostname>/<api-path>",
  files={
    "files": open("my-cool-file.pdf", "rb"),
    "output_schema": "labelstudio"
  }
)
```

If a pipeline notebook's `pipeline_api` function has `text: str` as the first argument, the implication is that one or more plain text files are expected in the auto-generated FastAPI route. E.g., it would expect a request that includes a `text_files` parameter:


```bash
curl -X 'POST' \
  'https://<hostname>/<api-path>' \
  -H 'Accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'text_files=<file-plain-text>' \
  -F 'output_schema=labelstudio'
```

or in Python using `requests` as follows:

```python
import requests

response = requests.post(
  "https://<hostname>/<api-path>",
  files={
    "text_files": "<file-plain-text>",
    "output_schema": "labelstudio"
  }
)
```

Alternatively, binary `files` may be posted instead of plain text `text_files`.

The API can accept multiple documents in a single call by passing in multiple `files` form fields.

A curl example looks like:


```bash
curl -X 'POST' \
  'https://<hostname>/<api-path>' \
  -H 'Accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'files=@my-cool-file.pdf' \
  -F 'files=@my-other-cool-file.pdf' \
  -F 'output_schema=labelstudio'
```

and in Python


```python
import requests

response = requests.post(
  "<api-url><api-path",
  files={
    "file": [
      open("my-cool-file.pdf", "rb"),
      open("my-other-cool-file.pdf", "rb"),
    ],
    "output_schema": "labelstudio"
  }
)
```

To post multiple files at once to the API use the parameter names `gzip_files` (each file is gzip'ed binary) or `gzip_texts` (each file is gzip'ed text).

For example, to post multiple gzip'ed text files with `curl`, repeat the parameter name for each file:

     -F 'gzip_files=@doc1.pdf.gz;type=text/plain' -F 'gzip_files=@doc2.pdf.gz;type=text/plain'

Refer to this [gist page](https://gist.github.com/cragwolfe/1da738e8710aa2781b20c12f795d00ae ) for additional examples.

The parameters that the HTTP API accepts is driven by the signature of the `pipeline_api` function defined in a pipeline notebook.

`pipeline_api` function may include the kwarg `response_type: str` which indicates that the function returns a payload suitable for csv (a string) or suitable for json (a dict or list object). If present, it must take a default argument of `"text/csv"` or `"application/json"`. The caller of the auto-generated API may then request the content type of the HTTP response through the usual Accept header when only one file is being posted to the api, for example:

```bash
curl -X 'POST' \
  'https://<hostname>/<api-path>' \
  -H 'Accept: text/csv' \
  -H 'Content-Type: multipart/form-data' \
  -F 'text_files=@my-cool-file.txt' \
  -F 'output_schema=labelstudio'
```

or in Python using `requests` as follows:

```python
import requests

response = requests.post(
  "https://<hostname>/<api-path>",
  files={
    "text_files": open("my-cool-file.txt", "rb"),
    "output_schema": "labelstudio",
  },
  headers={"Accept": "application/json"},
)
```

When multiple files are posted to the API, the caller may request the content type (per file within a multipart/mixed response) with the `output_format` parameter. In this case, the `Accept` header must be set to `multipart/mixed`, for example:

```bash
curl -X 'POST' \
  'https://<hostname>/<api-path>' \
  -H 'Accept: multipart/mixed' \
  -H 'Content-Type: multipart/form-data' \
  -F 'text_files=@my-cool-file.txt' \
  -F 'text_files=@my-cool-other-file.txt' \
  -F 'output_schema=labelstudio' \
  -F 'output_format=text/csv'
```

or in Python using `requests` as follows:

```python
import requests

response = requests.post(
  "https://<hostname>/<api-path>",
  files={
    "text_files": [
      open("my-cool-file.txt", "rb"),
      open("my-other-cool-file.txt", "rb"),
    ],
    "output_schema": "labelstudio",
    "output_format": "text/csv",
  },
  headers={"Accept": "multipart/mixed"},
)
```

### API Outputs

By default, an API endpoint returns initial structured data (ISD) as `application/json`. As previously mentioned, a `text/csv` could instead be requested if the `pipeline_api` defines the `response_type`. In addition, other schemas for a given content type may be requested through the use of the `output_schema` parameter. I.e., multiple json schemas may be supported for a given API. The ISD schema in Python is a list of dictionaries with at least the following fields:

- `text` (`str`) - The raw text for the section
- `type` (`str`) - The type of text the section represents (e.g. title, narrative text, table)

However, ISD schemas for different pipeline APIs may include additional fields as well.

The request's `Accept` header specifies the MIME type of the response the client expects. If a client specifies a MIME type in the `Accept` header that the API does not support, the API returns a `406: Not Acceptable` response.

## Github Repository Conventions for Pipeline Families

The directory layout for a preprocessing pipeline family follows, where "sec_filings" is the name of the pipeline family.

```
 - preprocessing-pipeline-family.yaml
 + pipeline-notebooks/
   - pipeline-<api-name-1>.ipynb
   - pipeline-<api-name-2>.ipynb
   - ...
 + exploration-notebooks/
   - exploration-notebook-1.ipynb
   - exploration-notebook-2.ipynb
   - ...
 + prepline_sec_filings/ (yes, prepline_)
   + your_module.py (optional)
   + api/
     - api_name_1.py (generated by script)
     - api_name_2.py (generated by script)
 + test_prepline_sec_filings/ (optional)
    + api
 + requirements/
 - Dockerfile
 - CHANGELOG.md
 ```

**preprocessing-pipeline-family.yaml** includes key configuration parameters, especially version and pipeline family name.

Pipelines are defined by their jupyter notebooks and tooling is provided by [unstructured-api-tools](https://github.com/Unstructured-IO/unstructured-api-tools#conversion-from-pipeline_api-to-fastapi) to convert the notebook to a FastAPI API.

Any notebook with a filename formatted as `pipeline-*.ipynb` is the jupyter notebook version of an API. By convention, a cell that begins with the line
 ```
 # pipeline-api
 ```
is included in the FastAPI module that defines the API. The path for the module within this repo is `prepline_<family_name>/api/<api_name>.py`.
 
Each pipeline notebook must define a function called `pipeline_api`, which is invoked during the API call for that pipeline. The `pipeline_api` function has a single input representing the document: either `text` which is a string representing contents of a text document or `fileb` which is the handle to an open read-only binary file. The `pipeline_api` function may also have optional keyword args to specify options such as `output_schema` or `response_type`.

`<api_name>.py` registers a REST endpoint for the pipeline. The code generation process creates a single FastAPI application for the pipeline family, with an endpoint for each pipeline.

The `unstructured_api_tools` repo houses the logic for converting pipeline notebooks to scripts.

Executing `make generate-api` from the root directory of the repo generates the HTTP APIs, i.e. creates prepline_<family_name>/api/<api_name>.py files.

### Releases

The latest stable or production release is indicated by the version in preprocessing-pipeline-family.yaml.

However, commits between releases may be merged to the main branch that are not intended to be released by
including a -dev suffix in the version in the CHANGELOG, e.g. `## 0.1.1-dev4`.

### Repo Names

By convention, all pipeline repositories begin with the prefix `pipeline-`.
