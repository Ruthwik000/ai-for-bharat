# ET Connect - Design Document

## Project Overview

**Project Name:** ET Connect  
**Platform:** Mobile-first Progressive Web Application  
**Design Philosophy:** Clean, trust-focused, number-driven, mobile-first  
**Target:** 450M+ Indian news readers  
**Tech Stack:** React.js + AWS Serverless (Lambda, Bedrock, DynamoDB, S3)  
**Hosting:** AWS Amplify / S3 + CloudFront

---

## System Architecture Design

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         User Interface Layer                         │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              React.js Frontend (SPA)                         │   │
│  │  • Responsive Design (Mobile-first)                          │   │
│  │  • Tailwind CSS Styling                                      │   │
│  │  • React Router Navigation                                   │   │
│  │  • Axios HTTP Client                                         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                        │
│                              │ HTTPS/REST                             │
│                              ▼                                        │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                         API Gateway Layer                            │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │           Amazon API Gateway (REST API)                      │   │
│  │  • Request Validation                                        │   │
│  │  • Rate Limiting & Throttling                                │   │
│  │  • CORS Configuration                                        │   │
│  │  • Lambda Authorizer (Cognito)                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                        │
│                              ▼                                        │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                      Business Logic Layer                            │
│                                                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │ AWS Lambda   │  │ AWS Lambda   │  │ AWS Lambda   │             │
│  │ (News Feed)  │  │ (Impact AI)  │  │ (Chatbot)    │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
│         │                  │                  │                      │
│         │                  ▼                  │                      │
│         │         ┌─────────────────┐         │                      │
│         │         │ Amazon Bedrock  │         │                      │
│         │         │ (Claude 3)      │         │                      │
│         │         │                 │         │                      │
│         │         │ Knowledge Bases │         │                      │
│         │         │ (RAG)           │         │                      │
│         │         └─────────────────┘         │                      │
│         │                  │                  │                      │
│         ▼                  ▼                  ▼                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                         Data Layer                                   │
│                                                                       │
│  ┌──────────────────────┐         ┌──────────────────────┐         │
│  │     DynamoDB         │         │      Amazon S3        │         │
│  │                      │         │                       │         │
│  │ • User Profiles      │         │ • News Articles       │         │
│  │ • Saved Articles     │         │ • Full Content        │         │
│  │ • Chat History       │         │ • Images              │         │
│  │ • Impact Reports     │         │ • Knowledge Base Docs │         │
│  └──────────────────────┘         └──────────────────────┘         │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                    External Services Layer                           │
│                                                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │ News APIs    │  │ RSS Feeds    │  │ AWS Cognito  │             │
│  │ (NewsAPI.org)│  │ (ET, Mint)   │  │ (Auth)       │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘
```

### Data Flow Architecture

**1. User Authentication Flow**
```
User → React App → API Gateway → Lambda Authorizer → Cognito
                                      ↓
                                  JWT Token
                                      ↓
                              Store in LocalStorage
```

**2. Personalized News Feed Flow**
```
User Request → API Gateway → Lambda (PersonalizedFeed)
                                      ↓
                              Get User Profile (DynamoDB)
                                      ↓
                              Query News Articles (S3)
                                      ↓
                              Rank by Relevance
                                      ↓
                              Return JSON Response
```

**3. Impact Analysis Flow**
```
User Clicks "View Impact" → API Gateway → Lambda (ImpactAnalysis)
                                              ↓
                                    Get Article (S3)
                                              ↓
                                    Get User Profile (DynamoDB)
                                              ↓
                                    Call Bedrock (Claude 3)
                                              ↓
                                    RAG: Query Knowledge Base
                                              ↓
                                    Generate Impact Report
                                              ↓
                                    Store Report (DynamoDB)
                                              ↓
                                    Return to Frontend (< 2 sec)
```

**4. Chatbot Conversation Flow**
```
User Message → API Gateway → Lambda (Chatbot)
                                  ↓
                        Get Chat History (DynamoDB)
                                  ↓
                        Get Article Context (S3)
                                  ↓
                        Call Bedrock with Context
                                  ↓
                        RAG: Retrieve Relevant Data
                                  ↓
                        Generate Response
                                  ↓
                        Save Message (DynamoDB)
                                  ↓
                        Return Response (< 3 sec)
