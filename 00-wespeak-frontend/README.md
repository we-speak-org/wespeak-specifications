# WeSpeak Frontend - Résumé des Écrans

## 📱 Index Complet des Écrans

| # | Écran | Route | Document |
|---|-------|-------|----------|
| **Zone Publique** |
| 1.1 | Landing Page | `/` | [03-SCREENS-PUBLIC-ONBOARDING.md](./03-SCREENS-PUBLIC-ONBOARDING.md) |
| 1.2 | Login | `/login` | [03-SCREENS-PUBLIC-ONBOARDING.md](./03-SCREENS-PUBLIC-ONBOARDING.md) |
| 1.3 | Register | `/register` | [03-SCREENS-PUBLIC-ONBOARDING.md](./03-SCREENS-PUBLIC-ONBOARDING.md) |
| 1.4 | Forgot Password | `/forgot-password` | - |
| 1.5 | Reset Password | `/reset-password/:token` | - |
| **Onboarding** |
| 2.1 | Choix Langue Cible | `/onboarding/target-language` | [03-SCREENS-PUBLIC-ONBOARDING.md](./03-SCREENS-PUBLIC-ONBOARDING.md) |
| 2.2 | Langue Maternelle | `/onboarding/native-language` | [03-SCREENS-PUBLIC-ONBOARDING.md](./03-SCREENS-PUBLIC-ONBOARDING.md) |
| 2.3 | Niveau Actuel | `/onboarding/level` | [03-SCREENS-PUBLIC-ONBOARDING.md](./03-SCREENS-PUBLIC-ONBOARDING.md) |
| 2.4 | Objectif | `/onboarding/goal` | [03-SCREENS-PUBLIC-ONBOARDING.md](./03-SCREENS-PUBLIC-ONBOARDING.md) |
| 2.5 | Goal Hebdomadaire | `/onboarding/weekly-goal` | [03-SCREENS-PUBLIC-ONBOARDING.md](./03-SCREENS-PUBLIC-ONBOARDING.md) |
| 2.6 | Test Placement | `/onboarding/placement-test` | [03-SCREENS-PUBLIC-ONBOARDING.md](./03-SCREENS-PUBLIC-ONBOARDING.md) |
| **Dashboard & Learning** |
| 3 | Dashboard | `/dashboard` | [04-SCREENS-DASHBOARD-LEARNING.md](./04-SCREENS-DASHBOARD-LEARNING.md) |
| 4 | Liste des Cours | `/learn` | [04-SCREENS-DASHBOARD-LEARNING.md](./04-SCREENS-DASHBOARD-LEARNING.md) |
| 5 | Détail Cours | `/learn/courses/:id` | [04-SCREENS-DASHBOARD-LEARNING.md](./04-SCREENS-DASHBOARD-LEARNING.md) |
| 6.1 | Lesson - Introduction | `/lessons/:id` | [04-SCREENS-DASHBOARD-LEARNING.md](./04-SCREENS-DASHBOARD-LEARNING.md) |
| 6.2 | Lesson - QCM | `/lessons/:id` | [04-SCREENS-DASHBOARD-LEARNING.md](./04-SCREENS-DASHBOARD-LEARNING.md) |
| 6.3 | Lesson - Fill Gap | `/lessons/:id` | [04-SCREENS-DASHBOARD-LEARNING.md](./04-SCREENS-DASHBOARD-LEARNING.md) |
| 6.4 | Lesson - Listen & Repeat | `/lessons/:id` | [04-SCREENS-DASHBOARD-LEARNING.md](./04-SCREENS-DASHBOARD-LEARNING.md) |
| 6.5 | Lesson - Translation | `/lessons/:id` | [04-SCREENS-DASHBOARD-LEARNING.md](./04-SCREENS-DASHBOARD-LEARNING.md) |
| 6.6 | Lesson - Feedback Correct | `/lessons/:id` | [04-SCREENS-DASHBOARD-LEARNING.md](./04-SCREENS-DASHBOARD-LEARNING.md) |
| 6.7 | Lesson - Feedback Incorrect | `/lessons/:id` | [04-SCREENS-DASHBOARD-LEARNING.md](./04-SCREENS-DASHBOARD-LEARNING.md) |
| 6.8 | Lesson - Fin de Leçon | `/lessons/:id` | [04-SCREENS-DASHBOARD-LEARNING.md](./04-SCREENS-DASHBOARD-LEARNING.md) |
| **Conversations** |
| 8 | Conversations Home | `/conversations` | [05-SCREENS-CONVERSATIONS.md](./05-SCREENS-CONVERSATIONS.md) |
| 9 | Sélection Topic | `/conversations/new` | [05-SCREENS-CONVERSATIONS.md](./05-SCREENS-CONVERSATIONS.md) |
| 10 | Préférences Partenaire | `/conversations/new/preferences` | [05-SCREENS-CONVERSATIONS.md](./05-SCREENS-CONVERSATIONS.md) |
| 11 | Matchmaking | `/conversations/matching` | [05-SCREENS-CONVERSATIONS.md](./05-SCREENS-CONVERSATIONS.md) |
| 12 | Partenaire Trouvé | `/conversations/matched` | [05-SCREENS-CONVERSATIONS.md](./05-SCREENS-CONVERSATIONS.md) |
| 13 | Appel Vidéo | `/conversations/session/:id` | [05-SCREENS-CONVERSATIONS.md](./05-SCREENS-CONVERSATIONS.md) |
| 14 | Post-Conv Rating | `/conversations/session/:id/rate` | [05-SCREENS-CONVERSATIONS.md](./05-SCREENS-CONVERSATIONS.md) |
| 15 | Post-Conv Summary | `/conversations/session/:id/summary` | [05-SCREENS-CONVERSATIONS.md](./05-SCREENS-CONVERSATIONS.md) |
| **Feedback & Progress** |
| 16 | Liste Feedbacks | `/feedback` | [06-SCREENS-FEEDBACK-PROGRESS.md](./06-SCREENS-FEEDBACK-PROGRESS.md) |
| 17 | Détail Feedback | `/feedback/:id` | [06-SCREENS-FEEDBACK-PROGRESS.md](./06-SCREENS-FEEDBACK-PROGRESS.md) |
| 18 | Dashboard Progression | `/progress` | [06-SCREENS-FEEDBACK-PROGRESS.md](./06-SCREENS-FEEDBACK-PROGRESS.md) |
| 19 | Carte des Skills | `/progress/skills` | [06-SCREENS-FEEDBACK-PROGRESS.md](./06-SCREENS-FEEDBACK-PROGRESS.md) |
| 20 | Transcription | `/feedback/:id/transcript` | [06-SCREENS-FEEDBACK-PROGRESS.md](./06-SCREENS-FEEDBACK-PROGRESS.md) |
| **Gamification** |
| 21 | Leaderboard | `/leaderboard` | [07-SCREENS-GAMIFICATION-PROFILE.md](./07-SCREENS-GAMIFICATION-PROFILE.md) |
| 22 | Badges | `/badges` | [07-SCREENS-GAMIFICATION-PROFILE.md](./07-SCREENS-GAMIFICATION-PROFILE.md) |
| 23 | Défis | `/challenges` | [07-SCREENS-GAMIFICATION-PROFILE.md](./07-SCREENS-GAMIFICATION-PROFILE.md) |
| **Profil** |
| 24 | Profil Utilisateur | `/profile` | [07-SCREENS-GAMIFICATION-PROFILE.md](./07-SCREENS-GAMIFICATION-PROFILE.md) |
| 25 | Abonnement | `/profile/subscription` | [07-SCREENS-GAMIFICATION-PROFILE.md](./07-SCREENS-GAMIFICATION-PROFILE.md) |
| 26 | Notifications | `/profile/notifications` | [07-SCREENS-GAMIFICATION-PROFILE.md](./07-SCREENS-GAMIFICATION-PROFILE.md) |

