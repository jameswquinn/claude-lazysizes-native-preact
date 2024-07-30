Certainly! Here's a detailed API documentation for the `LazyImage` component. This documentation covers the purpose, usage, and properties of the component, providing clear guidance for developers.

---

## `LazyImage` Component API Documentation

### Overview

The `LazyImage` component is a versatile, accessible component for loading images and iframes lazily. It supports fallback mechanisms for browsers that lack native support for lazy loading and provides accessibility features to enhance user experience.

### Usage

```jsx
import LazyImage from './LazyImage';

// Example usage for an image
<LazyImage
  src="https://example.com/image.jpg"
  alt="A description of the image"
  width="600px"
  height="400px"
  placeholderSrc="https://example.com/placeholder.jpg"
  placeholderContent={<div>Loading...</div>}
  placeholderStyle={{ backgroundColor: '#f0f0f0' }}
  srcSet="https://example.com/image.jpg 1x, https://example.com/image@2x.jpg 2x"
  sizes="(max-width: 600px) 100vw, 50vw"
  retryAttempts={5}
  retryDelay={2000}
  isIframe={false}
  backgroundImage="https://example.com/background.jpg"
/>

// Example usage for an iframe
<LazyImage
  src="https://example.com/iframe-content"
  alt="Iframe displaying content"
  width="800px"
  height="600px"
  isIframe={true}
  backgroundImage="https://example.com/iframe-placeholder.jpg"
/>
```

### Properties

#### Common Props

- **`src`** (string, required): The URL of the image or iframe content to be loaded.
  - **Type**: `string`
  - **Description**: The source URL of the image or iframe content.

- **`alt`** (string, required): The alt text for the image.
  - **Type**: `string`
  - **Description**: Descriptive text for the image or iframe content. Used for accessibility.

- **`width`** (string, optional): The width of the image or iframe container.
  - **Type**: `string`
  - **Default**: `'100%'`
  - **Description**: Defines the width of the container for the image or iframe. Accepts any valid CSS width value.

- **`height`** (string, optional): The height of the image or iframe container.
  - **Type**: `string`
  - **Default**: `'auto'`
  - **Description**: Defines the height of the container for the image or iframe. Accepts any valid CSS height value.

#### Image-Specific Props

- **`placeholderSrc`** (string, optional): The URL of a placeholder image to be shown before the actual image loads.
  - **Type**: `string`
  - **Default**: `''`
  - **Description**: URL of the placeholder image. Useful for showing a visual cue while the main image is loading.

- **`placeholderContent`** (node, optional): Custom content to display as a placeholder.
  - **Type**: `node`
  - **Default**: `null`
  - **Description**: Can be used to provide a custom loading message or element.

- **`placeholderStyle`** (object, optional): Inline styles for the placeholder content.
  - **Type**: `object`
  - **Default**: `{}`
  - **Description**: CSS properties to apply to the placeholder content.

- **`srcSet`** (string, optional): A set of image sources to use for responsive images.
  - **Type**: `string`
  - **Default**: `''`
  - **Description**: Defines the set of images for different screen resolutions (e.g., `image.jpg 1x, image@2x.jpg 2x`).

- **`sizes`** (string, optional): Specifies how the image sizes should be calculated.
  - **Type**: `string`
  - **Default**: `''`
  - **Description**: Defines the sizes for responsive image loading (e.g., `(max-width: 600px) 100vw, 50vw`).

- **`retryAttempts`** (number, optional): Number of times to retry loading the image if it fails.
  - **Type**: `number`
  - **Default**: `3`
  - **Description**: Specifies how many retry attempts should be made if the image fails to load.

- **`retryDelay`** (number, optional): Delay between retry attempts.
  - **Type**: `number`
  - **Default**: `1000` (milliseconds)
  - **Description**: Delay in milliseconds before retrying the image load after a failure.

#### Iframe-Specific Props

- **`isIframe`** (boolean, optional): Indicates whether the component is displaying an iframe.
  - **Type**: `boolean`
  - **Default**: `false`
  - **Description**: Set to `true` if the component is used to display an iframe instead of an image.

- **`backgroundImage`** (string, optional): The URL of a background image for iframes.
  - **Type**: `string`
  - **Default**: `''`
  - **Description**: URL of the background image to be shown while the iframe is loading.

### CSS Classes

- **`lazy-image-container`**: The container for the lazy-loaded image.
- **`lazy-image-placeholder`**: The class applied to the placeholder image or content.
- **`lazy-image-loading`**: The class applied to the loading spinner or indicator.
- **`lazy-image-loaded`**: The class applied to the successfully loaded image.
- **`lazy-image-error`**: The class applied to the error state of the image.
- **`lazy-iframe-container`**: The container for the lazy-loaded iframe.
- **`lazy-iframe-placeholder`**: The class applied to the placeholder for the iframe.
- **`lazy-iframe-loading`**: The class applied to the loading spinner or indicator for the iframe.
- **`lazy-iframe-error`**: The class applied to the error state of the iframe.

### Example

```jsx
<LazyImage
  src="https://example.com/image.jpg"
  alt="A beautiful landscape"
  width="600px"
  height="400px"
  placeholderSrc="https://example.com/placeholder.jpg"
  placeholderContent={<div>Loading...</div>}
  placeholderStyle={{ backgroundColor: '#f0f0f0' }}
  srcSet="https://example.com/image.jpg 1x, https://example.com/image@2x.jpg 2x"
  sizes="(max-width: 600px) 100vw, 50vw"
  retryAttempts={5}
  retryDelay={2000}
  isIframe={false}
  backgroundImage="https://example.com/background.jpg"
/>
```

This `LazyImage` component is designed to handle various loading scenarios for images and iframes, ensuring accessibility and compatibility with various browsers.