```

---

## Design Principles

### 1. Mobile-First
- Primary viewport: 320px - 428px (mobile devices)
- Touch-optimized interactions
- Thumb-friendly navigation zones
- Responsive scaling for tablet (768px+) and desktop (1280px+)

### 2. Trust & Transparency
- Clear data presentation with source attribution
- Confidence scores visible on all AI-generated content
- No sensationalism—calm, informational tone
- Professional, credible visual language

### 3. Number-Driven
- Specific numbers over vague statements
- "₹2,800 EMI increase" not "rates increased"
- Impact scores (0-10) prominently displayed
- Financial projections with timelines

### 4. Instant Gratification
- Impact visible in < 2 seconds
- No loading spinners for critical content
- Progressive disclosure (summary → details)
- Smooth, minimal animations (60fps)

### 5. Actionable Design
- Clear CTAs for next steps
- Action recommendations prominently displayed
- Easy access to "Wanna Know More?" chatbot
- One-tap save/bookmark functionality

---

## Color System

### Primary Colors
```css
--primary: #1E3A8A (Deep Blue) - Trust, professionalism
--primary-light: #3B82F6 (Blue) - Interactive elements
--primary-dark: #1E40AF (Dark Blue) - Headers, emphasis
```

### Impact Score Colors
```css
--negative: #EF4444 (Red) - High Impact (8-10)
--warning: #F59E0B (Amber) - Medium Impact (4-7)
--positive: #10B981 (Green) - Low Impact (0-3)
```

### Neutral Colors
```css
--neutral-950: #0A0A0A (Background - dark mode)
--neutral-900: #171717 (Cards - dark mode)
--neutral-100: #F5F5F5 (Background - light mode)
--white: #FFFFFF (Cards - light mode)
--secondary: #6B7280 (Secondary text)
```

### Semantic Colors
```css
--success: #10B981 (Green) - Positive actions
--error: #EF4444 (Red) - Errors, warnings
--info: #3B82F6 (Blue) - Information
```

---

## Typography

### Font Families
```css
--font-primary: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif
--font-headline: 'Merriweather', Georgia, serif (for news headlines)
--font-mono: 'JetBrains Mono', 'Courier New', monospace (for numbers)
```

### Font Scale
```css
--text-xs: 0.75rem (12px) - Timestamps, labels
--text-sm: 0.875rem (14px) - Body text, descriptions
--text-base: 1rem (16px) - Primary content
--text-lg: 1.125rem (18px) - Subheadings
--text-xl: 1.25rem (20px) - Section titles
--text-2xl: 1.5rem (24px) - Page titles
--text-3xl: 1.875rem (30px) - Hero headlines
--text-4xl: 2.25rem (36px) - Landing page
```

### Font Weights
```css
--font-light: 300
--font-normal: 400
--font-medium: 500
--font-semibold: 600
--font-bold: 700
```

---

## Component Design Specifications

### 1. News Card (Home Feed)

**Dimensions:**
- Width: 100% (with 16px padding)
- Height: Auto (min 280px)
- Border radius: 12px
- Shadow: 0 2px 8px rgba(0,0,0,0.1)

**Structure:**
```
┌─────────────────────────────────┐
│  [News Image - 200px height]    │
│  [Gradient overlay]              │
│  ┌─────────────────────────┐    │
│  │ Impact Score Badge      │    │
│  │ [8/10] High Impact      │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│ Category • Timestamp             │
│                                  │
│ Headline (2 lines max)           │
│                                  │
│ Summary (3 lines max)            │
│                                  │
│ [View Impact] [Save] [Share]     │
└─────────────────────────────────┘
```

**Impact Score Badge:**
- Position: Top-right corner of image
- Size: 60px × 32px
- Background: Semi-transparent with backdrop blur
- Font: Bold, 14px
- Color: Dynamic based on score (Red/Yellow/Green)

**Interactions:**
- Tap card → Navigate to full article
- Tap "View Impact" → Show Impact Report
- Tap badge → Show Impact Report
- Tap Save → Add to saved news
- Tap Share → Native share sheet

---

### 2. Personal Impact Report (Modal/Page)

**Layout:**
```
┌─────────────────────────────────┐
│ [Close Button]                   │
│                                  │
│ YOUR PERSONAL IMPACT             │
│                                  │
│ ┌─────────────────────────┐     │
│ │ Relevance Score: 8/10   │     │
│ │ High - directly affects │     │
│ │ your 2026 home loan     │     │
│ └─────────────────────────┘     │
│                                  │
│ Short-term Impact (0-6 months)   │
│ • EMI increases from ₹32,000    │
│   to ₹34,800/month              │
│ • Impact starts next month      │
│                                  │
│ Long-term Impact (6m - 5y)       │
│ • ₹6.72L more in interest       │
│   over 20 years                 │
│                                  │
│ What YOU Should Do:              │
│ ✓ Wait 6 months before buying   │
│ ✓ Increase down payment by ₹2L  │
│ ✓ Consider ₹35L apartment       │
│                                  │
│ Confidence: High (9/10)          │
│ Based on RBI data + 8 sources    │
│                                  │
│ [Wanna Know More?] [Save Report] │
└─────────────────────────────────┘
```

**Visual Elements:**
- Relevance Score: Large, bold number with color coding
- Impact sections: Clear hierarchy with icons
- Action items: Checkboxes (visual only) for clarity
- Confidence indicator: Progress bar + text
- CTA buttons: Primary (blue) and secondary (outline)

**Animations:**
- Slide up from bottom (mobile)
- Fade in with scale (desktop)
- Smooth transitions (300ms ease-out)

---

### 3. "Wanna Know More?" Chatbot Interface

**Layout:**
```
┌─────────────────────────────────┐
│ [← Back] ET Connect AI           │
├─────────────────────────────────┤
│ ┌─────────────────────────┐     │
│ │ Context Card (Collapsible)│    │
│ │ Article: RBI Rate Hike    │    │
│ │ Your Impact: 8/10         │    │
│ └─────────────────────────┘     │
│                                  │
│ [AI Avatar] Based on your        │
│             profile (planning    │
│             ₹40L home loan),     │
│             this rate hike...    │
│                                  │
│ Suggested Questions:             │
│ • Want to explore scenarios?     │
│ • Should you take action now?    │
│ • What are the risks?            │
│                                  │
│ [User] What if I buy now?        │
│                                  │
│ [AI Avatar] If you buy now...    │
│             [Response]           │
│                                  │
├─────────────────────────────────┤
│ [Type your question...]    [→]  │
└─────────────────────────────────┘
```

**Design Details:**
- **Context Card:**
  - Collapsible (tap to expand/collapse)
  - Shows article title + impact score
  - Light blue background (#EFF6FF)
  - 8px padding, 8px border radius

- **AI Messages:**
  - Left-aligned
  - Light gray background (#F3F4F6)
  - Avatar icon (robot/AI symbol)
  - Max width: 80% of screen
  - 12px padding, 16px border radius

- **User Messages:**
  - Right-aligned
  - Primary blue background (#3B82F6)
  - White text
  - Max width: 80% of screen
  - 12px padding, 16px border radius

- **Suggested Questions:**
  - Pill-shaped buttons
  - Border: 1px solid #E5E7EB
  - Tap to auto-fill input
  - Horizontal scroll on mobile

- **Input Field:**
  - Fixed at bottom
  - White background with shadow
  - 48px height (touch-friendly)
  - Send button: Primary blue

**Interactions:**
- Tap suggested question → Auto-send
- Type and tap send → Submit message
- Scroll to load history
- Tap context card → Expand/collapse

---

### 4. Bottom Navigation (Mobile)

**Layout:**
```
┌─────────────────────────────────┐
│ [Home] [Explore] [Ask AI] [Profile] │
│  Icon    Icon      Icon     Icon  │
│  Text    Text      Text     Text  │
└─────────────────────────────────┘
```

**Specifications:**
- Height: 64px
- Background: White with top border
- Icons: 24px × 24px
- Text: 11px, medium weight
- Active state: Primary blue color
- Inactive state: Gray (#6B7280)
- Safe area padding for iOS notch

**Navigation Items:**
1. **Home:** House icon - Personalized feed
2. **Explore:** Compass icon - Full-screen browsing
3. **Ask AI:** Message circle icon - Chatbot
4. **Profile:** User icon - Settings & saved

---

### 5. Top Navigation Bar

**Layout:**
```
┌─────────────────────────────────┐
│ [Logo] ET Connect    [🔔] [⚙️]  │
└─────────────────────────────────┘
```

**Specifications:**
- Height: 56px
- Background: White (light mode) / #171717 (dark mode)
- Logo: 32px height
- Icons: 24px × 24px
- Padding: 16px horizontal
- Shadow: 0 1px 3px rgba(0,0,0,0.1)

**Elements:**
- Logo + App name (left)
- Notification bell (right, with badge for unread)
- Settings gear (right)

---

### 6. Landing Page Hero

**Layout:**
```
┌─────────────────────────────────┐
│ [Background Image with overlay]  │
│                                  │
│     ET Connect Logo              │
│                                  │
│   News you understand,           │
│   not just consume.              │
│                                  │
│   Transform headlines into       │
│   personal impact                │
│                                  │
│   [Start Your Journey]           │
│   [Learn More]                   │
│                                  │
└─────────────────────────────────┘
```

**Design Details:**
- Full viewport height (100vh)
- Background: Dark gradient with image overlay
- Text: White, centered
- Headline: 36px (mobile), 48px (desktop)
- Subheading: 18px (mobile), 24px (desktop)
- CTA buttons: 48px height, 16px padding
- Primary button: Blue with white text
- Secondary button: Transparent with white border

---

## User Flow Diagrams

### Flow 1: First-Time User Journey

```
Landing Page
    ↓
