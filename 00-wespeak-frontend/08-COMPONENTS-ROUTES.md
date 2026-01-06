# WeSpeak Frontend - Composants & Routes

## 📦 Composants Partagés

---

## 27. Composants UI Réutilisables

### 27.1 Navigation

```typescript
// Bottom Navigation (Mobile)
interface BottomNavItem {
  icon: string;      // 🏠 📚 💬 📊 👤
  label: string;     // Home, Learn, Talk, Progress, Profile
  route: string;
  badge?: number;    // Notifications count
  isActive: boolean;
}

// Sidebar (Desktop)
interface SidebarItem {
  icon: string;
  label: string;
  route: string;
  children?: SidebarItem[];
  badge?: number;
}

// Header
interface HeaderConfig {
  showBackButton: boolean;
  title: string;
  rightActions?: {
    icon: string;
    action: () => void;
  }[];
  showLanguageSelector: boolean;
  showNotifications: boolean;
  showProfile: boolean;
}
```

### 27.2 Cards

```
┌─────────────────────────────────────────┐
│  LESSON CARD                            │
│  ┌────────────────────────────────────┐ │
│  │ 🍽️ Image/Icon                      │ │
│  │ Titre de la leçon                  │ │
│  │ ━━━━━━━━━━░░░░░ 75%               │ │
│  │ A2 · 8 min · +50 XP                │ │
│  │ [Continuer] ou 🔒                  │ │
│  └────────────────────────────────────┘ │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  COURSE CARD                            │
│  ┌────────────────────────────────────┐ │
│  │ 📚 Image/Icon                      │ │
│  │ Nom du cours                       │ │
│  │ Description courte                 │ │
│  │ ⭐ 4.8 · 15 leçons · ~2h           │ │
│  │ ━━━━━━━━░░░░░ 60%                 │ │
│  └────────────────────────────────────┘ │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  TOPIC CARD (Conversation)              │
│  ┌────────────────────────────────────┐ │
│  │ 🎯 Icon                            │ │
│  │ Nom du topic                       │ │
│  │ A2 · 15 min                        │ │
│  │ ⭐ Populaire (optionnel)           │ │
│  └────────────────────────────────────┘ │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  FEEDBACK CARD                          │
│  ┌────────────────────────────────────┐ │
│  │ 🍽️ Topic · avec Partner            │ │
│  │ Score: 78/100 ↑                    │ │
│  │ ━━━━━━━━━━━━━━━━━━░░░░             │ │
│  │ ✅ 3 points forts · ⚠️ 2 erreurs   │ │
│  │ Date · Durée                       │ │
│  │                    [Voir détail →] │ │
│  └────────────────────────────────────┘ │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  BADGE CARD                             │
│  ┌────────────────────────────────────┐ │
│  │                                    │ │
│  │        🏅 Icon (color/gray)        │ │
│  │                                    │ │
│  │        Nom du badge                │ │
│  │        Description                 │ │
│  │        ✓ Débloqué / 🔒 5/10        │ │
│  │                                    │ │
│  └────────────────────────────────────┘ │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  CHALLENGE CARD                         │
│  ┌────────────────────────────────────┐ │
│  │ ☑️/○ Titre du défi                 │ │
│  │ ━━━━━━━━━━━━━━░░░░░░  7/10        │ │
│  │ Récompense: +100 XP               │ │
│  │             [Récupérer] si complété│ │
│  └────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### 27.3 Progress Indicators

```
// Progress Bar
━━━━━━━━━━━━━━━━━░░░░░░░░░░░░  60%

// Circular Progress
    ┌───────┐
    │  72%  │  (avec animation)
    └───────┘

// Streak Indicator
🔥 8 jours

// Level Badge
🏆 Niveau 8 (1,240 / 1,600 XP)

// XP Gain Animation
+50 XP ⭐ (popup animé)
```

### 27.4 Exercise Components

```typescript
// MCQ Component
interface MCQExercise {
  question: string;
  options: string[];
  correctIndex: number;
  selectedIndex?: number;
  showFeedback: boolean;
}

// Fill Gap Component
interface FillGapExercise {
  sentence: string;       // "I would like _____ the steak"
  gapPosition: number;
  options: string[];
  correctAnswer: string;
  userAnswer?: string;
}

