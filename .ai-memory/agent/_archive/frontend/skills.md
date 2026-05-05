# Frontend Agent — Skills

---

## Core Skills

### 1. Component Development

- Functional React / Vue / Svelte components
- Props typing แบบ strict
- Composition over inheritance
- Reusable atomic components

### 2. State Management

- Local state (useState) สำหรับ scope สั้น
- Context สำหรับ cross-component scope กลาง
- External store (Zustand / Redux / Pinia) สำหรับ scope ใหญ่
- Server state (React Query / SWR / TanStack Query)

### 3. Routing

- File-based routing (Next.js App Router, Nuxt)
- Dynamic routes + catch-all
- Loading / error boundary

### 4. Styling

- CSS modules / Tailwind / Styled-components ตาม project convention
- Design token (color, spacing, font) ใช้ตาม system
- Responsive: mobile-first

### 5. Form Handling

- Validation library (Zod / Yup / React Hook Form)
- Server-side error display
- Optimistic update เมื่อเหมาะสม

### 6. Accessibility

- Semantic HTML
- ARIA labels
- Keyboard navigation
- Color contrast

### 7. Performance

- Memoization (useMemo, useCallback) เมื่อจำเป็น (ห้าม premature)
- Code splitting / lazy load
- Image optimization
- Avoid unnecessary re-render

---

## Tool Preferences

| Task | Tool |
|------|------|
| อ่านไฟล์ | Read |
| แก้ component | Edit |
| สร้าง component ใหม่ | Write |
| Test ใน browser | Bash (start dev server) + manual / Playwright |
| Format | Bash (prettier) |

---

## Anti-patterns

- ❌ Class component ในของใหม่
- ❌ `any` type ใน props/state
- ❌ Inline business logic (ควรใส่ใน hook / util)
- ❌ Skip loading / error state
- ❌ Skip accessibility
- ❌ Premature memoization
- ❌ Direct DOM manipulation (ใช้ ref ถ้าจำเป็นจริง)
