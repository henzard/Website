# Website Development Rules - Complete Index

**Version:** 3.0.0  
**Last Updated:** December 19, 2025  
**Purpose:** Comprehensive guidelines for building production-ready React + Firebase websites with security, quality, performance, and beginner-friendliness

---

## 📋 Quick Start

**If you're starting a new project:**
1. Read **00-project-setup** first
2. Complete requirements gathering (**08a**)
3. Generate color palette (**08b**)
4. Review all "Always Applied" rules below

**If you're adding a feature:**
1. Create todo breakdown (**09-task-management**)
2. Review existing code and database (**10-database-design**)
3. Follow architecture patterns (**01-clean-architecture**)
4. Document as you build (**11-documentation-standards**)

---

## 📚 Rule Categories

### 🏗️ Foundation (Always Applied)

#### [00-project-setup](00-project-setup/RULE.md)
**Purpose:** Pre-development checklist and project initialization  
**Key Topics:**
- Required documentation (design system, color palette, requirements)
- 5-step project setup process
- Beginner-friendly explanations
- AI decision gates

**When to use:** Before starting ANY new project

---

#### [01-clean-architecture](01-clean-architecture/RULE.md)
**Purpose:** Code organization and separation of concerns  
**Key Topics:**
- Layer architecture (Presentation → Business Logic → Data Access)
- Feature-based organization
- Dependency injection
- Repository pattern

**When to use:** When structuring any new feature or refactoring

---

### ⚛️ React & Frontend (Always Applied)

#### [02-react-standards](02-react-standards/RULE.md)
**Purpose:** React best practices and TypeScript standards  
**Key Topics:**
- Component structure and patterns
- Custom hooks
- State management (Context API)
- Performance optimization
- Accessibility requirements

**When to use:** When creating any React component or hook

---

#### [04-ui-ux-design](04-ui-ux-design/RULE.md)
**Purpose:** Design system and user experience standards  
**Key Topics:**
- Color system implementation
- Typography and spacing
- Component design patterns
- Responsive design
- WCAG 2.1 AA compliance

**When to use:** When building UI components or pages  
**Applied to:** `**/components/**`, `**/pages/**`, `**/*.css`, `**/*.tsx`

---

### 🔥 Firebase & Backend (Always Applied)

#### [03-firebase-standards](03-firebase-standards/RULE.md)
**Purpose:** Firebase security, data structure, and best practices  
**Key Topics:**
- Comprehensive security rules
- Data structure standards
- Repository pattern implementation
- Cloud Functions best practices
- Real-time listeners

**When to use:** When working with Firebase/database operations

---

#### [10-database-design](10-database-design/RULE.md)
**Purpose:** Database naming, normalization, and data integrity  
**Key Topics:**
- Naming conventions (collections, fields, variables)
- Data normalization (anti-dump rules)
- TypeScript type definitions
- Relationships (one-to-many, many-to-many)
- Indexing strategy

**When to use:** When designing collections, creating types, or storing data  
**Applied to:** `**/repositories/**`, `**/services/**`, `**/types/**`

---

### 🔒 Security & Compliance (Always Applied)

#### [05-data-privacy](05-data-privacy/RULE.md)
**Purpose:** GDPR/POPIA compliance and user data rights  
**Key Topics:**
- User rights implementation (data export, deletion)
- Consent management
- Data retention policies
- Audit logging
- Required legal pages

**When to use:** When collecting any user data, always review before launch

---

### ✅ Quality Assurance (Always Applied to Tests)

#### [06-testing-standards](06-testing-standards/RULE.md)
**Purpose:** Unit, integration, and E2E testing requirements  
**Key Topics:**
- Testing pyramid (80% unit, 15% integration, 5% E2E)
- Component testing
- Firebase testing
- 70% minimum coverage
- Test-driven development practices

**When to use:** When writing any feature code  
**Applied to:** `**/*.test.ts`, `**/*.test.tsx`, `**/*.spec.ts`

---

### 🤖 AI & Decision Making (Always Applied)

