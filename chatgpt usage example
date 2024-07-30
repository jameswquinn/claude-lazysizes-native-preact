Sure, let's walk through an example of how to use the `LazyImage` component in a Preact application, with some practical considerations for image optimization.

### Example Usage of `LazyImage`

In this example, we'll use the `LazyImage` component to display images with lazy loading, handling different resolutions and WebP support.

#### App Component

```jsx
import { h } from 'preact';
import LazyImage from './LazyImage'; // Adjust the path as necessary

const App = () => {
  return (
    <div style={{ padding: '20px' }}>
      <h1>Lazy Loaded Images Example</h1>
      <LazyImage
        src="https://example.com/images/large-image.webp"
        alt="A beautiful landscape"
        width="100%"
        height="auto"
        placeholderSrc="https://example.com/images/placeholder-image.jpg"
        srcSet="
          https://example.com/images/large-image-320w.webp 320w,
          https://example.com/images/large-image-480w.webp 480w,
          https://example.com/images/large-image-800w.webp 800w
        "
        sizes="
          (max-width: 600px) 100vw,
          (max-width: 1200px) 50vw,
          33vw
        "
      />
      <LazyImage
        src="https://example.com/images/another-image.webp"
        alt="Another view"
        width="100%"
        height="auto"
        placeholderSrc="https://example.com/images/another-placeholder.jpg"
        srcSet="
          https://example.com/images/another-image-320w.webp 320w,
          https://example.com/images/another-image-480w.webp 480w,
          https://example.com/images/another-image-800w.webp 800w
        "
        sizes="
          (max-width: 600px) 100vw,
          (max-width: 1200px) 50vw,
          33vw
        "
      />
    </div>
  );
};

export default App;
```

### Explanation

1. **`src`**: The URL of the WebP image to be loaded.
2. **`alt`**: Descriptive text for the image, for accessibility.
3. **`width` and `height`**: Set the dimensions for the image container. Use `width="100%"` and `height="auto"` for responsive designs.
4. **`placeholderSrc`**: A low-resolution placeholder image to show while the main image is loading.
5. **`srcSet`**: Defines different image sources for different viewport sizes or resolutions. This helps the browser select the appropriate image based on the device's screen size and resolution.
6. **`sizes`**: Specifies how much space the image should take up at different viewport sizes. This is used in conjunction with `srcSet` to choose the best image size.

### CSS for Spinner Animation

You might want to add some CSS for the spinner animation. Here's an example of CSS you can include in your global stylesheet:

```css
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loading-spinner {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 40px;
  height: 40px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}
```

### Summary

- **Lazy Loading**: Images are only loaded when they come into view, saving bandwidth and improving performance.
- **Responsive Images**: `srcSet` and `sizes` ensure the right image size is served based on the device’s viewport.
- **Placeholder Images**: While the main image is loading, a placeholder is shown to improve user experience.

This setup provides a performant and responsive image loading experience, accommodating different screen sizes and resolutions while leveraging modern browser capabilities.
