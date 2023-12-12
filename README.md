# Schema summary

- We have two schemas: music and album
- The music schema has a reference to the album type (being a global document)
- The music schema import the fields is_top and year from album reference

# How to reproduce the bug

- First of all, deploy this simple application
- After that, upload the json document located at ext using the following commands:

```bash
vespa document put ext/album.json
vespa document put ext/music.json
```

With the following commands, you will see that the visitor has different behaviours when running it as a GET operation (just returning the documents that matches the selection criteria) and using UPDATE operations (updating those document that matches the selection criteria) and using an imported an imported field for that criteria.

## Running it as a GET operation

```bash
vespa visit vespa visit --selection 'music.isTop == true' --field-set '[all]'
```

Running that command, you will get the following thing:

```json
{
    "id":"id:mynamespace:music::sultans-of-swing",
    "fields":{
        "title": "Sultans of swing",
        "album": "id:mynamespace:album::dire-straits",
        "score": 75
    }
}
```

## Running it as a UPDATE operation

Now imagine that we want those music documents whose album are top (music.isTop == true). In order to do that, we run the following command:

```bash
curl --location --request PUT 'http://localhost:8080/document/v1/mynamespace/music/docid/?cluster=music&bucketSpace=default&selection=music.isTop == true' \
--header 'Content-Type: application/json' \
--data '{
    "fields": {
        "score": {
            "assign": 100
        }
    }
}'
```

However, that request returns the following response:

```json
{
    "pathId": "/document/v1/mynamespace/music/docid/",
    "documentCount": 0
}
```

instead of 

```diff
{
    "pathId": "/document/v1/mynamespace/music/docid/",
-    "documentCount": 0
+    "documentCount": 1
}
```

This means that the selection criteria that we've used didn't matched and it's not working in this scenario.

In case that criteria matched, we should have the following content in the document that we are modifying:
```diff
{
    "pathId": "/document/v1/mynamespace/music/docid/sultans-of-swing",
    "id": "id:mynamespace:music::sultans-of-swing",
    "fields": {
        "title": "Sultans of swing",
        "album": "id:mynamespace:album::dire-straits",
-        "score": 75
+        "score": 100
    }
}
```