#### [07-ai-decision-framework](07-ai-decision-framework/RULE.md)
**Purpose:** AI-driven quality gates and decision-making process  
**Key Topics:**
- STOP → THINK → VERIFY → ACT framework
- Decision gates before every action
- "Five Whys" technique
- Emergency override protocol
- Preventing rushed decisions

**When to use:** AI uses this for every significant decision

---

### 👥 For Non-Technical Users (Always Applied)

#### [08a-requirements-gathering](08a-requirements-gathering/RULE.md)
**Purpose:** Structured requirements gathering for beginners  
**Key Topics:**
- 11-question discovery interview
- Plain English explanations
- Client communication templates
- Common mistake warnings
- Realistic expectations

**When to use:** At project start, before any development

---

#### [08b-ai-color-generation](08b-ai-color-generation/RULE.md)
**Purpose:** Using AI tools to generate professional color palettes  
**Key Topics:**
- Gemini/ChatGPT/Claude prompt templates
- Color palette generation workflow
- Accessibility testing
- Design psychology
- Complete example (yoga studio)

**When to use:** When design system doesn't have colors yet  
**Applied manually:** Reference with `@08b-ai-color-generation`

---

### 📋 Project Management (Always Applied)

#### [09-task-management](09-task-management/RULE.md)
**Purpose:** Task breakdown, todo tracking, and impact assessment  
**Key Topics:**
- The "3-Task Rule" (when to create todos)
- Feature breakdown templates
- Impact assessment before changes
- Definition of done
- No big work without planning

**When to use:** Before starting any complex feature (>3 steps)

---

#### [11-documentation-standards](11-documentation-standards/RULE.md)
**Purpose:** What to document, how much, and when  
**Key Topics:**
- Required documentation (README, REQUIREMENTS, DESIGN_SYSTEM, CHANGELOG)
- Code comment guidelines (why not what)
- JSDoc for public APIs
- Avoiding documentation anti-patterns
- Balance between under and over-documenting

**When to use:** Throughout development, especially when completing features

---

### ⚡ Performance & Optimization (Always Applied)

#### [12-performance-standards](12-performance-standards/RULE.md)
**Purpose:** Performance optimization for fast, efficient applications  
**Key Topics:**
- Bundle size optimization (code splitting, tree shaking)
- Image optimization (modern formats, lazy loading)
- React performance (memoization, virtual lists)
- Firebase optimization (efficient queries, caching)
- Network optimization (service workers, prefetching)
- Performance monitoring (Web Vitals)

**When to use:** Throughout development, especially before production

---

### 🔍 SEO & Discoverability (Always Applied)

#### [13-seo-standards](13-seo-standards/RULE.md)
**Purpose:** SEO optimization and search engine visibility  
**Key Topics:**
- Meta tags for every page
- Structured data (JSON-LD schemas)
- Sitemap generation
- Robots.txt configuration
- Semantic HTML
- Mobile SEO
- Analytics integration

**When to use:** For every page, especially content-heavy sites

---

### 🚨 Error Handling (Always Applied)

#### [14-error-handling](14-error-handling/RULE.md)
**Purpose:** Error handling, logging, and graceful failure strategies  
**Key Topics:**
- React Error Boundaries
- Custom error classes
- Error handler service
- User-friendly error messages
- Logging service
- Production monitoring (Sentry)
- Network error handling

**When to use:** For all async operations and critical features  
**Critical for:** Production deployments

---

### 💳 Advanced Features (Apply When Needed)

#### [15-payment-processing](15-payment-processing/RULE.md)
**Purpose:** Secure payment processing with Stripe  
**Key Topics:**
- Stripe integration
- Payment intent creation
- Webhook handling
- Refund processing
- Invoice generation
- PCI compliance
- Subscription billing

**When to use:** When implementing payments/e-commerce  
**Apply manually:** Only for projects with payment features

---

#### [16-email-system](16-email-system/RULE.md)
**Purpose:** Transactional and marketing email system  
**Key Topics:**
- SendGrid/Mailgun integration
- Email templates (welcome, verification, booking confirmation)
- HTML + plain text versions
- Unsubscribe management
- Email analytics
- Deliverability best practices

