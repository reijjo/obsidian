# next/image

## Optimizing images with next/image
In regular HTML, you would load an image with an _<img />_ tag:
```html
<img src="/profile.jpg" alt="User profile" width="200" height="200" />
```
This works, but browsers load the full-size image regardless of the device screen size, and there is no automatic optimization. Next.js provides the [Image](https://nextjs.org/docs/app/api-reference/components/image) component that automatically optimizes images for performance:

```tsx
import Image from "next/image"

const Profile = () => {
  return (
    <div>
      <Image
        src="/profile.jpg"
        alt="User profile"
        width={200}
        height={200}
      />
    </div>
  )
}

export default Profile
```
The Image component offers several key benefits:
- **Automatic format optimization:** Next.js serves modern formats like WebP and AVIF to browsers that support them, falling back to the original format for older browsers.
- **Responsive images:** The component automatically generates multiple sizes of the image and serves the appropriate one based on the user's device.
- **Lazy loading:** Images outside the viewport are not loaded until the user scrolls near them, reducing initial page load time.
- **Prevents layout shift:** By requiring explicit `width` and `height` props (or using `fill`), the browser reserves space before the image loads, preventing content from jumping around.

For images stored in the _public_ folder, the path starts with _/_:
```tsx
<Image src="/images/logo.png" alt="Logo" width={100} height={50} />
```

## External images hosted on other domains
For external images hosted on other domains, you need to configure the allowed domains in _next.config.js_:
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'example.com',
      },
    ],
  },
}

module.exports = nextConfig
```
Then you can use the external URL:
```tsx
<Image
  src="https://example.com/photo.jpg"
  alt="Photo"
  width={400}
  height={300}
/>
```

## Fill prop
If you do not know the image dimensions ahead of time (for example, user-uploaded images), you can use the _fill_ prop to make the image fill its parent container:
```tsx
<div style={{ position: 'relative', width: '100%', height: '400px' }}>
  <Image
    src="/dynamic-image.jpg"
    alt="Dynamic content"
    fill
    style={{ objectFit: 'cover' }}
  />
</div>
```
The parent container must have _position: relative_, _position: fixed_, or _position: absolute_ for _fill_ to work correctly.

The Image component is one of the most important performance optimizations in Next.js. Using it instead of plain _<img>_ tags ensures your images are delivered efficiently across all devices and network conditions.
## Related
- [[Next.js]]