[Sign Up]
    ↓
Email/Password Entry
    ↓
Profile Survey
    ↓
Complete Profile (Age, Income, Goals, Interests)
    ↓
Home Feed (Personalized News)
    ↓
Tap Article with Impact Score 8/10
    ↓
Read Article
    ↓
Tap "View Impact" or Impact Badge
    ↓
Personal Impact Report (2 seconds)
    ↓
Read Impact: "EMI increases ₹2,800/month"
    ↓
Tap "Wanna Know More?"
    ↓
Chatbot with Context Pre-loaded
    ↓
AI Suggests: "Want to explore scenarios?"
    ↓
User Asks: "Should I wait to buy?"
    ↓
AI Responds with Scenario Analysis
    ↓
User Makes Informed Decision
```

### Flow 2: Returning User Journey

```
Login
    ↓
Home Feed (Updated News)
    ↓
High-Impact Article Highlighted (Score: 9/10)
    ↓
Tap Article
    ↓
View Impact Report
    ↓
Save Report for Later
    ↓
Continue Browsing
    ↓
Tap "Ask AI" in Bottom Nav
    ↓
Chat About Multiple Articles
    ↓
Get Personalized Recommendations
```

---

## Responsive Design Breakpoints

### Mobile (Primary)
- **320px - 428px:** Optimized for iPhone SE to iPhone 14 Pro Max
- Single column layout
- Bottom navigation
- Full-width cards
- Touch-optimized (48px minimum tap targets)

### Tablet
- **768px - 1024px:** iPad, Android tablets
- Two-column layout for news feed
- Side navigation (optional)
- Larger cards with more content visible

### Desktop
- **1280px+:** Desktop browsers
- Three-column layout
- Sidebar navigation
- Hover states for interactions
- Keyboard shortcuts support

---

## Accessibility (WCAG 2.1 Level AA)

### Color Contrast
- Text on background: Minimum 4.5:1 ratio
- Large text (18px+): Minimum 3:1 ratio
- Interactive elements: Clear focus indicators

### Keyboard Navigation
- Tab order follows visual flow
- Enter/Space to activate buttons
- Escape to close modals
- Arrow keys for carousel navigation

### Screen Reader Support
- Semantic HTML (header, nav, main, article)
- ARIA labels for icons
- Alt text for images
- Live regions for dynamic content

### Touch Targets
- Minimum 48px × 48px for all interactive elements
- Adequate spacing between targets (8px minimum)

---

## Animation & Transitions

### Principles
- **Purposeful:** Animations guide attention, don't distract
- **Fast:** 200-300ms for most transitions
- **Smooth:** 60fps, use transform and opacity
- **Respectful:** Honor prefers-reduced-motion

### Common Animations

**Page Transitions:**
```css
.page-enter {
  opacity: 0;
  transform: translateY(20px);
}
.page-enter-active {
  opacity: 1;
  transform: translateY(0);
  transition: all 300ms ease-out;
}
```

**Card Hover (Desktop):**
```css
.card {
  transition: transform 200ms ease, box-shadow 200ms ease;
}
.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 16px rgba(0,0,0,0.15);
}
```

**Impact Score Badge Pulse:**
```css
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.8; }
}
.impact-badge-high {
  animation: pulse 2s ease-in-out infinite;
}
```

**Chatbot Message Appear:**
```css
.message-enter {
  opacity: 0;
  transform: scale(0.95);
}
.message-enter-active {
  opacity: 1;
  transform: scale(1);
  transition: all 200ms ease-out;
}
```

---

## Loading States

### Skeleton Screens
- Use for news feed loading
- Gray placeholders matching card layout
- Subtle shimmer animation
- Replace with actual content when loaded

### Spinners
- Use for Impact Report generation (< 2 seconds)
- Primary blue color
- 32px size
- Centered in container

### Progress Indicators
- Use for multi-step processes (profile survey)
- Show current step and total steps
- Visual progress bar

---

## Error States

### Network Error
```
┌─────────────────────────────────┐
│     [Icon: Wifi Off]             │
│                                  │
│   Unable to load news            │
│   Check your connection          │
│                                  │
│   [Retry]                        │
└─────────────────────────────────┘
```

### No Results
```
┌─────────────────────────────────┐
│     [Icon: Search]               │
│                                  │
│   No news found                  │
│   Try adjusting your interests   │
│                                  │
│   [Update Profile]               │
└─────────────────────────────────┘
```

### API Error
```
┌─────────────────────────────────┐
│     [Icon: Alert Triangle]       │
│                                  │
│   Something went wrong           │
│   We're working on it            │
│                                  │
│   [Go Back]                      │
└─────────────────────────────────┘
```

---

## Design System Components

### Buttons

**Primary Button:**
```css
background: #3B82F6
color: white
padding: 12px 24px
border-radius: 8px
font-weight: 600
hover: background #2563EB
```

**Secondary Button:**
```css
background: transparent
color: #3B82F6
border: 2px solid #3B82F6
padding: 12px 24px
border-radius: 8px
font-weight: 600
hover: background #EFF6FF
```

**Text Button:**
```css
background: transparent
color: #3B82F6
padding: 8px 16px
font-weight: 600
hover: text-decoration underline
```

### Input Fields

**Text Input:**
```css
border: 1px solid #E5E7EB
border-radius: 8px
padding: 12px 16px
font-size: 16px
focus: border-color #3B82F6, outline 2px #3B82F6
```

**Dropdown:**
```css
border: 1px solid #E5E7EB
border-radius: 8px
padding: 12px 16px
background: white
icon: chevron-down
```

### Badges

**Impact Score Badge:**
```css
High (8-10): background #FEE2E2, color #991B1B
Medium (4-7): background #FEF3C7, color #92400E
Low (0-3): background #D1FAE5, color #065F46
padding: 4px 12px
border-radius: 12px
font-weight: 700
font-size: 12px
```

---

## Dark Mode Support

### Color Adjustments
```css
/* Light Mode (Default) */
--bg-primary: #FFFFFF
--bg-secondary: #F9FAFB
--text-primary: #111827
--text-secondary: #6B7280

