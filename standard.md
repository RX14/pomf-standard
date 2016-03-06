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
A `POST` request is sent to `/upload.php` as `multipart/form-data`. The files to be uploaded are callled `files[]`.

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

For example `POST upload.php` returns JSON response. `POST upload.php?output=csv` returns CSV response.

#### JSON Response
The server should respond with a json object containing a success value and a list of files.

Example:
```js
{
    "success": true, // Whether the upload request was successful
    "files": [ // List of file objects
        {
            "hash": "06c8103e3e2c1659fd5831fb83d78a4b0a869b23", // sha1 checksum of file uploaded
            "name": "your_file.ext", // Name of file from the upload form
            "url": "https://your.clone/abcde.ext", // Full URL to file
            "size": 73483 // Size of file in bytes
        }
    ]
}
```

#### CSV Response
```csv
"name","url","hash","size"
"your_file.ext","https://your.clone/abcde.ext","06c8103e3e2c1659fd5831fb83d78a4b0a869b23","73483"
```

#### Gyazo Response
The server responds with only the URL to the first file uploaded.
```
https://your.clone/abcde.ext
```

#### Text Response
The server responds with 
```
your_file.ext: https://your.clone/abcde.ext
```

#### HTML Response
```html
<a href="https://your.clone/name.ext">https://your.clone/name.ext</a><br>
```
