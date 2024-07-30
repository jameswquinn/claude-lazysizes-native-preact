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

// ... [rest of the component code remains the same]

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

  // ... [rest of the component code remains the same]
};

LazyImage.propTypes = {
  // ... [other propTypes remain the same]
  format: PropTypes.string,
};

LazyImage.defaultProps = {
  // ... [other defaultProps remain the same]
  format: '',
};

export default LazyImage;
