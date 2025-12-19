---
description: "Updated project setup with beginner-friendly guidance and AI tool integration"
alwaysApply: true
---

# Project Setup & Prerequisites

## 🎯 START HERE (For Non-Technical Users)

**If you're new to web development, READ THIS FIRST:**

Building a website is like building a house. You need:
1. **Blueprints** (your design system) ← **DO THIS FIRST!**
2. **Materials** (your content and colors) ← **THEN THIS!**
3. **Foundation** (the technical setup) ← **WE'LL HELP WITH THIS!**
4. **Construction** (the actual coding) ← **ONLY AFTER ABOVE ARE DONE!**

**⚠️ IMPORTANT:** If you skip steps 1-2, the AI will have to guess. Guessing leads to mistakes. Mistakes cost time and money.

---

## 📋 Pre-Development Requirements

Before **ANY** development begins, complete these steps **IN ORDER:**

### ✅ STEP 1: Requirements Gathering (30-60 minutes)

**Action:** Complete the requirements interview in **Rule 08a**

```
📄 Go to: .cursor/rules/08a-requirements-gathering/RULE.md
🎯 Complete: All 11 questions
📝 Output: Requirements Document
```

**What you'll have after this:**
- Clear purpose statement
- List of all features needed
- List of all pages
- User personas
- Success criteria

**🚫 DO NOT PROCEED** until you have a completed requirements document.

---

### ✅ STEP 2: Color Palette Generation (15-30 minutes)

**Option A: You Have Existing Brand Colors**

```
If you already have:
☐ Logo with specific colors
☐ Brand guidelines
☐ Colors you must use

Action: Extract the hex codes
Example: #3B82F6, #10B981, #F59E0B

Document in: DESIGN_SYSTEM.md
```

**Option B: You Need to Generate Colors**

```
Action: Use AI to generate palette

📄 Go to: .cursor/rules/08b-ai-color-generation/RULE.md
🤖 Use: Gemini, ChatGPT, or Claude
⏱️ Time: 15-30 minutes
📝 Output: Complete color palette with hex codes
```

**What you'll have after this:**
- Primary color (your main brand color)
- Secondary color (supporting color)
- Accent color (for buttons and highlights)
- Neutral colors (grays for text/backgrounds)
- Semantic colors (success, warning, error)

**🚫 DO NOT PROCEED** without a documented, tested color palette.

---

### ✅ STEP 3: Design System Documentation (15 minutes)

Create `DESIGN_SYSTEM.md` in your project root:

```markdown
# Design System - [Project Name]

## Color Palette

### Primary: [Color Name]
- Hex: #______
- Use for: [When to use this color]

### Secondary: [Color Name]
- Hex: #______
- Use for: [When to use this color]

### Accent: [Color Name]
- Hex: #______
- Use for: [When to use this color]

### Neutrals
- Text Dark: #______ (for headings)
- Text Normal: #______ (for body text)
- Text Light: #______ (for secondary text)
- Background: #______ (page background)
- Surface: #______ (cards, panels)
- Border: #______ (dividers, borders)

### Semantic
- Success: #______ (green)
- Warning: #______ (amber)
- Error: #______ (red)
- Info: #______ (blue)

## Typography

### Fonts
- Heading Font: [Font Name] (backup: sans-serif)
- Body Font: [Font Name] (backup: sans-serif)
- Monospace: [Font Name] (backup: monospace)

### Font Sizes
- Heading 1: 48px (3rem)
- Heading 2: 36px (2.25rem)
- Heading 3: 30px (1.875rem)
- Heading 4: 24px (1.5rem)
- Body Large: 18px (1.125rem)
- Body: 16px (1rem)
- Body Small: 14px (0.875rem)
- Caption: 12px (0.75rem)

## Spacing

Use multiples of 4px:
- 4px, 8px, 12px, 16px, 24px, 32px, 48px, 64px

## Buttons

### Primary Button
- Background: [Accent Color]
- Text: White
- Hover: [Darker shade of accent]
- Padding: 12px 24px
- Border Radius: 8px

### Secondary Button
- Background: Transparent
- Text: [Accent Color]
- Border: 2px solid [Accent Color]
- Hover: Light tint of accent as background
- Padding: 12px 24px
- Border Radius: 8px

## Forms

### Input Fields
- Border: 1px solid [Border Color]
- Focus: 2px solid [Accent Color]
- Padding: 12px 16px
- Border Radius: 8px
- Error State: Border becomes [Error Color]

## Breakpoints

- Mobile: 0-639px
- Tablet: 640-1023px
- Desktop: 1024-1279px
- Large: 1280px+

## Shadows

- Small: 0 1px 3px rgba(0,0,0,0.12)
- Medium: 0 4px 6px rgba(0,0,0,0.10)
- Large: 0 10px 20px rgba(0,0,0,0.15)

## Reference Sites

[List 2-3 websites whose design you like]
- https://example.com - [What you like about it]
- https://example.com - [What you like about it]
```