**When to use:** When sending automated emails  
**Apply manually:** Only for projects with email features

---

#### [17-internationalization](17-internationalization/RULE.md)
**Purpose:** Multi-language support and localization  
**Key Topics:**
- react-i18next setup
- Translation file management
- Language switcher
- Date/number/currency formatting by locale
- RTL (right-to-left) language support
- SEO for multiple languages (hreflang)

**When to use:** When supporting multiple languages  
**Apply manually:** Only for multi-language projects

---

#### [18-blogging-standards](18-blogging-standards/RULE.md)
**Purpose:** Content creation, blog writing, and markdown formatting  
**Key Topics:**
- Writing compelling hooks (attention-grabbing openings)
- Blog post structure and formatting
- SEO for blog content
- Markdown best practices
- AI-guided writing assistance
- Content quality checklist
- Readability standards

**When to use:** When creating blog content in markdown  
**Applied to:** `**/blog/**/*.md`, `**/posts/**/*.md`

---

#### [19-gallery-products](19-gallery-products/RULE.md)
**Purpose:** Simple gallery and product display components  
**Key Topics:**
- Image gallery with lightbox
- Product card components
- Product detail pages
- Responsive image grids
- Touch-friendly interactions
- Mobile-optimized layouts
- Basic product showcase (3-10 items)

**When to use:** For showcasing images or small product catalogs  
**Applied to:** `**/components/Gallery*`, `**/components/Product*`, `**/pages/products/**`, `**/pages/gallery/**`

**Note:** For full e-commerce (cart, checkout), combine with Rule 15 (Payment Processing)

---

#### [20-authentication](20-authentication/RULE.md)
**Purpose:** User authentication, login, registration, and role-based access  
**Key Topics:**
- Firebase Auth integration
- Login/registration forms
- Social login (Google, Facebook)
- Password reset flow
- Role-based access control (RBAC)
- Protected routes
- Session management

**When to use:** Any project with user accounts  
**Applied to:** `**/auth/**`, `**/components/Auth*`, `**/hooks/useAuth*`

---

#### [21-forms-validation](21-forms-validation/RULE.md)
**Purpose:** Form handling, validation, and user input patterns  
**Key Topics:**
- React Hook Form + Zod
- Reusable form components
- Validation schemas
- Multi-step forms
- File uploads
- Error handling

**When to use:** Any form implementation  
**Applied to:** `**/components/Form*`, `**/components/*Form*`

---

#### [22-file-storage](22-file-storage/RULE.md)
**Purpose:** Firebase Storage for file uploads and management  
**Key Topics:**
- Storage security rules
- Image upload with preview
- Progress indicators
- Image optimization
- Unique filenames

**When to use:** Any file/image upload feature  
**Applied to:** `**/storage/**`, `**/upload/**`

---

#### [23-deployment](23-deployment/RULE.md)
**Purpose:** CI/CD, deployment, and environment management  
**Key Topics:**
- GitHub Actions workflows
- Firebase Hosting
- Staging vs Production
- Environment variables
- Preview deployments
- Rollback procedures

**When to use:** Setting up deployment pipelines  
**Applied to:** `**/.github/**`, `**/firebase.json`

---

#### [24-booking-system](24-booking-system/RULE.md)
**Purpose:** Appointment scheduling and reservation systems  
**Key Topics:**
- Availability management
- Time zone handling
- Double-booking prevention
- Calendar integration
- Booking notifications
- Cancellation flows

**When to use:** Any booking/scheduling feature  
**Applied to:** `**/booking/**`, `**/calendar/**`

---

#### [25-contact-forms](25-contact-forms/RULE.md)
**Purpose:** Contact forms and lead capture  
**Key Topics:**
- Form implementation
- Spam protection (reCAPTCHA)
- Honeypot fields
- Lead storage
- Auto-response emails

**When to use:** Contact pages, inquiry forms  
**Applied to:** `**/components/Contact*`

---

