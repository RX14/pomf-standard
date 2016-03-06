Pomf Standardisation
====================

## TODO

- DEL Requests & deletion keys.
- Standardized file expiry //default will be forever
- Searching?
- Max file size discovery

## `/upload.php`
This is the main entry point for uploading files in the API.

[An example of a complete request/response to a popular clone](https://aww.moe/p6low0.png)

### Request
A `POST` request is sent to `/upload.php` as `multipart/form-data`. The files to be uploaded are stored in the key `files[]`.

Example, uploading 2 files, one called `foo` containing `bar\n` and the other called `bar` containing `foo\n`:
```
--052b045b431e4118948c02fbadbd057a
Content-Disposition: form-data; name="files[]"; filename="foo"

bar

--052b045b431e4118948c02fbadbd057a
Content-Disposition: form-data; name="files[]"; filename="bar"

foo

--052b045b431e4118948c02fbadbd057a--
```

### Response
There are several response types, determined by the `output` argument to `upload.php`.
- json (default)
- csv
- gyazo
- text (only some clones)
- html (only some clones)
- info

For example `POST upload.php` returns JSON response. `POST upload.php?output=csv` returns CSV response.

#### JSON Response
The server should respond with a json object containing a success value and a list of files. `Content-Type` should be set to `application/json`. Clients **must not** assume any specific JSON formatting, including but not limited to whitespace and ordering of keys. Regex is *not* a good way to parse JSON.

Schema:
```js
{
    "success": bool, // Whether the upload request was successful

    "files": [ // List of file objects, only is success=true
        {
            "name": string, // Name of file from the upload form
            "url": string, // Full URL including domain to file
            "hash": string, // SHA-1 hash of file uploaded
            "size": int // Size of file in bytes
        }
    ],

    "errorcode": 400, // HTTP error code of the response. Only is success=false
    "description": "No input file(s)" // Human-readable description of the error. Only if success=false
}
```

Example:
```json
{
    "success": true,
    "files": [
        {
            "hash": "06c8103e3e2c1659fd5831fb83d78a4b0a869b23",
            "name": "your_file.ext",
            "url": "https://your.clone/abcde.ext",
            "size": 73483
        }
    ],
    "errorcode": 400,
    "description": "No input file(s)"
}
```

#### CSV Response
The server should respond with a CSV document listing name, url, hash and size of uploaded files. The meaning of name, url hash and size is the same as in the JSON response. `Content-Type` should be set to `text/csv`.

```csv
"name","url","hash","size"
"your_file.ext","https://your.clone/abcde.ext","06c8103e3e2c1659fd5831fb83d78a4b0a869b23","73483"
```

#### Gyazo Response
The server responds with complete URLs to uploaded files, in the order they are uploaded, seperated by newlines, with a trailing newline. `Content-Type` should be `text/plain`.
```
https://your.clone/abcde.ext
https://your.clone/fghij.ext
```

#### Text Response
//TODO: discuss more
The server responds with 
```
your_file.ext: https://your.clone/abcde.ext
```

#### HTML Response
A HTML page containing links to uploaded files. This can be formatted and styled to your preference, as it is for human consumption.

Example:
```html
<a href="https://your.clone/name.ext">https://your.clone/name.ext</a><br>
```

## `/upload.php?output=info`
Returns a JSON object containing machine-readable metadata about the pomf clone.

Schema:
```js
{
    "base_url": string, // URL of the HTML homepage of the pomf clone.
    "max_size": int, // Maximum file size in bytes.
    "forbidden_ext": [string], // Array containing forbidden file extensions, without dot.
    "forbidden_mime": [string], // Array containing forbidden MIME types.
    "output_modes": [string] // Array containing supported values for the output argument on upload.php.
}
```

Example:
```json
{
    "base_url": "https://your.clone",
    "max_size": 104857600,
    "forbidden_ext": [
        "exe",
        "html",
        "vbs"
    ],
    "forbidden_mime": [
        ""
    ],
    "output_modes": [
        "json",
        "gyazo",
        "csv",
        "info"
    ]
}
```
