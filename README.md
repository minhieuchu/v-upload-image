# v-upload-image

A simple UI component to upload images to a remote server, created with Vue 3.

A common approach to handle images is to store them on a database and then store the URL to access the image and other data on a different database. This component is designed specifically for that use case.

## Features

- Drag/drop to upload images
- Preview uploaded images
- Double-click to remove uploaded images

## Getting Started

Import the component and use it in your Vue component template as follows.

```javascript
import UploadImage from "v-upload-image";
```

```html
<upload-image
  :processImage="customProcessImage"
  :removeImage="customRemoveImage"
  @update-urls="customHandleUpdatedUrls"
></upload-image>
```

In order to upload images to a server, it's necessary to define some functions to use as component props:

`processImage` : Takes an async function that receives an image (an object of Javascript File interface) and uploads the image to a server. This function should return an object that contains `imageId` and `imageUrl` properties. Once the upload is completed, the image will appear in the preview area.

`removeImage` : Takes an async function that receives an image ID (returned from function passed to `processImage` above). This function is called when user double-clicks on a preview image to remove it. When the image has been deleted from server, it will be removed from preview area.

`update-urls` event : This event is emitted when the processing of images is finished. The emitted event contains the list of image urls, each of which is returned from the function passed to `processImage` prop.

## Example Usage

Product images for an e-commerce site are stored on Firebase Cloud Storage in the example below. The URLs returned from Firebase are then stored along with other product data in a different database.

```javascript
// Pass to `processImage` prop
async uploadImageToFirebase(image) {
    const imageId = uuidv4();
    const storageRef = firebaseRef(firebaseStorage, imageId);
    await uploadBytes(storageRef, image);
    const downloadUrl = await getDownloadURL(storageRef);
    return { imageId: imageId, imageUrl: downloadUrl };
}
// Pass to `removeImage` prop
async removeImageFromFirebase(imageId) {
    const storageRef = firebaseRef(firebaseStorage, imageId);
    await deleteObject(storageRef);
}
// Handler for `update-urls` event
handleUpdatedUrls(urlsList) {
    this.product.imageUrls = urlsList;
}
```

## Example Result

https://user-images.githubusercontent.com/25933120/180596474-0a050fe2-1667-4dc5-a9cc-8ade86a60d8e.mov

## Improvement

We plan to bundle the Vue components so that this package can be used directly in browser.

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## License

This project is licensed under the MIT License.
