import { h } from 'preact';
import { useEffect, useRef, useState } from 'preact/hooks';
import PropTypes from 'prop-types';
import './LazyImage.css';

// Base64 encoded light gray 1x1 pixel GIF
const lightGrayGif = 'data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///ywAAAAAAQABAAACAUwAOw==';

// Function to check if the browser supports WebP images
const checkWebPSupport = () => {
  return new Promise((resolve) => {
    const webP = new Image();
    webP.onload = webP.onerror = () => resolve(webP.height === 2);
    webP.src = 'data:image/webp;base64,UklGRjoAAABXRUJQVlA4IC4AAACyAgCdASoCAAIALmk0mk0iIiIiIgBoSygABc6WWgAA/veff/0PP8bA//LwYAAA';
  });
};

// Function to dynamically load polyfills for unsupported browser features
const loadPolyfill = (src) => {
  return new Promise((resolve, reject) => {
    const script = document.createElement('script');
    script.src = src;
    script.async = true;
    script.onload = resolve;
    script.onerror = () => reject(new Error(`Failed to load polyfill: ${src}`));
    document.head.appendChild(script);
  });
};

// Custom hook to detect browser capabilities and load necessary polyfills
const useFeatureDetection = (format) => {
  const [features, setFeatures] = useState({
    webPSupported: null,
    supportsLazyLoading: 'loading' in HTMLImageElement.prototype,
    intersectionObserverSupported: 'IntersectionObserver' in window,
  });

  useEffect(() => {
    const detectFeatures = async () => {
      if (format === 'webp') {
        const webPSupported = await checkWebPSupport();
        if (!webPSupported) {
          try {
            await loadPolyfill('https://unpkg.com/webp-hero@0.0.2/dist-cjs/polyfills.js');
            console.log('WebP polyfill loaded successfully');
          } catch (error) {
            console.error(error.message);
          }
        }
        setFeatures((prev) => ({ ...prev, webPSupported }));
      }

      if (!features.intersectionObserverSupported) {
        try {
          await loadPolyfill('https://polyfill.io/v3/polyfill.min.js?features=IntersectionObserver');
          setFeatures((prev) => ({ ...prev, intersectionObserverSupported: 'IntersectionObserver' in window }));
          console.log('IntersectionObserver polyfill loaded successfully');
        } catch (error) {
          console.error(error.message);
        }
      }
    };

    detectFeatures();
  }, [format, features.intersectionObserverSupported]);

  return features;
};

// Custom hook to handle lazy loading based on browser support
const useLazyLoading = (features, imgRef, iframeRef, setInView, setLoadState) => {
  useEffect(() => {
    if (features.supportsLazyLoading) {
      setInView(true);
      setLoadState('loading');
      return;
    }

    if (features.intersectionObserverSupported) {
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

      if (imgRef.current) observer.observe(imgRef.current);
      if (iframeRef.current) observer.observe(iframeRef.current);

      return () => {
        if (imgRef.current) observer.unobserve(imgRef.current);
        if (iframeRef.current) observer.unobserve(iframeRef.current);
      };
    } else {
      setInView(true);
      setLoadState('loading');
    }
  }, [features, imgRef, iframeRef, setInView, setLoadState]);
};

// Main LazyImage component for lazy loading images and iframes
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
  isIframe = false,
  backgroundImage = lightGrayGif,
  format = '',
  ...props
}) => {
  const [inView, setInView] = useState(false);
  const [loadState, setLoadState] = useState('initial');
  const [attemptsLeft, setAttemptsLeft] = useState(retryAttempts);
  const imgRef = useRef(null);
  const iframeRef = useRef(null);

  const features = useFeatureDetection(format);
  useLazyLoading(features, imgRef, iframeRef, setInView, setLoadState);

  const handleLoad = () => setLoadState('loaded');
  const handleError = () => {
    if (attemptsLeft > 0) {
      setAttemptsLeft(attemptsLeft - 1);
      setLoadState('initial');
      setTimeout(() => setLoadState('loading'), retryDelay);
    } else {
      setLoadState('error');
    }
  };

  if (isIframe) {
    return (
      <div className="lazy-iframe-container" style={{ width, height }}>
        {backgroundImage && !inView && (
          <div
            className="lazy-iframe-placeholder"
            style={{ backgroundImage: `url(${backgroundImage})`, ...placeholderStyle }}
          >
            {placeholderContent}
          </div>
        )}
        {inView && (
          <iframe
            ref={iframeRef}
            src={src}
            width={width}
            height={height}
            title={alt}
            onLoad={handleLoad}
            onError={handleError}
            {...props}
          />
        )}
        {loadState === 'loading' && (
          <div className="lazy-iframe-loading" aria-live="polite">
            <div className="spinner" />
          </div>
        )}
        {loadState === 'error' && (
          <div className="lazy-iframe-error" role="alert">
            <span role="img" aria-label="Error" style={{ fontSize: '24px' }}>⚠️</span>
            <p>Iframe failed to load</p>
          </div>
        )}
      </div>
    );
  }

  return (
    <div className="lazy-image-container" style={{ width, height }}>
      {placeholderSrc && loadState !== 'loaded' && !placeholderContent && (
        <img
          src={placeholderSrc}
          alt={`Placeholder for ${alt}`}
          className="lazy-image-placeholder"
          aria-hidden="true"
        />
      )}
      {placeholderContent && loadState !== 'loaded' && (
        <div className="lazy-image-placeholder" style={{ backgroundImage: `url(${backgroundImage})`, ...placeholderStyle }}>
          {placeholderContent}
        </div>
      )}
      {loadState === 'loading' && (
        <div className="lazy-image-loading" aria-live="polite">
          <div className="spinner" />
        </div>
      )}
      {(inView || features.supportsLazyLoading) && (
        <img
          ref={imgRef}
          src={features.supportsLazyLoading ? src : undefined}
          srcSet={srcSet}
          sizes={sizes}
          alt={alt}
          onLoad={handleLoad}
          onError={handleError}
          className={`lazy-image-loaded ${loadState === 'error' ? 'lazy-image-error' : ''}`}
          loading={features.supportsLazyLoading ? 'lazy' : undefined}
          aria-live={loadState === 'error' ? 'assertive' : 'polite'}
          {...props}
        />
      )}
      {loadState === 'error' && (
        <div className="lazy-image-error" role="alert">
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
  isIframe: PropTypes.bool,
  backgroundImage: PropTypes.string,
  format: PropTypes.string,
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
  isIframe: false,
  backgroundImage: lightGrayGif,
  format: '',
};

export default LazyImage;
