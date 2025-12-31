# Daily Vocab Vision — Product Requirements Document

**Version:** 2.0  
**Date:** December 30, 2025  
**Audience:** Engineering Team  
**Status:** Ready for Implementation

---

## 1. Product Overview

### 1.1 One-Liner

A daily Spanish vocabulary quiz that rewards correct guesses with beautifully illustrated mnemonics—each image earned is a trophy in your growing collection.

### 1.2 Core Loop

1. User receives push notification at their chosen time
2. User opens app, sees English translation + heavily blurred mnemonic image
3. User types Spanish word guess
4. Wrong? Image sharpens, hints progressively revealed, try again
5. Correct? Full reveal animation, frame awarded based on attempts
6. Word + image + frame saved to collection
7. Streak increments
8. Share trophy (optional)
9. Repeat tomorrow

### 1.3 Product Principles

- **Earn the image.** The mnemonic is a trophy, not content.
- **One word, everyone, daily.** The shared challenge is the product.
- **Craft over scale.** 365 handcrafted entries beat 10,000 generated ones.
- **Collection as pride.** Frames tell your story—no shame, just journey.
- **Respect attention.** One notification. No guilt mechanics. No dark patterns.

---

## 2. Quiz Mechanic: "Reveal"

### 2.1 Attempt Structure

| Attempt | Image Blur | Hints Shown | Frame Earned |
|---------|------------|-------------|--------------|
| 1st try correct | 85% blur → full reveal | English only | 🥇 Gold |
| 2nd try correct | 60% blur → full reveal | + Mnemonic sentence 1 | 🥈 Silver |
| 3rd try correct | 35% blur → full reveal | + Full mnemonic | 🥉 Bronze |
| 4th try correct | 0% blur (full image) | + Pronunciation | Frameless |
| 5th+ correct | 0% blur | All hints | Frameless |
| Give up (after 2+) | 0% blur | All hints | Frameless |

### 2.2 Blur Implementation

```
Blur Level Calculation:
- Attempt 0 (initial): blur(20px)
- Attempt 1 wrong: blur(14px)  
- Attempt 2 wrong: blur(8px)
- Attempt 3 wrong: blur(0px)
- Correct at any point: animate blur(current) → blur(0px) over 600ms
```

Use CSS/RN `blurRadius` or shader-based blur. Test on low-end Android for performance.

### 2.3 Hint Progression

| State | User Sees |
|-------|-----------|
| Initial | English translation |
| After 1st wrong | English + first sentence of mnemonic |
| After 2nd wrong | English + full mnemonic + [Give Up] button appears |
| After 3rd wrong | English + full mnemonic + pronunciation (IPA + audio) |

Hints accumulate—they don't replace. Each wrong answer adds information.

### 2.4 Input Validation

**Forgiving matching rules:**
- Case insensitive: "Madrugada" = "madrugada" ✓
- Accent insensitive: "madrugada" = "madrúgada" ✓
- Trim whitespace: " madrugada " = "madrugada" ✓
- No partial matching: "madru" ≠ "madrugada" ✗

**Normalization function:**
```typescript
function normalizeAnswer(input: string): string {
  return input
    .toLowerCase()
    .trim()
    .normalize("NFD")
    .replace(/[\u0300-\u036f]/g, ""); // Strip diacritics
}

function checkAnswer(userInput: string, correctWord: string): boolean {
  return normalizeAnswer(userInput) === normalizeAnswer(correctWord);
}
```

### 2.5 Give Up Flow

- Button appears after attempt 2
- Tap → confirmation: "Give up? You'll still learn the word."
- Confirm → full reveal, frameless, word saved to collection
- No penalty beyond missing the frame

### 2.6 Already Completed Today

If user returns after completing today's quiz:
- Show completed word detail view (not quiz)
- Display earned frame prominently
- No re-quiz option (one chance per day, like Wordle)

---

## 3. Frame System

### 3.1 Frame Types

| Frame | Visual Treatment | Earned When |
|-------|------------------|-------------|
| 🥇 Gold | Thick gold border, subtle shimmer animation | 1st attempt correct |
| 🥈 Silver | Medium silver border, subtle shine | 2nd attempt correct |
| 🥉 Bronze | Thin bronze border | 3rd attempt correct |
| None | No border treatment | 4th+ attempt or give up |

### 3.2 Frame Rendering

**In collection grid:**
```
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│ ╔═════════╗ │   │ ┌─────────┐ │   │             │
│ ║         ║ │   │ │         │ │   │    img      │
│ ║   img   ║ │   │ │   img   │ │   │             │
│ ║         ║ │   │ │         │ │   │             │
│ ╚═════════╝ │   │ └─────────┘ │   │             │
└─────────────┘   └─────────────┘   └─────────────┘
    Gold              Silver           Frameless
```

**Frame specifications:**
- Gold: 4px border, `#D4AF37`, outer glow `rgba(212,175,55,0.3)`
- Silver: 3px border, `#C0C0C0`, subtle gradient
- Bronze: 2px border, `#CD7F32`
- Frameless: 0px border, standard image display

### 3.3 Frame in Detail View