/* Dark Mode */
--bg-primary: #0A0A0A
--bg-secondary: #171717
--text-primary: #F9FAFB
--text-secondary: #9CA3AF
```

### Implementation
- Use CSS custom properties
- Respect system preference (prefers-color-scheme)
- Toggle in settings
- Persist user choice in localStorage

---

## Performance Optimization

### Image Optimization
- WebP format with JPEG fallback
- Lazy loading for below-fold images
- Responsive images (srcset)
- Blur-up placeholder technique

### Code Splitting
- Route-based code splitting
- Lazy load chatbot component
- Defer non-critical JavaScript

### Caching Strategy
- Service Worker for offline support
- Cache news articles (24 hours)
- Cache user profile locally
- Invalidate on update

---

## Design Tokens (Tailwind Config)

```javascript
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: {
          DEFAULT: '#1E3A8A',
          light: '#3B82F6',
          dark: '#1E40AF',
        },
        negative: '#EF4444',
        warning: '#F59E0B',
        positive: '#10B981',
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
        serif: ['Merriweather', 'serif'],
        mono: ['JetBrains Mono', 'monospace'],
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
      },
      borderRadius: {
        '4xl': '2rem',
      },
      boxShadow: {
        'card': '0 2px 8px rgba(0,0,0,0.1)',
        'card-hover': '0 8px 16px rgba(0,0,0,0.15)',
      },
    },
  },
}
```

---

## Figma Design File Structure

### Pages
1. **Cover & Overview**
2. **Design System** (Colors, Typography, Components)
3. **Mobile Screens** (320px - 428px)
4. **Tablet Screens** (768px - 1024px)
5. **Desktop Screens** (1280px+)
6. **User Flows**
7. **Prototypes**

### Artboards (Mobile)
- Landing Page
- Sign Up / Login
- Profile Survey (Multi-step)
- Home Feed
- Article View
- Impact Report Modal
- Chatbot Interface
- Explore Full-Screen
- Profile Page
- Saved News
- Notifications
- Settings

---

## Development Handoff

### Assets Export
- Icons: SVG format
- Images: WebP + JPEG fallback
- Logos: SVG with PNG fallback
- Illustrations: SVG

### Specifications
- Spacing: 4px, 8px, 12px, 16px, 24px, 32px
- Border radius: 8px, 12px, 16px
- Shadows: Defined in design tokens
- Animations: Duration and easing specified

### Component Library
- Storybook for component documentation
- Props and variants documented
- Accessibility notes included

---

## Document Control

**Version:** 2.0  
**Last Updated:** January 25, 2026  
**Created By:** Kiro AI Assistant  
**Status:** Draft for Review  
**Tech Stack:** React.js + AWS Serverless Architecture  
**Related Documents:** requirements.md, matter.txt

---

## Appendix: AWS Infrastructure Design

### Frontend Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS Amplify Hosting                       │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Git Repository (GitHub/GitLab)                      │   │
│  │         ↓                                             │   │
│  │  Automatic Build & Deploy                            │   │
│  │         ↓                                             │   │
│  │  React Build (npm run build)                         │   │
│  │         ↓                                             │   │
│  │  Deploy to Amplify CDN                               │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                               │
│  Features:                                                    │
│  • Automatic HTTPS                                           │
│  • Custom Domain Support                                     │
│  • Global CDN (CloudFront)                                   │
│  • Atomic Deployments                                        │
│  • Rollback Support                                          │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### Backend Lambda Functions Design

**Function: PersonalizedNewsFeed**
```python
# Lambda Handler
def lambda_handler(event, context):
    # Extract user ID from JWT token
    user_id = extract_user_id(event['headers']['Authorization'])
    
    # Get user profile from DynamoDB
    user_profile = dynamodb.get_item(
        TableName='Users',
        Key={'userId': user_id}
    )
    
    # Query news articles from S3
    articles = s3.list_objects_v2(
        Bucket='et-connect-news',
        Prefix='articles/'
    )
    
    # Rank articles by relevance
    ranked_articles = rank_by_interests(
        articles, 
        user_profile['interests']
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps(ranked_articles)
    }
