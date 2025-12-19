---
description: "Beginner-friendly requirements gathering and documentation guide for non-technical users"
alwaysApply: true
---

# Requirements Gathering for Non-Technical Users

## 🎯 Purpose

This guide helps **anyone** - even if you've never built a website before - clearly define what they need so the AI can build it correctly. Think of this as a conversation template to extract all the information needed.

## The "Discovery Interview" Framework

### Before ANY code is written, conduct this interview with yourself or the client.

---

## 📋 PART 1: The Basics (Must Have)

### Question 1: What is this website for?

**Prompt for the user (dev asks client):**
```
"In one sentence, what does this website do?"

Examples:
✅ "A booking system for my yoga studio"
✅ "A portfolio to showcase my photography"
✅ "An online store for handmade jewelry"
❌ "A website" (too vague)
❌ "Something cool" (not specific)
```

**AI Action:** Document this as the project purpose. If vague, keep asking until clear.

---

### Question 2: Who will use this website?

**Prompt for the user:**
```
"Describe your typical user in detail:"

- How old are they?
- Are they tech-savvy or beginners?
- Will they use phones, tablets, or computers?
- What do they want to accomplish?

Example Answer:
"My users are women aged 25-45 who want to book yoga classes. 
They'll mostly use their phones while commuting. They're not 
very tech-savvy, so it needs to be simple and obvious."
```

**AI Action:** Document user persona. This drives all design decisions.

---

### Question 3: What are the main things users need to do?

**Prompt for the user:**
```
"List 3-5 main actions users need to take:"

Example for Yoga Studio:
1. Browse class schedule
2. Book a class
3. Cancel a booking
4. See their upcoming classes
5. Pay for classes

Example for Portfolio:
1. View photo galleries
2. Read about the photographer
3. Contact for bookings
4. Download pricing info
```

**AI Action:** Create feature checklist. Each becomes a development task.

---

### Question 4: Do you need user accounts?

**Prompt for the user:**
```
"Answer YES or NO to each:"

☐ Do users need to log in? (YES/NO)
☐ Do users need to save information? (YES/NO)
☐ Do different users see different things? (YES/NO)
☐ Do you need an admin area? (YES/NO)
☐ Do users need to pay for things? (YES/NO)

If ANY answer is YES → You need Firebase Authentication + Database
If ALL are NO → You might only need a simple static website
```

**AI Action:** Determine architecture complexity. Stop if Firebase not needed but specified in stack.

---

## 🎨 PART 2: Design & Branding (Required Before Coding)

### Question 5: What's your brand personality?

**Prompt for the user:**
```
"Choose 3-5 words that describe how your brand should FEEL:"

Examples:
- Professional, Trustworthy, Modern
- Fun, Playful, Colorful
- Calm, Peaceful, Natural
- Bold, Edgy, Innovative
- Luxurious, Elegant, Sophisticated

Your choices: ________________
```

**AI Action:** Use these keywords for the AI color palette generator (see rule 08b).

---

### Question 6: Do you have existing branding?

**Prompt for the user:**
```
"Do you already have:"

☐ A logo? (Attach file or describe colors)
☐ Brand colors? (Provide hex codes like #FF5733)
☐ Specific fonts you must use?
☐ Example websites you love? (Provide URLs)

If YES to any: Provide details
If NO to all: We'll generate a complete design system for you
```

**AI Action:** 
- IF has branding → Extract colors and use existing palette
- IF no branding → Guide to rule 08b (AI Color Palette Generation)

---

## 📝 PART 3: Content (What Goes On The Site)

### Question 7: What pages do you need?

**Prompt for the user:**
```
"List every page your website needs:"

Example for Yoga Studio:
1. Home page (what we do)
2. Class schedule page
3. About the instructor
4. Pricing page
5. Booking page
6. My Account page (after login)
7. Contact page

Your pages:
1. ________________
2. ________________
3. ________________
(etc.)
```

**AI Action:** Create site map document. Each page becomes a component task.

---

### Question 8: What content do you have ready?

**Prompt for the user:**
```
"For each page, do you have:"

☐ Text/copy written?
☐ Images/photos ready?
☐ Videos to include?
☐ Documents to download (like price lists)?

If NO: We'll use placeholder content, but you'll need to provide 
real content before launch.
```

**AI Action:** 
- Flag missing content in documentation
- Use appropriate placeholders
- Create content checklist for launch

---

## 🔒 PART 4: Legal & Privacy (Critical!)

### Question 9: Where are your users located?

**Prompt for the user:**
```
"Where are your users/customers?"

☐ Europe (GDPR applies - strict privacy rules)
☐ South Africa (POPIA applies)
☐ United States (various state laws)
☐ Other: ________________

This determines what privacy features are REQUIRED BY LAW.
```

**AI Action:** 
- Document compliance requirements
- MUST implement data export/deletion if GDPR/POPIA applies
- Refer to rule 05-data-privacy

---

### Question 10: What data will you collect?

