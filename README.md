Certainly! I'll modify our LazyImage component to specifically handle WebP images, including a test for WebP support and a dynamic polyfill loader for browsers that don't support WebP. This approach will ensure optimal performance across different browsers while maintaining the benefits of WebP format where supported.



```javascript
import { h } from 'preact';
import { useEffect, useRef, useState } from 'preact/hooks';

// Helper function to check WebP support
const checkWebPSupport = () => {
  return new Promise((resolve) => {
    const webP = new Image();
    webP.onload = webP.onerror = function () {
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
        <div 
          style={{
            position: 'absolute',
            top: '50%',
            left: '50%',
            transform: 'translate(-50%, -50%)',
            width: '40px',
            height: '40px',
            border: '4px solid #f3f3f3',
            borderTop: '4px solid #3498db',
            borderRadius: '50%',
            animation: 'spin 1s linear infinite',
          }}
        />
      )}
      {inView && webPSupported !== null && (
        <img
          ref={imgRef}
          src={src}
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

Let's break down the new WebP-specific features:

1. WebP Support Check:
   - We've added a `checkWebPSupport` function that tests if the browser supports WebP images.
   - This function creates a small WebP image and checks if it loads successfully.

2. Dynamic Polyfill Loading:
   - The `loadWebPPolyfill` function dynamically loads the WebP polyfill script if WebP is not supported.
   - We're using the `webp-hero` library as our polyfill. It's loaded from a CDN (unpkg) for simplicity, but in a production environment, you might want to host this file yourself.

3. WebP Support State:
   - We've added a `webPSupported` state to track whether WebP is supported or if the polyfill has been loaded.

4. Conditional Rendering:
   - The main image is only rendered once we've determined WebP support status (`webPSupported !== null`).
   - This ensures that the image doesn't start loading before we've had a chance to load the polyfill if necessary.

To use this WebP-focused LazyImage component:

```jsx
import LazyImage from './WebPLazyImage';

function App() {
  return (
    <div>
      <h1>My WebP Image Gallery</h1>
      <LazyImage 
        src="path/to/image.webp"
        placeholderSrc="path/to/low-quality-image.webp"
        alt="Description 1" 
        width={300} 
        height={200} 
      />
      {/* Add more LazyImage components as needed */}
    </div>
  );
}
```

This implementation provides several benefits:

1. Efficient lazy loading of WebP images.
2. Automatic detection of WebP support.
3. Dynamic loading of a WebP polyfill for browsers that don't support WebP.
4. Maintains all previous features like blur-up effect, loading spinner, and error handling.

A few important notes:

1. The polyfill is loaded from a CDN for simplicity. In a production environment, you should consider hosting the polyfill file yourself to have more control over its availability and version.

2. The polyfill (webp-hero) works by converting WebP images to PNG on the client side. This process happens in the background but may cause a slight delay in image display on unsupported browsers.

3. While this solution allows you to use WebP images universally, it's still a good practice to provide fallback formats (like JPEG or PNG) on the server side for optimal performance. You could modify this component to accept both WebP and fallback URLs if needed.

4. Remember that the polyfill adds extra JavaScript to be downloaded and executed in browsers without native WebP support, which could impact performance on older or less capable devices.

Is there anything specific about this WebP implementation you'd like me to explain further?
