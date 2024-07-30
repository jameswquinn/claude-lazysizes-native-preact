Certainly! Here is the final, integrated version of the `LazyImage` component along with an example usage and necessary CSS for the spinner animation.

### Final `LazyImage` Component

```jsx
import { h } from 'preact';
import { useEffect, useRef, useState } from 'preact/hooks';

// Helper function to check WebP support
const checkWebPSupport = () => {
  return new Promise((resolve) => {
    const webP = new Image();
    webP.onload = webP.onerror = () => {
      resolve(webP.height === 2);
    };
    webP.src = 'data:image/webp;base64,UklGRjoAAABXRUJQVlA4IC4AAACyAgCdASoCAAIALmk0mk0iIiIiIgBoSygABc6WWgAA/veff/0PP8bA//LwYAAA';
  });
};

// Helper function to dynamically load WebP polyfill
const loadWebPPolyfill = () => {
  return new Promise((resolve, reject) => {
    const script = document.createElement('script');
    script.src = 'https://unpkg.com/webp-hero@0.0.2/dist-cjs/polyfills.js';
    script.onload = () => {
      const webpMachine = new window.webpHero.WebpMachine();
      webpMachine.polyfillDocument().then(resolve).catch(reject);
    };
    script.onerror = reject;
    document.head.appendChild(script);
  });
};

const LazyImage = ({ 
  src, 
  alt, 
  width, 
  height, 
  placeholderSrc,
  srcSet,
  sizes,
  ...props 
}) => {
  const [inView, setInView] = useState(false);
  const [loadState, setLoadState] = useState('initial'); // 'initial', 'loading', 'loaded', 'error'
  const [webPSupported, setWebPSupported] = useState(null);
  const imgRef = useRef(null);

  useEffect(() => {
    const checkSupport = async () => {
      const isSupported = await checkWebPSupport();
      setWebPSupported(isSupported);
      if (!isSupported) {
        try {
          await loadWebPPolyfill();
          console.log('WebP polyfill loaded successfully');
        } catch (error) {
          console.error('Failed to load WebP polyfill:', error);
        }
      }
    };

    checkSupport();

    let observer;

    if ('IntersectionObserver' in window) {
      observer = new IntersectionObserver(
        (entries) => {
          entries.forEach((entry) => {
            if (entry.isIntersecting) {
              setInView(true);
              setLoadState('loading');
              observer.unobserve(entry.target);
            }
          });
        },
        {
          rootMargin: '200px 0px',
          threshold: 0.01,
        }
      );

      if (imgRef.current) {
        observer.observe(imgRef.current);
      }
    } else {
      setInView(true);
      setLoadState('loading');
    }

    return () => {
      if (observer && imgRef.current) {
        observer.unobserve(imgRef.current);
      }
    };
  }, []);

  const handleLoad = () => {
    setLoadState('loaded');
  };

  const handleError = () => {
    setLoadState('error');
  };

  return (
    <div 
      style={{ 
        width, 
        height, 
        background: '#f0f0f0',
        overflow: 'hidden',
        position: 'relative'
      }}
    >
      {placeholderSrc && loadState !== 'loaded' && (
        <img
          src={placeholderSrc}
          alt=""
          style={{
            position: 'absolute',
            top: 0,
            left: 0,
            width: '100%',
            height: '100%',
            objectFit: 'cover',
            filter: 'blur(10px)',
            transform: 'scale(1.1)',
          }}
        />
      )}
      {loadState === 'loading' && (
        <div className="loading-spinner" />
      )}
      {inView && webPSupported !== null && (
        <img
          ref={imgRef}
          src={src}
          srcSet={srcSet}
          sizes={sizes}
          alt={alt}
          onLoad={handleLoad}
          onError={handleError}
          style={{
            position: 'relative',
            width: '100%',
            height: '100%',
            objectFit: 'cover',
            opacity: loadState === 'loaded' ? 1 : 0,
            transition: 'opacity 0.5s ease-in-out',
          }}
          {...props}
        />
      )}
      {loadState === 'error' && (
        <div
          style={{
            position: 'absolute',
            top: '50%',
            left: '50%',
            transform: 'translate(-50%, -50%)',
            textAlign: 'center',
            color: '#d63031',
          }}
        >
          <span style={{ fontSize: '24px' }}>⚠️</span>
          <p>Image failed to load</p>
        </div>
      )}
    </div>
  );
};

export default LazyImage;
```

### Example Usage in `App` Component

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

### CSS for Spinner Animation

Add this CSS to your global stylesheet (e.g., `styles.css`):

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

This setup ensures that:

- **Images are lazy-loaded**: Images are only loaded when they come into view, improving performance.
- **Responsive image handling**: Using `srcSet` and `sizes` ensures the right image size is served based on the device's viewport.
- **WebP support**: The component checks for WebP support and loads a polyfill if necessary.
- **Placeholder and error handling**: Placeholder images and error messages improve user experience during image loading.

Feel free to adjust the image URLs and other parameters to fit your specific needs.