// Listen & Repeat Component
interface ListenRepeatExercise {
  audioUrl: string;
  targetText: string;
  isPlaying: boolean;
  isRecording: boolean;
  userRecordingUrl?: string;
  score?: number;
}

// Translation Component
interface TranslationExercise {
  sourceText: string;
  sourceLang: string;
  targetLang: string;
  correctTranslation: string;
  userTranslation?: string;
  hints?: string[];
}

// Match Pairs Component
interface MatchPairsExercise {
  leftItems: string[];
  rightItems: string[];
  correctPairs: [number, number][];
  userPairs: [number, number][];
}

// Ordering Component
interface OrderingExercise {
  shuffledWords: string[];
  correctOrder: number[];
  userOrder: number[];
}
```

### 27.5 Feedback Components

```
// Correct Answer Feedback
┌────────────────────────────────────────┐
│  ✅ Excellent !                        │
│                                        │
│  Votre réponse était parfaite.        │
│                                        │
│  📝 Note explicative (optionnelle)    │
│                                        │
│                          +15 XP ⭐     │
└────────────────────────────────────────┘

// Incorrect Answer Feedback
┌────────────────────────────────────────┐
│  ❌ Pas tout à fait...                │
│                                        │
│  Votre réponse : "..."                │
│  Correction : "..."                   │
│                                        │
│  💡 Explication détaillée             │
│                                        │
│  📚 Leçon recommandée (optionnel)     │
└────────────────────────────────────────┘
```

---

## 28. Structure des Routes Angular

```typescript
// app.routes.ts
export const routes: Routes = [
  // Public routes
  { path: '', component: LandingComponent },
  { path: 'login', component: LoginComponent },
  { path: 'register', component: RegisterComponent },
  { path: 'forgot-password', component: ForgotPasswordComponent },
  { path: 'reset-password/:token', component: ResetPasswordComponent },
  { path: 'verify-email/:token', component: VerifyEmailComponent },

  // Onboarding (auth required, first time)
  {
    path: 'onboarding',
    canActivate: [AuthGuard, FirstTimeGuard],
    children: [
      { path: 'target-language', component: TargetLanguageComponent },
      { path: 'native-language', component: NativeLanguageComponent },
      { path: 'level', component: LevelComponent },
      { path: 'goal', component: GoalComponent },
      { path: 'weekly-goal', component: WeeklyGoalComponent },
      { path: 'placement-test', component: PlacementTestComponent },
    ]
  },

  // Main app (auth required)
  {
    path: '',
    canActivate: [AuthGuard],
    children: [
      // Dashboard
      { path: 'dashboard', component: DashboardComponent },

      // Learning
      { 
        path: 'learn',
        children: [
          { path: '', component: CoursesListComponent },
          { path: 'courses/:courseId', component: CourseDetailComponent },
          { path: 'lessons/:lessonId', component: LessonPlayerComponent },
        ]
      },

      // Conversations
      {
        path: 'conversations',
        children: [
          { path: '', component: ConversationsHomeComponent },
          { path: 'new', component: TopicSelectionComponent },
          { path: 'new/preferences', component: PreferencesComponent },
          { path: 'matching', component: MatchmakingComponent },
          { path: 'matched', component: MatchedComponent },
          { path: 'session/:sessionId', component: VideoCallComponent },
          { path: 'session/:sessionId/rate', component: RatePartnerComponent },
          { path: 'session/:sessionId/summary', component: SummaryComponent },
          { path: 'history', component: ConversationHistoryComponent },
        ]
      },

      // Feedback & Progress
      {
        path: 'feedback',
        children: [
          { path: '', component: FeedbackListComponent },
          { path: ':feedbackId', component: FeedbackDetailComponent },
          { path: ':feedbackId/transcript', component: TranscriptComponent },
        ]
      },
      {
        path: 'progress',
        children: [
          { path: '', component: ProgressDashboardComponent },
          { path: 'skills', component: SkillsMapComponent },
        ]
      },

      // Gamification
      { path: 'leaderboard', component: LeaderboardComponent },
      { path: 'badges', component: BadgesComponent },
      { path: 'challenges', component: ChallengesComponent },

      // Profile
      {
        path: 'profile',
        children: [
          { path: '', component: ProfileComponent },
          { path: 'edit', component: EditProfileComponent },
          { path: 'subscription', component: SubscriptionComponent },
          { path: 'notifications', component: NotificationsSettingsComponent },
          { path: 'security', component: SecurityComponent },
          { path: 'learning-profiles', component: LearningProfilesComponent },
          { path: 'learning-profiles/:id', component: EditLearningProfileComponent },
        ]
      },
    ]
  },

  // Fallback
  { path: '**', redirectTo: '/dashboard' }
];
```

---

## 29. Services Angular Requis

```typescript
// Services principaux
interface AngularServices {
  // Auth
  AuthService: {
    login(email: string, password: string): Observable<AuthResponse>;
    register(data: RegisterData): Observable<AuthResponse>;
    logout(): void;
    refreshToken(): Observable<string>;
    getCurrentUser(): Observable<User>;
    isAuthenticated(): boolean;
  };

