# Uploading Files

To make files work as expected, there's a lot going on behind the scenes. Make
sure to read through the [Files](../getting-started/files.md) section in
Getting Started first as we'll be building on that information.

This section only talks about file uploading. For non-uploaded files such as
URLs and file IDs, you just need to pass a string.

## Fields

Let's start by talking about how the library represents files as part of a
Config.

### Static Fields

Most endpoints use static file fields. For example, `sendPhoto` expects a single
file named `photo`. All we have to do is set that single field with the correct
value (either a string or multipart file). Methods like `sendDocument` take two
file uploads, a `document` and a `thumb`. These are pretty straightforward.

Remembering that the `Fileable` interface only requires one method, let's
implement it for `DocumentConfig`.

```go
func (config DocumentConfig) files() []RequestFile {
    // We can have multiple files, so we'll create an array. We also know that
    // there always is a document file, so initialize the array with that.
	files := []RequestFile{{
		Name: "document",
		Data: config.File,
	}}

    // We'll only add a file if we have one.
	if config.Thumb != nil {
		files = append(files, RequestFile{
			Name: "thumb",
			Data: config.Thumb,
		})
	}

	return files
}
```

Bale also supports the `attach://` syntax (discussed more later) for
thumbnails, but there's no reason to make things more complicated.

### Dynamic Fields

Of course, not everything can be so simple. Methods like `sendMediaGroup`
can accept many files, and each file can have custom markup. Using a static
field isn't possible because we need to specify which field is attached to each
item. Bale introduced the `attach://` syntax for this.

Let's follow through creating a new media group with string and file uploads.

First, we start by creating some `InputMediaPhoto`.

```go
photo := tgbotapi.NewInputMediaPhoto(tgbotapi.FilePath("tests/image.jpg"))
url := tgbotapi.NewInputMediaPhoto(tgbotapi.FileURL("https://i.imgur.com/unQLJIb.jpg"))
```

This created a new `InputMediaPhoto` struct, with a type of `photo` and the
media interface that we specified.

We'll now create our media group with the photo and URL.

```go
mediaGroup := NewMediaGroup(ChatID, []interface{}{
    photo,
    url,
})
```

A `MediaGroupConfig` stores all the media in an array of interfaces. We now
have all the data we need to upload, but how do we figure out field names for
uploads? We didn't specify `attach://unique-file` anywhere.

When the library goes to upload the files, it looks at the `params` and `files`
for the Config. The params are generated by transforming the file into a value
more suitable for uploading, file IDs and URLs are untouched but uploaded types
are all changed into `attach://file-%d`. When collecting a list of files to
upload, it names them the same way. This creates a nearly transparent way of
handling multiple files in the background without the user having to consider
what's going on.
