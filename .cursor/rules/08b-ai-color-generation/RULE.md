---
description: "Guide for using AI tools (Gemini, ChatGPT, Claude) to generate professional color palettes and design systems"
alwaysApply: false
---

# AI-Powered Color Palette Generation

## 🎨 Purpose

This guide helps you use AI tools like **Gemini Nano**, **ChatGPT**, **Claude**, or **Midjourney** to generate professional color palettes when you don't have existing brand colors.

---

## 🚀 Quick Start: Generating a Color Palette

### Step 1: Gather Your Brand Keywords

From your requirements document (rule 08a), you have 3-5 brand personality words.

**Example:**
- Professional, Trustworthy, Modern (Law Firm)
- Fun, Playful, Energetic (Kids Activity Center)
- Calm, Peaceful, Natural (Yoga Studio)
- Bold, Innovative, Tech-Forward (Tech Startup)

---

### Step 2: Choose Your AI Tool

#### Option A: Gemini (Google)
- **Best for:** Quick color generation, understanding context
- **Access:** gemini.google.com

#### Option B: ChatGPT (OpenAI)
- **Best for:** Detailed explanations, multiple options
- **Access:** chat.openai.com

#### Option C: Claude (Anthropic)
- **Best for:** Comprehensive design systems
- **Access:** claude.ai

#### Option D: Coolors AI
- **Best for:** Visual palette generation
- **Access:** coolors.co

---

## 📝 Prompt Templates for AI Color Generation

### 🟢 BASIC PROMPT (Good for Beginners)

```
I'm building a website for [TYPE OF BUSINESS]. 
My brand personality is: [KEYWORDS FROM REQUIREMENTS].

Please generate a professional color palette with:
- 1 Primary color (most used)
- 1 Secondary color (used sometimes)
- 1 Accent color (for buttons, highlights)
- Neutral colors (grays for text and backgrounds)

For each color, provide:
- Hex code (e.g., #3B82F6)
- Color name (e.g., Ocean Blue)
- When to use it
- Accessibility notes

Make sure colors work well together and are WCAG AA compliant.
```

**Example Usage:**
```
I'm building a website for a yoga studio.
My brand personality is: Calm, Peaceful, Natural, Welcoming, Zen.

Please generate a professional color palette with:
[rest of prompt above]
```

---

### 🔵 ADVANCED PROMPT (More Control)

```
Create a comprehensive color system for my [TYPE] website.

BRAND PERSONALITY:
- Keywords: [Your 3-5 words]
- Industry: [Your industry]
- Target audience: [Age, demographics]
- Emotional response desired: [How should users feel?]

EXISTING CONSTRAINTS:
- Must work with: [Existing logo colors if any]
- Must avoid: [Colors competitors use or colors you dislike]
- Inspiration: [URLs of sites with colors you like]

REQUIREMENTS:
- Primary color: [Any preferences?]
- Secondary color: [Any preferences?]
- Accent color: [Prefer warm or cool tones?]
- Neutral palette: [Prefer warm grays or cool grays?]
- Semantic colors needed: Success (green), Warning (yellow), Error (red), Info (blue)

TECHNICAL NEEDS:
- Must be WCAG AA compliant (accessible)
- Provide shades (50, 100, 200... 900) for each color
- Provide HSL, RGB, and Hex codes
- Show color contrast ratios
- Suggest Tailwind CSS configuration

OUTPUT FORMAT:
Provide as JSON and visual preview.
```

---

### 🟡 PROMPT FOR DESCRIBING EXISTING SITES

If you have example sites you love:

```
I want colors similar to [WEBSITE URL].

Please analyze that website's color palette and:
1. Identify their primary, secondary, and accent colors
2. Extract the hex codes
3. Explain why these colors work well together
4. Suggest how I can adapt this palette for my [TYPE OF BUSINESS]
5. Ensure the adapted palette is distinct enough to be unique

My business: [DESCRIPTION]
My brand keywords: [YOUR KEYWORDS]

Create a similar but unique palette for me.
```

---

### 🟣 PROMPT FOR COLOR PSYCHOLOGY

For deeper understanding:

```
Explain the psychology of colors for a [TYPE OF BUSINESS].

My brand personality: [KEYWORDS]

For each potential color choice (blue, green, red, yellow, purple, orange), explain:
1. What emotions it evokes
2. What industries typically use it
3. Whether it fits my brand personality
4. Accessibility considerations
5. Cultural considerations

Then recommend the best color palette for my specific needs.
```