  // Learning
  CourseService: {
    getCourses(filters?: CourseFilters): Observable<Course[]>;
    getCourse(id: string): Observable<Course>;
    getUnits(courseId: string): Observable<Unit[]>;
  };

  LessonService: {
    getLesson(id: string): Observable<Lesson>;
    getExercises(lessonId: string): Observable<Exercise[]>;
    startLesson(id: string): Observable<LessonSession>;
    submitExercise(exerciseId: string, answer: any): Observable<ExerciseResult>;
    completeLesson(id: string, data: LessonCompletionData): Observable<LessonResult>;
  };

  ProgressService: {
    getUserProgress(): Observable<UserProgress>;
    getSkills(): Observable<Skill[]>;
    getProgressHistory(period: string): Observable<ProgressDataPoint[]>;
  };

  // Conversations
  ConversationService: {
    getTopics(filters?: TopicFilters): Observable<ConversationTopic[]>;
    createMatchRequest(data: MatchRequestData): Observable<MatchRequest>;
    cancelMatchRequest(id: string): Observable<void>;
    getSession(id: string): Observable<ConversationSession>;
    rateSession(sessionId: string, rating: Rating): Observable<void>;
    getHistory(): Observable<ConversationSummary[]>;
  };

  WebRTCService: {
    connect(sessionId: string): Observable<RTCConnection>;
    disconnect(): void;
    toggleAudio(): void;
    toggleVideo(): void;
    getLocalStream(): Observable<MediaStream>;
    getRemoteStream(): Observable<MediaStream>;
  };

  SignalingService: {
    connect(sessionId: string): Observable<WebSocket>;
    sendOffer(sdp: RTCSessionDescription): void;
    sendAnswer(sdp: RTCSessionDescription): void;
    sendIceCandidate(candidate: RTCIceCandidate): void;
    onMessage(): Observable<SignalingMessage>;
  };

  // Feedback
  FeedbackService: {
    getFeedbackList(): Observable<FeedbackSummary[]>;
    getFeedback(id: string): Observable<FeedbackReport>;
    getTranscript(feedbackId: string): Observable<Transcript>;
    getUserStats(): Observable<UserStats>;
  };

  // Gamification
  GamificationService: {
    getProfile(): Observable<GamificationProfile>;
    getBadges(): Observable<Badge[]>;
    getChallenges(type: string): Observable<Challenge[]>;
    claimReward(challengeId: string): Observable<RewardResult>;
    getLeaderboard(filters: LeaderboardFilters): Observable<LeaderboardEntry[]>;
  };

  // Recommendations
  RecommendationService: {
    getNextLesson(): Observable<LessonRecommendation>;
    getRecommendedTopics(): Observable<TopicRecommendation[]>;
    getSkillGaps(): Observable<SkillGap[]>;
  };

  // User
  UserService: {
    getProfile(): Observable<User>;
    updateProfile(data: UpdateProfileData): Observable<User>;
    getLearningProfiles(): Observable<LearningProfile[]>;
    createLearningProfile(data: CreateLearningProfileData): Observable<LearningProfile>;
    updateLearningProfile(id: string, data: UpdateLearningProfileData): Observable<LearningProfile>;
    setActiveLearningProfile(id: string): Observable<void>;
  };