```

**Function: ImpactAnalysis**
```python
# Lambda Handler with Bedrock
def lambda_handler(event, context):
    article_id = event['pathParameters']['articleId']
    user_id = extract_user_id(event['headers']['Authorization'])
    
    # Get article and user profile
    article = get_article_from_s3(article_id)
    user_profile = get_user_profile(user_id)
    
    # Prepare prompt for Bedrock
    prompt = f"""
    Analyze the impact of this news article on the user:
    
    Article: {article['headline']}
    Content: {article['content']}
    
    User Profile:
    - Age: {user_profile['age']}
    - Income: {user_profile['income']}
    - Goals: {', '.join(user_profile['goals'])}
    
    Provide:
    1. Impact Score (0-10)
    2. Short-term impact (0-6 months)
    3. Long-term impact (6 months - 5 years)
    4. Specific action recommendations
    5. Confidence score
    """
    
    # Call Bedrock with RAG
    response = bedrock_runtime.invoke_model(
        modelId='anthropic.claude-3-sonnet-20240229-v1:0',
        body=json.dumps({
            'prompt': prompt,
            'max_tokens': 2000,
            'temperature': 0.7
        })
    )
    
    # Parse and structure response
    impact_report = parse_bedrock_response(response)
    
    # Save to DynamoDB
    save_impact_report(user_id, article_id, impact_report)
    
    return {
        'statusCode': 200,
        'body': json.dumps(impact_report)
    }