---

## 🎯 Complete Example Workflow

### Example: Yoga Studio Website

#### Step 1: Define Requirements
```
Business Type: Yoga Studio
Target Users: Women 25-45, health-conscious, seeking peace
Brand Keywords: Calm, Peaceful, Natural, Welcoming, Zen
Goal: Make users feel relaxed and eager to book classes
```

#### Step 2: Generate Palette with AI

**Prompt to AI:**
```
I'm building a website for a yoga and meditation studio.

BRAND PERSONALITY:
- Calm, Peaceful, Natural, Welcoming, Zen
- Target: Women 25-45 who want stress relief
- Should feel: Relaxed, safe, inspired

REQUIREMENTS:
- Must evoke calmness and nature
- Avoid bright, aggressive colors
- Should work well with photos of yoga poses
- Need warm, inviting feel (not cold/clinical)
- Accessible (WCAG AA compliant)

Please generate:
- Primary color (main brand color)
- Secondary color (supports primary)
- Accent color (for CTAs like "Book Class")
- Neutral palette (backgrounds, text)
- Semantic colors (success, warning, error)

Provide shades (50-900) and Tailwind CSS config.
```

#### Step 3: Review AI Output

**Example AI Response:**
```json
{
  "palette": {
    "primary": {
      "name": "Sage Green",
      "50": "#f0f7f4",
      "100": "#dcede3",
      "200": "#bcdbc9",
      "300": "#90c2a6",
      "400": "#5fa17d",
      "500": "#3d8560",
      "600": "#2d6a4c",
      "700": "#25563f",
      "800": "#204534",
      "900": "#1c392c"
    },
    "secondary": {
      "name": "Warm Sand",
      "50": "#faf8f5",
      "100": "#f2ede3",
      "200": "#e7dcc8",
      "300": "#d7c5a4",
      "400": "#c5a978",
      "500": "#b89159",
      "600": "#a77b4d",
      "700": "#8b6441",
      "800": "#715239",
      "900": "#5d4530"
    },
    "accent": {
      "name": "Terracotta",
      "50": "#fdf4f3",
      "100": "#fbe8e5",
      "200": "#f7d5d0",
      "300": "#f1b5ac",
      "400": "#e88b7d",
      "500": "#dc6b59",
      "600": "#c84d3b",
      "700": "#a73f2f",
      "800": "#8b372a",
      "900": "#753228"
    },
    "neutral": {
      "white": "#FFFFFF",
      "50": "#fafafa",
      "100": "#f5f5f5",
      "200": "#e5e5e5",
      "300": "#d4d4d4",
      "400": "#a3a3a3",
      "500": "#737373",
      "600": "#525252",
      "700": "#404040",
      "800": "#262626",
      "900": "#171717",
      "black": "#000000"
    },
    "semantic": {
      "success": "#10B981",
      "warning": "#F59E0B",
      "error": "#EF4444",
      "info": "#3B82F6"
    }
  },
  "usage": {
    "primary": "Main brand color. Use for: headers, navigation, yoga instructor photos overlay",
    "secondary": "Supporting color. Use for: section backgrounds, cards, subtle highlights",
    "accent": "Call-to-action color. Use for: 'Book Class' button, important links, hover states",
    "neutral": "Use for: text, backgrounds, borders"
  },
  "accessibility": {
    "primary500OnWhite": "4.52:1 (AA Pass)",
    "accentOnWhite": "4.85:1 (AA Pass)",
    "notes": "All text colors meet WCAG AA standards"
  }
}
```

#### Step 4: Validate with Visual Tools

Use Coolors.co to visualize:
```
https://coolors.co/3d8560-b89159-dc6b59-f5f5f5-262626
```

#### Step 5: Test Accessibility

Use WebAIM Contrast Checker:
```
https://webaim.org/resources/contrastchecker/

Test combinations:
- Primary (#3d8560) on White (#FFFFFF) ✅ Pass
- Accent (#dc6b59) on White (#FFFFFF) ✅ Pass
- Neutral-900 (#171717) on White (#FFFFFF) ✅ Pass
```

#### Step 6: Document in Design System

