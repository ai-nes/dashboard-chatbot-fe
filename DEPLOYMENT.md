# Deployment Guide

This project is configured as a **Next.js Static Export** application, which means it generates static HTML files that can be deployed to any static hosting service.

## 🚀 Quick Start

### Development
```bash
npm run dev
# or
bun run dev
```
- Runs on `http://localhost:3333`
- Uses Bun for faster development experience

### Production Build
```bash
npm run build
```
- Creates optimized static files in `/out` directory
- Uses npm for stable production builds

### Preview Production
```bash
npm run start
# or
npm run preview
# or  
npm run serve
```
- Serves the static files from `/out` directory
- Runs on `http://localhost:3333`

## 📋 Available Scripts

| Script | Command | Description |
|--------|---------|-------------|
| `dev` | `bun --bun next dev` | Start development server with Bun |
| `build` | `next build` | Build for production (static export) |
| `start` | `npx serve@latest out` | Serve production build |
| `preview` | `npx serve@latest out` | Preview production build |
| `serve` | `npx serve@latest out` | Serve static files |
| `lint` | `next lint` | Run ESLint |
| `type-check` | `tsc --noEmit` | Check TypeScript types |
| `build:bun` | `bun --bun next build` | Build with Bun (fallback) |
| `clean` | `rm -rf .next out` | Clean build artifacts |

## 🌐 Production: Vercel

Domain **`qna.faip.pro`**. Backend API: **`https://api.qna.faip.pro`** (`core-tuyensinh` trên EC2).

### Lần đầu

1. Vercel → **Add New Project** → import repo `ai-nes/dashboard-chatbot-fe`.
   - Framework preset: **Next.js** (tự nhận). Root directory: `./`.
   - Build command / output: để mặc định — Next `output: 'export'`, Vercel serve `out/`.
2. **Settings → Environment Variables** (Production, và Preview nếu cần):

   | Name | Value |
   |------|-------|
   | `NEXT_PUBLIC_API_BASE_URL` | `https://api.qna.faip.pro` |

   Biến `NEXT_PUBLIC_*` bake vào bundle lúc build → đổi giá trị phải **redeploy**.
3. **Settings → Domains** → thêm `qna.faip.pro`.
   - Cloudflare DNS: `CNAME qna → cname.vercel-dns.com`, **DNS only** (tắt proxy màu cam) để Vercel cấp cert.
4. Vercel Git integration tự deploy mỗi khi push `main` (và tạo Preview cho PR).

### CI

`.github/workflows/ci.yml` chỉ chạy lint + type-check + build check. Không build/push Docker nữa — Vercel lo deploy.

### Fallback: static hosting khác

`bun run build` → thư mục `out/` là static thuần, deploy được lên Netlify / Cloudflare Pages / S3+CloudFront / GitHub Pages (set `index.html` default, `404.html` cho lỗi). Dockerfile trong `docker/` vẫn dùng được nếu cần self-host.

## ⚙️ Configuration

### Next.js Config (`next.config.js`)
```javascript
const nextConfig = {
  output: 'export',           // Enable static export
  eslint: {
    ignoreDuringBuilds: true, // Skip linting during build
  },
  images: { 
    unoptimized: true         // Required for static export
  },
};
```

### Why Static Export?
- **No server required**: Pure static files
- **CDN friendly**: Fast global distribution
- **Cost effective**: Cheap hosting options
- **High performance**: Pre-rendered pages
- **Easy deployment**: Upload and serve

## 🔧 Troubleshooting

### Build Issues with Bun
If `bun run build` fails, use npm:
```bash
npm run build
```

### Port Already in Use
Dev/preview dùng cố định port **3333**. Nếu port bận, tắt process đang chiếm 3333 rồi chạy lại:
```bash
lsof -ti :3333 | xargs kill -9
```

### Clean Build
If you encounter caching issues:
```bash
npm run clean
npm run build
```

### TypeScript Errors
Check types before building:
```bash
npm run type-check
```

## 📊 Build Output

After running `npm run build`, you'll see:
```
Route (app)                    Size     First Load JS
┌ ○ /                         2.5 kB   81.9 kB
├ ○ /dashboard/departments    5.92 kB  119 kB
├ ○ /dashboard/tuition        12.4 kB  140 kB
└ ○ /login                    3.98 kB  94.1 kB

○ (Static) automatically rendered as static HTML
```

## 🎯 Production Checklist

- [ ] Vercel project imported, env `NEXT_PUBLIC_API_BASE_URL=https://api.qna.faip.pro`
- [ ] Domain `qna.faip.pro` added; Cloudflare `CNAME qna → cname.vercel-dns.com` (DNS only)
- [ ] BE `CORS_ORIGINS` chứa `https://qna.faip.pro`
- [ ] Push `main` → Vercel deploy xanh, `ci.yml` xanh
- [ ] Đăng nhập được, API calls không lỗi CORS (DevTools → Network)
- [ ] Check console for errors, responsive, performance

## 📝 Notes

- **Development**: Uses Bun for faster hot reload
- **Production**: Uses npm for stability
- **Static Export**: All pages pre-rendered at build time
- **No SSR**: Server-side rendering disabled
- **API Routes**: Not supported in static export mode