When viewing a word from collection, frame is visible around the image. Small badge in corner shows:
- 🥇 "First try"
- 🥈 "Second try"  
- 🥉 "Third try"
- (nothing for frameless)

### 3.4 Frame in Share Card

Shared images include the earned frame:

```
┌─────────────────────────────────────┐
│ ╔═════════════════════════════════╗ │
│ ║                                 ║ │
│ ║        [Mnemonic Image]         ║ │
│ ║                                 ║ │
│ ╚═════════════════════════════════╝ │
├─────────────────────────────────────┤
│                                     │
│  madrugada                    🥇    │
│  dawn, early morning                │
│                                     │
│                    [App Logo]       │
└─────────────────────────────────────┘
```

Gold frame visible. Badge shows "🥇" indicating first-try success. Bragging rights encoded.

---

## 4. User Flows

### 4.1 First Launch (Onboarding)

```
[Splash Screen]
    │
    ▼
[Welcome Screen]
    "Learn Spanish, one word at a time"
    Visual: Example gold-framed image
    [Continue]
    │
    ▼
[How It Works]
    "Guess the Spanish word"
    "Earn gold, silver, or bronze"
    "Build your collection"
    Visual: Three frames shown
    [Continue]
    │
    ▼
[Notification Permission]
    "When should we send your daily challenge?"
    [Time Picker - default 8:00 AM]
    [Allow Notifications]
    │
    ▼
[Auth Screen]
    "Sign in to save your trophies"
    [Continue with Apple]
    [Continue with Google]
    [Continue with Email]
    │
    ▼
[First Quiz]
    Today's word challenge
    │
    ▼
[Home Screen]
```

### 4.2 Daily Quiz Flow (Returning User)

```
[Push Notification]
    "Tu palabra del día está lista"
    │
    ▼
[Home Screen]
    │
    ├── Quiz not yet attempted today
    │       │
    │       ▼
    │   [Quiz Screen - Initial State]
    │       - Heavily blurred image (85%)
    │       - English translation displayed
    │       - Text input field
    │       - [Submit] button (disabled until input)
    │       │
    │       ├── User submits CORRECT answer
    │       │       │
    │       │       ▼
    │       │   [Victory Animation]
    │       │       - Blur animates to 0 (600ms)
    │       │       - Frame materializes around image
    │       │       - Haptic: success pattern
    │       │       - Frame badge animates in
    │       │       │
    │       │       ▼
    │       │   [Word Detail View]
    │       │       - Full image with earned frame
    │       │       - Spanish word + pronunciation
    │       │       - English translation
    │       │       - Full mnemonic explanation
    │       │       - Example sentence
    │       │       - [Share Trophy]
    │       │       - Streak updated inline
    │       │
    │       └── User submits WRONG answer
    │               │
    │               ▼
    │           [Quiz Screen - Next Attempt]
    │               - Image sharpens one level
    │               - New hint revealed
    │               - Input cleared, ready for retry
    │               - Subtle shake animation on input
    │               - Haptic: light error tap
    │               - [Give Up] appears after attempt 2
    │               │
    │               └── (Loop until correct or give up)
    │
    └── Quiz already completed today
            │
            ▼
        [Word Detail View]
            - Full image with earned frame
            - "Completed" state
            - Cannot re-attempt
```

### 4.3 Quiz Screen States

**State Machine:**

```
┌─────────────┐
│   INITIAL   │ ← App opened, quiz not started
│ blur: 85%   │
│ hints: EN   │
└──────┬──────┘
       │ submit wrong
       ▼
┌─────────────┐
│  ATTEMPT_1  │
│ blur: 60%   │
│ hints: +M1  │
└──────┬──────┘
       │ submit wrong
       ▼
┌─────────────┐
│  ATTEMPT_2  │ ← [Give Up] button appears
│ blur: 35%   │
│ hints: +MF  │
└──────┬──────┘
       │ submit wrong
       ▼
┌─────────────┐
│  ATTEMPT_3  │
│ blur: 0%    │
│ hints: +PR  │
└──────┬──────┘
       │ submit wrong
       ▼
┌─────────────┐
│ ATTEMPT_4+  │ ← Can keep trying, no more blur/hints
│ blur: 0%    │
│ hints: ALL  │
└─────────────┘

Any state + correct answer → VICTORY
Any state (≥2) + give up → COMPLETE (frameless)
```

**Hint Legend:**
- EN = English translation
- M1 = Mnemonic sentence 1
- MF = Mnemonic full
- PR = Pronunciation

### 4.4 Collection Gallery

```
[Tab: Collection]
    │
    ▼
[Gallery Grid View]
    - 3-column grid
    - Chronological, newest first
    - Each cell: image with frame (if earned)
    - Tap to open full view
    │
    ├── [Filter: All] (default)
    │
    ├── [Filter: Favorites]
    │       Heart icon in nav
    │
    ├── [Filter: 🥇 Gold Only]
    │       Trophy filter option
    │
    └── [Search]
            Search by Spanish/English
```

### 4.5 Profile / Stats

