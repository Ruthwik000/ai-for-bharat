# ImapactFlow - Requirements Document

## Project Overview

**Project Name:** Impact flow 
**Program:** AI for Bharat  
**Platform:** Mobile-first Progressive Web Application  

## Vision

Transform passive news consumption into active, informed decision-making. Every article shows personal impact with specific numbers, timelines, and clear action steps.

## The Problem

450M+ Indians read news daily but <5% take action. Users read "RBI Increases Repo Rate" but don't realize their ₹40L home loan EMI will increase from ₹32,000 to ₹34,800/month—costing ₹6.72L over 20 years.

---

## Core Features

### 1. Personal Impact Score & Report

**Impact Score Badge (0-10)**
- Displayed on every article before clicking
- Color-coded: High (8-10) Red, Medium (4-7) Yellow, Low (0-3) Green
- Generated in < 2 seconds based on user profile

**Impact Report (Tap badge to view)**
- **Relevance Score:** Why this matters to you
- **Short-term Impact (0-6 months):** Specific numbers, immediate effects
- **Long-term Impact (6m-5y):** Total financial/career impact
- **Action Steps:** 3-5 specific, prioritized recommendations
- **Confidence Score:** High/Medium/Low with source transparency

**Example:**
```
Relevance: 8/10 - Directly affects your 2026 home loan

Short-term:
• EMI increases from ₹32,000 to ₹34,800/month
• Impact starts next month

Long-term:
• ₹6.72L more in interest over 20 years

What YOU Should Do:
✓ Wait 6 months before buying
✓ Increase down payment by ₹2L
✓ Consider ₹35L apartment instead

Confidence: High (9/10)
Based on RBI data + 8 sources
```

### 2. "Wanna Know More?" AI Chatbot

**Context Pre-loaded**
- AI has already read the article and your Impact Report
- No need to explain context

**Auto-generated Summary**
- Personalized brief displayed immediately
- Example: "Based on your profile (₹40L home loan plan), this rate hike increases your EMI by ₹2,800/month"

**Proactive Suggested Questions**
- "Want to explore future scenarios?"
- "Should you take action now or wait?"
- "What are the risks?"
- "How likely are property prices to drop?"

**Conversational Q&A**
- Natural language responses
- Scenario analysis on demand
- Comparison of alternatives (wait vs buy now)
- Response time: < 3 seconds

### 3. Future Scenarios (Phase 3 - Coming Soon)

- Interactive timeline simulations (3, 5, 10 years)
- Multi-scenario comparison
- Long-term financial modeling
- AI-recommended optimal decisions

---

## User Authentication & Profile

### Sign Up / Login
- Email/password via Firebase Authentication
- Password minimum 8 characters
- Session persistence

### User Profile (Required for personalization)
- Age
- Life Stage (Student, Early Career, Mid Career, Professional, Investor)
- Profession
- Monthly Income (₹25k-₹50k, ₹50k-₹1L, ₹1L-₹2L, ₹2L-₹5L, >₹5L)
- Current Commitments (loans, EMIs, savings)
- Financial Goals (Buying House, Job Change, Investing, Education, Retirement)
- Interests (Technology, Business, Finance, Politics, Real Estate, Career)
- Planning Timeline (0-6m, 6-12m, 1-2y, 2-5y)

---

## News Feed & Discovery

### Home Feed
- Personalized news based on user profile
- High-impact articles at top
- Auto-sliding carousel (5 articles)
- Trending section (3 articles with live indicator)
- Recommended section (2+ articles)

### Explore Full-Screen
- Full-screen browsing with page-flip animation
- Swipe navigation
- Large images with overlay
- "Personal Impact" and "Future Scenarios" buttons
- Save/bookmark functionality

### News Categories
- Technology, Business, Finance, Politics, Real Estate, Career, Health, Education

---

## Additional Features

### Saved News
- Bookmark articles for later
- Sync via Firestore

### Notifications
- High-impact news alerts
- In-app notification center

### Profile Management
- View/edit profile information
- Update interests and goals
- Privacy settings

---

## Technical Requirements

### Frontend Stack
- React 18 (JavaScript)
- React Router v6
- Vite build tool
- Tailwind CSS
- Lucide React icons
- Firebase Auth & Firestore

### Backend Integration
- RESTful API: `/api/personalized-news`
- Parameters: age, goals, interests, k (articles count), userId
- Response: news_by_interest with articles array

### AI/LLM Integration
- Impact Report generation (< 2 seconds)
- Interest recommendation
- Chatbot conversation (RAG + LLM)
- Scenario planning

### Data Models

**User Profile:**
```javascript
{
  uid, email, age, profession, income,
  goals: [], interests: [],
  commitments: {}, timeline: string,
  createdAt, updatedAt
}
```

**News Article:**
```javascript
{
  id, headline, summary, content,
  category, source, timestamp,
  impactScore: 0-10,
  impactLevel: "High|Medium|Low",
  impactSummary, image, link
}
```

### Deployment
- Vercel for frontend hosting
- Environment variables for Firebase & API
- Automatic Git deployments

---

## Performance Requirements

- Initial load: < 3 seconds (4G)
- News feed refresh: < 2 seconds
- Impact Report generation: < 2 seconds
- Chatbot response: < 3 seconds
- Smooth animations: 60fps

---

## Responsive Design

- **Mobile (Primary):** 320px - 428px
- **Tablet:** 768px - 1024px
- **Desktop:** 1280px+
- Touch-optimized (48px minimum tap targets)

---

## Security & Privacy

- HTTPS-only
- Firebase Auth security
- User data encryption
- No third-party data sharing
- GDPR considerations

---

## Success Metrics

- Daily Active Users (DAU)
- User retention (7-day, 30-day)
- Articles read per session
- Impact Report views
- Chatbot engagement
- Profile completion rate
- Average session duration

---

## Development Phases

**Phase 1: MVP (3 months)**
- Personal Impact Score & Report (top 5 categories)
- "Wanna Know More?" chatbot
- User profiling
- ET news feed integration

**Phase 2: Enhanced (6 months)**
- 15+ news categories
- Advanced profiling
- Proactive notifications
- Confidence scoring

**Phase 3: Future Ready (12 months)**
- Interactive scenario simulations
- Multi-scenario comparison
- Long-term financial modeling
- AI-recommended decisions

---

**Version:** 1.0  
**Last Updated:** January 25, 2026  
**Status:** Final