```

### DynamoDB Table Design

**Users Table Schema**
```
Table Name: et-connect-users
Partition Key: userId (String)

Attributes:
{
  "userId": "user-123",
  "email": "priya@example.com",
  "age": 26,
  "profession": "Software Engineer",
  "income": "₹1L-₹2L",
  "lifeStage": "Early Career",
  "goals": ["Buying House", "Investing"],
  "interests": ["Technology", "Finance", "Real Estate"],
  "commitments": {
    "savings": 800000,
    "plannedLoan": 4000000
  },
  "planningTimeline": "6-12 months",
  "createdAt": 1706169600000,
  "updatedAt": 1706169600000
}

Indexes:
- None (simple key-value lookups)

Capacity: On-Demand
```

**NewsArticles Table Schema**
```
Table Name: et-connect-news-metadata
Partition Key: articleId (String)
Sort Key: publishedDate (Number)

Attributes:
{
  "articleId": "article-123",
  "headline": "RBI Increases Repo Rate",
  "category": "Finance",
  "source": "Economic Times",
  "publishedDate": 1706169600000,
  "s3Key": "articles/2026/01/25/article-123.json",
  "entities": ["RBI", "Repo Rate"],
  "topics": ["Monetary Policy"],
  "imageUrl": "https://cdn.../image.jpg"
}