```
[Tab: Profile]
    │
    ├── User avatar + name
    │
    ├── Streak Display
    │       "47 day streak 🔥"
    │
    ├── Trophy Stats
    │       ┌─────┬─────┬─────┬─────┐
    │       │ 🥇  │ 🥈  │ 🥉  │  ○  │
    │       │ 23  │ 15  │  7  │  2  │
    │       └─────┴─────┴─────┴─────┘
    │       Gold  Silver Bronze Bare
    │
    ├── Total Words: 47
    │
    ├── Settings
    │       - Notification time
    │       - Sign out
    │
    └── Footer links
```

---

## 5. Data Models (Convex Schema)

### 5.1 Tables

```typescript
// schema.ts

import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  // Content table - loaded by editorial team
  words: defineTable({
    slug: v.string(),
    scheduledDate: v.string(), // YYYY-MM-DD
    
    // Spanish content
    spanishWord: v.string(),
    pronunciation: v.string(), // IPA
    
    // English content
    englishTranslation: v.string(),
    
    // Mnemonic content (split for progressive reveal)
    mnemonicSentenceOne: v.string(),
    mnemonicFull: v.string(),
    
    // Example usage
    exampleSentenceSpanish: v.string(),
    exampleSentenceEnglish: v.string(),
    
    // Media
    imageUrl: v.string(),
    audioUrl: v.optional(v.string()),
    
    // For answer validation
    normalizedAnswer: v.string(), // Pre-computed normalized form
    
    // Metadata
    createdAt: v.number(),
    updatedAt: v.number(),
  })
    .index("by_scheduled_date", ["scheduledDate"])
    .index("by_slug", ["slug"]),

  // User profile and preferences
  users: defineTable({
    clerkId: v.string(),
    
    displayName: v.string(),
    avatarUrl: v.optional(v.string()),
    
    notificationTime: v.string(), // HH:MM
    timezone: v.string(),
    
    // Streak
    currentStreak: v.number(),
    longestStreak: v.number(),
    lastEngagementDate: v.optional(v.string()),
    
    // Aggregate stats (denormalized for fast reads)
    totalWords: v.number(),
    goldCount: v.number(),
    silverCount: v.number(),
    bronzeCount: v.number(),
    
    createdAt: v.number(),
    updatedAt: v.number(),
  })
    .index("by_clerk_id", ["clerkId"]),

  // User's word collection with quiz results
  userWords: defineTable({
    userId: v.id("users"),
    wordId: v.id("words"),
    
    // Quiz result
    attemptsUsed: v.number(), // 1-N, or 0 if gave up
    frame: v.union(
      v.literal("gold"),
      v.literal("silver"),
      v.literal("bronze"),
      v.literal("none")
    ),
    gaveUp: v.boolean(),
    
    // Engagement
    completedAt: v.number(),
    completedDate: v.string(), // YYYY-MM-DD local
    isFavorite: v.boolean(),
  })
    .index("by_user", ["userId"])
    .index("by_user_and_word", ["userId", "wordId"])
    .index("by_user_and_date", ["userId", "completedDate"])
    .index("by_user_and_frame", ["userId", "frame"]),

  // Push tokens
  pushTokens: defineTable({
    userId: v.id("users"),
    token: v.string(),
    platform: v.union(v.literal("ios"), v.literal("android")),
    createdAt: v.number(),
  })
    .index("by_user", ["userId"])
    .index("by_token", ["token"]),

  // Quiz sessions (tracks in-progress attempts)
  quizSessions: defineTable({
    userId: v.id("users"),
    wordId: v.id("words"),
    date: v.string(), // YYYY-MM-DD
    currentAttempt: v.number(), // 0 = not started, 1+ = attempts made
    completed: v.boolean(),
    createdAt: v.number(),
    updatedAt: v.number(),
  })
    .index("by_user_and_date", ["userId", "date"]),
});
```

### 5.2 Frame Determination Logic

```typescript
function determineFrame(attemptsUsed: number, gaveUp: boolean): Frame {
  if (gaveUp) return "none";
  if (attemptsUsed === 1) return "gold";
  if (attemptsUsed === 2) return "silver";
  if (attemptsUsed === 3) return "bronze";
  return "none";
}
```

---

## 6. Convex Functions (API Contracts)

### 6.1 Queries

