# ZipArchive and ZipBuilder

This repository contains two libs to handle ZIP archives, `zip-archive.js` to read them, and `zip-builder.js` to write them. Note that this repository is managed as described in https://xkcd.com/1597/.

## Why yet another library for ZIP archives?

This library originates from a project where I needed both raw deflate compression and ZIP archives. I used `pako` for the compression and so decided to write my own ZIP library that internally uses `pako`.

## How to use

The two libraries are independent from each other, both require [pako](https://github.com/nodeca/pako) (unless your ZIP archives are just containers with uncompressed contents). Simply include both `pako` and this library via `<script>` tags into your project. Then you can use the globals `ZipArchive` and `ZipBuilder`.

## Reading ZIP files

First of all you need the ZIP file as `ArrayBuffer`. If you are using XHR, set `responseType = 'arraybuffer'`; if you are using `fetch`, then call the `arrayBuffer()` method on the response. If you are working with files, then use the `readAsArrayBuffer()` method of the `FileReader`.

Then call the `ZipArchive` constructor with the arraybuffer as argument. Note that this will throw an error if the ZIP file is broken. Otherwise you get a `ZipArchive` object with the following methods:

### getComment(legacy)

Get the comment of the ZIP archive. This assumes UTF-8 encoding by default, set the optional argument to `true` if that’s not the case.

### getEntries()

Get the list of all entries of the ZIP archive.

### getMetadata(name)

Get the metadata of an entry of the ZIP archive. You can specify the entry either by using its name, or its index. The metadata object contains following entries:
* `name`
* `versionCreated`
* `version`
* `bitflag`
* `compressionMethod`
* `lastMod` (as `Date`)
* `crc32`
* `compressedSize`
* `uncompressedSize`
* `disk`
* `internalAttr`
* `externalAttr`
* `extra` (as array)
* `comment`

Refer to the [ZIP standard](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT) for the meaning.

### getCompressedData(name)

Get the compressed data of an entry of the ZIP archive (as arraybuffer).

### getData(name, string)

Get the uncompressed data of an entry of the ZIP archive, either as arraybuffer or as string (if the second optional argument is set to `true`, assumes UTF-8 encoding).

## Writing ZIP files

Call the `ZipBuilder` constructor with an optional argument for default data (see below for `addFile`). The resulting object has the following methods:

### addFile(data)

This adds a file the the ZIP archive. The data object can contain the following entries:
* `name`: the name of the file
* `comment`: the file comment
* `extra`: extra data (as array)
* `date`: the timestamp (as `Date`)
* `content`: the content (as array)
* `isText`: whether the content is text (as `Boolean`)
* `isDir`: whether the content is a directory (as `Boolean`)
* `version`: the version
* `attr`: the numeric attributes
* `compressionLevel`: the compression level
* `compressionRate`: the maximal compression rate (if the compressed data is larger it will be stored without compression)

Missing options fall back to the default you specified in the constructor, or to sensible defaults.

### close(comment)

Closes the ZIP archive with an optional comment. Note that the comment will be encoded as UTF-8. If you are not sure whether your reader will accept this, use only ASCII letters (or no comment at all).

### getCurrentCount()

The current number of entries in the ZIP archive.

### getCurrentSize()

The current byte size of the ZIP archive (assuming you don’t add a comment).

### getBlob(name)

Returns the ZIP archive as `File` (if you give a name) or as `Blob` (without name). This will automatically close the archive if not done yet.

