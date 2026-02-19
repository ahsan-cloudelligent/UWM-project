# Frontend Engineering Skills

## Modern Framework Expertise

### React
- **Component Architecture**: Functional components, hooks, composition patterns
- **State Management**: useState, useReducer, Context API, Zustand, Redux Toolkit
- **Performance**: useMemo, useCallback, React.memo, lazy loading, code splitting
- **Side Effects**: useEffect, useLayoutEffect, custom hooks
- **Advanced Patterns**: Compound components, render props, HOCs, controlled vs uncontrolled
- **Server Components**: Next.js App Router, React Server Components, streaming

### Vue.js
- **Composition API**: setup(), reactive(), ref(), computed(), watch()
- **Component Communication**: Props, events, provide/inject
- **State Management**: Pinia, Vuex
- **Directives**: v-if, v-for, v-model, custom directives
- **Performance**: Lazy loading, async components, keep-alive

### Angular
- **Components & Modules**: Standalone components, NgModules
- **Dependency Injection**: Services, providers, hierarchical injectors
- **RxJS**: Observables, operators, subjects, async pipe
- **Forms**: Template-driven, reactive forms, validation
- **Change Detection**: Zones, OnPush strategy

## UI/UX Implementation

### Responsive Design
- **Mobile-First Approach**: Design for smallest screens first, progressively enhance
- **Breakpoints**: Define consistent breakpoints (sm: 640px, md: 768px, lg: 1024px, xl: 1280px)
- **Flexible Layouts**: CSS Grid, Flexbox, container queries
- **Responsive Images**: srcset, picture element, lazy loading
- **Touch Interactions**: Touch-friendly targets (min 44x44px), swipe gestures

### CSS Architecture
- **Methodologies**: BEM, SMACSS, OOCSS
- **CSS-in-JS**: Styled-components, Emotion, CSS Modules
- **Utility-First**: Tailwind CSS, custom utility classes
- **Design Tokens**: Colors, spacing, typography, shadows as variables
- **CSS Variables**: Dynamic theming, runtime customization

### Component Libraries
- **Material UI (React)**: Comprehensive component library, theming
- **Ant Design**: Enterprise-grade UI components
- **Chakra UI**: Accessible, composable components
- **Shadcn/ui**: Copy-paste component library with Radix UI
- **Headless UI**: Unstyled, accessible components

### Animation & Transitions
- **CSS Animations**: Keyframes, transitions, transforms
- **JavaScript Libraries**: Framer Motion, GSAP, React Spring
- **Performance**: Use transform and opacity, avoid layout thrashing
- **Accessibility**: Respect prefers-reduced-motion

## State Management

### Local State
- Component state for UI-specific data
- Lift state only when necessary
- Keep state minimal and normalized

### Global State
- **Context API**: Simple global state, avoid overuse
- **Redux Toolkit**: Predictable state container, time-travel debugging
- **Zustand**: Lightweight, hooks-based state management
- **Jotai/Recoil**: Atomic state management
- **MobX**: Observable state, automatic reactivity

### Server State
- **React Query (TanStack Query)**: Caching, synchronization, background updates
- **SWR**: Stale-while-revalidate pattern
- **Apollo Client**: GraphQL state management
- **RTK Query**: Redux Toolkit's data fetching solution

### Form State
- **React Hook Form**: Performant, flexible form validation
- **Formik**: Form state management with validation
- **Zod/Yup**: Schema validation libraries
- **Native Validation**: HTML5 validation attributes

## Performance Optimization

### Code Splitting
- Route-based splitting with React.lazy() or dynamic imports
- Component-based splitting for heavy components
- Vendor splitting for third-party libraries
- Analyze bundle size with webpack-bundle-analyzer

### Lazy Loading
- Images: Intersection Observer, native lazy loading
- Components: React.lazy(), Suspense
- Routes: Dynamic imports in routing
- Third-party scripts: Load on interaction

### Rendering Optimization
- **Memoization**: React.memo, useMemo, useCallback
- **Virtualization**: React Window, React Virtualized for long lists
- **Debouncing/Throttling**: Limit expensive operations
- **Web Workers**: Offload heavy computations