```typescript
// queries/getTodaysQuiz.ts
export const getTodaysQuiz = query({
  args: { timezone: v.string() },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) return null;
    
    const user = await getUserByClerkId(ctx, identity.subject);
    if (!user) return null;
    
    const today = getLocalDate(args.timezone);
    
    // Get today's word
    const word = await ctx.db
      .query("words")
      .withIndex("by_scheduled_date", (q) => q.eq("scheduledDate", today))
      .unique();
    
    if (!word) return null;
    
    // Check if already completed
    const userWord = await ctx.db
      .query("userWords")
      .withIndex("by_user_and_word", (q) =>
        q.eq("userId", user._id).eq("wordId", word._id)
      )
      .unique();
    
    // Get or create quiz session
    let session = await ctx.db
      .query("quizSessions")
      .withIndex("by_user_and_date", (q) =>
        q.eq("userId", user._id).eq("date", today)
      )
      .unique();
    
    if (userWord) {
      // Already completed - return full word with result
      return {
        status: "completed",
        word: {
          spanishWord: word.spanishWord,
          pronunciation: word.pronunciation,
          englishTranslation: word.englishTranslation,
          mnemonicFull: word.mnemonicFull,
          exampleSentenceSpanish: word.exampleSentenceSpanish,
          exampleSentenceEnglish: word.exampleSentenceEnglish,
          imageUrl: word.imageUrl,
          audioUrl: word.audioUrl,
        },
        result: {
          frame: userWord.frame,
          attemptsUsed: userWord.attemptsUsed,
          gaveUp: userWord.gaveUp,
        },
      };
    }
    
    // Quiz in progress - return appropriate hints based on attempts
    const attempts = session?.currentAttempt ?? 0;
    
    return {
      status: "in_progress",
      wordId: word._id,
      attempts,
      quiz: {
        englishTranslation: word.englishTranslation,
        imageUrl: word.imageUrl,
        // Progressive hints
        mnemonicSentenceOne: attempts >= 1 ? word.mnemonicSentenceOne : null,
        mnemonicFull: attempts >= 2 ? word.mnemonicFull : null,
        pronunciation: attempts >= 3 ? word.pronunciation : null,
        audioUrl: attempts >= 3 ? word.audioUrl : null,
      },
    };
  },
});

// queries/getUserProfile.ts
export const getUserProfile = query({
  args: {},
  handler: async (ctx) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) return null;
    
    const user = await ctx.db
      .query("users")
      .withIndex("by_clerk_id", (q) => q.eq("clerkId", identity.subject))
      .unique();
    
    if (!user) return null;
    
    return {
      displayName: user.displayName,
      avatarUrl: user.avatarUrl,
      currentStreak: user.currentStreak,
      longestStreak: user.longestStreak,
      totalWords: user.totalWords,
      goldCount: user.goldCount,
      silverCount: user.silverCount,
      bronzeCount: user.bronzeCount,
      notificationTime: user.notificationTime,
      createdAt: user.createdAt,
    };
  },
});

// queries/getUserCollection.ts
export const getUserCollection = query({
  args: {
    favoritesOnly: v.optional(v.boolean()),
    frameFilter: v.optional(v.union(
      v.literal("gold"),
      v.literal("silver"),
      v.literal("bronze"),
      v.literal("none")
    )),
    searchQuery: v.optional(v.string()),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) return [];
    
    const user = await getUserByClerkId(ctx, identity.subject);
    if (!user) return [];
    
    let userWords = await ctx.db
      .query("userWords")
      .withIndex("by_user", (q) => q.eq("userId", user._id))
      .collect();
    
    if (args.favoritesOnly) {
      userWords = userWords.filter((uw) => uw.isFavorite);
    }
    
    if (args.frameFilter) {
      userWords = userWords.filter((uw) => uw.frame === args.frameFilter);
    }
    
    const wordsWithDetails = await Promise.all(
      userWords.map(async (uw) => {
        const word = await ctx.db.get(uw.wordId);
        return {
          id: uw._id,
          wordId: uw.wordId,
          spanishWord: word?.spanishWord,
          englishTranslation: word?.englishTranslation,
          imageUrl: word?.imageUrl,
          frame: uw.frame,
          isFavorite: uw.isFavorite,
          completedAt: uw.completedAt,
        };
      })
    );
    
    // Apply search filter
    if (args.searchQuery) {
      const query = args.searchQuery.toLowerCase();
      return wordsWithDetails.filter(
        (w) =>
          w.spanishWord?.toLowerCase().includes(query) ||
          w.englishTranslation?.toLowerCase().includes(query)
      );
    }
    
    // Sort by completedAt descending
    return wordsWithDetails.sort((a, b) => b.completedAt - a.completedAt);
  },
});
```

### 6.2 Mutations

