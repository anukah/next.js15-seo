# The Complete Guide to SEO in Next.js 15

This guide summarizes the essential practices and advanced techniques for achieving top-tier SEO in **Next.js 15**, transforming your website into a **search engine powerhouse**.

---

## 1. Foundational SEO with the Metadata API

Next.js 15 simplifies SEO with its **file-based Metadata API**, which is:
- **Type-safe**
- **Automatically optimized**
- Supports **dynamic content**

### Root Layout (`app/layout.tsx`)

Your root layout is the foundation. Configure it to establish a **consistent SEO baseline** across your entire site.

```tsx
// app/layout.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  // Essential for resolving relative image and URL paths
  metadataBase: new URL("https://yourdomain.com/"),

  // Creates dynamic page titles like "Page Name | Your Business Name"
  title: {
    default: "Your Business Name | Professional Service Description",
    template: "%s | Your Business Name",
  },

  description: "A compelling business description (under 160 characters) with your main keywords.",
  keywords: ["primary service", "secondary service", "local keywords"],

  // Fine-tuned instructions for search engine crawlers
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      "max-image-preview": "large",
      "max-snippet": -1,
    },
  },

  // Optimizes how your site appears on social media
  openGraph: {
    type: "website",
    locale: "en_US",
    url: "https://yourdomain.com",
    title: "Your Business - Professional Service",
    description: "Engaging description for social sharing.",
    images: [
      {
        url: "/hero-image.jpg", // Relative to metadataBase
        width: 1200,
        height: 630,
        alt: "Descriptive alt text for your main image.",
      },
    ],
    siteName: "Your Business Name",
  },

  twitter: {
    card: "summary_large_image",
    title: "Your Business - Professional Service",
    description: "Engaging description for Twitter sharing.",
    images: ["/hero-image.jpg"], // Relative to metadataBase
  },

  // Prevents duplicate content issues
  alternates: {
    canonical: "https://yourdomain.com",
  },
};
```
## 2. Structured Data (Schema)

Structured data helps search engines **understand the meaning** of your content, leading to **rich search results**.

Add these scripts to your root layout's `<head>`.

### Organization & Website Schema

Defines your **business identity** and **website structure**.

```tsx
<script
  type="application/ld+json"
  dangerouslySetInnerHTML={{
    __html: JSON.stringify({
      "@context": "https://schema.org",
      "@type": "Organization",
      "name": "Your Business Name",
      "url": "https://yourdomain.com",
      "logo": "https://yourdomain.com/logo.png"
      // ... more details like address
    })
  }}
/>

<script
  type="application/ld+json"
  dangerouslySetInnerHTML={{
    __html: JSON.stringify({
      "@context": "https://schema.org",
      "@type": "WebSite",
      "url": "https://yourdomain.com",
      "name": "Your Business Name",
      "publisher": { "@id": "https://yourdomain.com/#organization" }
    })
  }}
/>
```
## 3. Dynamic Sitemap & Robots.txt

These files are crucial for guiding **search engine crawlers**.  
Next.js 15 generates them dynamically from **TypeScript files in your `app/` directory**.

### Dynamic Sitemap (`app/sitemap.ts`)

A sitemap is a roadmap of your website. Next.js automatically generates `/sitemap.xml` from this file.  

Combine **static pages** with **dynamically fetched content**.

```tsx
// app/sitemap.ts
import { MetadataRoute } from 'next'
import { getAllPosts } from '@/lib/blog-service' // Example fetch function

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = 'https://yourdomain.com'

  // Static pages
  const staticPages: MetadataRoute.Sitemap = [
    { url: baseUrl, lastModified: new Date(), priority: 1.0 },
    { url: `${baseUrl}/about`, lastModified: new Date(), priority: 0.9 },
  ];

  // Dynamic pages (e.g., blog posts)
  const posts = await getAllPosts();
  const blogPages = posts.map(post => ({
    url: `${baseUrl}/blog/${post.slug}`,
    lastModified: new Date(post.updatedAt),
    priority: 0.6,
  }));

  return [...staticPages, ...blogPages];
}
```
### Dynamic Robots.txt (`app/robots.ts`)

The `robots.txt` file tells bots **which pages they can and cannot crawl**.  

The dynamic approach allows for different rules in **development vs. production**.

```tsx
// app/robots.ts
import { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  const baseUrl = 'https://yourdomain.com'
  const isProduction = process.env.NODE_ENV === 'production'

  // Block all crawlers in non-production environments
  if (!isProduction) {
    return { rules: { userAgent: '*', disallow: '/' } };
  }

  return {
    rules: [{
      userAgent: '*',
      allow: '/',
      disallow: ['/api/', '/admin/'],
    }],
    sitemap: `${baseUrl}/sitemap.xml`,
  }
}
```
## 4. Page-Level and Advanced SEO

