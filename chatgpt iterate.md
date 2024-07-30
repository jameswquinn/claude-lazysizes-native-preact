Sure, let’s enhance your `LazyImage` component to address the suggested improvements:

### Placeholder Customization

We’ll update the `LazyImage` component to allow customization of placeholder styles and content via props. You can pass custom styles or a React component for the placeholder.

### Error Handling with Retry Logic

We’ll add retry logic to handle image load failures more gracefully. If the image fails to load, it will attempt to reload a few times before showing an error message.

### Prop Types and Default Values

We’ll use PropTypes to define the expected prop types and provide default values for props to ensure the component behaves predictably even when props are omitted.

### Loading Spinner

We’ll add a loading spinner to indicate that the image is being loaded, enhancing the user experience.

Here’s the updated implementation:

### `LazyImage.jsx`

```jsx
import { h } from 'preact';
import { useEffect, useRef, useState } from 'preact/hooks';
import PropTypes from 'prop-types';
import './LazyImage.css';

// Check WebP support
const checkWebPSupport = () => {
  return new Promise((resolve) => {
    const webP = new Image();
    webP.onload = webP.onerror = () => resolve(webP.height === 2);
    webP.src = 'data:image/webp;base64,UklGRjoAAABXRUJQVlA4IC4AAACyAgCdASoCAAIALmk0mk0iIiIiIgBoSygABc6WWgAA/veff/0PP8bA//LwYAAA';
  });
};

// Load WebP polyfill
const loadWebPPolyfill = () => {
  return new Promise((resolve, reject) => {
    const script = document.createElement('script');
    script.src = 'https://unpkg.com/webp-hero@0.0.2/dist-cjs/polyfills.js';
    script.async = true;
    script.onload = () => resolve();
    script.onerror = reject;
    document.head.appendChild(script);
  });
};

// Check for native lazy loading support
const checkLazyLoadingSupport = () => 'loading' in HTMLImageElement.prototype;

const LazyImage = ({
  src,
  alt,
  width,
  height,
  placeholderSrc,
  placeholderContent,
  placeholderStyle,
  srcSet,
  sizes,
  retryAttempts = 3,
  retryDelay = 1000,
  ...props
}) => {
  const [inView, setInView] = useState(false);
  const [loadState, setLoadState] = useState('initial'); // 'initial', 'loading', 'loaded', 'error'
  const [webPSupported, setWebPSupported] = useState(null);
  const [supportsLazyLoading, setSupportsLazyLoading] = useState(true);
  const [attemptsLeft, setAttemptsLeft] = useState(retryAttempts);
  const imgRef = useRef(null);

  useEffect(() => {
    const checkSupport = async () => {
      const isWebPSupported = await checkWebPSupport();
      setWebPSupported(isWebPSupported);
      if (!isWebPSupported) {
        try {
          await loadWebPPolyfill();
          console.log('WebP polyfill loaded successfully');
        } catch (error) {
          console.error('Failed to load WebP polyfill:', error);
        }
      }
    };

    checkSupport();

    setSupportsLazyLoading(checkLazyLoadingSupport());

    if (supportsLazyLoading) {
      setInView(true);
      setLoadState('loading');
    } else if ('IntersectionObserver' in window) {
      const observer = new IntersectionObserver(
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

      return () => {
        if (imgRef.current) {
          observer.unobserve(imgRef.current);
        }
      };
    } else {
      setInView(true);
      setLoadState('loading');
    }
  }, [supportsLazyLoading]);

  const handleLoad = () => {
    setLoadState('loaded');
  };

  const handleError = () => {
    if (attemptsLeft > 0) {
      setAttemptsLeft(attemptsLeft - 1);
      setLoadState('initial');
      setTimeout(() => {
        setLoadState('loading');
      }, retryDelay);
    } else {
      setLoadState('error');
    }
  };

  return (
    <div className="lazy-image-container" style={{ width, height }}>
      {placeholderSrc && loadState !== 'loaded' && !placeholderContent && (
        <img
          src={placeholderSrc}
          alt={`Placeholder for ${alt}`}
          className="lazy-image-placeholder"
        />
      )}
      {placeholderContent && loadState !== 'loaded' && (
        <div className="lazy-image-placeholder" style={placeholderStyle}>
          {placeholderContent}
        </div>
      )}
      {loadState === 'loading' && (
        <div className="lazy-image-loading">
          <div className="spinner" />
        </div>
      )}
      {(inView || supportsLazyLoading) && (
        <img
          ref={imgRef}
          src={supportsLazyLoading ? src : undefined}
          srcSet={srcSet}
          sizes={sizes}
          alt={alt}
          onLoad={handleLoad}
          onError={handleError}
          className={`lazy-image-loaded ${loadState === 'error' ? 'lazy-image-error' : ''}`}
          loading={supportsLazyLoading ? 'lazy' : undefined}
          {...props}
        />
      )}
      {loadState === 'error' && (
        <div className="lazy-image-error">
          <span role="img" aria-label="Error" style={{ fontSize: '24px' }}>⚠️</span>
          <p>Image failed to load</p>
        </div>
      )}
    </div>
  );
};

LazyImage.propTypes = {
  src: PropTypes.string.isRequired,
  alt: PropTypes.string.isRequired,
  width: PropTypes.string,
  height: PropTypes.string,
  placeholderSrc: PropTypes.string,
  placeholderContent: PropTypes.node,
  placeholderStyle: PropTypes.object,
  srcSet: PropTypes.string,
  sizes: PropTypes.string,
  retryAttempts: PropTypes.number,
  retryDelay: PropTypes.number,
};

LazyImage.defaultProps = {
  width: '100%',
  height: 'auto',
  placeholderSrc: '',
  placeholderContent: null,
  placeholderStyle: {},
  srcSet: '',
  sizes: '',
  retryAttempts: 3,
  retryDelay: 1000,
};

export default LazyImage;
```

### `LazyImage.css`

Add styles for the spinner and adjust existing styles as needed:

```css
.lazy-image-container {
  position: relative;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

.lazy-image-placeholder {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: flex;
  align-items: center;
  justify-content: center;
}

.lazy-image-loading {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(255, 255, 255, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
}

.spinner {
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-left: 4px solid #3498db;
  border-radius: 50%;
  width: 36px;
  height: 36px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.lazy-image-loaded {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.lazy-image-error {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(255, 255, 255, 0.9);
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
}

.lazy-image-error span {
  font-size: 2rem;
  margin-right: 8px;
}
```

### Summary of Enhancements

1. **Placeholder Customization**: Allows customization of the placeholder with either an image or a React component.
2. **Error Handling**: Implements retry logic with configurable attempts and delay before showing an error message.
3. **Prop Types and Default Values**: Adds PropTypes for type checking and default values to handle missing props gracefully.
4. **Loading Spinner**: Adds a spinner to indicate that the image is loading, improving the user experience.

With these updates, your `LazyImage` component will be more flexible, user-friendly, and robust in handling different scenarios.