```typescript
// mutations/submitAnswer.ts
export const submitAnswer = mutation({
  args: {
    wordId: v.id("words"),
    answer: v.string(),
    timezone: v.string(),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Unauthorized");
    
    const user = await getUserByClerkId(ctx, identity.subject);
    if (!user) throw new Error("User not found");
    
    const word = await ctx.db.get(args.wordId);
    if (!word) throw new Error("Word not found");
    
    const today = getLocalDate(args.timezone);
    
    // Check if already completed
    const existingCompletion = await ctx.db
      .query("userWords")
      .withIndex("by_user_and_word", (q) =>
        q.eq("userId", user._id).eq("wordId", args.wordId)
      )
      .unique();
    
    if (existingCompletion) {
      return { status: "already_completed", result: existingCompletion };
    }
    
    // Get or create session
    let session = await ctx.db
      .query("quizSessions")
      .withIndex("by_user_and_date", (q) =>
        q.eq("userId", user._id).eq("date", today)
      )
      .unique();
    
    if (!session) {
      const sessionId = await ctx.db.insert("quizSessions", {
        userId: user._id,
        wordId: args.wordId,
        date: today,
        currentAttempt: 0,
        completed: false,
        createdAt: Date.now(),
        updatedAt: Date.now(),
      });
      session = await ctx.db.get(sessionId);
    }
    
    // Check answer
    const isCorrect = checkAnswer(args.answer, word.spanishWord);
    const newAttemptCount = session.currentAttempt + 1;
    
    if (isCorrect) {
      // Determine frame
      const frame = determineFrame(newAttemptCount, false);
      
      // Create userWord entry
      await ctx.db.insert("userWords", {
        userId: user._id,
        wordId: args.wordId,
        attemptsUsed: newAttemptCount,
        frame,
        gaveUp: false,
        completedAt: Date.now(),
        completedDate: today,
        isFavorite: false,
      });
      
      // Update session
      await ctx.db.patch(session._id, {
        currentAttempt: newAttemptCount,
        completed: true,
        updatedAt: Date.now(),
      });
      
      // Update user stats
      const newStreak = calculateNewStreak(user, today);
      await ctx.db.patch(user._id, {
        currentStreak: newStreak,
        longestStreak: Math.max(user.longestStreak, newStreak),
        lastEngagementDate: today,
        totalWords: user.totalWords + 1,
        goldCount: user.goldCount + (frame === "gold" ? 1 : 0),
        silverCount: user.silverCount + (frame === "silver" ? 1 : 0),
        bronzeCount: user.bronzeCount + (frame === "bronze" ? 1 : 0),
        updatedAt: Date.now(),
      });
      
      return {
        status: "correct",
        frame,
        attemptsUsed: newAttemptCount,
        newStreak,
        word: {
          spanishWord: word.spanishWord,
          pronunciation: word.pronunciation,
          mnemonicFull: word.mnemonicFull,
          exampleSentenceSpanish: word.exampleSentenceSpanish,
          exampleSentenceEnglish: word.exampleSentenceEnglish,
        },
      };
    } else {
      // Wrong answer - update session
      await ctx.db.patch(session._id, {
        currentAttempt: newAttemptCount,
        updatedAt: Date.now(),
      });
      
      // Return next hints based on attempt count
      return {
        status: "incorrect",
        attempts: newAttemptCount,
        hints: {
          mnemonicSentenceOne: newAttemptCount >= 1 ? word.mnemonicSentenceOne : null,
          mnemonicFull: newAttemptCount >= 2 ? word.mnemonicFull : null,
          pronunciation: newAttemptCount >= 3 ? word.pronunciation : null,
          audioUrl: newAttemptCount >= 3 ? word.audioUrl : null,
        },
        canGiveUp: newAttemptCount >= 2,
      };
    }
  },
});

// mutations/giveUp.ts
export const giveUp = mutation({
  args: {
    wordId: v.id("words"),
    timezone: v.string(),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Unauthorized");
    
    const user = await getUserByClerkId(ctx, identity.subject);
    if (!user) throw new Error("User not found");
    
    const word = await ctx.db.get(args.wordId);
    if (!word) throw new Error("Word not found");
    
    const today = getLocalDate(args.timezone);
    
    // Get session to check attempt count
    const session = await ctx.db
      .query("quizSessions")
      .withIndex("by_user_and_date", (q) =>
        q.eq("userId", user._id).eq("date", today)
      )
      .unique();
    
    if (!session || session.currentAttempt < 2) {
      throw new Error("Cannot give up before attempt 2");
    }
    
    // Create userWord entry with gaveUp = true
    await ctx.db.insert("userWords", {
      userId: user._id,
      wordId: args.wordId,
      attemptsUsed: 0, // 0 indicates gave up
      frame: "none",
      gaveUp: true,
      completedAt: Date.now(),
      completedDate: today,
      isFavorite: false,
    });
    
    // Update session
    await ctx.db.patch(session._id, {
      completed: true,
      updatedAt: Date.now(),
    });
    
    // Update user stats (streak still increments - they engaged)
    const newStreak = calculateNewStreak(user, today);
    await ctx.db.patch(user._id, {
      currentStreak: newStreak,
      longestStreak: Math.max(user.longestStreak, newStreak),
      lastEngagementDate: today,
      totalWords: user.totalWords + 1,
      updatedAt: Date.now(),
    });
    
    return {
      status: "gave_up",
      newStreak,
      word: {
        spanishWord: word.spanishWord,
        pronunciation: word.pronunciation,
        mnemonicFull: word.mnemonicFull,
        exampleSentenceSpanish: word.exampleSentenceSpanish,
        exampleSentenceEnglish: word.exampleSentenceEnglish,
        audioUrl: word.audioUrl,
      },
    };
  },
});

// mutations/toggleFavorite.ts
export const toggleFavorite = mutation({
  args: { userWordId: v.id("userWords") },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Unauthorized");
    
    const userWord = await ctx.db.get(args.userWordId);
    if (!userWord) throw new Error("Word not found in collection");
    
    await ctx.db.patch(args.userWordId, {
      isFavorite: !userWord.isFavorite,
    });
    
    return { isFavorite: !userWord.isFavorite };
  },
});

// mutations/createUser.ts
export const createUser = mutation({
  args: {
    notificationTime: v.string(),
    timezone: v.string(),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Unauthorized");
    
    const existing = await ctx.db
      .query("users")
      .withIndex("by_clerk_id", (q) => q.eq("clerkId", identity.subject))
      .unique();
    
    if (existing) return existing;
    
    const userId = await ctx.db.insert("users", {
      clerkId: identity.subject,
      displayName: identity.name || "Learner",
      avatarUrl: identity.pictureUrl,
      notificationTime: args.notificationTime,
      timezone: args.timezone,
      currentStreak: 0,
      longestStreak: 0,
      lastEngagementDate: undefined,
      totalWords: 0,
      goldCount: 0,
      silverCount: 0,
      bronzeCount: 0,
      createdAt: Date.now(),
      updatedAt: Date.now(),
    });
    
    return await ctx.db.get(userId);
  },
});

// mutations/updateNotificationTime.ts
export const updateNotificationTime = mutation({
  args: {
    notificationTime: v.string(),
    timezone: v.string(),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Unauthorized");
    
    const user = await getUserByClerkId(ctx, identity.subject);
    if (!user) throw new Error("User not found");
    
    await ctx.db.patch(user._id, {
      notificationTime: args.notificationTime,
      timezone: args.timezone,
      updatedAt: Date.now(),
    });
    
    return { success: true };
  },
});

// mutations/registerPushToken.ts
export const registerPushToken = mutation({
  args: {
    token: v.string(),
    platform: v.union(v.literal("ios"), v.literal("android")),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Unauthorized");
    
    const user = await getUserByClerkId(ctx, identity.subject);
    if (!user) throw new Error("User not found");
    
    const existing = await ctx.db
      .query("pushTokens")
      .withIndex("by_token", (q) => q.eq("token", args.token))
      .unique();
    
    if (existing) {
      if (existing.userId !== user._id) {
        await ctx.db.patch(existing._id, { userId: user._id });
      }
      return { success: true };
    }
    
    await ctx.db.insert("pushTokens", {
      userId: user._id,
      token: args.token,
      platform: args.platform,
      createdAt: Date.now(),
    });
    
    return { success: true };
  },
});
```