### Asset Optimization
- **Images**: WebP format, responsive images, compression
- **Fonts**: Font subsetting, font-display: swap, variable fonts
- **CSS**: Minification, critical CSS, unused CSS removal
- **JavaScript**: Minification, tree shaking, compression (gzip/brotli)

### Core Web Vitals
- **LCP (Largest Contentful Paint)**: < 2.5s - Optimize images, server response
- **FID (First Input Delay)**: < 100ms - Reduce JavaScript execution
- **CLS (Cumulative Layout Shift)**: < 0.1 - Reserve space for dynamic content
- **INP (Interaction to Next Paint)**: < 200ms - Optimize event handlers

## Accessibility (WCAG 2.1 AA)

### Semantic HTML
- Use appropriate elements (button, nav, main, article, section)
- Heading hierarchy (h1-h6) for document structure
- Lists (ul, ol) for grouped content
- Forms with proper labels and fieldsets

### ARIA (Accessible Rich Internet Applications)
- **Roles**: role="button", role="dialog", role="navigation"
- **Properties**: aria-label, aria-labelledby, aria-describedby
- **States**: aria-expanded, aria-hidden, aria-disabled, aria-selected
- **Live Regions**: aria-live, aria-atomic for dynamic content

### Keyboard Navigation
- All interactive elements must be keyboard accessible
- Logical tab order (tabindex management)
- Focus indicators (visible focus styles)
- Keyboard shortcuts (avoid conflicts with screen readers)
- Escape key to close modals/dropdowns

### Screen Reader Support
- Meaningful alt text for images
- Skip navigation links
- Announce dynamic content changes
- Form validation messages
- Test with NVDA, JAWS, VoiceOver

### Color & Contrast
- Minimum contrast ratio 4.5:1 for normal text
- Minimum contrast ratio 3:1 for large text (18pt+)
- Don't rely on color alone to convey information
- Support high contrast mode

### Focus Management
- Trap focus in modals
- Return focus after closing dialogs
- Manage focus on route changes
- Visible focus indicators

## Testing Strategies

### Unit Testing
- Test component logic in isolation
- Mock external dependencies
- Test custom hooks
- Tools: Jest, Vitest, Testing Library

### Component Testing
- Test component rendering and interactions
- Test props and state changes
- Test accessibility (jest-axe)
- Tools: React Testing Library, Vue Test Utils

### Integration Testing
- Test user flows across multiple components
- Test API integration
- Test routing and navigation
- Tools: Testing Library, Cypress Component Testing

### End-to-End Testing
- Test complete user journeys
- Test across browsers
- Visual regression testing
- Tools: Playwright, Cypress, Puppeteer

### Visual Testing
- Screenshot comparison
- Detect unintended UI changes
- Tools: Percy, Chromatic, BackstopJS

## Build Tools & Bundlers

### Vite
- Fast dev server with HMR
- Optimized production builds
- Plugin ecosystem
- Native ESM support

### Webpack
- Highly configurable bundler
- Code splitting, lazy loading
- Loaders and plugins
- Module federation

### Turbopack / Rspack
- Next-generation bundlers
- Rust-based for performance
- Webpack-compatible

### Build Optimization
- Tree shaking for unused code
- Minification and compression
- Source maps for debugging
- Cache optimization

## API Integration

### REST APIs
- Fetch API, Axios for HTTP requests
- Error handling and retry logic
- Request/response interceptors
- Authentication token management

### GraphQL
- Apollo Client, urql for GraphQL
- Query, mutation, subscription patterns
- Caching and normalization
- Optimistic updates

### WebSockets
- Real-time bidirectional communication
- Socket.io, native WebSocket API
- Reconnection logic
- Message queuing

### Server-Sent Events (SSE)
- One-way server-to-client streaming
- Automatic reconnection
- Simpler than WebSockets for read-only updates

## Routing