---

## 🎨 Palette de Couleurs Suggérée

```scss
// Primary Colors
$primary: #6366f1;        // Indigo - Actions principales
$primary-dark: #4f46e5;
$primary-light: #a5b4fc;

// Accent Colors
$accent: #22c55e;         // Green - Succès, XP
$warning: #f59e0b;        // Amber - Attention
$error: #ef4444;          // Red - Erreurs
$info: #3b82f6;           // Blue - Information

// Neutral Colors
$gray-50: #f9fafb;
$gray-100: #f3f4f6;
$gray-200: #e5e7eb;
$gray-300: #d1d5db;
$gray-400: #9ca3af;
$gray-500: #6b7280;
$gray-600: #4b5563;
$gray-700: #374151;
$gray-800: #1f2937;
$gray-900: #111827;

// Special Colors
$streak: #f97316;         // Orange - Streaks
$xp: #eab308;             // Yellow - XP
$badge-common: #9ca3af;
$badge-rare: #3b82f6;
$badge-epic: #a855f7;
$badge-legendary: #f59e0b;
```

---

## 📐 Spacing & Typography

```scss
// Spacing Scale (rem)
$spacing: (
  0: 0,
  1: 0.25rem,   // 4px
  2: 0.5rem,    // 8px
  3: 0.75rem,   // 12px
  4: 1rem,      // 16px
  5: 1.25rem,   // 20px
  6: 1.5rem,    // 24px
  8: 2rem,      // 32px
  10: 2.5rem,   // 40px
  12: 3rem,     // 48px
  16: 4rem,     // 64px
);

// Typography
$font-family: 'Inter', -apple-system, sans-serif;

$font-sizes: (
  'xs': 0.75rem,    // 12px
  'sm': 0.875rem,   // 14px
  'base': 1rem,     // 16px
  'lg': 1.125rem,   // 18px
  'xl': 1.25rem,    // 20px
  '2xl': 1.5rem,    // 24px
  '3xl': 1.875rem,  // 30px
  '4xl': 2.25rem,   // 36px
);

$font-weights: (
  'normal': 400,
  'medium': 500,
  'semibold': 600,
  'bold': 700,
);
```