### 6.3 Helper Functions

```typescript
// lib/helpers.ts

export function getLocalDate(timezone: string): string {
  const now = new Date();
  const formatter = new Intl.DateTimeFormat("en-CA", {
    timeZone: timezone,
    year: "numeric",
    month: "2-digit",
    day: "2-digit",
  });
  return formatter.format(now);
}

export function calculateNewStreak(
  user: { currentStreak: number; lastEngagementDate?: string },
  today: string
): number {
  if (!user.lastEngagementDate) return 1;
  
  const lastDate = new Date(user.lastEngagementDate);
  const todayDate = new Date(today);
  const diffDays = Math.floor(
    (todayDate.getTime() - lastDate.getTime()) / (1000 * 60 * 60 * 24)
  );
  
  if (diffDays === 0) return user.currentStreak;
  if (diffDays === 1) return user.currentStreak + 1;
  return 1;
}

export function normalizeAnswer(input: string): string {
  return input
    .toLowerCase()
    .trim()
    .normalize("NFD")
    .replace(/[\u0300-\u036f]/g, "");
}

export function checkAnswer(userInput: string, correctWord: string): boolean {
  return normalizeAnswer(userInput) === normalizeAnswer(correctWord);
}

type Frame = "gold" | "silver" | "bronze" | "none";

export function determineFrame(attemptsUsed: number, gaveUp: boolean): Frame {
  if (gaveUp) return "none";
  if (attemptsUsed === 1) return "gold";
  if (attemptsUsed === 2) return "silver";
  if (attemptsUsed === 3) return "bronze";
  return "none";
}
```

---

## 7. Screen Specifications

### 7.1 Quiz Screen

**Layout - Initial State:**

```
┌─────────────────────────────────────┐
│ [Status Bar]                        │
├─────────────────────────────────────┤
│                                     │
│                                     │
│    ┌───────────────────────────┐    │
│    │                           │    │
│    │   [Blurred Image 85%]     │    │
│    │                           │    │
│    │                           │    │
│    └───────────────────────────┘    │
│                                     │
│    What's the Spanish word for:     │
│                                     │
│    "dawn, early morning"            │
│                                     │
│    ┌───────────────────────────┐    │
│    │ Type your answer...       │    │
│    └───────────────────────────┘    │
│                                     │
│    [    Submit    ]                 │
│                                     │
├─────────────────────────────────────┤
│ [Home]     [Collection]   [Profile] │
└─────────────────────────────────────┘
```

**Layout - After Wrong Attempts:**

```
┌─────────────────────────────────────┐
│ [Status Bar]                        │
├─────────────────────────────────────┤
│                                     │
│    ┌───────────────────────────┐    │
│    │                           │    │
│    │   [Blurred Image 35%]     │    │
│    │                           │    │
│    └───────────────────────────┘    │
│                                     │
│    "dawn, early morning"            │
│                                     │
│    ─────────────────────────────    │
│    💡 Hint                          │
│    Imagine a MADRE (mother)         │
│    DRAGging the sun up over         │
│    the horizon in the early         │
│    morning...                       │
│    ─────────────────────────────    │
│                                     │
│    ┌───────────────────────────┐    │
│    │                           │    │
│    └───────────────────────────┘    │
│                                     │
│    [Submit]          [Give Up]      │
│                                     │
├─────────────────────────────────────┤
│ [Home]     [Collection]   [Profile] │
└─────────────────────────────────────┘
```

