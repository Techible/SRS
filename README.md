# Techible — Interview Prep & Learning Hub
## Intern Build Guide

---

## What You're Building

A new section of the Techible platform with five modules:

| Module | URL | What It Does |
|---|---|---|
| Company Interview Prep | `/interview-prep` | Browse companies, view interview process, questions, stories, compensation |
| Technical Challenges | `/learn/challenges` | Admin-extensible practice hub — DSA, Design Patterns, LLD, Behavioral, AI, etc. |
| AI Learning Hub | `/learn/ai` | Tiered AI curriculum (Tier 1 free → Tier 3 premium) |
| Interview Prep (Leveled) | `/learn/interview-prep` | Level-based interview Q&A (Beginner / Intermediate / Advanced) |
| Projects | `/learn/projects` | Project ideas and challenges |

You will build this as a **standalone React + Node app**. Nitin will integrate it into the main platform when you're done. This README tells you how to get started and what the daily workflow looks like.

For the full specification (schemas, APIs, code patterns), read `INTERN_SRS.md`.

---

## Prerequisites

| Tool | Version | Install |
|---|---|---|
| Node.js | v18+ | [nodejs.org](https://nodejs.org) |
| npm | v9+ | Comes with Node |
| MongoDB | v6+ (local) | [mongodb.com/try/download](https://www.mongodb.com/try/download/community) |
| Git | Any | [git-scm.com](https://git-scm.com) |
| VS Code | Latest | [code.visualstudio.com](https://code.visualstudio.com) |

---

## Project Setup

### 1. Create your frontend

```bash
npm create vite@latest techible-learn-frontend -- --template react
cd techible-learn-frontend

# Core dependencies
npm install react-router-dom @tanstack/react-query @reduxjs/toolkit react-redux axios

# UI
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# shadcn/ui
npx shadcn-ui@latest init
# When prompted: Style = New York, Base color = Zinc, CSS variables = Yes

# Utilities
npm install lucide-react clsx tailwind-merge date-fns

# shadcn components you'll need
npx shadcn-ui@latest add button card badge tabs accordion dialog progress separator alert
```

### 2. Configure Tailwind

In `tailwind.config.js`, update `content`:

```javascript
content: [
  "./index.html",
  "./src/**/*.{js,ts,jsx,tsx}",
],
```

### 3. Create your backend

```bash
mkdir techible-learn-backend
cd techible-learn-backend
npm init -y

npm install express mongoose dotenv cors express-session connect-mongo multer
npm install -D nodemon

# Create entry point
touch server.js
touch .env
```

In `.env`:
```
PORT=3000
MONGO_URI=mongodb://localhost:27017/techible-learn
NODE_ENV=development
SESSION_SECRET=dev-secret-key-change-in-prod
```

In `package.json`, add:
```json
"scripts": {
  "start": "nodemon server.js",
  "seed": "node scripts/seed.js"
}
```

### 4. Basic backend `server.js`

```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const session = require('express-session');
require('dotenv').config();

const app = express();

app.use(cors({ origin: 'http://localhost:5173', credentials: true }));
app.use(express.json({ limit: '2mb' }));
app.use(express.urlencoded({ extended: true, limit: '2mb' }));
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: { httpOnly: true, maxAge: 24 * 60 * 60 * 1000 }
}));

// Routes (add as you build them)
app.use('/api/companies', require('./routes/companyRoutes'));
app.use('/api/dsa', require('./routes/dsaRoutes'));
app.use('/api/ai-topics', require('./routes/aiTopicRoutes'));
app.use('/api/interview-prep', require('./routes/interviewPrepRoutes'));

app.get('/health', (req, res) => res.json({ status: 'ok' }));

mongoose.connect(process.env.MONGO_URI)
  .then(() => {
    console.log('MongoDB connected');
    app.listen(process.env.PORT || 3000, () => {
      console.log(`Server running on port ${process.env.PORT || 3000}`);
    });
  })
  .catch(err => {
    console.error('MongoDB connection error:', err);
    process.exit(1);
  });
```

### 5. Frontend `src/services/axiosConfig.js`

```javascript
import axios from 'axios';

const BACKEND_URL = import.meta.env.VITE_BACKEND_URL || 'http://localhost:3000';

const apiClient = axios.create({
  baseURL: BACKEND_URL,
  withCredentials: true,
  timeout: 20000,
  headers: { 'Content-Type': 'application/json' },
});

apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.code === 'ECONNABORTED') {
      console.error('Request timed out');
    }
    return Promise.reject(error);
  }
);

export default apiClient;
export { BACKEND_URL };
```

### 6. Frontend `src/hooks/useDebounce.js`

```javascript
import { useState, useEffect } from 'react';

export function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}
```

### 7. Frontend `.env`

```
VITE_BACKEND_URL=http://localhost:3000
```

---

## Folder Structure

### Frontend (`techible-learn-frontend/src/`)

```
src/
├── components/
│   ├── ui/                          ← shadcn/ui (auto-generated, don't edit)
│   ├── InterviewPrepComponents/     ← cards, tables, accordions for company prep
│   └── LearnComponents/             ← cards, code blocks for learning hub
├── pages/
│   ├── InterviewPrepPages/
│   │   ├── CompanyListing.jsx       ← /interview-prep
│   │   └── CompanyDetail.jsx        ← /interview-prep/:slug
│   └── LearnPages/
│       ├── DsaLanding.jsx           ← /learn/dsa
│       ├── DsaCategoryPage.jsx      ← /learn/dsa/:category
│       ├── DsaProblemPage.jsx       ← /learn/dsa/:category/:slug
│       ├── AiLearningHub.jsx        ← /learn/ai
│       ├── AiTopicPage.jsx          ← /learn/ai/:topicSlug
│       ├── InterviewPrepHub.jsx     ← /learn/interview-prep
│       ├── InterviewPrepLevel.jsx   ← /learn/interview-prep/:level
│       ├── InterviewPrepTopic.jsx   ← /learn/interview-prep/:level/:topic
│       └── ProjectsHub.jsx          ← /learn/projects
├── services/
│   ├── axiosConfig.js               ← Centralized axios (required)
│   ├── companyApi.js                ← API functions + React Query hooks
│   ├── dsaApi.js
│   └── aiTopicsApi.js
├── store/
│   ├── store.js                     ← Redux store config
│   └── slices/
│       ├── companyFilterSlice.js    ← Filters for company listing
│       └── dsaFilterSlice.js
└── hooks/
    └── useDebounce.js               ← Required utility
```

### Backend (`techible-learn-backend/`)

```
/
├── models/
│   ├── companySchema.js
│   ├── interviewProcessSchema.js
│   ├── interviewQuestionSchema.js
│   ├── successStorySchema.js
│   ├── compensationSchema.js
│   ├── dsaCategorySchema.js
│   ├── codingProblemSchema.js
│   ├── aiTopicSchema.js
│   └── interviewPrepTopicSchema.js
├── routes/
│   ├── companyRoutes.js
│   ├── dsaRoutes.js
│   ├── aiTopicRoutes.js
│   └── interviewPrepRoutes.js
├── scripts/
│   └── seed.js                      ← Seed script for dummy data
├── utils/
│   └── logger.js                    ← Simple console logger
├── server.js
└── .env
```

---

## Build Order (Recommended)

Work in this sequence to avoid blockers:

```
Week 1: Foundation + Company Prep
  Day 1-2: Setup + backend models + seed data
  Day 3-4: Company listing page (frontend)
  Day 5:   Company detail page (tabs: Overview + Questions)

Week 2: Complete Company Prep + Start DSA
  Day 1-2: Remaining tabs (Stories, Compensation, Resources)
  Day 3-4: DSA landing + category page
  Day 5:   DSA problem detail page

Week 3: AI Learning + Interview Prep
  Day 1-2: AI Learning Hub + topic pages
  Day 3-4: Interview Prep leveled system
  Day 5:   Projects page

Week 4: Polish
  Day 1-2: Mobile responsiveness
  Day 3:   Loading/error/empty states audit
  Day 4:   Seed data quality pass
  Day 5:   Handoff prep
```

---

## Daily Development Workflow

```bash
# Terminal 1 — Backend
cd techible-learn-backend
npm start

# Terminal 2 — Frontend
cd techible-learn-frontend
npm run dev

# After adding new models — seed the data
cd techible-learn-backend
node scripts/seed.js
```

---

## The 10 Rules You Must Not Break

These will cause integration failures if violated:

1. **Use Tailwind CSS only** — no CSS modules, no styled-components, no inline styles
2. **Server data goes in React Query, never Redux** — Redux is for filter/search/pagination state only
3. **All API calls go through `axiosConfig.js`** — never use `fetch()` directly
4. **Every card component must be wrapped in `React.memo()`**
5. **File naming matches the SRS exactly** — `companySchema.js`, not `Company.js` or `company.model.js`
6. **URL slugs are always kebab-case** — `two-sum`, `dynamic-programming`, not `twoSum` or `Two_Sum`
7. **MongoDB field names are always camelCase** — `dateAsked`, not `date_asked` or `DateAsked`
8. **Every backend route file uses the `handleError()` and `checkDb` pattern from the SRS**
9. **All imports use relative paths** — `../../services/companyApi` not absolute paths
10. **Never hardcode the backend URL** — always use `import.meta.env.VITE_BACKEND_URL`

---

## Seed Data Minimum Requirements

Before submitting, your seed script must produce:

| Collection | Minimum Records |
|---|---|
| Companies | 20 (mix of industries and difficulties) |
| Interview Processes | 1 per company (20 total) |
| Interview Questions | 10+ per company (200+ total) |
| Success Stories | 3+ per company (60+ total) |
| Compensation Records | 5+ per company (100+ total) |
| Challenge Domains | 7 (the initial list in the SRS — admin can add more later) |
| Challenge Categories | 4–6 per domain |
| Challenges | 8–10 per category |
| AI Topics | 16+ across all 3 tiers (intern team decides topic titles and content) |
| Interview Prep Topics | 4–6 topic categories per level, 8+ questions per topic |

**Companies to seed (minimum):** Google, Amazon, Microsoft, Adobe, Flipkart, Infosys, TCS, Wipro, PhonePe, Zomato — plus 10 more of your choice.

**Challenge content and AI topics:** Intern team's responsibility. Research what is most commonly asked and needed. The system is fully DB-driven — admin can expand any domain or add new ones with zero code changes. Design your seed script to demonstrate this flexibility.

---

## Quick Reference: shadcn/ui Components

```jsx
// Tabs (for Company Detail page)
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
<Tabs defaultValue="overview">
  <TabsList><TabsTrigger value="overview">Overview</TabsTrigger></TabsList>
  <TabsContent value="overview">...</TabsContent>
</Tabs>

// Badge (difficulty, category labels)
import { Badge } from '@/components/ui/badge';
<Badge className="bg-green-100 text-green-700">Easy</Badge>

// Accordion (interview phases, tips)
import { Accordion, AccordionContent, AccordionItem, AccordionTrigger } from '@/components/ui/accordion';
<Accordion type="single" collapsible>
  <AccordionItem value="phase-1">
    <AccordionTrigger>Phase 1: Online Assessment</AccordionTrigger>
    <AccordionContent>...</AccordionContent>
  </AccordionItem>
</Accordion>

// Dialog (submission modals)
import { Dialog, DialogContent, DialogHeader, DialogTitle } from '@/components/ui/dialog';
<Dialog open={isOpen} onOpenChange={setIsOpen}>
  <DialogContent>
    <DialogHeader><DialogTitle>Submit Your Story</DialogTitle></DialogHeader>
  </DialogContent>
</Dialog>

// Progress (topic completion)
import { Progress } from '@/components/ui/progress';
<Progress value={33} className="h-2" />
```

---

## Quick Reference: Lucide Icons

```jsx
import {
  Building2,        // Companies
  Code,             // DSA / coding
  Brain,            // AI / ML
  BookOpen,         // Learning
  Trophy,           // Success stories
  DollarSign,       // Compensation
  ChevronRight,     // Navigation
  ArrowLeft,        // Back button
  Search,           // Search input
  Filter,           // Filter button
  Star,             // Ratings
  Clock,            // Duration
  Users,            // Team / count
  CheckCircle,      // Completed
  Lock,             // Premium locked
  Sparkles,         // AI / premium
  BarChart2,        // Analytics
  Briefcase,        // Jobs / roles
  GraduationCap,    // Education
  Target,           // Goals
} from 'lucide-react';

// Usage: <Building2 size={16} className="text-gray-500" />
```

---

## Handoff Checklist

Before submitting your work to Nitin:

```
Code Quality
□ All 10 mandatory rules followed
□ No console.log statements in production code
□ No hardcoded URLs or secrets
□ No unused imports
□ All components handle loading, error, and empty states

Data
□ Seed script runs without errors on fresh MongoDB
□ All minimum seed counts met
□ Slugs are unique, lowercase, kebab-case
□ All seed data has approved: true

Testing
□ All pages render without console errors
□ Search works with debounce (300ms)
□ Filters work and reset pagination
□ Mobile layout tested (Chrome DevTools device mode)
□ Tabs switch correctly on company detail page
□ Problem pages show code with language tabs

Folder Structure
□ Matches the structure in this README exactly
□ File names match conventions in SRS
□ No extra files or temp files committed

Documentation
□ How to run the project (already in this README — just verify it works)
□ Any environment variables needed are documented
□ Seed script location and usage is clear
```

---

## Getting Help

1. **Read the SRS first** (`INTERN_SRS.md`) — most questions are answered there
2. **Use AI prompts from Section 10 of the SRS** — they are designed to produce code that matches these patterns
3. **Look at the pattern examples in SRS Section 8** — every file type has a full working example
4. **Check the "10 Rules" above** — if something feels off during integration, one of these is usually the cause

---

*Techible Platform — Intern Build*
*Read INTERN_SRS.md for complete specifications*