#### [26-notifications](26-notifications/RULE.md)
**Purpose:** Push and in-app notification systems  
**Key Topics:**
- Firebase Cloud Messaging
- In-app notifications
- Notification preferences
- Real-time updates
- Permission handling

**When to use:** Any notification feature  
**Applied to:** `**/notifications/**`

---

#### [27-analytics](27-analytics/RULE.md)
**Purpose:** Analytics, tracking, and user behavior  
**Key Topics:**
- Google Analytics 4
- Event tracking
- Consent management
- E-commerce events
- Page view tracking

**When to use:** Any production site  
**Applied to:** `**/analytics/**`

---

#### [28-search](28-search/RULE.md)
**Purpose:** Search implementation with Algolia or Firebase  
**Key Topics:**
- Algolia integration
- Autocomplete
- Search indexing
- Typo tolerance
- Result ranking

**When to use:** Content-heavy sites needing search  
**Applied to:** `**/search/**`

---

#### [29-admin-dashboard](29-admin-dashboard/RULE.md)
**Purpose:** Admin panels and content management  
**Key Topics:**
- Role-protected routes
- Data tables
- CRUD operations
- Dashboard stats
- Admin layout

**When to use:** Sites needing admin interface  
**Applied to:** `**/admin/**`, `**/dashboard/**`

---

#### [30-pwa-mobile](30-pwa-mobile/RULE.md)
**Purpose:** Progressive Web App and mobile optimization  
**Key Topics:**
- Web app manifest
- Service workers
- Offline support
- Install prompts
- Touch-friendly design

**When to use:** Mobile-focused or installable apps  
**Applied to:** `**/sw.js`, `**/manifest.json`

---

#### [31-theming-darkmode](31-theming-darkmode/RULE.md)
**Purpose:** Theming system and dark mode  
**Key Topics:**
- CSS custom properties
- System preference detection
- Theme toggle
- Persistent preferences
- Smooth transitions

**When to use:** Sites with theme customization  
**Applied to:** `**/theme/**`, `**/contexts/Theme*`

---

## 🎯 Rule Priority Matrix

### Must Follow (Critical)
1. **00-project-setup** - No code without prerequisites
2. **07-ai-decision-framework** - Think before acting
3. **14-error-handling** - Production-grade error handling
4. **03-firebase-standards** - Security is non-negotiable
5. **05-data-privacy** - Legal compliance required
6. **09-task-management** - Plan complex work

### Should Follow (Important)
7. **01-clean-architecture** - Maintainable code structure
8. **02-react-standards** - Quality React code
9. **10-database-design** - Proper data structure
10. **12-performance-standards** - Fast, efficient apps
11. **06-testing-standards** - Quality assurance
12. **11-documentation-standards** - Knowledge preservation
13. **13-seo-standards** - Discoverability and ranking

### Good to Follow (Enhancing)
14. **04-ui-ux-design** - Professional UI/UX
15. **08a-requirements-gathering** - Clear requirements
16. **08b-ai-color-generation** - Design assistance

### Apply When Needed (Feature-Specific)
17. **15-payment-processing** - If implementing payments
18. **16-email-system** - If sending emails
19. **17-internationalization** - If multi-language support
20. **18-blogging-standards** - If creating blog content
21. **19-gallery-products** - If displaying images/products
22. **20-authentication** - If user accounts required
23. **21-forms-validation** - For all forms
24. **22-file-storage** - If file uploads needed
25. **23-deployment** - For CI/CD setup
26. **24-booking-system** - If scheduling features
27. **25-contact-forms** - For contact/inquiry forms
28. **26-notifications** - If push/in-app notifications
29. **27-analytics** - For production tracking
30. **28-search** - If search functionality
31. **29-admin-dashboard** - If admin panel needed
32. **30-pwa-mobile** - For PWA features
33. **31-theming-darkmode** - For theme customization

---

## 🚦 Decision Flow