---

## 🔔 Animations & Transitions

```scss
// Transitions
$transition-fast: 150ms ease;
$transition-base: 250ms ease;
$transition-slow: 350ms ease;

// Animations clés
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from { transform: translateY(20px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

@keyframes xpGain {
  0% { transform: scale(0.5); opacity: 0; }
  50% { transform: scale(1.2); }
  100% { transform: scale(1); opacity: 1; }
}

@keyframes streakFlame {
  0%, 100% { transform: scaleY(1); }
  50% { transform: scaleY(1.1); }
}
```

---

## 📁 Structure des Fichiers

```
src/
├── app/
│   ├── core/
│   │   ├── guards/
│   │   ├── interceptors/
│   │   ├── services/
│   │   └── models/
│   ├── shared/
│   │   ├── components/
│   │   │   ├── bottom-nav/
│   │   │   ├── header/
│   │   │   ├── sidebar/
│   │   │   ├── cards/
│   │   │   ├── progress/
│   │   │   └── modals/
│   │   ├── directives/
│   │   ├── pipes/
│   │   └── utils/
│   ├── features/
│   │   ├── auth/
│   │   │   ├── login/
│   │   │   ├── register/
│   │   │   └── onboarding/
│   │   ├── learning/
│   │   │   ├── courses/
│   │   │   ├── lessons/
│   │   │   └── exercises/
│   │   ├── conversations/
│   │   │   ├── home/
│   │   │   ├── matchmaking/
│   │   │   ├── video-call/
│   │   │   └── post-conversation/
│   │   ├── feedback/
│   │   │   ├── list/
│   │   │   ├── detail/
│   │   │   └── progress/
│   │   ├── gamification/
│   │   │   ├── leaderboard/
│   │   │   ├── badges/
│   │   │   └── challenges/
│   │   └── profile/
│   │       ├── settings/
│   │       ├── subscription/
│   │       └── learning-profiles/
│   ├── state/
│   │   ├── auth/
│   │   ├── learning/
│   │   ├── conversation/
│   │   ├── feedback/
│   │   └── gamification/
│   └── app.routes.ts
├── assets/
│   ├── icons/
│   ├── images/
│   ├── audio/
│   └── i18n/
├── styles/
│   ├── _variables.scss
│   ├── _mixins.scss
│   ├── _typography.scss
│   ├── _animations.scss
│   └── styles.scss
└── environments/
```

---

## 🚀 Prochaines Étapes Frontend

1. **Phase 1 - Setup** (1 semaine)
   - [ ] Création projet Angular 20
   - [ ] Configuration routing
   - [ ] Setup state management (NgRx/Signals)
   - [ ] Composants de base (Header, Nav, Cards)
   - [ ] Système de design tokens

2. **Phase 2 - Auth & Onboarding** (1 semaine)
   - [ ] Pages Login/Register
   - [ ] Intégration OAuth (Google)
   - [ ] Flux onboarding complet
   - [ ] Guards et interceptors

3. **Phase 3 - Learning Module** (2 semaines)
   - [ ] Liste des cours
   - [ ] Détail cours/unités
   - [ ] Player de leçons
   - [ ] Tous les types d'exercices
   - [ ] Système de feedback

4. **Phase 4 - Conversations** (2 semaines)
   - [ ] Sélection topics
   - [ ] Matchmaking UI
   - [ ] WebRTC video call
   - [ ] Post-conversation flow

5. **Phase 5 - Feedback & Gamification** (1 semaine)
   - [ ] Liste et détail feedbacks
   - [ ] Dashboard progression
   - [ ] Leaderboard
   - [ ] Badges & Défis

6. **Phase 6 - Profile & Polish** (1 semaine)
   - [ ] Pages profil
   - [ ] Paramètres
   - [ ] Abonnement
   - [ ] Tests E2E
   - [ ] Accessibilité

---

*Spécifications frontend WeSpeak v1.0*  
*Janvier 2026*