```markdown
# Color Palette: Peaceful Yoga Studio

## Brand Colors

### Primary: Sage Green
**Purpose:** Main brand color representing nature and growth
- Hex: #3d8560
- RGB: rgb(61, 133, 96)
- HSL: hsl(149, 37%, 38%)
- Use for: Navigation, headers, logo accent, section dividers

### Secondary: Warm Sand
**Purpose:** Creates warmth and comfort
- Hex: #b89159
- RGB: rgb(184, 145, 89)
- HSL: hsl(35, 40%, 54%)
- Use for: Background sections, card backgrounds, subtle highlights

### Accent: Terracotta
**Purpose:** Draws attention without being aggressive
- Hex: #dc6b59
- RGB: rgb(220, 107, 89)
- HSL: hsl(8, 65%, 61%)
- Use for: CTA buttons ("Book Class", "Sign Up"), important notifications, hover effects

## Neutral Palette

### Text Colors
- Heading: #171717 (Neutral-900)
- Body: #404040 (Neutral-700)
- Muted: #737373 (Neutral-500)

### Backgrounds
- Primary: #FFFFFF (White)
- Secondary: #fafafa (Neutral-50)
- Tertiary: #f5f5f5 (Neutral-100)

## Semantic Colors

- Success: #10B981 (Green) - Booking confirmed
- Warning: #F59E0B (Amber) - Class almost full
- Error: #EF4444 (Red) - Booking failed
- Info: #3B82F6 (Blue) - General information

## Accessibility

All color combinations meet WCAG 2.1 AA standards:
- Text contrast: Minimum 4.5:1
- Large text: Minimum 3:1
- UI components: Minimum 3:1
```

---

## 🧪 Testing Your Generated Palette

### Test 1: The Squint Test

Display your palette full-screen and squint. Do the colors:
- ✅ Feel harmonious?
- ✅ Have clear hierarchy (primary stands out)?
- ✅ Look professional?
- ❌ Clash or feel jarring?

### Test 2: The Mobile Test

View on your phone. Colors often look different on mobile screens.

### Test 3: The Gray Scale Test

Convert your design to grayscale. Can you still tell what's important?

### Test 4: The Accessibility Test

```
CHECKLIST:
☐ Text on backgrounds has 4.5:1 contrast ratio minimum
☐ Large text (18pt+) has 3:1 contrast ratio minimum
☐ Buttons/links have 3:1 contrast with surrounding colors
☐ Color is not the ONLY way to convey information
☐ Tested with color blindness simulator
```

### Test 5: The Real Content Test

Apply colors to real content (not lorem ipsum). Does it still work?

---

## 🎨 Alternative: Using Visual AI Tools

### Midjourney/DALL-E for Color Inspiration

**Prompt:**
```
A color palette mood board for a [TYPE] website,
[BRAND KEYWORDS], professional branding colors,
color swatches with hex codes, clean layout
--v 5 --ar 16:9
```

Then extract colors from the generated image using:
- Adobe Color (color.adobe.com)
- ImageColorPicker.com
- Coolors.co (upload image)

---

## 🔄 Iterating on Your Palette

If the first palette doesn't feel right:

**Refinement Prompt:**
```
The palette you generated is close, but:

KEEP: [What works]
CHANGE: [What doesn't work]
DIRECTION: [Make it more/less __________]

Reasons it's not perfect:
- [Specific feedback]

Generate 3 variations based on this feedback.
```

**Example:**
```
The Sage Green and Sand work perfectly, but:

KEEP: Primary and Secondary
CHANGE: The Terracotta accent feels too aggressive
DIRECTION: Make the accent more subtle and peaceful

Reasons:
- Terracotta feels too energetic for a meditation space
- Would prefer something more aligned with sunset/twilight
- Should still be noticeable for CTA buttons

Generate 3 accent color alternatives.
```

---

## 📋 Checklist: Before Finalizing Palette

```
BEFORE YOU COMMIT TO THIS PALETTE:

Design Quality
☐ Colors represent the brand personality
☐ Palette feels cohesive (colors work together)
☐ Clear visual hierarchy (primary > secondary > accent)
☐ Appropriate for target audience
☐ Distinct from competitors

Technical Requirements
☐ All colors have hex codes
☐ All combinations pass WCAG AA
☐ Shades provided (50-900) for each color
☐ Semantic colors defined
☐ Works in light and dark mode (if needed)

Documentation
☐ Each color has a name
☐ Usage guidelines written
☐ "When to use" examples provided
☐ Saved in design system document
☐ Tailwind config generated

Testing
☐ Tested on multiple devices
☐ Tested in grayscale
☐ Tested with color blindness simulator
☐ Tested with real content
☐ Approved by stakeholder/client

IF ALL CHECKED ✅ → Proceed with development
IF ANY UNCHECKED ❌ → Address before coding
```

