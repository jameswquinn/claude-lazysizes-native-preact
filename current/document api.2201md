Here's a comprehensive API documentation for the `LazyImage` component. This documentation will help developers understand how to use the component, its props, and expected behaviors.

---

## `LazyImage` Component

The `LazyImage` component is designed to efficiently handle lazy loading of images and iframes, with support for WebP, IntersectionObserver polyfills, error handling, and accessibility features.

### **Component Overview**

```jsx
import LazyImage from './LazyImage';
```

### **Props**

#### **Common Props**

- **`src`** (`string`): *Required*  
  The source URL of the image or iframe. 

- **`alt`** (`string`): *Required*  
  The alternative text for the image or iframe, which improves accessibility.

- **`width`** (`string`):  
  The width of the image or iframe. Default is `'100%'`.

- **`height`** (`string`):  
  The height of the image or iframe. Default is `'auto'`.

- **`retryAttempts`** (`number`):  
  Number of retry attempts in case of loading errors. Default is `3`.

- **`retryDelay`** (`number`):  
  Delay between retry attempts in milliseconds. Default is `1000`.

#### **Image-Specific Props**

- **`placeholderSrc`** (`string`):  
  The URL of the placeholder image to show while the main image is loading. 

- **`placeholderContent`** (`node`):  
  React node to be displayed as a placeholder if `placeholderSrc` is not provided.

- **`placeholderStyle`** (`object`):  
  Inline styles to apply to the placeholder content.

- **`srcSet`** (`string`):  
  A set of image sources with different resolutions for responsive images.

- **`sizes`** (`string`):  
  Specifies the sizes for different viewports.

#### **Iframe-Specific Props**

- **`isIframe`** (`bool`):  
  Indicates whether the component should render an iframe instead of an image. Default is `false`.

- **`backgroundImage`** (`string`):  
  The URL of the background image to display while the iframe is loading.

#### **Styling and Accessibility Props**

- **`className`** (`string`):  
  Custom class name for the container.

- **`style`** (`object`):  
  Inline styles to apply to the container.

### **Behavior**

1. **Lazy Loading**:
   - Uses native lazy loading if supported by the browser (`loading="lazy"`).
   - Falls back to IntersectionObserver for browsers that do not support native lazy loading.
   - Directly loads the content if neither lazy loading nor IntersectionObserver is supported.

2. **WebP Support**:
   - Checks if WebP format is supported and loads a polyfill if necessary.

3. **Error Handling**:
   - Provides retry logic with configurable attempts and delay for handling transient loading errors.
   - Displays error messages if loading fails after all retry attempts.

4. **Accessibility**:
   - Uses `aria-live` attributes and roles to announce loading states and errors to screen readers.

### **Example Usage**

#### **Image Example**

```jsx
<LazyImage
  src="https://example.com/image.jpg"
  alt="Description of image"
  width="600px"
  height="400px"
  placeholderSrc="https://example.com/placeholder.jpg"
  retryAttempts={5}
  retryDelay={2000}
/>
```

#### **Iframe Example**

```jsx
<LazyImage
  src="https://example.com/iframe.html"
  alt="Description of iframe"
  width="800px"
  height="600px"
  isIframe={true}
  backgroundImage="https://example.com/background.jpg"
/>
```

### **CSS Styling**

Ensure you include the following CSS for styling:

```css
.lazy-image-container,
.lazy-iframe-container {
  position: relative;
  overflow: hidden;
}

.lazy-image-placeholder,
.lazy-iframe-placeholder {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-size: cover;
  background-position: center;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1;
}

.lazy-image-loading,
.lazy-iframe-loading {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2;
}

.spinner {
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-left: 4px solid #333;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.lazy-image-error,
.lazy-iframe-error {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  z-index: 2;
}
```

### **Notes**

- **Browser Compatibility**: Ensure that polyfills are correctly loaded for browsers that do not support WebP or IntersectionObserver.
- **Placeholder Handling**: Use `placeholderSrc` or `placeholderContent` to provide a visual placeholder during loading.

---

This documentation provides a detailed overview of the `LazyImage` component's API, usage, and styling. It should help developers integrate and use the component effectively in their projects.
