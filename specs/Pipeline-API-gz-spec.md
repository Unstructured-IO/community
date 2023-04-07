## Pipeline APIs: Support for gzip'ed files and single JSON response payloads for multiple file inputs

The current Pipeline API pattern allows for single or multiple files POST'ed as inputs.

This spec proposes extending this pattern to allow accepting gzip'ed files as well, and, the ability to return an `application/json` response instead of only `multipart/mixed` responses when multiple files are posted.

## Explicitly (or not) Specifying the content type of a gzip'ed payload


### Current Behaviour

The API-framework passes along the `Content-Type` for each file as the `file_content_type` in `pipeline_api`. For example, in a request such as:

```
curl -X 'POST' https://<hostname>/<api-path>' \
  -H 'Accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'files=receipt.pdf' \
  -F 'output_schema=labelstudio'
```
The content type of `receipt.pdf` is included in the multipart/form-data payload, e.g.:

```
--some_random_boundary
Content-Disposition: form-data; name="files"; filename="reciept.pdf"
Content-Type: application/pdf

%PDF-1.7
...
```
 &nbsp; &nbsp; &nbsp; _note: curl adds the Content-Type behind the scenes as in the case above_

This content type is then passed along to `pipeline_api` by the FastAPI framework provided by `unstructured-api-tools`, i.e., the `file_content_type` in

```
pipeline_api(file,
             file_content_type=None,
             filename=None,
             response_schema="isd",
             response_format="application/json,
             ... <other optional mulit-string form parameters,
                  e.g. m_sections>)
```
### Allowing .gz files

Within this framework, a compressed file such as `receipt.pdf.gz`, the `Content-Type` would be posted with a Content-Type of `application/gzip`. No problems there!

### The uncompressed Content-Type

However, that leaves the question of the content-type of the uncompressed file(s). To indicate the uncompressed Content-Type of the all `files` uploaded, the client may pass a form variable, `gz_uncompressed_content_type`.

> The alternative of adding additional form-data headers per file to indicate the "uncompressed content type" would require more client and back end complexity, so is not supported. This may be revisited in the future.

However, if `gz_uncompressed_content_type` is not provided, the `unstructured-api-tools` framework will "guess" the content type with the python `magic` library to pass along as `file_content_type` . E.g.,:

```
import magic

# Create a Magic instance
m = magic.Magic(mime=True)

# Use the Magic instance to get the MIME type of the file
mime_type = m.from_buffer(file_handle.read())
```
_**Warning:** above code not tested!_

## .tar's As Inputs Are Not Supported (yet)

.tar's or .tar.gz's are not yet supported by the framework.

Reasons to avoid .tar input file support:

* Since these APIs are stateless, small batches of files are better than large ones.
* Large or extremely large batches of files should be handled in a workflow-ish manner, which outside the scope of stateless **Transform** API.
* .tar's tend to be large batches of files, which should be discouraged.
* Expanding .tar's (esp. .tar.gz's) into its constituent files on the server side adds additional complexity and risks.
* If an `unstructured-api-tools` pipeline API did accept a .tar, the output is not yet well defined and would need to be spec'ed. I.e., there is no longer an known order of file outputs to return.

That said, this current spec does not preclude adding support for .tar's later if the reasons for doing so outweigh the points above.

_.tar's (typically with not many files) as **outputs** are fair game in the future_ (outside the scope of this spec).

## Enumerating the Use Cases

There are more than a few permutations on how to POST files to an API driven by `unstructured-api-tools` , all of which ultimately pass the files (one file at a time) to `pipeline_api`. The format of the response is also though

For completeness, existing use cases covered by the framework are included below as well as the new possibilities introduced by this spec.

### Single text file, CSV response

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: text/csv' \
  -H 'Content-Type: multipart/form-data' \
  -F 'text_files=<file-plain-text>'
```

### Single Text File, JSON response with particular schema

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'text_files=<file-plain-text> \
  -F 'output_schema=labelstudio'
```

### Multiple Text Files, Multipart/mixed response, each part is a CSV

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: multipart/mixed' \
  -H 'Content-Type: multipart/form-data' \
  -F 'text_files=@my-cool-file.txt' \
  -F 'text_files=@my-other-cool-file.txt' \
  -F 'output_schema=a_particular_csv_schema'
  -F 'output_format=text/csv'
```

### Multiple Text Files, Multipart/mixed Json response

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: multipart/mixed' \
  -H 'Content-Type: multipart/form-data' \
  -F 'text_files=@my-cool-file.txt' \
  -F 'text_files=@my-cool-other-file.txt' \
  -F 'output_schema=labelstudio' \
  -F 'output_format=application/json'
```