**🚫 DO NOT PROCEED** without this file completed.

---

### ✅ STEP 4: Content Inventory (Variable Time)

**Check what content you have:**

```
☐ Logo (SVG or high-res PNG)
☐ Text for all pages
☐ Images/photos
☐ Videos (if applicable)
☐ Contact information
☐ Social media links
☐ Privacy policy text
☐ Terms of service text

If MISSING content:
- Use placeholders during development
- Mark clearly what needs replacement
- Set deadline for content delivery
```

**⚠️ You CAN proceed with placeholders, but document what's missing!**

---

### ✅ STEP 5: Technical Setup (AI Can Help With This!)

Now the AI can help set up the technical foundation:

```
Firebase Project:
☐ Create account at firebase.google.com
☐ Create new project
☐ Enable Authentication (if users log in)
☐ Enable Firestore (if storing data)
☐ Enable Storage (if uploading files)
☐ Copy project config (API keys)

Domain (Optional for start):
☐ Domain purchased (e.g., from Namecheap, Google Domains)
☐ DNS configured

Development Tools:
☐ Node.js installed (nodejs.org)
☐ Code editor installed (VS Code recommended)
☐ Git installed (for version control)
```

**AI can guide you through this step-by-step!**

---

## 🗂️ Initial Project Structure

After above steps complete, create this structure:

```
project-root/
├── .cursor/
│   └── rules/              # This folder (rules you're reading now)
├── docs/
│   ├── REQUIREMENTS.md     # From Step 1
│   └── DESIGN_SYSTEM.md    # From Step 3
├── src/
│   ├── components/         # Reusable UI components (AI creates these)
│   ├── features/           # Feature modules (AI creates these)
│   ├── hooks/              # Custom React hooks (AI creates these)
│   ├── services/           # Firebase services (AI creates these)
│   ├── utils/              # Helper functions (AI creates these)
│   ├── types/              # TypeScript types (AI creates these)
│   ├── config/             # Configuration (AI creates these)
│   └── App.tsx             # Main app file (AI creates this)
├── public/
│   ├── images/             # Your images go here (YOU provide these)
│   └── favicon.ico         # Site icon (YOU provide this)
├── .env.example            # Environment variables template
├── .gitignore              # Git ignore file
├── package.json            # Dependencies
├── README.md               # Project overview
└── tailwind.config.js      # Styling configuration
```

**The AI will create the technical files. You provide the content and design specs.**

---

## 🛡️ AI Protection System

The AI will **STOP and ASK** if these are missing:

### Checkpoint 1: Before Starting Development
```
AI CHECKS:
☐ REQUIREMENTS.md exists and is complete
☐ DESIGN_SYSTEM.md exists with color palette
☐ All colors have hex codes
☐ Typography is defined
☐ Spacing system is defined

IF MISSING → AI will ask you to complete them first
```

### Checkpoint 2: Before Creating Components
```
AI CHECKS:
☐ Component is defined in requirements
☐ Colors to use are specified
☐ Layout/structure is described

IF UNCLEAR → AI will ask for clarification
```

### Checkpoint 3: Before Collecting User Data
```
AI CHECKS:
☐ Privacy policy exists
☐ Data collection is justified
☐ User consent mechanism planned
☐ Data deletion feature planned

IF MISSING → AI will warn you and require confirmation
```

---

## 📝 Required Documentation Files

Create these files before development starts:

### 1. README.md

```markdown
# [Project Name]

## What This Project Does
[One sentence description]

## Tech Stack
- Frontend: React + TypeScript
- Backend: Firebase (Firestore, Auth, Storage)
- Styling: Tailwind CSS
- Hosting: Firebase Hosting

## Setup Instructions

1. Clone repository
2. Install dependencies: `npm install`
3. Copy `.env.example` to `.env` and fill in Firebase config
4. Run development server: `npm run dev`
5. Open browser to `http://localhost:5173`

## Project Status
- [x] Requirements documented
- [x] Design system created
- [ ] Development in progress
- [ ] Testing
- [ ] Deployed

