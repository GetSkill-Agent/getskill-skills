# Next.js Landing Page

> Create a modern, responsive landing page using Next.js 14 with App Router, Tailwind CSS, and TypeScript.

## Trigger

- User asks to create a landing page, marketing page, or promotional website
- User needs a SaaS landing page, product page, or startup website
- User wants a modern, responsive single-page site

## Prerequisites

- Node.js 18+
- npm or pnpm

## Instructions

### 1. Create Next.js project

```bash
npx create-next-app@latest my-landing --typescript --tailwind --app --src-dir --no-eslint
cd my-landing
```

### 2. Set up project structure

```
src/
├── app/
│   ├── page.tsx        # Landing page
│   ├── layout.tsx      # Root layout
│   └── globals.css     # Global styles
└── components/
    ├── Hero.tsx        # Hero section
    ├── Features.tsx    # Features grid
    ├── Testimonials.tsx # Social proof
    ├── Pricing.tsx     # Pricing table (optional)
    ├── CTA.tsx         # Call to action
    └── Footer.tsx      # Footer
```

### 3. Create Hero component

Must include:
- **Headline**: Clear value proposition (5-10 words)
- **Subheadline**: Supporting text (1-2 sentences)
- **CTA button**: Primary action with contrasting color
- **Social proof**: Logos, user count, or testimonials snippet

```tsx
// components/Hero.tsx
export function Hero() {
  return (
    <section className="py-20 px-4 text-center">
      <h1 className="text-5xl font-bold mb-4">
        Your Value Proposition Here
      </h1>
      <p className="text-xl text-gray-600 mb-8 max-w-2xl mx-auto">
        Supporting text that explains the benefit in one or two sentences.
      </p>
      <button className="bg-blue-600 text-white px-8 py-3 rounded-lg text-lg font-semibold hover:bg-blue-700">
        Get Started Free
      </button>
      <p className="mt-4 text-sm text-gray-500">
        Trusted by 10,000+ users
      </p>
    </section>
  );
}
```

### 4. Create Features section

Use a 3-column grid with icons:

```tsx
// components/Features.tsx
const features = [
  { icon: "⚡", title: "Fast", description: "Lightning quick performance" },
  { icon: "🔒", title: "Secure", description: "Enterprise-grade security" },
  { icon: "🎯", title: "Simple", description: "Easy to use interface" },
];

export function Features() {
  return (
    <section className="py-16 px-4 bg-gray-50">
      <div className="max-w-6xl mx-auto grid md:grid-cols-3 gap-8">
        {features.map((f) => (
          <div key={f.title} className="text-center p-6">
            <div className="text-4xl mb-4">{f.icon}</div>
            <h3 className="text-xl font-semibold mb-2">{f.title}</h3>
            <p className="text-gray-600">{f.description}</p>
          </div>
        ))}
      </div>
    </section>
  );
}
```

### 5. Add responsive design

Use Tailwind breakpoints:
- Mobile first: default styles
- `md:` for tablet (768px+)
- `lg:` for desktop (1024px+)

### 6. Optimize for performance

```tsx
// app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Your Product - Tagline Here',
  description: 'Clear description for SEO (150-160 chars)',
  openGraph: {
    title: 'Your Product - Tagline Here',
    description: 'Clear description for social sharing',
  },
};
```

### 7. Add CTA section

```tsx
// components/CTA.tsx
export function CTA() {
  return (
    <section className="py-20 px-4 bg-blue-600 text-white text-center">
      <h2 className="text-3xl font-bold mb-4">Ready to get started?</h2>
      <p className="mb-8 text-blue-100">Join thousands of happy users today.</p>
      <button className="bg-white text-blue-600 px-8 py-3 rounded-lg font-semibold">
        Start Free Trial
      </button>
    </section>
  );
}
```

## Success Criteria

- [ ] Page loads in < 3 seconds
- [ ] Mobile responsive (test at 375px width)
- [ ] Clear value proposition visible above the fold
- [ ] Primary CTA button is prominent
- [ ] Lighthouse score > 90

## Common Pitfalls

- Don't use too many fonts (max 2)
- Don't make CTA buttons too small on mobile
- Don't forget meta tags for SEO
- Don't use placeholder text in production