**Prompt for the user:**
```
"List everything you'll collect from users:"

Example for Yoga Studio:
- Name
- Email
- Phone number
- Class bookings
- Payment information
- Health conditions (for safety)

Your data:
- ________________
- ________________
```

**AI Action:**
- For EACH data point, verify it's necessary (data minimization)
- Ensure privacy policy covers all data collected
- Implement proper security rules

---

## 🚀 PART 5: Technical Details (Dev Fills This Out)

### Question 11: Deployment & Hosting

**Prompt for the dev:**
```
"Confirm technical setup:"

☐ Firebase project created? (Project ID: _________)
☐ Domain name purchased? (www.________.com)
☐ Who will maintain this? (Client / Agency / Freelancer)
☐ Budget for hosting? ($___/month)
☐ Expected traffic? (_____ users/month)
```

**AI Action:** Document in deployment section. Create deployment guide.

---

## 📄 THE REQUIREMENTS DOCUMENT

After completing the interview, create this document:

```markdown
# Project Requirements Document
**Project Name:** [Name]
**Date:** [Date]
**Created by:** [Name]

## 1. Project Overview
**Purpose:** [One sentence from Question 1]
**Target Users:** [From Question 2]

## 2. Core Features
[List from Question 3]
- Feature 1
- Feature 2
(etc.)

## 3. Authentication Required
- User Login: YES/NO
- Admin Panel: YES/NO
- Payment Processing: YES/NO

## 4. Design System
**Brand Personality:** [Keywords from Question 5]

**Color Palette:**
- Primary: #______ (Name: e.g., Ocean Blue)
- Secondary: #______ (Name: e.g., Coral)
- Accent: #______ (Name: e.g., Sunshine Yellow)
- Neutral: #______ (Name: e.g., Soft Gray)

**Typography:**
- Heading Font: [Font Name]
- Body Font: [Font Name]

**Reference Sites:** [URLs from Question 6]

## 5. Site Structure
[List from Question 7]
1. Page Name → Purpose → Key Content
2. Page Name → Purpose → Key Content
(etc.)

## 6. Content Status
- [ ] All text written
- [ ] Images collected
- [ ] Videos ready
- [ ] Forms defined

## 7. Legal Compliance
**Applicable Laws:** [From Question 9]
- [ ] Privacy Policy drafted
- [ ] Terms of Service drafted
- [ ] Cookie Consent implemented
- [ ] Data Export feature
- [ ] Data Deletion feature

**Data Collected:**
[List from Question 10]

## 8. Technical Setup
- Firebase Project: [ID]
- Domain: [URL]
- Expected Launch: [Date]

## 9. Success Criteria
**How do we know this project is successful?**
[User defines success metrics]

Example:
- Users can book classes in under 2 minutes
- Mobile-friendly (works on all phones)
- 99% uptime
- Passes all accessibility tests
```

---

## 🛡️ AI PROTECTION RULES FOR NON-TECHNICAL USERS

### Rule 1: Never Assume - Always Ask

**❌ DON'T:**
```
User: "Build me a website"
AI: *starts coding*
```

**✅ DO:**
```
User: "Build me a website"
AI: "I'd love to help! Before I start, let me ask a few questions 
to make sure I build exactly what you need. 

First question: What is this website for? Describe it in one sentence."
```

---

### Rule 2: Explain in Plain English

**❌ DON'T:**
```
"We need to implement a stateful component with useContext for 
global state management and implement Firebase Auth HOC..."
```

**✅ DO:**
```
"I'm going to create a login system. When users log in, the 
website will remember who they are and show them their personal 
information. This is stored securely in Google's Firebase database."
```

---

### Rule 3: Show Don't Just Tell

**When explaining concepts, use analogies:**

```
Firebase Security Rules = Bouncers at a nightclub
- They check if you're allowed in
- They check if you're old enough
- They keep bad people out
- They protect everyone inside

Color Palette = A wardrobe
- Primary color = Your favorite outfit (use most)
- Secondary color = Your second favorite (use sometimes)
- Accent color = Jewelry/accessories (use sparingly)
- Neutral color = Basic t-shirt (background, text)
```

---

### Rule 4: Provide Examples for Everything

**Never give instructions without an example:**

```
❌ "Provide your brand colors"
✅ "Provide your brand colors like this:
   - Primary: #3B82F6 (Blue)
   - Secondary: #10B981 (Green)
   - Accent: #F59E0B (Orange)"
```

---

### Rule 5: Create Checklists, Not Paragraphs

**People complete checklists. They skip paragraphs.**

```
Before we start coding, verify:
☐ I have answered all 11 questions above
☐ I have my color palette (or will generate one)
☐ I know what pages I need
☐ I know what user data I'm collecting
☐ I understand the privacy requirements
☐ I have realistic timeline expectations

IF ANY BOX IS UNCHECKED → Go back and complete it
```

---

### Rule 6: Warn About Common Mistakes

**Proactively prevent problems:**