```
New Project Starting
    ↓
08a: Requirements Gathering (30-60 min)
    ↓
08b: Color Palette Generation (15-30 min)
    ↓
00: Project Setup & Prerequisites
    ↓
09: Break Down Features into Tasks
    ↓
For Each Feature:
    ↓
    07: AI Decision Gate (Think, Verify)
    ↓
    14: Error Handling Strategy
    ↓
    10: Database Design (if needed)
    ↓
    01: Clean Architecture Layer
    ↓
    02/04: Implement React/UI
    ↓
    12: Performance Optimization
    ↓
    03: Firebase Integration
    ↓
    05: Privacy Compliance (if collecting data)
    ↓
    13: SEO Implementation (if public page)
    ↓
    15/16/17: Advanced Features (if needed)
    ↓
    06: Write Tests
    ↓
    11: Document
    ↓
    09: Mark Todo Complete
    ↓
Next Feature
    ↓
All Features Complete
    ↓
Final Review: All Rules Compliant?
    ↓
Deploy! 🚀
```

---

## 🛡️ Protection Features

### For Beginners
- Plain English explanations throughout
- Mandatory checkpoints before critical actions
- AI stops without required information
- Examples for every concept
- Realistic timeline setting

### For Security
- No code without security rules (03)
- Privacy compliance checks (05)
- Data validation required (10)
- Audit logging enforced (05)

### For Quality
- Testing required before completion (06)
- Code review checklist (07, 09)
- Architecture patterns enforced (01)
- Database design validation (10)

### For Maintainability
- Documentation standards (11)
- Task breakdown required (09)
- Clean architecture (01)
- TypeScript types (02, 10)

---

## 📊 Coverage Map

| Task | Primary Rules | Supporting Rules |
|------|--------------|------------------|
| Starting new project | 00, 08a, 08b | 07, 09, 11, 14 |
| Building authentication | 01, 02, 03, 05 | 06, 10, 11, 14 |
| Creating UI components | 02, 04, 12 | 01, 06, 11, 13 |
| Database work | 03, 10 | 01, 05, 09, 14 |
| Collecting user data | 03, 05, 10 | 02, 11, 14 |
| Complex feature (>3 steps) | 09, 14 | 01, 07, 11, 12 |
| E-commerce/Payments | 15 | 03, 05, 14, 16 |
| Email notifications | 16 | 05, 11, 14 |
| Multi-language site | 17 | 04, 11, 13 |
| Blog/content site | 18 | 13, 11, 04 |
| Gallery/portfolio site | 19 | 12, 04, 13 |
| Small product catalog | 19 | 13, 04, 12 |
| User authentication | 20, 21 | 03, 05, 14 |
| File uploads | 22 | 03, 12, 14 |
| Booking/scheduling | 24 | 16, 26, 20 |
| Contact forms | 25, 21 | 16, 14 |
| Admin panel | 29, 20 | 10, 14 |
| Search feature | 28 | 12, 13 |
| PWA/mobile app | 30 | 12, 04 |
| Dark mode | 31 | 04, 02 |
| Performance issues | 12 | 02, 03, 13 |
| Before deployment | 23 | 06, 14, 27 |

---

## 🔄 Maintenance

### When to Update Rules
- New best practices emerge
- Team encounters repeated issues
- Technology stack changes
- Legal requirements change
- Team feedback indicates confusion

### How to Update
1. Update specific RULE.md file
2. Update this INDEX.md if structure changes
3. Test with AI to ensure it follows new rules
4. Communicate changes to team
5. Update project documentation

---

## 📞 Quick Reference

**"I'm starting a new project"**  
→ Start with 00, 08a, 08b

**"I need to add a complex feature"**  
→ Create todo breakdown (09), follow 01-02-03

**"I'm collecting user data"**  
→ Must review 05 (privacy), 10 (database), 03 (security)

**"I'm not sure what to do"**  
→ Ask AI, it will guide you through 07 (decision framework)

**"Something feels wrong with my code"**  
→ Review 01 (architecture), 02 (React standards), 10 (database design)

**"I'm a beginner and overwhelmed"**  
→ Start with 08a, let AI guide you with 07

**"My app is slow"**  
→ Review 12 (performance), check bundle size and queries