GSI: category-publishedDate-index
Partition Key: category
Sort Key: publishedDate
```

### S3 Bucket Structure

```
Bucket: et-connect-news-production

Structure:
├── articles/
│   ├── 2026/
│   │   ├── 01/
│   │   │   ├── 25/
│   │   │   │   ├── article-123.json
│   │   │   │   ├── article-124.json
│   │   │   │   └── article-125.json
│   │   │   └── 26/
│   │   └── 02/
│   └── index.json (latest articles)
│
├── images/
│   ├── articles/
│   │   ├── article-123-hero.jpg
│   │   └── article-124-hero.jpg
│   └── ui/
│       ├── logo.svg
│       └── placeholder.jpg
│
├── knowledge-base/
│   ├── financial-policies/
│   │   ├── rbi-policies.pdf
│   │   └── gst-regulations.pdf
│   ├── economic-data/
│   │   └── india-gdp-data.csv
│   └── historical-events/
│       └── past-rate-hikes.json
│
└── user-uploads/ (future)
    └── profile-pictures/

Lifecycle Policies:
- articles/: Move to Glacier after 90 days
- images/: Keep in Standard for 1 year
- knowledge-base/: Keep in Standard (frequently accessed)
```

### API Gateway Configuration

```yaml
API Name: et-connect-api
Protocol: REST
Stage: production

