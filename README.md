# Software Requirements Specification
## Techible Platform — Interview Prep & Learning Hub
### Version 1.0 | For Intern Development Team

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [System Overview & Feature Map](#2-system-overview--feature-map)
3. [Feature Specifications](#3-feature-specifications)
   - 3.1 Company Interview Prep Module
   - 3.2 Coding Challenges (DSA) Module
   - 3.3 AI Learning Module
   - 3.4 Interview Prep (Level-Based) Module
   - 3.5 Projects Module
4. [Database Schema](#4-database-schema)
5. [API Specification](#5-api-specification)
6. [Frontend Pages & Components](#6-frontend-pages--components)
7. [Codebase Integration Guide](#7-codebase-integration-guide)
8. [Development Guidelines for Interns](#8-development-guidelines-for-interns)
9. [Design System Reference](#9-design-system-reference)
10. [AI Prompts Toolkit](#10-ai-prompts-toolkit)

---

## 1. Introduction

### 1.1 Purpose

This document specifies the complete requirements for the **Interview Prep & Learning Hub** — a major new section of the Techible platform. It is written for **interns who will build this independently** and hand off to the platform owner (Nitin) for integration into the main codebase.

You will **not** have access to the live codebase. You will build a **standalone React + Node app** that mirrors the exact patterns of the main platform so that integration is seamless.

### 1.2 Scope

The feature covers five interconnected modules:

| Module | Description |
|---|---|
| Company Interview Prep | Browse companies, view interview process, questions, stories, compensation |
| Coding Challenges | DSA problems organized by category with full solution walkthroughs |
| AI Learning Hub | Tiered curriculum (Fundamentals → Advanced) with interactive content |
| Interview Prep (Leveled) | Level-based interview Q&A with patterns and approaches |
| Projects | Creative project challenges (scope left to team) |

### 1.3 How Integration Will Work

Nitin will take your finished code and:
1. Copy your **backend route files** into `/backend/routes/` and mount them in `index.js`
2. Copy your **Mongoose models** into `/backend/models/`
3. Copy your **frontend pages** into `/frontend/src/pages/`
4. Copy your **frontend components** into `/frontend/src/components/`
5. Add your **routes** to `/frontend/src/routes/publicRoutes.jsx`
6. Add your **Redux slices** to `/frontend/src/store/`

**This means: every file you create must follow the exact same conventions as the main codebase.** Deviation = integration pain.

### 1.4 Definitions

| Term | Meaning |
|---|---|
| SOURCE-OF-TRUTH | Canonical file — defines the authoritative shape of data or logic |
| `refPath` | Mongoose polymorphic reference (one field referencing different models) |
| shadcn/ui | The UI component library used (built on Radix UI + Tailwind) |
| React Query | TanStack Query v5 — used for all server state / API calls |
| Redux Toolkit | Used for UI filter/search/pagination state only (not server data) |
| BullMQ | Redis-backed job queue for async tasks |

---

## 2. System Overview & Feature Map

```
/interview-prep                    ← Company listing page
  /interview-prep/:companySlug     ← Company detail page (tabs)
    → Overview Tab
    → Common Questions Tab
    → Recent Questions Tab (last 6 months)
    → Success Stories Tab
    → Tips Tab
    → Compensation Tab
    → Resources Tab

/learn                             ← Learning hub landing
  /learn/dsa                       ← DSA coding challenges
    /learn/dsa/:category           ← Category problems list
      /learn/dsa/:category/:slug   ← Single problem detail
  /learn/ai                        ← AI learning curriculum
    /learn/ai/:topicSlug           ← Single topic module
  /learn/interview-prep            ← Level-based interview prep
    /learn/interview-prep/:level   ← Level detail (topics + questions)
  /learn/projects                  ← Projects section
```

### User Roles That Access This Feature

- **All users** (public, no auth required) — can browse company pages, see questions, read challenges
- **Logged-in users** — can track progress, mark topics complete, unlock premium content (future)
- **Admin** — can create/edit companies, add questions, approve success stories

---

## 3. Feature Specifications

---

### 3.1 Company Interview Prep Module

#### 3.1.1 Company Listing Page (`/interview-prep`)

**What it shows:**
- Search bar (search by company name)
- Filter chips: Industry (Tech, Finance, Consulting, E-Commerce, etc.)
- Filter: Difficulty (Easy / Medium / Hard interview process)
- Sort: Most Popular, Alphabetical, Recently Added
- Grid of Company Cards (4 per row on desktop, 2 on tablet, 1 on mobile)
- Pagination (8 companies per page)

**Company Card contains:**
- Company logo
- Company name
- Industry badge
- Difficulty badge (color-coded: green=easy, yellow=medium, red=hard)
- Number of interview questions available
- "View Interview Guide" button → navigates to `/interview-prep/:slug`

#### 3.1.2 Company Detail Page (`/interview-prep/:companySlug`)

This is a **tabbed page**. The URL slug is the company's unique identifier (e.g., `google`, `amazon`, `adobe`).

**Page Header (always visible):**
- Company logo + name
- Industry + headquarters
- Overall difficulty rating (stars or label)
- Quick stats: X questions, Y success stories, Z people hired
- Link to company website

**Tab 1: Interview Overview**

Sections:
- **About the Interview Process** — paragraph description of what to expect overall
- **Interview Phases** — ordered list of phases, each showing:
  - Phase number and name (e.g., "Phase 1: Online Assessment")
  - Duration (e.g., "60–90 minutes")
  - Format (Online / In-Person / Video Call)
  - What this phase tests
  - What to expect (bullet points)
  - Common tools/platforms used (e.g., HackerRank, Zoom)
- **Overall Tips** — 3–5 general tips for this company's interview

**Tab 2: Common Interview Questions**

- Filter: Category (DSA / System Design / Behavioral / HR / Domain-Specific)
- Filter: Difficulty (Easy / Medium / Hard)
- List of question cards, each showing:
  - Question text
  - Category badge
  - Difficulty badge
  - **Pattern** — what kind of question this is (e.g., "Two Pointer", "STAR Method", "LLD")
  - **Things to Keep in Mind** — 3–5 bullet points (NOT the answer, just guidance)
  - Upvote count
  - Which round it was asked in

> **IMPORTANT**: We show the question and guidance on HOW to think about it. We do NOT show the actual answer/solution.

**Tab 3: Recent Questions (Last 6 Months)**

Same layout as Tab 2 but filtered to questions with `dateAsked` within 6 months. Badge shows "Asked X months ago".

**Tab 4: Success Stories**

- Cards showing:
  - Author name (or "Anonymous") + profile photo (or avatar)
  - Role they got hired for
  - Package (CTC) — shown as range if anonymous
  - Year of joining
  - **Story** — their full experience (collapsible, "Read more")
  - **Preparation approach** — what they did to prepare
  - **Key advice** — their tip for future candidates
- Submit Your Story button (for logged-in users)

**Tab 5: Tips to Crack the Interview**

- Phase-by-phase tips (accordion layout)
- General preparation tips
- Resources that helped most candidates (links to platform resources)
- Do's and Don'ts section (two-column)

**Tab 6: Compensation**

- Table showing:
  - Role / Level / Min Salary / Max Salary / Location / Source
- Note: "Data crowdsourced and may not reflect current offers"
- Average CTC highlight cards for: Fresher / 2–4 years / Senior

**Tab 7: Resources**

- "Based on what's asked at [Company], we recommend:"
- Cards linking to:
  - DSA categories most frequently tested
  - AI Learning topics relevant to this company
  - External links (GeeksforGeeks, LeetCode company-specific)

---

### 3.2 Coding Challenges (DSA) Module

#### 3.2.1 DSA Landing (`/learn/dsa`)

- Page title (intern team decides final copy)
- Subtitle: short description of the section's purpose (intern team writes this)
- Search bar for searching by category or keyword
- **Category Cards Grid** (3 per row on desktop):
  - Category name — intern team decides which categories to cover (minimum 8). Categories should cover the most commonly tested DSA topics in technical interviews.
  - Problem count badge (auto-computed from DB)
  - Short description of what the category teaches/tests
  - "View problems" button

> **Content decision**: The specific categories and the problems within each are the intern team's responsibility. Research what topics are most asked in SDE interviews at product companies. Categories are stored in the DB (`dsaCategorySchema`), not hardcoded.

#### 3.2.2 Category Problems List (`/learn/dsa/:category`)

URL pattern: `/learn/dsa/:categorySlug` where slug is the category's slug field

- Breadcrumb: `Coding Challenges > [Category Name]`
- Title: "[Category Name] Challenges — Explore all X problems"
- Back button to `/learn/dsa`
- List of problem cards (vertical list, not grid), each showing:
  - Difficulty badge (Easy=green, Medium=yellow, Hard=red)
  - Problem title
  - One-line description
  - "View problem" button → `/learn/dsa/:category/:slug`

#### 3.2.3 Single Problem (`/learn/dsa/:category/:slug`)

**Breadcrumb**: `Coding Challenges > [Category] > [Problem Title]`

**Page sections (vertical scroll):**

1. **Problem Header**
   - Title, Difficulty badge, Category badge

2. **Problem Description**
   - Full problem statement

3. **Examples** (accordion or always visible)
   - Input / Output / Explanation for each example

4. **Solution Approach** (the main teaching section)
   - Approach Overview (paragraph)
   - Step-by-Step Explanation:
     - Each step has: title + description + code snippet (language tabs: JavaScript / Python / Java)

5. **Complexity Analysis**
   - Time Complexity: label + explanation
   - Space Complexity: label + explanation

6. **Key Insights**
   - Bullet list of insights, tradeoffs, applicability

7. **Solution Code**
   - Language tabs (JavaScript / Python / Java)
   - Full working solution

8. **External Resources**
   - Cards: LeetCode, GeeksforGeeks, HackerRank, CodeSignal — with "Practice interactively" links

9. **Additional Learning**
   - Link to related Learning Resources
   - Link to Community Forum

---

### 3.3 AI Learning Module

#### 3.3.1 AI Hub Landing (`/learn/ai`)

The AI Learning Hub is a structured, tiered curriculum. Content is divided into three tiers of increasing depth. Tier 1 is free; Tiers 2 and 3 are premium (locked behind a subscribe gate for now — UI shows lock, no payment flow needed in MVP).

**Page elements:**
- Page title and a subtitle that communicates the structured, progressive nature of the curriculum (intern team writes the copy)
- Search bar (searches across topic titles and subtitles)
- Filter tabs: All / Tier 1 / Tier 2 / Tier 3
- Progress indicator per tier showing how many topics the user has completed (e.g., "2 / 5 completed") — requires user auth; show 0 for guests
- Three tier sections, each with a header, description, and a grid/list of topic cards

**Tier structure:**
- **Tier 1** — foundational topics (free). Minimum 5 topics. Covers the basics a beginner needs before touching frameworks or advanced patterns.
- **Tier 2** — intermediate topics (premium). Minimum 6 topics. Covers practical implementation — integrating with APIs, building real features, retrieval, tooling.
- **Tier 3** — advanced topics (premium). Minimum 5 topics. Covers production patterns, multi-component systems, evaluation, and observability.

> **Content decision**: The intern team decides the actual topic titles and content. Research what people genuinely need to learn to go from zero to building real AI applications. Topics are stored in the DB (`aiTopicSchema`), not hardcoded. The tier/order fields control display sequence.

**Topic Card shows:**
- Tier badge (color-coded per tier)
- "New" tag if `isNew: true`
- Topic title + subtitle
- Progress state: not started / in progress / complete (based on user data, or "not started" for guests)
- Lock icon + "Premium — subscribe to unlock" for Tier 2/3 topics
- "Open topic" button (disabled/locked for premium if not subscribed)

#### 3.3.2 Single AI Topic (`/learn/ai/:topicSlug`)

Each topic is a self-contained learning module with structured content.

**Page header (sticky on scroll):**
- Back button → AI hub
- Tier badge + "New" badge (if applicable)
- Topic title + subtitle
- "Mark module complete" button (logged-in users only — fires a POST to mark completion)

**Content layout (vertical scroll, fixed reading width ~750px):**

1. **Overview** — 2–4 paragraph intro. Sets up the mental model before diving in. Rendered as markdown.

2. **Numbered Sections** (as many as the topic needs, typically 3–6):
   - Section number + title
   - Explanation text (markdown — can include callout boxes, lists, comparisons)
   - **Interactive Element** (optional, per section) — can be one of:
     - Step-through diagram (user clicks Next/Previous to walk through a process)
     - Parameter slider (adjust values and see output change — useful for concept demos)
     - Static diagram/image with annotations
     - None (most sections will be text + code only)
   - **Code snippet(s)** with language tabs. Support at minimum: Python + one framework (e.g., LangChain). Each tab has: label, language, code block, one-line description below.

3. **Tips & Best Practices** — numbered list of 3–6 practical tips

4. **Learning Resources** — cards linking to external articles, docs, or videos. Each card: title, type badge (Article / Documentation / Video / Tool), description, external link.

5. **Continue Learning** — link back to all topics + adjacent topic navigation (Previous / Next)

> **MVP note**: Topic content (sections, code snippets, tips, resources) can be seeded as static data in MongoDB. No CMS needed for MVP. Interactive elements are React components driven by config data from the `interactiveConfig` field in the schema.

---

### 3.4 Interview Prep (Level-Based) Module

#### 3.4.1 Interview Prep Landing (`/learn/interview-prep`)

Entry point where the user selects their experience level before diving into preparation material.

**Page elements:**
- Title + brief description of the module
- Three level selection cards: **Beginner**, **Intermediate**, **Advanced**
- Each level card shows:
  - Level name + short description (e.g., "For those starting their first job search")
  - Number of topic categories available
  - Estimated preparation time
  - "Start Prep" button → navigates to that level's page

#### 3.4.2 Level Detail (`/learn/interview-prep/:level`)

URL pattern: `/learn/interview-prep/beginner`, `/learn/interview-prep/intermediate`, `/learn/interview-prep/advanced`

Shows all **topic categories** within the chosen level. Each level should have at minimum 4–6 topic categories spanning different question types (DSA, System Design, Behavioral, HR, etc. — intern team decides appropriate distribution per level).

**Page elements:**
- Breadcrumb + back button
- Level title + description
- Grid of topic category cards:
  - Category title
  - Category type badge (DSA / System Design / Behavioral / HR)
  - Question count
  - "View Questions" button → navigates to topic page

#### 3.4.3 Topic Questions (`/learn/interview-prep/:level/:topic`)

The question list for a specific topic category within a level.

**Page elements:**
- Breadcrumb: `Interview Prep > [Level] > [Topic]`
- Topic title + brief description
- List of question cards (minimum 8 questions per topic), each showing:
  - Question text (the actual interview question)
  - Difficulty badge (Easy / Medium / Hard)
  - **Pattern** — classifies the question type (e.g., "Two Pointer approach", "STAR method", "Capacity estimation")
  - **Approach** — 2–3 sentences on how to structure your thinking. NOT a full answer.
  - **Things to Keep in Mind** — 3–5 bullet points of guidance
  - Expandable / collapsible (collapsed by default to avoid overwhelming the page)

> **Content decision**: The intern team writes the question bank. Aim for questions that are actually asked in SDE interviews at product-based companies at the corresponding level. Questions live in the DB (`interviewPrepTopicSchema`), not hardcoded.

---

### 3.5 Projects Module

> **Scope left to intern team creativity.** Suggested directions:
> - Curated project ideas with guided briefs
> - Project challenge cards with requirements and resources
> - Difficulty levels (Beginner / Intermediate / Advanced)
> - Tech stack tags (React, Node, Python, etc.)
> - "Start Project" flow that generates a project brief

Minimum requirement: A page at `/learn/projects` with at least 10 project cards.

---

## 4. Database Schema

All schemas use **Mongoose** and follow these conventions:
- Field names: `camelCase`
- Schema file names: `entityNameSchema.js`
- All schemas include `createdAt` and `updatedAt` timestamps
- Approval workflow (where needed): `approved: Boolean` + `approvalStatus: enum['pending','approved','rejected']`

---

### 4.1 Company Schema

**File**: `companySchema.js`

```javascript
const mongoose = require('mongoose');

const companySchema = new mongoose.Schema({
  name: { type: String, required: true, trim: true },
  slug: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    trim: true
    // e.g., "google", "amazon", "adobe"
  },
  logo: { url: String, filename: String },
  description: { type: String, trim: true },
  industry: {
    type: String,
    enum: ['Tech', 'Finance', 'Consulting', 'E-Commerce', 'Healthcare', 'EdTech', 'Other'],
    default: 'Tech'
  },
  headquarters: { type: String, trim: true },
  founded: { type: Number },
  website: { type: String, trim: true },
  glassdoorUrl: { type: String, trim: true },
  linkedinUrl: { type: String, trim: true },
  interviewDifficulty: {
    type: String,
    enum: ['Easy', 'Medium', 'Hard'],
    default: 'Medium'
  },
  approved: { type: Boolean, default: false },
  approvalStatus: {
    type: String,
    enum: ['pending', 'approved', 'rejected'],
    default: 'pending'
  },
  featured: { type: Boolean, default: false },
  views: { type: Number, default: 0, min: 0 },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Company', companySchema);
```

---

### 4.2 Interview Process Schema

**File**: `interviewProcessSchema.js`

```javascript
const interviewProcessSchema = new mongoose.Schema({
  company: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Company',
    required: true,
    unique: true  // One process per company
  },
  overview: { type: String, trim: true },
  phases: [{
    order: { type: Number, required: true },
    name: { type: String, required: true, trim: true },
    // e.g., "Online Assessment", "Technical Round 1", "HR Round"
    duration: { type: String, trim: true },
    // e.g., "60-90 minutes"
    format: {
      type: String,
      enum: ['Online', 'In-Person', 'Video Call', 'Phone'],
      default: 'Online'
    },
    whatItTests: { type: String, trim: true },
    whatToExpect: [{ type: String, trim: true }],
    platforms: [{ type: String, trim: true }]
    // e.g., ["HackerRank", "Zoom"]
  }],
  generalTips: [{ type: String, trim: true }],
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('InterviewProcess', interviewProcessSchema);
```

---

### 4.3 Interview Question Schema

**File**: `interviewQuestionSchema.js`

```javascript
const interviewQuestionSchema = new mongoose.Schema({
  company: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Company',
    required: true
  },
  question: { type: String, required: true, trim: true },
  category: {
    type: String,
    enum: ['DSA', 'System Design', 'Behavioral', 'HR', 'Domain-Specific', 'LLD', 'HLD'],
    required: true
  },
  difficulty: {
    type: String,
    enum: ['Easy', 'Medium', 'Hard'],
    default: 'Medium'
  },
  round: { type: String, trim: true },
  // e.g., "Technical Round 1", "HR Round"
  pattern: { type: String, trim: true },
  // e.g., "Two Pointer", "STAR Method", "System Design Framework"
  thingsToKeepInMind: [{ type: String, trim: true }],
  // Guidance bullets — NOT the answer
  isRecent: { type: Boolean, default: false },
  dateAsked: { type: Date },
  // Used to determine "last 6 months" filter
  upvotes: { type: Number, default: 0, min: 0 },
  approved: { type: Boolean, default: false },
  approvalStatus: {
    type: String,
    enum: ['pending', 'approved', 'rejected'],
    default: 'pending'
  },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

// Index for fast filtering
interviewQuestionSchema.index({ company: 1, category: 1, difficulty: 1 });
interviewQuestionSchema.index({ company: 1, dateAsked: -1 });

module.exports = mongoose.model('InterviewQuestion', interviewQuestionSchema);
```

---

### 4.4 Success Story Schema

**File**: `successStorySchema.js`

```javascript
const successStorySchema = new mongoose.Schema({
  company: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Company',
    required: true
  },
  authorName: { type: String, trim: true },
  isAnonymous: { type: Boolean, default: false },
  role: { type: String, required: true, trim: true },
  // e.g., "Software Engineer - L3"
  package: {
    min: { type: Number },
    max: { type: Number },
    currency: { type: String, default: 'INR' }
  },
  yearOfJoining: { type: Number },
  story: { type: String, required: true, trim: true },
  preparationApproach: { type: String, trim: true },
  keyAdvice: { type: String, trim: true },
  approved: { type: Boolean, default: false },
  approvalStatus: {
    type: String,
    enum: ['pending', 'approved', 'rejected'],
    default: 'pending'
  },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('SuccessStory', successStorySchema);
```

---

### 4.5 Compensation Data Schema

**File**: `compensationSchema.js`

```javascript
const compensationSchema = new mongoose.Schema({
  company: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Company',
    required: true
  },
  role: { type: String, required: true, trim: true },
  level: {
    type: String,
    enum: ['Fresher', 'Junior', 'Mid-Level', 'Senior', 'Staff', 'Principal'],
    default: 'Fresher'
  },
  minSalary: { type: Number, required: true },
  maxSalary: { type: Number, required: true },
  currency: { type: String, default: 'INR' },
  location: { type: String, trim: true },
  source: { type: String, trim: true },
  // e.g., "Glassdoor", "Anonymous submission"
  yearReported: { type: Number },
  createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Compensation', compensationSchema);
```

---

### 4.6 DSA Category Schema

**File**: `dsaCategorySchema.js`

```javascript
const dsaCategorySchema = new mongoose.Schema({
  name: { type: String, required: true, trim: true },
  // e.g., "Arrays", "Dynamic Programming"
  slug: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    trim: true
    // e.g., "arrays", "dynamic-programming"
  },
  description: { type: String, trim: true },
  icon: { type: String },
  // Icon name from Lucide icons
  order: { type: Number, default: 0 },
  problemCount: { type: Number, default: 0 },
  createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('DsaCategory', dsaCategorySchema);
```

---

### 4.7 Coding Problem Schema

**File**: `codingProblemSchema.js`

```javascript
const codingProblemSchema = new mongoose.Schema({
  category: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'DsaCategory',
    required: true
  },
  title: { type: String, required: true, trim: true },
  slug: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    trim: true
    // e.g., "two-sum", "contains-duplicate"
  },
  difficulty: {
    type: String,
    enum: ['Easy', 'Medium', 'Hard'],
    required: true
  },
  description: { type: String, required: true },
  examples: [{
    input: { type: String, required: true },
    output: { type: String, required: true },
    explanation: { type: String }
  }],
  solutionApproach: {
    overview: { type: String },
    steps: [{
      title: { type: String },
      description: { type: String },
      codeSnippets: [{
        language: {
          type: String,
          enum: ['javascript', 'python', 'java', 'cpp'],
          default: 'javascript'
        },
        code: { type: String }
      }]
    }]
  },
  complexityAnalysis: {
    time: { type: String },
    // e.g., "O(n)"
    timeExplanation: { type: String },
    space: { type: String },
    spaceExplanation: { type: String }
  },
  keyInsights: [{ type: String, trim: true }],
  solutionCode: [{
    language: {
      type: String,
      enum: ['javascript', 'python', 'java', 'cpp'],
      default: 'javascript'
    },
    code: { type: String, required: true }
  }],
  externalResources: [{
    platform: { type: String, trim: true },
    // e.g., "LeetCode", "GeeksforGeeks"
    url: { type: String, trim: true },
    label: { type: String, trim: true }
    // e.g., "Practice interactively"
  }],
  relatedCompanies: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Company'
  }],
  order: { type: Number, default: 0 },
  approved: { type: Boolean, default: true },
  views: { type: Number, default: 0 },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('CodingProblem', codingProblemSchema);
```

---

### 4.8 AI Learning Topic Schema

**File**: `aiTopicSchema.js`

```javascript
const aiTopicSchema = new mongoose.Schema({
  tier: {
    type: String,
    enum: ['Fundamentals', 'Intermediate', 'Advanced'],
    required: true
  },
  tierLevel: { type: Number, enum: [1, 2, 3], required: true },
  order: { type: Number, default: 0 },
  title: { type: String, required: true, trim: true },
  slug: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    trim: true
  },
  subtitle: { type: String, trim: true },
  isPremium: { type: Boolean, default: false },
  isNew: { type: Boolean, default: false },
  content: {
    overview: { type: String },
    sections: [{
      sectionNumber: { type: Number },
      title: { type: String, required: true },
      explanation: { type: String },
      interactiveType: {
        type: String,
        enum: ['none', 'tokenizer', 'stepper', 'slider', 'diagram'],
        default: 'none'
      },
      interactiveConfig: { type: mongoose.Schema.Types.Mixed },
      // JSON config for the interactive component
      codeSnippets: [{
        tabLabel: { type: String },
        // e.g., "Generic Python", "LangChain"
        language: { type: String },
        code: { type: String },
        description: { type: String }
      }]
    }],
    tipsAndBestPractices: [{ type: String, trim: true }]
  },
  learningResources: [{
    title: { type: String, trim: true },
    type: {
      type: String,
      enum: ['Article', 'Documentation', 'Video', 'Tool'],
      default: 'Article'
    },
    url: { type: String, trim: true },
    description: { type: String, trim: true }
  }],
  status: {
    type: String,
    enum: ['draft', 'published'],
    default: 'draft'
  },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('AiTopic', aiTopicSchema);
```

---

### 4.9 Interview Prep Topic Schema

**File**: `interviewPrepTopicSchema.js`

```javascript
const interviewPrepTopicSchema = new mongoose.Schema({
  level: {
    type: String,
    enum: ['Beginner', 'Intermediate', 'Advanced'],
    required: true
  },
  category: {
    type: String,
    enum: ['DSA', 'System Design', 'Behavioral', 'HR', 'Domain-Specific'],
    required: true
  },
  title: { type: String, required: true, trim: true },
  slug: {
    type: String,
    required: true,
    unique: true,
    lowercase: true
  },
  order: { type: Number, default: 0 },
  questions: [{
    question: { type: String, required: true, trim: true },
    pattern: { type: String, trim: true },
    approach: { type: String, trim: true },
    thingsToKeepInMind: [{ type: String, trim: true }],
    difficulty: {
      type: String,
      enum: ['Easy', 'Medium', 'Hard'],
      default: 'Medium'
    }
  }],
  approved: { type: Boolean, default: true },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('InterviewPrepTopic', interviewPrepTopicSchema);
```

---

## 5. API Specification

All routes mount under `/api/`. All responses follow the pattern:

```json
{
  "success": true,
  "data": { ... },
  "pagination": { "total": 100, "currentPage": 1, "pageSize": 8, "totalPages": 13 }
}
```

Error responses:
```json
{
  "success": false,
  "message": "Human-readable error",
  "error": "ERROR_CODE",
  "statusCode": 400
}
```

---

### 5.1 Company Routes (`/api/companies`)

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/api/companies` | Public | List all approved companies (paginated, filterable) |
| GET | `/api/companies/:slug` | Public | Get company by slug |
| GET | `/api/companies/:slug/interview-process` | Public | Get interview process for company |
| GET | `/api/companies/:slug/questions` | Public | Get interview questions (filterable) |
| GET | `/api/companies/:slug/questions/recent` | Public | Get questions from last 6 months |
| GET | `/api/companies/:slug/success-stories` | Public | Get approved success stories |
| GET | `/api/companies/:slug/compensation` | Public | Get compensation data |
| POST | `/api/companies` | Admin | Create company |
| POST | `/api/companies/:slug/questions` | Admin | Add interview question |
| POST | `/api/companies/:slug/success-stories` | Auth | Submit success story (pending approval) |
| PATCH | `/api/companies/:slug` | Admin | Update company |

**Query params for GET `/api/companies`:**
```
?page=1&limit=8&search=google&industry=Tech&difficulty=Medium&sort=views&order=desc
```

**Query params for GET `/api/companies/:slug/questions`:**
```
?category=DSA&difficulty=Medium&page=1&limit=10
```

---

### 5.2 DSA Routes (`/api/dsa`)

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/api/dsa/categories` | Public | List all DSA categories |
| GET | `/api/dsa/categories/:slug` | Public | Get category with problem list |
| GET | `/api/dsa/problems/:slug` | Public | Get single problem by slug |
| POST | `/api/dsa/categories` | Admin | Create category |
| POST | `/api/dsa/problems` | Admin | Create problem |
| PATCH | `/api/dsa/problems/:slug` | Admin | Update problem |

---

### 5.3 AI Learning Routes (`/api/ai-topics`)

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/api/ai-topics` | Public | List all published topics (grouped by tier) |
| GET | `/api/ai-topics/:slug` | Public | Get single topic with full content |
| POST | `/api/ai-topics` | Admin | Create topic |
| PATCH | `/api/ai-topics/:slug` | Admin | Update topic |
| POST | `/api/ai-topics/:slug/complete` | Auth | Mark topic as complete (user progress) |

---

### 5.4 Interview Prep Routes (`/api/interview-prep`)

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/api/interview-prep/levels` | Public | List available levels |
| GET | `/api/interview-prep/:level` | Public | Get all topics for a level |
| GET | `/api/interview-prep/:level/:slug` | Public | Get topic with questions |
| POST | `/api/interview-prep/topics` | Admin | Create topic |

---

## 6. Frontend Pages & Components

### 6.1 Folder Structure You Must Follow

```
src/
├── pages/
│   ├── InterviewPrepPages/
│   │   ├── CompanyListing.jsx         ← /interview-prep
│   │   └── CompanyDetail.jsx          ← /interview-prep/:slug
│   └── LearnPages/
│       ├── LearnHub.jsx               ← /learn (landing)
│       ├── DsaLanding.jsx             ← /learn/dsa
│       ├── DsaCategoryPage.jsx        ← /learn/dsa/:category
│       ├── DsaProblemPage.jsx         ← /learn/dsa/:category/:slug
│       ├── AiLearningHub.jsx          ← /learn/ai
│       ├── AiTopicPage.jsx            ← /learn/ai/:topicSlug
│       ├── InterviewPrepHub.jsx       ← /learn/interview-prep
│       ├── InterviewPrepLevel.jsx     ← /learn/interview-prep/:level
│       ├── InterviewPrepTopic.jsx     ← /learn/interview-prep/:level/:topic
│       └── ProjectsHub.jsx            ← /learn/projects
├── components/
│   ├── InterviewPrepComponents/
│   │   ├── CompanyCard.jsx
│   │   ├── QuestionCard.jsx
│   │   ├── SuccessStoryCard.jsx
│   │   ├── CompensationTable.jsx
│   │   ├── InterviewPhaseAccordion.jsx
│   │   └── ResourceLinkCard.jsx
│   └── LearnComponents/
│       ├── DsaCategoryCard.jsx
│       ├── DsaProblemCard.jsx
│       ├── ProblemCodeBlock.jsx
│       ├── AiTopicCard.jsx
│       ├── TierSection.jsx
│       ├── InteractiveStepper.jsx
│       └── LanguageTabsCode.jsx
├── services/
│   ├── companyApi.js                  ← API functions + React Query hooks
│   ├── dsaApi.js
│   └── aiTopicsApi.js
└── store/
    └── slices/
        ├── companyFilterSlice.js      ← Redux slice for company filters
        └── dsaFilterSlice.js
```

### 6.2 Routes to Register

These are the exact entries to add to `publicRoutes.jsx`:

```jsx
// Interview Prep
const CompanyListing = React.lazy(() => import('../pages/InterviewPrepPages/CompanyListing'));
const CompanyDetail = React.lazy(() => import('../pages/InterviewPrepPages/CompanyDetail'));

// Learn Hub
const DsaLanding = React.lazy(() => import('../pages/LearnPages/DsaLanding'));
const DsaCategoryPage = React.lazy(() => import('../pages/LearnPages/DsaCategoryPage'));
const DsaProblemPage = React.lazy(() => import('../pages/LearnPages/DsaProblemPage'));
const AiLearningHub = React.lazy(() => import('../pages/LearnPages/AiLearningHub'));
const AiTopicPage = React.lazy(() => import('../pages/LearnPages/AiTopicPage'));
const InterviewPrepHub = React.lazy(() => import('../pages/LearnPages/InterviewPrepHub'));
const InterviewPrepLevel = React.lazy(() => import('../pages/LearnPages/InterviewPrepLevel'));
const InterviewPrepTopic = React.lazy(() => import('../pages/LearnPages/InterviewPrepTopic'));
const ProjectsHub = React.lazy(() => import('../pages/LearnPages/ProjectsHub'));

// Route entries:
<Route key="interview-prep" path="/interview-prep" element={<CompanyListing />} />
<Route key="company-detail" path="/interview-prep/:slug" element={<CompanyDetail />} />
<Route key="dsa" path="/learn/dsa" element={<DsaLanding />} />
<Route key="dsa-category" path="/learn/dsa/:category" element={<DsaCategoryPage />} />
<Route key="dsa-problem" path="/learn/dsa/:category/:slug" element={<DsaProblemPage />} />
<Route key="ai-hub" path="/learn/ai" element={<AiLearningHub />} />
<Route key="ai-topic" path="/learn/ai/:topicSlug" element={<AiTopicPage />} />
<Route key="interview-prep-hub" path="/learn/interview-prep" element={<InterviewPrepHub />} />
<Route key="interview-prep-level" path="/learn/interview-prep/:level" element={<InterviewPrepLevel />} />
<Route key="interview-prep-topic" path="/learn/interview-prep/:level/:topic" element={<InterviewPrepTopic />} />
<Route key="projects-hub" path="/learn/projects" element={<ProjectsHub />} />
```

---

## 7. Codebase Integration Guide

This section tells Nitin exactly what to do when integrating your code.

### 7.1 Backend Integration Checklist

```
□ Copy model files to /backend/models/
  - companySchema.js
  - interviewProcessSchema.js
  - interviewQuestionSchema.js
  - successStorySchema.js
  - compensationSchema.js
  - dsaCategorySchema.js
  - codingProblemSchema.js
  - aiTopicSchema.js
  - interviewPrepTopicSchema.js

□ Copy route files to /backend/routes/
  - companyRoutes.js
  - dsaRoutes.js
  - aiTopicRoutes.js
  - interviewPrepRoutes.js

□ Add to /backend/routes/index.js:
  router.use('/companies', require('./companyRoutes'));
  router.use('/dsa', require('./dsaRoutes'));
  router.use('/ai-topics', require('./aiTopicRoutes'));
  router.use('/interview-prep', require('./interviewPrepRoutes'));

□ Add seed script if provided (to /backend/scripts/)
```

### 7.2 Frontend Integration Checklist

```
□ Copy page files to /frontend/src/pages/
  - InterviewPrepPages/ folder
  - LearnPages/ folder

□ Copy component files to /frontend/src/components/
  - InterviewPrepComponents/ folder
  - LearnComponents/ folder

□ Copy API service files to /frontend/src/services/
  - companyApi.js
  - dsaApi.js
  - aiTopicsApi.js

□ Copy Redux slices to /frontend/src/store/slices/
  - companyFilterSlice.js
  - dsaFilterSlice.js

□ Register slices in /frontend/src/store/store.js

□ Add routes to /frontend/src/routes/publicRoutes.jsx
```

---

## 8. Development Guidelines for Interns

### 8.1 Your Project Setup

Since you don't have access to the main repo, set up a standalone project:

```bash
# Frontend
npm create vite@latest techible-learn-frontend -- --template react
cd techible-learn-frontend
npm install react-router-dom@7 @tanstack/react-query @reduxjs/toolkit react-redux axios
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# Shadcn/ui setup
npx shadcn-ui@latest init
# Choose: New York style, Zinc base color, CSS variables: yes

# Backend
mkdir techible-learn-backend
cd techible-learn-backend
npm init -y
npm install express mongoose dotenv cors express-session connect-mongo multer
npm install -D nodemon
```

### 8.2 Mandatory Code Patterns

#### Pattern 1: Every API Service File

```javascript
// src/services/companyApi.js
import apiClient from './axiosConfig';

const BACKEND_URL = import.meta.env.VITE_BACKEND_URL || 'http://localhost:3000';

export const companyApi = {
  getCompanies: async (params) => {
    const queryParams = new URLSearchParams();
    if (params.page) queryParams.append('page', params.page);
    if (params.limit) queryParams.append('limit', params.limit);
    if (params.search?.trim()) queryParams.append('search', params.search.trim());
    if (params.industry && params.industry !== 'All') queryParams.append('industry', params.industry);
    if (params.difficulty && params.difficulty !== 'All') queryParams.append('difficulty', params.difficulty);

    const response = await apiClient.get(`${BACKEND_URL}/api/companies?${queryParams.toString()}`);
    return response.data;
  },

  getCompanyBySlug: async (slug) => {
    const response = await apiClient.get(`${BACKEND_URL}/api/companies/${slug}`);
    return response.data;
  },
};

// React Query hooks — always export these alongside the API functions
export const useCompanies = (params) => {
  return useQuery({
    queryKey: ['companies', params],
    queryFn: () => companyApi.getCompanies(params),
    staleTime: 5 * 60 * 1000,
    keepPreviousData: true,
  });
};

export const useCompany = (slug) => {
  return useQuery({
    queryKey: ['company', slug],
    queryFn: () => companyApi.getCompanyBySlug(slug),
    staleTime: 10 * 60 * 1000,
    enabled: !!slug,
  });
};
```

#### Pattern 2: Every Redux Filter Slice

```javascript
// src/store/slices/companyFilterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const initialState = {
  searchQuery: '',
  debouncedSearchQuery: '',
  filters: {
    industry: 'All',
    difficulty: 'All',
    sort: 'views',
    order: 'desc'
  },
  pagination: {
    currentPage: 1,
    pageSize: 8
  }
};

const companyFilterSlice = createSlice({
  name: 'companyFilters',
  initialState,
  reducers: {
    setSearchQuery: (state, action) => {
      state.searchQuery = action.payload;
    },
    setDebouncedSearchQuery: (state, action) => {
      state.debouncedSearchQuery = action.payload;
      state.pagination.currentPage = 1; // Always reset pagination on search
    },
    setFilter: (state, action) => {
      const { key, value } = action.payload;
      state.filters[key] = value;
      state.pagination.currentPage = 1;
    },
    setPage: (state, action) => {
      state.pagination.currentPage = action.payload;
    },
    resetFilters: () => initialState,
  }
});

export const { setSearchQuery, setDebouncedSearchQuery, setFilter, setPage, resetFilters } = companyFilterSlice.actions;

// Selectors — build query params from slice state
export const selectCompanyQueryParams = (state) => ({
  page: state.companyFilters.pagination.currentPage,
  limit: state.companyFilters.pagination.pageSize,
  search: state.companyFilters.debouncedSearchQuery,
  industry: state.companyFilters.filters.industry,
  difficulty: state.companyFilters.filters.difficulty,
  sort: state.companyFilters.filters.sort,
  order: state.companyFilters.filters.order,
});

export default companyFilterSlice.reducer;
```

#### Pattern 3: Every Page Component

```jsx
// src/pages/InterviewPrepPages/CompanyListing.jsx
import { useDispatch, useSelector } from 'react-redux';
import { useDebounce } from '../../hooks/useDebounce';
import { useCompanies } from '../../services/companyApi';
import {
  setSearchQuery, setDebouncedSearchQuery, setFilter, setPage,
  selectCompanyQueryParams
} from '../../store/slices/companyFilterSlice';

const CompanyListing = () => {
  const dispatch = useDispatch();
  const queryParams = useSelector(selectCompanyQueryParams);
  const searchQuery = useSelector(state => state.companyFilters.searchQuery);
  const filters = useSelector(state => state.companyFilters.filters);
  const pagination = useSelector(state => state.companyFilters.pagination);

  const debouncedSearch = useDebounce(searchQuery, 300);
  useEffect(() => {
    dispatch(setDebouncedSearchQuery(debouncedSearch));
  }, [debouncedSearch, dispatch]);

  const { data, isLoading, isFetching, isError, error } = useCompanies(queryParams);
  const companies = data?.companies || [];
  const paginationData = data?.pagination || {};
  const loading = isLoading || isFetching;

  if (isError) {
    return (
      <div className="text-center py-12 text-red-600">
        {error?.message || 'Failed to load companies'}
      </div>
    );
  }

  return (
    <div className="max-w-7xl mx-auto px-4 py-8">
      {/* Header */}
      <div className="mb-8">
        <h1 className="text-3xl font-bold text-gray-900">Interview Prep</h1>
        <p className="text-gray-600 mt-2">Browse companies and ace your next interview</p>
      </div>

      {/* Search */}
      <input
        type="text"
        placeholder="Search companies..."
        value={searchQuery}
        onChange={(e) => dispatch(setSearchQuery(e.target.value))}
        className="w-full px-4 py-3 border border-gray-200 rounded-xl mb-6 focus:outline-none focus:ring-2 focus:ring-blue-500"
      />

      {/* Filter chips */}
      <div className="flex gap-2 flex-wrap mb-6">
        {['All', 'Tech', 'Finance', 'Consulting', 'E-Commerce'].map(ind => (
          <button
            key={ind}
            onClick={() => dispatch(setFilter({ key: 'industry', value: ind }))}
            className={`px-4 py-2 rounded-full text-sm font-medium transition-colors ${
              filters.industry === ind
                ? 'bg-blue-600 text-white'
                : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
            }`}
          >
            {ind}
          </button>
        ))}
      </div>

      {/* Grid */}
      {loading && (
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
          {Array(8).fill(0).map((_, i) => (
            <div key={i} className="h-48 bg-gray-100 rounded-xl animate-pulse" />
          ))}
        </div>
      )}

      {!loading && companies.length === 0 && (
        <div className="text-center py-16 text-gray-500">
          No companies found matching your search.
        </div>
      )}

      {!loading && companies.length > 0 && (
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
          {companies.map(company => (
            <CompanyCard key={company._id} company={company} />
          ))}
        </div>
      )}

      {/* Pagination */}
      {paginationData.totalPages > 1 && (
        <div className="mt-8 flex justify-center gap-2">
          {Array.from({ length: paginationData.totalPages }, (_, i) => i + 1).map(page => (
            <button
              key={page}
              onClick={() => dispatch(setPage(page))}
              className={`w-10 h-10 rounded-lg font-medium transition-colors ${
                pagination.currentPage === page
                  ? 'bg-blue-600 text-white'
                  : 'border border-gray-200 hover:bg-gray-50'
              }`}
            >
              {page}
            </button>
          ))}
        </div>
      )}
    </div>
  );
};

export default CompanyListing;
```

#### Pattern 4: Every Backend Route File

```javascript
// backend/routes/companyRoutes.js
const express = require('express');
const router = express.Router();
const mongoose = require('mongoose');
const Company = require('../models/companySchema');
const logger = require('../utils/logger');

// Reuse this exact handleError pattern in every route file
const handleError = (res, error, defaultMessage = 'An error occurred') => {
  logger.error('Error:', { message: error.message, name: error.name });

  if (error.name === 'ValidationError') {
    return res.status(400).json({ success: false, message: 'Validation error', error: error.message, statusCode: 400 });
  }
  if (error.name === 'CastError') {
    return res.status(400).json({ success: false, message: 'Invalid ID format', error: 'INVALID_ID', statusCode: 400 });
  }

  return res.status(500).json({
    success: false,
    message: defaultMessage,
    error: process.env.NODE_ENV === 'production' ? 'INTERNAL_SERVER_ERROR' : error.message,
    statusCode: 500
  });
};

// DB connection check middleware
const checkDb = (req, res, next) => {
  if (mongoose.connection.readyState !== 1) {
    return res.status(503).json({ success: false, message: 'Database unavailable', error: 'DATABASE_UNAVAILABLE', statusCode: 503 });
  }
  next();
};

router.use(checkDb);

// GET /api/companies
router.get('/', async (req, res) => {
  try {
    const { page = 1, limit = 8, search, industry, difficulty, sort = 'views', order = 'desc' } = req.query;

    let query = { approved: true };

    if (search) {
      query.$or = [
        { name: { $regex: search, $options: 'i' } },
        { description: { $regex: search, $options: 'i' } }
      ];
    }
    if (industry && industry !== 'All') query.industry = industry;
    if (difficulty && difficulty !== 'All') query.interviewDifficulty = difficulty;

    const skip = (parseInt(page) - 1) * parseInt(limit);
    const sortObj = { [sort]: order === 'desc' ? -1 : 1 };

    const [companies, total] = await Promise.all([
      Company.find(query).sort(sortObj).skip(skip).limit(parseInt(limit)).lean(),
      Company.countDocuments(query)
    ]);

    return res.status(200).json({
      success: true,
      companies,
      pagination: {
        total,
        currentPage: parseInt(page),
        pageSize: parseInt(limit),
        totalPages: Math.ceil(total / parseInt(limit))
      }
    });
  } catch (error) {
    handleError(res, error, 'Failed to fetch companies');
  }
});

// GET /api/companies/:slug
router.get('/:slug', async (req, res) => {
  try {
    const company = await Company.findOne({ slug: req.params.slug, approved: true }).lean();
    if (!company) {
      return res.status(404).json({ success: false, message: 'Company not found', statusCode: 404 });
    }
    return res.status(200).json({ success: true, company });
  } catch (error) {
    handleError(res, error, 'Failed to fetch company');
  }
});

module.exports = router;
```

#### Pattern 5: Every Card Component

```jsx
// src/components/InterviewPrepComponents/CompanyCard.jsx
import { Link } from 'react-router-dom';
import { Building2, Globe } from 'lucide-react';
import { Badge } from '../ui/badge';

// Color coding for difficulty (matches the visual spec)
const DIFFICULTY_CONFIG = {
  Easy: { className: 'bg-green-100 text-green-700', label: 'Easy' },
  Medium: { className: 'bg-yellow-100 text-yellow-700', label: 'Medium' },
  Hard: { className: 'bg-red-100 text-red-700', label: 'Hard' },
};

const CompanyCard = ({ company }) => {
  const diffConfig = DIFFICULTY_CONFIG[company.interviewDifficulty] || DIFFICULTY_CONFIG.Medium;

  return (
    <div className="bg-white rounded-xl shadow-md hover:shadow-lg transition-shadow duration-300 overflow-hidden">
      {/* Logo area */}
      <div className="p-6 flex items-center justify-center bg-gray-50 h-32">
        {company.logo?.url ? (
          <img src={company.logo.url} alt={company.name} className="max-h-16 max-w-32 object-contain" />
        ) : (
          <div className="w-16 h-16 rounded-full bg-blue-100 flex items-center justify-center">
            <span className="text-2xl font-bold text-blue-600">
              {company.name.charAt(0)}
            </span>
          </div>
        )}
      </div>

      {/* Content */}
      <div className="p-4">
        <h3 className="font-semibold text-gray-900 text-lg mb-1">{company.name}</h3>
        <p className="text-sm text-gray-500 mb-3">{company.industry}</p>

        <div className="flex gap-2 mb-4">
          <span className={`text-xs font-medium px-2 py-1 rounded-full ${diffConfig.className}`}>
            {diffConfig.label}
          </span>
        </div>

        <Link
          to={`/interview-prep/${company.slug}`}
          className="block w-full text-center bg-white hover:bg-blue-600 text-blue-600 hover:text-white px-4 py-2 rounded-lg font-medium transition-colors border border-blue-500 text-sm"
        >
          View Interview Guide
        </Link>
      </div>
    </div>
  );
};

// Always memo card components — they appear in lists
export default React.memo(CompanyCard);
```

### 8.3 Naming Conventions — MANDATORY

| Type | Convention | Example |
|---|---|---|
| React components | PascalCase | `CompanyCard.jsx`, `QuestionCard.jsx` |
| Pages | PascalCase | `CompanyListing.jsx`, `DsaProblemPage.jsx` |
| API service files | camelCase | `companyApi.js`, `dsaApi.js` |
| Redux slice files | camelCase | `companyFilterSlice.js` |
| Backend model files | camelCase + Schema | `companySchema.js` |
| Backend route files | camelCase + Routes | `companyRoutes.js` |
| Backend service files | camelCase + Service | `companyCacheService.js` |
| URL slugs | kebab-case | `two-sum`, `dynamic-programming` |
| MongoDB field names | camelCase | `dateAsked`, `approvalStatus` |
| CSS classes | Tailwind only | No custom CSS files |

### 8.4 State Management Rules

| Type of State | Where to Put It |
|---|---|
| API data (companies, problems, etc.) | React Query — never in Redux |
| Search query (raw input) | Redux slice |
| Debounced search query | Redux slice |
| Filter selections | Redux slice |
| Pagination | Redux slice |
| Modal open/close | Local `useState` |
| Tab selection | Local `useState` |
| Form state | Local `useState` |

### 8.5 Do's and Don'ts

**DO:**
- Use `React.memo()` on all card components
- Use `useCallback` and `useMemo` for expensive computations
- Add `.lean()` to all MongoDB read queries (faster, plain JS objects)
- Use `Promise.all([...])` when running parallel DB queries
- Export both the API function AND the React Query hook from the same file
- Always handle loading, error, and empty states in every list component
- Use skeleton loaders (pulsing gray boxes) not spinners for list loading
- Add `index` props on frequently-queried fields in Mongoose schemas

**DON'T:**
- Don't use `useEffect` to fetch data — use React Query
- Don't call APIs directly in components — always go through service files
- Don't put server data in Redux — Redux is for UI state only
- Don't write inline styles — use Tailwind classes only
- Don't create custom CSS files — use Tailwind config extensions
- Don't put business logic in React components — put it in service files or backend
- Don't use `console.log` in production code — use a logger
- Don't add auth to public read routes — these endpoints are public

---

## 9. Design System Reference

### 9.1 Color Palette (Tailwind classes)

| Purpose | Class | Hex |
|---|---|---|
| Primary action | `bg-blue-600 text-white` | #2563EB |
| Primary hover | `hover:bg-blue-700` | #1D4ED8 |
| Primary outline | `border-blue-500 text-blue-600` | #3B82F6 |
| Success/Easy | `bg-green-100 text-green-700` | #DCFCE7 / #15803D |
| Warning/Medium | `bg-yellow-100 text-yellow-700` | #FEF9C3 / #A16207 |
| Error/Hard | `bg-red-100 text-red-700` | #FEE2E2 / #B91C1C |
| Premium badge | `bg-amber-100 text-amber-700` | #FEF3C7 / #B45309 |
| Neutral card bg | `bg-white` | #FFFFFF |
| Page bg | `bg-gray-50` | #F9FAFB |
| Border | `border-gray-200` | #E5E7EB |
| Muted text | `text-gray-500` | #6B7280 |
| Body text | `text-gray-700` | #374151 |
| Heading | `text-gray-900` | #111827 |

### 9.2 Typography Scale

```
Page Title:    text-3xl font-bold text-gray-900
Section Title: text-xl font-semibold text-gray-900
Card Title:    text-lg font-semibold text-gray-900
Subtitle:      text-sm text-gray-500
Body:          text-sm text-gray-700
Badge/Label:   text-xs font-medium
Code:          font-mono text-sm
```

### 9.3 Spacing & Layout

```
Page wrapper:    max-w-7xl mx-auto px-4 py-8
Section gap:     mb-8 or space-y-8
Card gap:        gap-4
Card padding:    p-4 (sm) or p-6 (lg)
Card radius:     rounded-xl
Card shadow:     shadow-md hover:shadow-lg transition-shadow
```

### 9.4 Button Styles (Exact)

```jsx
// Primary
<button className="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-lg font-medium transition-colors">

// Outline (card CTA)
<button className="px-4 py-2 bg-white hover:bg-blue-600 text-blue-600 hover:text-white border border-blue-500 rounded-lg font-medium transition-colors">

// Ghost
<button className="px-4 py-2 text-gray-700 hover:bg-gray-100 rounded-lg font-medium transition-colors">

// Chip (selected)
<button className="px-4 py-2 bg-blue-600 text-white rounded-full text-sm font-medium">

// Chip (unselected)
<button className="px-4 py-2 bg-gray-100 text-gray-700 hover:bg-gray-200 rounded-full text-sm font-medium">
```

### 9.5 shadcn/ui Components to Use

Import from `@/components/ui/`:

| Component | Use For |
|---|---|
| `<Tabs>` | Company detail page tabs |
| `<Accordion>` | Interview phases, Do's & Don'ts |
| `<Badge>` | Difficulty, category, tier labels |
| `<Card>` | Base card wrapper |
| `<Dialog>` | Success story submission modal |
| `<Tabs>` | Language tabs in code blocks |
| `<Progress>` | Topic completion progress |
| `<Separator>` | Section dividers |
| `<Alert>` | Premium lock notices |

---

## 10. AI Prompts Toolkit

Use these prompts when asking AI (Claude, GPT-4, etc.) to help you build parts of this feature. They are written to produce output that matches Techible's patterns exactly.

---

### Prompt 1: Generate a Page Component

```
Create a React page component for [PAGE NAME] following these exact constraints:

Tech stack:
- React 18 + Vite
- React Router v7 (useParams, useNavigate, Link)
- Redux Toolkit (useSelector, useDispatch)
- TanStack React Query v5 (useQuery)
- Tailwind CSS only (no custom CSS)
- Lucide React for icons
- shadcn/ui components (import from '@/components/ui/')

Patterns to follow:
1. Filter state lives in Redux slice, not component state
2. API data fetching uses React Query hooks (imported from service files)
3. Show skeleton loaders (animate-pulse gray boxes) while loading
4. Show error state if isError is true
5. Show empty state if data is empty array
6. Use React.lazy pattern (the component itself, not a wrapper)
7. No useEffect for data fetching
8. Debounce search input with 300ms delay via Redux dispatch

The page is: [DESCRIBE THE PAGE]

The Redux selector to use: [SELECTOR NAME from slice file]
The React Query hook to use: [HOOK NAME from api file]
The data shape returned: [DESCRIBE THE DATA STRUCTURE]

Return only the JSX component. No explanations.
```

---

### Prompt 2: Generate a Redux Filter Slice

```
Create a Redux Toolkit slice for filter state management following this exact pattern:

Slice name: [e.g., companyFilters]
State shape needed:
- searchQuery: string (raw input)
- debouncedSearchQuery: string (debounced)
- filters: { [key]: value } — list the filter keys: [e.g., industry: 'All', difficulty: 'All']
- pagination: { currentPage: 1, pageSize: 8 }

Rules:
1. All filter changes reset currentPage to 1
2. setFilter reducer takes { key, value } payload
3. Export a selectQueryParams selector that builds the API query params object
4. Use Immer-based mutations (direct state assignment inside reducers)
5. Include a resetFilters action that returns to initialState

Return the complete slice file with exports. No explanations.
```

---

### Prompt 3: Generate a Mongoose Schema

```
Create a Mongoose schema for [ENTITY NAME] following these rules:

1. File name convention: [entityName]Schema.js
2. Use camelCase for all field names
3. Add trim: true to all String fields
4. Use enums with explicit lists where applicable
5. Add required: true only to genuinely required fields
6. Include these standard fields:
   - approved: { type: Boolean, default: false }
   - approvalStatus: { type: String, enum: ['pending', 'approved', 'rejected'], default: 'pending' }
   - createdAt: { type: Date, default: Date.now }
   - updatedAt: { type: Date, default: Date.now }
7. Add mongoose.Schema.Types.ObjectId refs for relationships
8. Add index() calls for fields that will be frequently filtered or sorted
9. Use .lean() note in comments for expected query usage

Entity fields needed: [DESCRIBE THE FIELDS]
Relationships: [DESCRIBE ANY REFS TO OTHER MODELS]

Return only the schema file code. No explanations.
```

---

### Prompt 4: Generate an Express Route File

```
Create an Express.js route file for [RESOURCE NAME] following these exact patterns:

1. Require express, mongoose, the model, and a logger utility
2. Include a handleError(res, error, defaultMessage) function at the top
3. Include a checkDb middleware that checks mongoose.connection.readyState === 1
4. Apply router.use(checkDb) before all routes
5. Use async/await with try/catch for every route handler
6. Use Promise.all([]) for parallel DB queries
7. Add .lean() to all read queries
8. Return responses in this format:
   Success: { success: true, [data], pagination: { total, currentPage, pageSize, totalPages } }
   Error: { success: false, message, error, statusCode }
9. Use req.query destructuring with defaults for GET list routes
10. Build MongoDB query object from filters (skip 'All' values)

Routes to implement:
- GET / (list with pagination and filters)
- GET /:slug (single by slug)
- POST / (create — no auth for now, add ensureAuthenticated comment)
- PATCH /:slug (update — no auth for now)

Model to use: [MODEL NAME]
Fields that can be filtered: [LIST FIELDS]
Fields that can be searched (regex): [LIST FIELDS]
Default sort: [FIELD and DIRECTION]

Return only the route file. No explanations.
```

---

### Prompt 5: Generate a Card Component

```
Create a React card component for [ENTITY] following these exact rules:

1. Accept a single prop: { [entity] } (the full data object)
2. Wrap in React.memo() for performance
3. Use Tailwind CSS only — no style props, no CSS modules
4. Card structure:
   - Outer: div with "bg-white rounded-xl shadow-md hover:shadow-lg transition-shadow duration-300 overflow-hidden"
   - Top section (image/logo area): "bg-gray-50 h-32 flex items-center justify-center"
   - Body: "p-4"
   - CTA button at bottom using outline style: "bg-white hover:bg-blue-600 text-blue-600 hover:text-white border border-blue-500"
5. Use Link from react-router-dom for navigation
6. Use lucide-react for any icons
7. Color-code badges: Easy=green-100/green-700, Medium=yellow-100/yellow-700, Hard=red-100/red-700
8. Handle missing logo/image with an initial-letter fallback div

The entity data shape: [DESCRIBE THE OBJECT SHAPE]
Navigation target URL pattern: [e.g., /interview-prep/:slug]
Badges to show: [LIST BADGES]

Return only the component code. No explanations.
```

---

### Prompt 6: Generate Seed Data

```
Generate a MongoDB seed script for [ENTITY] with [NUMBER] realistic sample records.

Rules:
1. Use Node.js with mongoose
2. Connect to MongoDB at process.env.MONGO_URI or 'mongodb://localhost:27017/techible-learn'
3. Clear existing records before inserting (deleteMany)
4. Use insertMany for bulk insert
5. Generate slugs from names (lowercase, spaces to hyphens)
6. Set approved: true and approvalStatus: 'approved' for all seed data
7. Use realistic, India-focused content where applicable
8. Close connection after completion

Entity: [ENTITY NAME]
Fields: [DESCRIBE FIELDS]
Sample categories/types to cover: [LIST]

Return the complete seed script. No explanations.
```

---

### Prompt 7: Generate a Tabbed Detail Page

```
Create a React tabbed detail page component following these patterns:

1. Use shadcn/ui Tabs component: import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs'
2. Fetch main entity data using React Query (useQuery) by URL param (useParams)
3. Fetch tab-specific data lazily — only when tab is selected (use enabled: activeTab === 'questions')
4. Show a full-page skeleton loader while main data loads
5. Show 404-style message if entity not found
6. Sticky header with entity name and key stats
7. Tabs scroll horizontally on mobile (overflow-x-auto, whitespace-nowrap)
8. Default active tab: first tab

Tabs to include: [LIST TAB NAMES]
For each tab: [DESCRIBE WHAT EACH TAB RENDERS]
Main entity: [NAME and API hook]
Page URL pattern: /[path]/:slug

Return only the component. No explanations.
```

---

### Prompt 8: Fix Integration Issues

```
I'm integrating a standalone React module into an existing codebase. The module was built separately and uses different patterns. Help me adapt [FILE NAME] to match the target codebase patterns.

Target codebase patterns:
- Routing: React Router v7 (not v6)
- API: axios via centralized apiClient from '../../services/axiosConfig' (not fetch)
- Auth: useAuth() from '../../contexts/AuthContext' (not local state)
- Store: Redux via useSelector/useDispatch (not Zustand/Context)
- UI: Tailwind + shadcn/ui (not MUI/Chakra)
- Backend URL: import.meta.env.VITE_BACKEND_URL (not hardcoded)

Current file: [PASTE FILE CONTENT]

Adapt this file to match the target patterns exactly. Keep the same functionality. Return only the adapted file. No explanations.
```

---

*End of SRS Document*
*Version 1.0 — Techible Platform Intern Build*