---

## 🔗 Useful AI Tools & Resources

### Color Generation
- **Coolors.co** - Generate and adjust palettes visually
- **Realtime Colors** - See colors on real UI mockups
- **Adobe Color** - Color wheel and harmony rules
- **Paletton** - Color scheme designer
- **Khroma** - AI color combinations trained on your preferences

### Accessibility Testing
- **WebAIM Contrast Checker** - Test color contrast
- **Whocanuse.com** - See how colors appear to users with different vision
- **Color Oracle** - Simulate color blindness
- **Stark Plugin** (Figma) - Accessibility checker

### AI Assistants for Design
- **Gemini** (gemini.google.com) - Fast palette generation
- **ChatGPT** (chat.openai.com) - Detailed design reasoning
- **Claude** (claude.ai) - Comprehensive design systems
- **Midjourney** (midjourney.com) - Visual mood boards

---

## 🎓 Understanding Color Theory Basics

### The 60-30-10 Rule

**In your design:**
- **60%** - Primary/Neutral (backgrounds, main sections)
- **30%** - Secondary (supporting elements, sections)
- **10%** - Accent (buttons, links, highlights)

```
Example Yoga Studio Homepage:
- 60%: White/Warm Sand backgrounds
- 30%: Sage Green (header, sections, images)
- 10%: Terracotta (CTA buttons, links)
```

### Color Harmony Types

Ask AI to generate specific harmony types:

1. **Complementary** - Opposite colors (high contrast)
   - "Generate complementary color scheme for [business]"

2. **Analogous** - Adjacent colors (harmonious)
   - "Generate analogous color scheme for [business]"

3. **Triadic** - Three evenly spaced colors (balanced)
   - "Generate triadic color scheme for [business]"

4. **Monochromatic** - Shades of one color (simple)
   - "Generate monochromatic scheme based on [color]"

---

## 💡 Pro Tips

### Tip 1: Start with Inspiration, Not Generation

**Better approach:**
1. Find 3-5 websites you love
2. Ask AI: "Analyze the color palettes of these sites"
3. Then: "Create a unique palette inspired by these, but for [my business]"

### Tip 2: Consider Your Content

**If your site has:**
- **Lots of photos** → Use muted, neutral palette (let photos shine)
- **Minimal images** → Use bolder colors for visual interest
- **Text-heavy** → Prioritize readability (high contrast)
- **E-commerce** → Use colors that make products pop

### Tip 3: Cultural Considerations

Ask AI:
```
"Does this color palette have any negative cultural associations 
in [COUNTRIES WHERE USERS ARE]? Suggest alternatives if needed."
```

### Tip 4: Save Multiple Options

Generate 3 palettes, then choose. It's easier to pick than to perfect one.

```
"Generate 3 different color palettes for my [business]:
1. Bold and energetic
2. Professional and trustworthy
3. Calm and minimal

For each, provide full specifications."
```

---

## 🚫 Common Mistakes to Avoid

### Mistake 1: Too Many Colors
```
❌ BAD: 5 primary colors, 3 secondaries, 4 accents = CHAOS
✅ GOOD: 1 primary, 1 secondary, 1 accent, neutrals
```

### Mistake 2: Ignoring Accessibility
```
❌ BAD: Light gray text on white background (can't read)
✅ GOOD: Dark gray (#404040) on white (4.5:1 contrast)
```

### Mistake 3: Following Trends Over Brand
```
❌ BAD: "Everyone uses purple now, let's use purple"
✅ GOOD: "Purple represents luxury, which fits our jewelry brand"
```

### Mistake 4: No Documentation
```
❌ BAD: "I'll remember what #3d8560 is for"
✅ GOOD: Document every color with name and usage
```

---

## ✅ AI Decision Gate

**BEFORE proceeding to development with your palette:**

```
☐ Palette generated using one of the prompt templates above
☐ All colors have hex codes documented
☐ All combinations tested for accessibility (WCAG AA)
☐ Colors align with brand personality keywords
☐ Shades (50-900) provided for primary colors
☐ Usage guidelines written for each color
☐ Semantic colors defined
☐ Tested on multiple devices
☐ Client/stakeholder approved
☐ Saved in DESIGN_SYSTEM.md

IF ANY BOX UNCHECKED → Don't start development
IF ALL CHECKED → Proceed with confidence!
```

---

**Remember: A thoughtfully chosen color palette is the foundation of great design. Take the time to get it right!**