Endpoints:
  /auth:
    POST /signup:
      Integration: Lambda (UserManagement)
      Auth: None
      
    POST /login:
      Integration: Lambda (UserManagement)
      Auth: None
      
  /news:
    GET /personalized:
      Integration: Lambda (PersonalizedFeed)
      Auth: Cognito Authorizer
      Query Params: limit, offset
      
    GET /{articleId}:
      Integration: Lambda (ArticleRetrieval)
      Auth: Cognito Authorizer
      
  /impact:
    POST /analyze:
      Integration: Lambda (ImpactAnalysis)
      Auth: Cognito Authorizer
      Body: { articleId, userId }
      
  /chat:
    POST /message:
      Integration: Lambda (Chatbot)
      Auth: Cognito Authorizer
      Body: { sessionId, message, articleId }
      
  /user:
    GET /profile:
      Integration: Lambda (UserManagement)
      Auth: Cognito Authorizer
      
    PUT /profile:
      Integration: Lambda (UserManagement)
      Auth: Cognito Authorizer
      
    GET /saved-news:
      Integration: Lambda (SavedNews)
      Auth: Cognito Authorizer

Settings:
  - Throttling: 1000 requests/second
  - Burst: 2000 requests
  - API Key: Required for external access
  - CORS: Enabled for frontend domain
  - Logging: Full request/response logging
```

### Monitoring Dashboard Design

**CloudWatch Dashboard Layout**
```
┌─────────────────────────────────────────────────────────────┐
│                  ET Connect Monitoring                       │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  API Gateway Metrics                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │ Total       │  │ Error Rate  │  │ Latency     │        │
│  │ Requests    │  │ 0.5%        │  │ 250ms       │        │
│  │ 1.2M/day    │  │             │  │             │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                               │
│  Lambda Performance                                          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Function          │ Invocations │ Duration │ Errors │   │
│  │ PersonalizedFeed  │ 500K        │ 180ms    │ 0.2%   │   │
│  │ ImpactAnalysis    │ 100K        │ 1.8s     │ 0.5%   │   │
│  │ Chatbot           │ 50K         │ 2.5s     │ 0.3%   │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                               │
│  Bedrock Usage                                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │ Total       │  │ Avg Tokens  │  │ Cost/Day    │        │
│  │ Requests    │  │ 1,200       │  │ $45         │        │
│  │ 150K/day    │  │             │  │             │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                               │
│  DynamoDB Metrics                                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │ Read Units  │  │ Write Units │  │ Throttles   │        │
│  │ 2.5K/sec    │  │ 500/sec     │  │ 0           │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## Implementation Checklist

### Phase 1: Infrastructure Setup
- [ ] Create AWS account and configure IAM roles
- [ ] Set up AWS CDK project (Python/TypeScript)
- [ ] Create DynamoDB tables (Users, NewsArticles, etc.)
- [ ] Create S3 buckets with lifecycle policies
- [ ] Configure Cognito User Pool
- [ ] Set up API Gateway with endpoints
- [ ] Deploy initial Lambda functions
- [ ] Configure Bedrock access and Knowledge Bases
- [ ] Set up CloudWatch dashboards and alarms

### Phase 2: Frontend Development
- [ ] Initialize React project with Vite
- [ ] Set up Tailwind CSS configuration
- [ ] Implement authentication flow (Cognito)
- [ ] Build component library (buttons, cards, etc.)
- [ ] Implement news feed page
- [ ] Implement impact report modal
- [ ] Implement chatbot interface
- [ ] Add responsive design for mobile/tablet/desktop
- [ ] Deploy to AWS Amplify

### Phase 3: Backend Development
- [ ] Implement news aggregation Lambda
- [ ] Implement personalized feed Lambda
- [ ] Implement impact analysis Lambda with Bedrock
- [ ] Implement chatbot Lambda with RAG
- [ ] Implement user management Lambda
- [ ] Set up EventBridge for scheduled news fetching
- [ ] Configure API Gateway throttling and caching
- [ ] Implement error handling and logging

### Phase 4: Testing & Optimization
- [ ] Unit tests for Lambda functions
- [ ] Integration tests for API endpoints
- [ ] Load testing with Artillery/JMeter
- [ ] Security testing (penetration testing)
- [ ] Performance optimization (caching, CDN)
- [ ] Cost optimization (reserved capacity, caching)
- [ ] User acceptance testing (UAT)

### Phase 5: Launch
- [ ] Production deployment
- [ ] DNS configuration
- [ ] SSL certificate setup
- [ ] Monitoring and alerting
- [ ] Documentation (API docs, user guide)
- [ ] Marketing and user onboarding