  // Notifications
  NotificationService: {
    getNotifications(): Observable<Notification[]>;
    markAsRead(id: string): Observable<void>;
    updateSettings(settings: NotificationSettings): Observable<void>;
  };
}
```

---

## 30. State Management (NgRx ou Signals)

```typescript
// Global State Shape
interface AppState {
  auth: AuthState;
  user: UserState;
  learning: LearningState;
  conversation: ConversationState;
  feedback: FeedbackState;
  gamification: GamificationState;
  ui: UIState;
}

interface AuthState {
  isAuthenticated: boolean;
  accessToken: string | null;
  refreshToken: string | null;
  isLoading: boolean;
  error: string | null;
}

interface UserState {
  currentUser: User | null;
  activeLearningProfile: LearningProfile | null;
  learningProfiles: LearningProfile[];
  isLoading: boolean;
}

interface LearningState {
  courses: Course[];
  currentCourse: Course | null;
  currentLesson: Lesson | null;
  currentExercises: Exercise[];
  currentExerciseIndex: number;
  sessionScore: number;
  sessionErrors: ExerciseError[];
  userProgress: UserProgress | null;
  skills: Skill[];
  isLoading: boolean;
}

interface ConversationState {
  topics: ConversationTopic[];
  matchRequest: MatchRequest | null;
  matchStatus: 'idle' | 'searching' | 'found' | 'timeout' | 'error';
  currentSession: ConversationSession | null;
  localStream: MediaStream | null;
  remoteStream: MediaStream | null;
  isConnected: boolean;
  currentPrompt: string | null;
  history: ConversationSummary[];
}

interface FeedbackState {
  feedbackList: FeedbackSummary[];
  currentFeedback: FeedbackReport | null;
  currentTranscript: Transcript | null;
  userStats: UserStats | null;
  progressHistory: ProgressDataPoint[];
  isLoading: boolean;
}

interface GamificationState {
  profile: GamificationProfile | null;
  badges: Badge[];
  challenges: Challenge[];
  leaderboard: LeaderboardEntry[];
  recentXPGains: XPTransaction[];
  isLoading: boolean;
}

interface UIState {
  theme: 'light' | 'dark' | 'system';
  sidebarOpen: boolean;
  currentLanguage: string;
  notifications: AppNotification[];
  modals: ModalState[];
  toasts: Toast[];
}
```

---

## 31. Responsive Breakpoints

```scss
// breakpoints.scss
$breakpoints: (
  'mobile': 320px,
  'mobile-lg': 425px,
  'tablet': 768px,
  'laptop': 1024px,
  'desktop': 1440px,
  'desktop-xl': 1920px
);

// Usage
@mixin mobile {
  @media (max-width: 767px) { @content; }
}

@mixin tablet {
  @media (min-width: 768px) and (max-width: 1023px) { @content; }
}

@mixin desktop {
  @media (min-width: 1024px) { @content; }
}
```

### Layout Adaptatif

| Écran | Mobile (<768px) | Tablet (768-1023px) | Desktop (≥1024px) |
|-------|-----------------|---------------------|-------------------|
| Navigation | Bottom bar | Bottom bar | Sidebar |
| Dashboard | Stack vertical | 2 colonnes | 3 colonnes |
| Course list | 1 carte/ligne | 2 cartes/ligne | 3-4 cartes/ligne |
| Video call | Full screen | Full screen | Split view |
| Lesson player | Full screen | Centered modal | Centered 800px |

---

## 32. Accessibilité (a11y)

### Standards à respecter

- **WCAG 2.1 AA** minimum
- Contraste couleurs ≥ 4.5:1
- Navigation clavier complète
- Labels ARIA sur tous les éléments interactifs
- Skip links pour navigation
- Focus visible
- Support lecteur d'écran

### Implémentation

```html
<!-- Exemple bouton accessible -->
<button 
  aria-label="Commencer la leçon"
  aria-describedby="lesson-description"
  [attr.aria-pressed]="isStarted"
  (keydown.enter)="startLesson()"
  (keydown.space)="startLesson()"
>
  Commencer
</button>

<!-- Exemple progress bar accessible -->
<div 
  role="progressbar"
  [attr.aria-valuenow]="progress"
  aria-valuemin="0"
  aria-valuemax="100"
  [attr.aria-label]="'Progression: ' + progress + '%'"
>
  <div class="progress-fill" [style.width.%]="progress"></div>
</div>
```