```
⚠️ COMMON MISTAKE: "Just use placeholder images"
❌ Problem: Placeholder images often have copyright restrictions
✅ Solution: Use free stock photos from:
   - Unsplash.com
   - Pexels.com
   - Pixabay.com

⚠️ COMMON MISTAKE: "We'll add privacy policy later"
❌ Problem: Collecting user data without privacy policy is ILLEGAL
✅ Solution: Privacy policy MUST be ready before collecting any data

⚠️ COMMON MISTAKE: "Make it look modern"
❌ Problem: "Modern" means different things to everyone
✅ Solution: Show 3 example websites that have the look you want
```

---

### Rule 7: Set Realistic Expectations

**Be honest about complexity and time:**

```
"Based on your requirements, this project includes:
- 7 pages
- User authentication
- Payment processing
- Booking system
- Admin dashboard

REALISTIC TIMELINE:
- Design & Planning: 1 week
- Development: 3-4 weeks
- Testing: 1 week
- Total: 5-6 weeks

CANNOT be done in:
❌ "A few days"
❌ "Over the weekend"
❌ "By tomorrow"

Quality takes time. Rushing leads to security issues and bugs."
```

---

## 🎓 Educational Mode (Explain As You Build)

When building for non-technical users, explain each major step:

```
AI: "I'm now creating the login page. Here's what I'm doing:

1. FORM CREATION
   - Email input (where users type their email)
   - Password input (hides password as they type)
   - Login button

2. SECURITY
   - Checking password is strong enough
   - Encrypting password (scrambling it so hackers can't read it)
   - Adding "forgot password" link

3. ERROR HANDLING
   - If email is wrong, show friendly message
   - If password is wrong, don't say which is wrong (security!)
   - If internet is down, explain what happened

Would you like me to proceed?"
```

---

## 🚨 MANDATORY CHECKPOINTS

The AI **MUST STOP** at these points and require user confirmation:

### Checkpoint 1: After Requirements Gathering
```
"I've documented your requirements. Please review this document:
[Show requirements document]

Does this accurately reflect what you want? (YES/NO)
Any changes needed?"
```

### Checkpoint 2: Before Design System Generation
```
"I'm about to generate your color palette based on these keywords:
[Show keywords]

Are these correct? Should I proceed? (YES/NO)"
```

### Checkpoint 3: Before Collecting Personal Data
```
"Your booking form will collect:
- Name
- Email
- Phone number
- Credit card information

This means you MUST have:
✅ Privacy Policy
✅ Terms of Service
✅ Secure data storage
✅ Data export feature
✅ Data deletion feature

Do you understand these legal requirements? (YES/NO)"
```

### Checkpoint 4: Before Going Live
```
"FINAL CHECKLIST before launch:

☐ All pages tested on mobile
☐ All pages tested on desktop
☐ Privacy policy published
☐ Terms of service published
☐ Cookie consent banner working
☐ Contact information correct
☐ Payment system tested
☐ Backup system in place
☐ Domain name connected
☐ SSL certificate active (https://)

Everything checked? Ready to launch? (YES/NO)"
```

---

## 📚 Quick Reference Card

Print this or keep it handy:

```
╔══════════════════════════════════════════════════════════╗
║          BEFORE ASKING AI TO BUILD ANYTHING              ║
╠══════════════════════════════════════════════════════════╣
║ ✅ I know exactly what the website does                  ║
║ ✅ I know who will use it                                ║
║ ✅ I have a list of all pages needed                     ║
║ ✅ I have my color palette (or know my brand words)      ║
║ ✅ I know what user data I'm collecting                  ║
║ ✅ I understand privacy law requirements                 ║
║ ✅ I have realistic timeline expectations                ║
║                                                          ║
║ IF ANY ❌ → Don't start coding yet!                      ║
╚══════════════════════════════════════════════════════════╝
```

---

## 🤝 Communication Templates

### For Clients (Agency/Freelancer Use)

**Email Template: Gathering Requirements**

```
Subject: Quick Questions Before We Build Your Website

Hi [Client Name],

Before I start building your website, I need some information to 
make sure I create exactly what you need.

Please answer these questions:

1. In one sentence, what does this website do?

2. Describe your typical user (age, tech skill level, device they'll use)

3. What are the 3-5 main things users need to do?

4. Do users need to log in and save information?

5. Choose 3-5 words that describe your brand personality

6. Do you have existing brand colors or should I generate a palette?

7. List all pages you need

8. Where are your users located? (For privacy law compliance)

9. What information will you collect from users?

Once I have these answers, I can provide:
- Accurate timeline
- Design mockups
- Cost estimate
- Technical architecture

Looking forward to your responses!

[Your Name]
```

---

## 🎯 Success Metrics

This rule is successful when:

✅ Zero code written before requirements documented
✅ Non-technical users can complete requirements doc alone
✅ Every data point has a clear "why we collect this"
✅ Client understands exactly what they're getting
✅ Timeline is realistic and agreed upon
✅ Privacy requirements are crystal clear
✅ No assumptions were made by the AI

---

**Remember: 30 minutes of questions saves 30 hours of rework.**