### Multiple Text Files and Files as inputs, Multipart/mixed CSV response

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: multipart/mixed' \
  -H 'Content-Type: multipart/form-data' \
  -F 'files=@my-cool-file.pdf' \
  -F 'files=@my-other-cool-file.pdf' \
  -F 'text_files=@my-text-file.txt' \
  -F 'output_format=text/csv'
```

* It's OK to mix Text Files and Files

### Multiple Files, Multipart/mixed CSV response

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: multipart/mixed' \
  -H 'Content-Type: multipart/form-data' \
  -F 'files=@my-cool-file.pdf' \
  -F 'files=@my-other-cool-file.pdf' \
  -F 'output_format=text/csv'
```

### Multiple Files, Multipart/mixed Json response

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: multipart/mixed' \
  -H 'Content-Type: multipart/form-data' \
  -F 'files=@my-cool-file.pdf' \
  -F 'files=@my-other-cool-file.pdf' \
  -F 'output_format=application/json'
```

### Multiple Files, Single Json response (new)

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'files=@my-cool-file.pdf' \
  -F 'files=@my-other-cool-file.pdf' \
  -F 'output_schema=labelstudio'
```

* If `-F 'output_format` is specified, it must also be `application/json`

### Single Text File compressed, Json response  (new)

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'text_files=a_wishlist.txt.gz'
```
* Assumes the Content-Type for the form part of `text_files` is `application/gzip`

### Multiple Text Files compressed, Multipart/mixed CSV response (new)

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: multipart/mixed' \
  -H 'Content-Type: multipart/form-data' \
  -F 'text_files=@my-cool-file.txt.gz' \
  -F 'text_files=@my-other-cool-file.txt.gz' \
  -F 'output_schema=a_particular_csv_schema'
  -F 'output_format=text/csv'
```
* Assumes the Content-Type for the form part of `text_files` is `application/gzip`

### Multiple Text Files compressed, Application/json response (new)

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'text_files=@my-cool-file.txt.gz' \
  -F 'text_files=@my-other-cool-file.txt.gz' \
  -F 'text_files=@my-previous-cool-file.txt.gz' \
  -F 'output_schema=isd'
  -F 'output_format=application/json'
```
* Assumes the Content-Type for the form part of `text_files` is `application/gzip`
* The json result is a list with length of the number of files (in this case 3), where each item is the result of a call to `pipeline_api`. The json schema of each item in the list is "isd" (AKA initial structured data).

### Single File compressed, Application/json response (new)

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'gz_uncompressed_content_type=image/png` `
  -F 'files=a_beach.png.gz'
```
In this case, the API is accepting a .png and responding with a .jpeg.

* Assumes the Content-Type for the form part of `files` is `application/gzip`
* Passing `gz_uncompressed_content_type` is optional but preferred

### Multiple Files compressed, Multipart/mixed response (new)

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: multipart/mixed' \
  -H 'Content-Type: multipart/form-data' \
  -F 'gz_uncompressed_content_type=applicaton/pdf` `
  -F 'files=@my-cool-file.pdf.gz' \
  -F 'files=@my-other-cool-file.pdf.gz' \
  -F 'output_format=text/csv'
```
* Assumes the Content-Type for each form part of `files` is `application/gzip`
* Passing `gz_uncompressed_content_type` is optional but preferred

### Multiple Files compressed, Application/json response (new)

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: Application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'gz_uncompressed_content_type=applicaton/pdf` `
  -F 'files=@my-cool-file.pdf.gz' \
  -F 'files=@my-other-cool-file.pdf.gz' \
  -F 'output_schema=labelstudio
```
* Assumes the Content-Type for each form part of `files` is `application/gzip`
* Passing `gz_uncompressed_content_type` is optional but preferred
* The json result is a list with length of the number of files (in this case 2), where each item is the result of a call to `pipeline_api`. The json schema of each item in the list is "labelstudio."


### Multiple Files compressed and uncompressed, Application/json response (new)

```
curl -X 'POST' 'https://<hostname>/<api-path>' \
  -H 'Accept: Application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'gz_uncompressed_content_type=applicaton/pdf` `
  -F 'files=@my-cool-file.pdf.gz' \
  -F 'files=@my-other-cool-file.pdf.gz' \
  -F 'files=@my-additional.pdf' \
  -F 'output_schema=labelstudio
```

* It's OK to pass uncompressed and compressed files, and the order of files is preserved in the response.
* Assumes the Content-Type for each .gz file is `application/gzip`
* Passing `gz_uncompressed_content_type` is optional but preferred, and only pertains to `applicaton/gzip` `files`.
* The json result is a list with length of the number of files (in this case 3), where each item is the result of a call to `pipeline_api`. The json schema of each item in the list is "labelstudio."
