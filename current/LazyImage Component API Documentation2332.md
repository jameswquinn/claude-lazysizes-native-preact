# LazyImage Component API Documentation

## Overview

The `LazyImage` component is a versatile React/Preact component designed for efficient lazy loading of images and iframes. It supports various browser capabilities and provides fallback mechanisms for optimal performance across different environments.

## Props

| Prop Name | Type | Default | Description |
|-----------|------|---------|-------------|
| `src` | `string` | Required | The source URL of the image or iframe to be loaded. |
| `alt` | `string` | Required | Alternative text for the image. For iframes, it's used as the title. |
| `width` | `string` | `'100%'` | The width of the image or iframe container. |
| `height` | `string` | `'auto'` | The height of the image or iframe container. |
| `placeholderSrc` | `string` | `''` | URL of a placeholder image to show while the main image is loading. |
| `placeholderContent` | `node` | `null` | React node to display as placeholder content. |
| `placeholderStyle` | `object` | `{}` | Custom styles for the placeholder container. |
| `srcSet` | `string` | `''` | The srcset attribute for responsive images. |
| `sizes` | `string` | `''` | The sizes attribute for responsive images. |
| `retryAttempts` | `number` | `3` | Number of retry attempts for failed loads. |
| `retryDelay` | `number` | `1000` | Delay in milliseconds between retry attempts. |
| `isIframe` | `boolean` | `false` | Set to `true` to render an iframe instead of an image. |
| `backgroundImage` | `string` | Light gray GIF | Base64 or URL of the background image for placeholders. |
| `format` | `string` | `''` | Image format type. If set to 'webp', WebP support will be checked. |

## Usage Examples

### Basic Image Usage

```jsx
<LazyImage
  src="https://example.com/image.jpg"
  alt="Example image"
  width="300px"
  height="200px"
/>
```

### WebP Image Usage

```jsx
<LazyImage
  src="https://example.com/image.webp"
  alt="WebP example"
  width="300px"
  height="200px"
  format="webp"
/>
```

### Responsive Image with Placeholder

```jsx
<LazyImage
  src="https://example.com/large-image.jpg"
  alt="Responsive example"
  placeholderSrc="https://example.com/placeholder.jpg"
  srcSet="https://example.com/small.jpg 300w, https://example.com/medium.jpg 600w, https://example.com/large.jpg 1200w"
  sizes="(max-width: 300px) 100vw, (max-width: 600px) 50vw, 33vw"
/>
```

### Iframe Usage

```jsx
<LazyImage
  src="https://example.com/embed"
  alt="Embedded content"
  width="560px"
  height="315px"
  isIframe={true}
  placeholderContent={<div>Loading iframe...</div>}
/>
```

## Notes

- The component automatically detects browser support for Intersection Observer and native lazy loading.
- WebP support is only checked when the `format` prop is set to 'webp'.
- It dynamically loads polyfills for unsupported features when necessary.
- Custom error handling and retry logic are implemented for failed loads.
- The component provides loading indicators and error messages for better user experience.

For more detailed information about the implementation, please refer to the component's source code and inline comments.