Individual pages can define their own **titles, descriptions, Open Graph, and Twitter Card metadata**.

### Example: `app/about/page.tsx`

```tsx
import { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'About Us | Your Business Name',
  description: 'Learn more about Your Business Name, our mission, and the team behind our success.',
  openGraph: {
    title: 'About Us | Your Business Name',
    description: 'Learn more about Your Business Name, our mission, and the team behind our success.',
    url: 'https://yourdomain.com/about',
    siteName: 'Your Business Name',
    images: [
      {
        url: 'https://yourdomain.com/og-about.jpg',
        width: 1200,
        height: 630,
        alt: 'About Us - Your Business Name',
      },
    ],
    locale: 'en_US',
    type: 'website',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'About Us | Your Business Name',
    description: 'Learn more about Your Business Name, our mission, and the team behind our success.',
    images: ['https://yourdomain.com/og-about.jpg'],
  },
  alternates: {
    canonical: 'https://yourdomain.com/about',
  },
}
```

### Breadcrumb Schema

Breadcrumbs help **Google display a navigational trail** in search results.

```tsx
// app/components/SeoBreadcrumbs.tsx
export default function SeoBreadcrumbs() {
  const breadcrumbSchema = {
    "@context": "https://schema.org",
    "@type": "BreadcrumbList",
    "itemListElement": [
      {
        "@type": "ListItem",
        "position": 1,
        "name": "Home",
        "item": "https://yourdomain.com"
      },
      {
        "@type": "ListItem",
        "position": 2,
        "name": "About",
        "item": "https://yourdomain.com/about"
      }
    ]
  }

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(breadcrumbSchema) }}
    />
  )
}
```

## 5. Local SEO

If your business has a **physical presence**, implementing `LocalBusiness` schema helps you rank in **Google Maps and local searches**.

### Example: LocalBusiness Schema

```tsx
// app/components/SeoLocalBusiness.tsx
export default function SeoLocalBusiness() {
  const localBusinessSchema = {
    "@context": "https://schema.org",
    "@type": "LocalBusiness",
    "name": "Your Business Name",
    "image": "https://yourdomain.com/logo.png",
    "@id": "https://yourdomain.com",
    "url": "https://yourdomain.com",
    "telephone": "+61-123-456-789",
    "address": {
      "@type": "PostalAddress",
      "streetAddress": "123 Main Street",
      "addressLocality": "Sydney",
      "addressRegion": "NSW",
      "postalCode": "2000",
      "addressCountry": "AU"
    },
    "geo": {
      "@type": "GeoCoordinates",
      "latitude": -33.8688,
      "longitude": 151.2093
    },
    "openingHoursSpecification": [
      {
        "@type": "OpeningHoursSpecification",
        "dayOfWeek": [
          "Monday",
          "Tuesday",
          "Wednesday",
          "Thursday",
          "Friday"
        ],
        "opens": "09:00",
        "closes": "18:00"
      }
    ]
  }

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(localBusinessSchema) }}
    />
  )
}
```

## Overall Summary

Implementing SEO in a **Next.js 13+ (App Router)** project involves multiple layers working together:

1. **Global SEO (Metadata in `app/layout.tsx`)**  
   - Define default `title`, `description`, Open Graph, Twitter, and canonical URLs.

2. **Structured Data (Schema)**  
   - Add `Organization`, `WebSite`, and `BreadcrumbList` schemas.  
   - For local businesses, include `LocalBusiness` schema to improve **Google Maps** visibility.

3. **Dynamic `robots.txt` and `sitemap.xml`**  
   - Block crawlers in development.  
   - Allow production crawling with a **dynamic sitemap**.

4. **Page-Level Metadata**  
   - Each page defines unique `title`, `description`, Open Graph, and Twitter previews.  
   - Use `generateMetadata` for **dynamic pages** (e.g., blogs, product pages).

5. **Local SEO Enhancements**  
   - Add `LocalBusiness` schema with **address, phone, geo-coordinates, and hours**.  
   - Ensures visibility in **local pack and Google Maps**.

---

With this setup, your Next.js app is optimized for:  
- **Search engine rankings** (technical SEO foundations).  
- **Rich snippets** (structured data like breadcrumbs, local business info).  
- **Social sharing** (Open Graph and Twitter cards).  
- **Local discoverability** (Google Maps + local search).  

This provides a **scalable SEO foundation** while keeping everything integrated into your Next.js project structure.