**"I need to accept payments"**  
→ Review 15 (payment processing), never store card data

**"I need to send emails"**  
→ Review 16 (email system), use SendGrid/Mailgun

**"I need multi-language support"**  
→ Review 17 (i18n), set up from day one

**"Errors are crashing my app"**  
→ Review 14 (error handling), implement error boundaries

**"My site doesn't show up on Google"**  
→ Review 13 (SEO), check meta tags and sitemap

**"I need to write blog posts"**  
→ Review 18 (blogging), use AI to guide content creation

**"How do I structure markdown blog posts?"**  
→ Review 18 (blogging), includes templates and examples

**"I need to display a product gallery"**  
→ Review 19 (gallery-products), simple layouts for 3-10 items

**"How do I create a lightbox for images?"**  
→ Review 19 (gallery-products), includes lightbox component

**"I need user login and registration"**  
→ Review 20 (authentication), complete Firebase Auth setup

**"How do I handle form validation?"**  
→ Review 21 (forms-validation), React Hook Form + Zod

**"I need to upload files/images"**  
→ Review 22 (file-storage), Firebase Storage patterns

**"How do I set up CI/CD?"**  
→ Review 23 (deployment), GitHub Actions workflows

**"I need a booking/appointment system"**  
→ Review 24 (booking-system), complete scheduling

**"I need a contact form with spam protection"**  
→ Review 25 (contact-forms), reCAPTCHA integration

**"I need push notifications"**  
→ Review 26 (notifications), FCM setup

**"How do I add Google Analytics?"**  
→ Review 27 (analytics), GA4 with consent

**"I need search functionality"**  
→ Review 28 (search), Algolia integration

**"I need an admin dashboard"**  
→ Review 29 (admin-dashboard), CRUD patterns

**"I want my app to work offline"**  
→ Review 30 (pwa-mobile), service worker setup

**"I need dark mode"**  
→ Review 31 (theming-darkmode), theme system

---

## ✅ Success Indicators

**This rule system is working when:**
- No security vulnerabilities in production
- New developers onboard smoothly
- Code is consistently maintainable
- Features are completed (not 80% done)
- No "surprise" requirements at end
- Minimal technical debt
- Happy users and happy developers

---

## 📝 Version History

**v3.0.0 - December 19, 2025** ⭐ MAJOR RELEASE
- Added 12 new comprehensive rules (20-31)
- **20-authentication** - Complete auth system with Firebase Auth
- **21-forms-validation** - React Hook Form + Zod patterns
- **22-file-storage** - Firebase Storage with security
- **23-deployment** - CI/CD with GitHub Actions
- **24-booking-system** - Scheduling and appointments
- **25-contact-forms** - Lead capture with spam protection
- **26-notifications** - Push and in-app notifications
- **27-analytics** - GA4 with consent management
- **28-search** - Algolia and Firebase search
- **29-admin-dashboard** - Admin panels and CRUD
- **30-pwa-mobile** - PWA with offline support
- **31-theming-darkmode** - Theme system and dark mode
- **Total Rules: 32** (was 20)
- Complete production-ready coverage for any web application

**v2.2.0 - December 19, 2025**
- Added gallery & product display (19)

**v2.1.0 - December 19, 2025**
- Added blogging standards (18) - Content creation guide
- Comprehensive blog writing rules
- Hook writing techniques (not React hooks!)
- Markdown formatting best practices
- AI-guided content improvement
- SEO for blog posts

**v2.0.0 - December 19, 2025**
- Added 6 advanced rules (12-17)
- Error handling & logging (14) - Critical for production
- Performance optimization (12)
- SEO standards (13)
- Payment processing (15)
- Email system (16)
- Internationalization (17)
- Complete production-ready coverage

**v1.0.0 - December 19, 2025**
- Initial complete rule set
- 12 comprehensive rules (00-11)
- Beginner-friendly focus
- AI-driven quality gates
- Complete React + Firebase coverage

---

**Need Help?** Ask the AI - it has access to all these rules and will guide you! 🤖✨