### 7.2 Victory Screen

**Animation Sequence:**

1. Input field fades out (200ms)
2. Blur animates from current level to 0 (600ms, ease-out)
3. Frame materializes around image (400ms, scale 0.9→1.0, starts at 400ms)
4. Haptic: success pattern (medium impact + light at 200ms offset)
5. Frame badge pulses in (300ms, starts at 800ms)
6. "Correct!" text animates up (200ms, starts at 600ms)
7. Word details fade in below (300ms, starts at 1000ms)

**Layout - Victory:**

```
┌─────────────────────────────────────┐
│ [Status Bar]                        │
├─────────────────────────────────────┤
│                                     │
│    ╔═══════════════════════════╗    │
│    ║                           ║    │
│    ║   [Full Image + Frame]    ║    │
│    ║                           ║    │
│    ║                       🥇  ║    │
│    ╚═══════════════════════════╝    │
│                                     │
│           ¡Correcto!                │
│                                     │
│    madrugada                        │
│    /ma.dɾu.ˈɣa.da/        [🔊]     │
│    dawn, early morning              │
│                                     │
│    ─────────────────────────────    │
│    Imagine a MADRE (mother)...      │
│    ─────────────────────────────    │
│                                     │
│    "Salí de casa en la madrugada."  │
│    I left the house at dawn.        │
│                                     │
│    [  Share Trophy  ]               │
│                                     │
│         47 day streak 🔥            │
│                                     │
├─────────────────────────────────────┤
│ [Home]     [Collection]   [Profile] │
└─────────────────────────────────────┘
```

### 7.3 Collection Screen

```
┌─────────────────────────────────────┐
│ [Status Bar]                        │
├─────────────────────────────────────┤
│  Collection              [🔍] [♡]   │
│                                     │
│  [All] [🥇] [🥈] [🥉]               │
│                                     │
├─────────────────────────────────────┤
│                                     │
│  ╔═════╗ ┌─────┐ ╔═════╗           │
│  ║     ║ │     │ ║     ║           │
│  ║ img ║ │ img │ ║ img ║           │
│  ║  🥇 ║ │  🥈 │ ║  🥇 ║           │
│  ╚═════╝ └─────┘ ╚═════╝           │
│  palabra palabra palabra            │
│                                     │
│  ┌─────┐         ┌─────┐           │
│  │     │   img   │     │           │
│  │ img │         │ img │           │
│  │  🥉 │         │     │           │
│  └─────┘         └─────┘           │
│  palabra palabra palabra            │
│                                     │
├─────────────────────────────────────┤
│ [Home]     [Collection]   [Profile] │
└─────────────────────────────────────┘
```

**Filter chips:**
- [All] - default, shows everything
- [🥇] - gold only
- [🥈] - silver only  
- [🥉] - bronze only
- Frameless words visible in "All" but not filterable (no chip for "none")

### 7.4 Profile Screen

```
┌─────────────────────────────────────┐
│ [Status Bar]                        │
├─────────────────────────────────────┤
│                                     │
│         ┌───────────┐               │
│         │  Avatar   │               │
│         └───────────┘               │
│          User Name                  │
│                                     │
├─────────────────────────────────────┤
│                                     │
│              47 🔥                   │
│           day streak                │
│                                     │
├─────────────────────────────────────┤
│                                     │
│    ┌──────┬──────┬──────┬──────┐   │
│    │  🥇  │  🥈  │  🥉  │  ○   │   │
│    │  23  │  15  │   7  │  2   │   │
│    └──────┴──────┴──────┴──────┘   │
│                                     │
│         47 words learned            │
│                                     │
├─────────────────────────────────────┤
│                                     │
│  Notification Time                  │
│  8:00 AM                      [>]   │
│                                     │
│  ─────────────────────────          │
│                                     │
│  Sign Out                           │
│                                     │
├─────────────────────────────────────┤
│ [Home]     [Collection]   [Profile] │
└─────────────────────────────────────┘
```

---

## 8. Animations & Haptics

### 8.1 Animation Specifications

| Animation | Duration | Easing | Notes |
|-----------|----------|--------|-------|
| Blur sharpen (wrong answer) | 400ms | ease-out | Blur radius decreases one level |
| Blur sharpen (correct) | 600ms | ease-out | Blur to 0, slightly slower for drama |
| Frame materialize | 400ms | spring(200, 20) | Scale 0.9→1.0, slight overshoot |
| Frame badge pulse | 300ms | ease-out | Scale 0→1.0 |
| Input shake (wrong) | 300ms | ease-in-out | translateX: 0→-8→8→-4→4→0 |
| Hint reveal | 250ms | ease-out | Fade in + slide up 8px |
| Victory text | 200ms | ease-out | Fade in + slide up 12px |

### 8.2 Haptic Patterns

| Event | iOS | Android |
|-------|-----|---------|
| Wrong answer | `notificationError` | `EFFECT_TICK` |
| Correct - Gold | `notificationSuccess` + `impactHeavy` | `EFFECT_HEAVY_CLICK` |
| Correct - Silver | `notificationSuccess` | `EFFECT_CLICK` |
| Correct - Bronze | `notificationSuccess` | `EFFECT_CLICK` |
| Correct - No frame | `impactLight` | `EFFECT_TICK` |
| Give up | None | None |
| Button tap | `impactLight` | `EFFECT_TICK` |