## Contact
[Your name/email]
```

### 2. REQUIREMENTS.md
(Created in Step 1 using Rule 08a)

### 3. DESIGN_SYSTEM.md
(Created in Step 3)

### 4. API_DOCUMENTATION.md
(AI will help create this based on your data needs)

### 5. DEPLOYMENT.md
(AI will create this when you're ready to deploy)

---

## 🎓 For Non-Technical Users: What Each File Does

**Think of it like building a house:**

```
REQUIREMENTS.md = Architectural blueprints
"What rooms do you need? How many bathrooms? Kitchen layout?"

DESIGN_SYSTEM.md = Interior design plan
"What colors? What style furniture? Modern or traditional?"

package.json = Shopping list of materials
"Lumber, nails, paint, fixtures needed"

src/components/ = Prefabricated parts
"Pre-built doors, windows, cabinets"

src/features/ = Room-by-room construction
"Build the kitchen, then the bathroom, then bedrooms"

Firebase = Utility connections
"Electricity, water, internet connections"

.env = House address and utility account numbers
"Your specific account details, kept private"
```

---

## 🚀 Ready to Start Checklist

```
BEFORE ASKING AI TO START CODING:

Documentation
☐ REQUIREMENTS.md exists with all 11 questions answered
☐ DESIGN_SYSTEM.md exists with complete color palette
☐ README.md exists with project description
☐ Content inventory completed (or placeholders noted)

Design
☐ Color palette has hex codes for all colors
☐ Colors tested for accessibility (WCAG AA)
☐ Typography fonts chosen
☐ Spacing system defined (we recommend 4px increments)
☐ Button styles defined

Technical
☐ Firebase project created (if needed)
☐ Firebase config copied
☐ Domain purchased (optional, can add later)
☐ Development environment ready (Node.js, editor)

Legal (If collecting user data)
☐ Privacy policy drafted
☐ Terms of service drafted
☐ Cookie consent planned
☐ Data export feature planned
☐ Data deletion feature planned

Timeline
☐ Realistic deadline set
☐ Milestones defined
☐ Review schedule planned

IF ALL CHECKED ✅ → Ready for development!
IF ANY UNCHECKED ❌ → Complete before proceeding
```

---

## 💬 What to Say to AI to Start

**After completing all above steps:**

```
"I'm ready to start building my website. I have:

✅ Completed requirements document
✅ Complete color palette with hex codes
✅ Design system documented
✅ Content inventory
✅ Firebase project created

Here's my project:
- Type: [e.g., Yoga studio booking website]
- Features: [List top 3-5 features]
- Color palette: [Your primary, secondary, accent]
- Tech stack: React + TypeScript + Firebase

I'm ready to start with [SPECIFIC FEATURE].
Can you help me build [FEATURE] following clean architecture 
and all the rules we have in place?"
```

---

## ⚠️ What NOT to Say

**❌ DON'T SAY:**
- "Just build me a website" (too vague)
- "Make it look good" (no design specs)
- "Use whatever colors" (no brand identity)
- "Skip the documentation" (recipe for disaster)
- "I'll figure out privacy later" (legal risk!)

**✅ DO SAY:**
- "Let's start with the requirements gathering"
- "Help me generate a color palette"
- "I need to document my design system first"
- "What information do you need from me?"
- "Let's make sure we follow all best practices"

---

## 🎯 AI Decision Gate

**The AI will enforce these gates:**

```
┌─────────────────────────────────────────┐
│  BEFORE ANY CODE IS WRITTEN             │
├─────────────────────────────────────────┤
│  ☐ Requirements document exists         │
│  ☐ Design system document exists        │
│  ☐ Color palette has ALL hex codes      │
│  ☐ Typography is defined                │
│  ☐ Spacing system is defined            │
│  ☐ Firebase project created (if needed) │
│                                         │
│  IF ANY ☐ UNCHECKED:                    │
│  ➜ AI STOPS and guides you to complete │
│                                         │
│  NEVER PROCEEDS WITHOUT THESE! ⛔       │
└─────────────────────────────────────────┘
```

---

## 📚 Additional Resources

### For Design
- **Coolors.co** - Visualize color palettes
- **Fonts.google.com** - Free fonts to use
- **Unsplash.com** - Free stock photos
- **Font Awesome** - Free icons

### For Learning
- **React Tutorial** - react.dev/learn
- **Firebase Docs** - firebase.google.com/docs
- **Web Accessibility** - web.dev/accessibility

### For Tools
- **VS Code** - code.visualstudio.com (code editor)
- **Node.js** - nodejs.org (required for React)
- **Git** - git-scm.com (version control)

---

**Remember: Proper preparation prevents poor performance. Take time to set up correctly, and development will be smooth and fast! 🚀**
