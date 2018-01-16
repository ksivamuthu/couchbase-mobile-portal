### Blobs

We've renamed "attachments" to "blobs", for clarity. The new behavior should be clearer too: a `Blob` is now a normal object that can appear in a document as a property value. In other words, there's no special API for creating or accessing attachments; you just instantiate an `Blob` and set it as the value of a property, and then later you can get the property value, which will be a `Blob` object. The following code example adds a blob to the document under the `avatar` property.

```swift
let appleImage = UIImage(named: "avatar.jpg")!
let imageData = UIImageJPEGRepresentation(appleImage, 1)!

let blob = Blob(contentType: "image/jpg", data: imageData)
newTask.setBlob(blob, forKey: "avatar")
try? database.save(newTask)

if let taskBlob = newTask.blob(forKey: "image") {
	UIImage(data: taskBlob.content!)
}
```

`Blob` itself has a simple API that lets you access the contents as in-memory data (a `Data` object) or as a `InputStream`. It also supports an optional `type` property that by convention stores the MIME type of the contents. Unlike attachments, blobs don't have names; if you need to associate a name you can put it in another document property, or make the filename be the property name (e.g. `document.set(imageBlob, forKey: "thumbnail.jpg")`)

> **Note:** A blob is stored in the document's raw JSON as an object with a property `"_cbltype":"blob"`. It also has properties such as `"digest"` (a SHA-1 digest of the data), `"length"` (the length in bytes), and optionally `"type"` (the MIME type.) As always, the data is not stored in the document, but in a separate content-addressable store, indexed by the digest.