---

## 9. Sharing

### 9.1 Share Card Design

```
┌─────────────────────────────────────┐
│ ╔═════════════════════════════════╗ │
│ ║                                 ║ │
│ ║        [Mnemonic Image]         ║ │
│ ║                                 ║ │
│ ║                             🥇  ║ │
│ ╚═════════════════════════════════╝ │
├─────────────────────────────────────┤
│                                     │
│  madrugada                          │
│  dawn, early morning                │
│                                     │
│  Got it in 1! 🥇                    │
│                                     │
│                    [App Logo]       │
└─────────────────────────────────────┘
```

**Frame-specific text:**
- Gold: "Got it in 1! 🥇"
- Silver: "Got it in 2! 🥈"
- Bronze: "Got it in 3! 🥉"
- Frameless (solved): "Got it! 💪"
- Frameless (gave up): "Learned this today 📚"

---

## 10. Content Pipeline

### 10.1 Content Package (Updated)

```json
{
  "slug": "madrugada",
  "scheduledDate": "2026-01-15",
  "spanishWord": "madrugada",
  "pronunciation": "/ma.dɾu.ˈɣa.da/",
  "englishTranslation": "dawn, early morning",
  "mnemonicSentenceOne": "Imagine a MADRE (mother) DRAGging the sun up over the horizon.",
  "mnemonicFull": "Imagine a MADRE (mother) DRAGging the sun up over the horizon in the early morning. She's determined to start the day, pulling that glowing orb into the sky while everyone else sleeps.",
  "exampleSentenceSpanish": "Salí de casa en la madrugada.",
  "exampleSentenceEnglish": "I left the house at dawn.",
  "normalizedAnswer": "madrugada",
  "imageUrl": "https://cdn.example.com/words/madrugada.png",
  "audioUrl": "https://cdn.example.com/audio/madrugada.mp3"
}
```

**Note:** `mnemonicSentenceOne` is first hint (after 1 wrong). `mnemonicFull` is complete hint (after 2 wrong).

---

## 11. Edge Cases

### 11.1 Quiz Edge Cases

| Scenario | Handling |
|----------|----------|
| User closes app mid-quiz | Session persists. Reopening shows current attempt state with appropriate blur/hints. |
| User submits empty string | Ignore, don't count as attempt |
| User submits only whitespace | Ignore, don't count as attempt |
| User on attempt 47 | Still can submit. No upper limit. They're determined. |
| Correct answer with trailing punctuation | Normalize strips it. "madrugada!" matches "madrugada" |
| Network fails on submit | Retry 3x silently. If all fail, show error: "Couldn't submit. Tap to retry." Don't clear input. |

### 11.2 Streak Edge Cases

| Scenario | Handling |
|----------|----------|
| User gives up | Streak still increments. Engagement counts, not correctness. |
| User views completed quiz but doesn't engage | No streak increment (already incremented on completion) |
| User changes timezone across midnight | Use new timezone. Accept date weirdness. |

### 11.3 Content Edge Cases

| Scenario | Handling |
|----------|----------|
| Word has multiple valid Spanish translations | normalizedAnswer field contains primary. Consider: comma-separated alternatives in future version. |
| Mnemonic doesn't split cleanly into one sentence | Content team responsibility. First sentence should stand alone. |

---

## 12. Performance Targets

| Metric | Target |
|--------|--------|
| Quiz screen load | < 300ms |
| Answer submission round-trip | < 500ms |
| Blur transition | 60fps |
| Frame animation | 60fps |
| Collection scroll (365 items) | 60fps |
| Image load (pre-cached) | < 200ms |

---

## 13. Analytics Events

| Event | Properties |
|-------|------------|
| `quiz_started` | `word_slug`, `date` |
| `answer_submitted` | `word_slug`, `attempt_number`, `is_correct` |
| `quiz_completed` | `word_slug`, `frame`, `attempts_used`, `gave_up` |
| `word_shared` | `word_slug`, `frame` |
| `favorite_toggled` | `word_slug`, `is_favorited` |
| `streak_milestone` | `days: 7 \| 30 \| 100 \| 365` |

---

## 14. Launch Checklist

### 14.1 Content
- [ ] 365 words with split mnemonics (sentence one + full)
- [ ] All images reviewed for quality
- [ ] All normalizedAnswer fields verified
- [ ] No scheduling gaps

### 14.2 Core Features
- [ ] Quiz flow: all states, all attempt counts
- [ ] Blur animation: all levels, transitions
- [ ] Frame system: award logic, display in collection
- [ ] Give up flow: confirmation, completion
- [ ] Streak: calculation, display, milestone animations

### 14.3 Polish
- [ ] All haptics tuned per platform
- [ ] All animations at 60fps
- [ ] Share card generation < 500ms
- [ ] Empty states designed
- [ ] Error states designed

### 14.4 Platform
- [ ] iOS TestFlight build passing
- [ ] Android internal testing build passing
- [ ] Push notifications working both platforms
- [ ] Deep links working

---