### Client-Side Routing
- **React Router**: Declarative routing, nested routes, loaders
- **TanStack Router**: Type-safe routing, search params
- **Vue Router**: Vue's official router
- **Angular Router**: Lazy loading, guards, resolvers

### Server-Side Routing
- **Next.js**: File-based routing, App Router, Pages Router
- **Remix**: Nested routes, loaders, actions
- **Nuxt**: Vue's meta-framework
- **SvelteKit**: Svelte's application framework

### Route Protection
- Authentication guards
- Role-based access control
- Redirect to login
- Loading states during auth checks

## Forms & Validation

### Form Handling
- Controlled vs uncontrolled components
- Form submission and validation
- Multi-step forms
- File uploads with progress

### Validation Strategies
- Client-side validation for UX
- Server-side validation for security
- Real-time vs on-submit validation
- Schema validation (Zod, Yup, Joi)

### Form Libraries
- React Hook Form: Performance-focused
- Formik: Feature-rich form management
- Final Form: Framework-agnostic

## Security Best Practices

### XSS Prevention
- Sanitize user input before rendering
- Use framework's built-in escaping (React, Vue auto-escape)
- Avoid dangerouslySetInnerHTML / v-html with user content
- Content Security Policy (CSP) headers

### CSRF Protection
- Use CSRF tokens for state-changing requests
- SameSite cookie attribute
- Verify origin headers

### Authentication
- Store tokens securely (httpOnly cookies preferred)
- Implement token refresh
- Handle expired sessions gracefully
- Never store sensitive data in localStorage

### Dependency Security
- Regular dependency updates
- Use npm audit / yarn audit
- Automated security scanning (Snyk, Dependabot)
- Review third-party packages before use

## Progressive Web Apps (PWA)

### Service Workers
- Offline functionality
- Background sync
- Push notifications
- Cache strategies (cache-first, network-first, stale-while-revalidate)

### Web App Manifest
- App icons and splash screens
- Display mode (standalone, fullscreen)
- Theme colors
- Install prompts

### Offline Support
- Cache critical assets
- Offline fallback pages
- Queue failed requests
- Sync when online

## Modern Web APIs

### Storage
- **localStorage**: Persistent, synchronous, 5-10MB limit
- **sessionStorage**: Session-scoped
- **IndexedDB**: Large structured data, asynchronous
- **Cache API**: Service worker caching

### Media
- **Canvas API**: 2D graphics, image manipulation
- **WebGL**: 3D graphics
- **Web Audio API**: Audio processing
- **MediaStream API**: Camera, microphone access

### Performance
- **Intersection Observer**: Lazy loading, infinite scroll
- **Resize Observer**: Responsive components
- **Performance API**: Measure page performance
- **Web Workers**: Background threads

### Modern Features
- **Web Components**: Custom elements, shadow DOM
- **CSS Container Queries**: Component-based responsive design
- **View Transitions API**: Smooth page transitions
- **Popover API**: Native popovers and tooltips

## Internationalization (i18n)

### Translation Management
- **react-i18next**: React internationalization
- **vue-i18n**: Vue internationalization
- **FormatJS**: ICU message format
- **Lingui**: Minimal runtime, compile-time optimization

### Localization Considerations
- Date and time formatting (Intl.DateTimeFormat)
- Number formatting (Intl.NumberFormat)
- Currency formatting
- Pluralization rules
- RTL (right-to-left) language support

## Developer Experience

### TypeScript
- Type safety for props, state, API responses
- Interfaces and types for contracts
- Generics for reusable components
- Strict mode for maximum safety

### Linting & Formatting
- ESLint for code quality
- Prettier for consistent formatting
- Husky for pre-commit hooks
- lint-staged for staged files only

### Documentation
- Component documentation (Storybook)
- JSDoc comments for complex logic
- README for setup and architecture
- ADRs (Architecture Decision Records)

### Debugging
- React DevTools, Vue DevTools
- Browser DevTools (Elements, Network, Performance)
- Source maps for production debugging
- Error boundaries for graceful error handling

---

**These skills represent senior-level frontend engineering expertise. Master these to build modern, performant, accessible, and maintainable user interfaces.**
