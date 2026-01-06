# WeSpeak Frontend - Flux Utilisateur Principal

## 2. Parcours Utilisateur Type

```mermaid
journey
    title Parcours Utilisateur WeSpeak - Jour 1
    section Découverte
      Arrive sur Landing: 5: Visiteur
      Clique "Commencer": 5: Visiteur
    section Inscription
      Choisit Google OAuth: 5: Nouveau User
      Accepte permissions: 4: Nouveau User
    section Onboarding
      Choisit langue à apprendre: 5: Nouveau User
      Indique langue maternelle: 5: Nouveau User
      Sélectionne niveau estimé: 4: Nouveau User
      Définit objectif: 5: Nouveau User
      Choisit goal hebdo: 4: Nouveau User
    section Première Leçon
      Arrive sur Dashboard: 5: Apprenant
      Clique première leçon: 5: Apprenant
      Complète exercices: 4: Apprenant
      Gagne premiers XP: 5: Apprenant
    section Gamification
      Voit niveau monter: 5: Apprenant
      Débloque badge: 5: Apprenant
```

---

## 3. Flux de Navigation Détaillé

### 3.1 Flux Authentification

```mermaid
stateDiagram-v2
    [*] --> Landing
    Landing --> Login: Clic "Se connecter"
    Landing --> Register: Clic "Commencer"
    
    Login --> Dashboard: Auth OK + Profil existe
    Login --> Onboarding: Auth OK + 1ère fois
    Login --> Login: Erreur credentials
    
    Register --> OAuthGoogle: Clic Google
    Register --> OAuthFacebook: Clic Facebook
    Register --> EmailRegister: Email/Password
    
    OAuthGoogle --> Onboarding
    OAuthFacebook --> Onboarding
    EmailRegister --> VerifyEmail
    VerifyEmail --> Onboarding
    
    Login --> ForgotPassword: Clic "Mot de passe oublié"
    ForgotPassword --> ResetSent
    ResetSent --> ResetPassword: Clic email
    ResetPassword --> Login
```

### 3.2 Flux Onboarding

```mermaid
stateDiagram-v2
    [*] --> ChoixLangueCible
    ChoixLangueCible --> ChoixLangueNative: Next
    ChoixLangueNative --> ChoixNiveau: Next
    ChoixNiveau --> ChoixObjectif: Next
    ChoixObjectif --> ChoixGoalHebdo: Next
    ChoixGoalHebdo --> TestPlacement: "Tester mon niveau"
    ChoixGoalHebdo --> Dashboard: "Commencer directement"
    TestPlacement --> ResultatTest
    ResultatTest --> Dashboard
    
    ChoixLangueNative --> ChoixLangueCible: Back
    ChoixNiveau --> ChoixLangueNative: Back
    ChoixObjectif --> ChoixNiveau: Back
    ChoixGoalHebdo --> ChoixObjectif: Back
```

### 3.3 Flux Leçon

```mermaid
stateDiagram-v2
    [*] --> CoursesList
    CoursesList --> CourseDetail: Sélection cours
    CourseDetail --> LessonDetail: Sélection leçon
    
    LessonDetail --> ExercisePlayer: "Commencer"
    
    state ExercisePlayer {
        [*] --> Exercise1
        Exercise1 --> Exercise2: Correct
        Exercise1 --> Exercise1: Retry (si erreur)
        Exercise2 --> Exercise3: Correct
        Exercise3 --> ExerciseN: ...
        ExerciseN --> [*]
    }
    
    ExercisePlayer --> LessonComplete: Tous exercices OK
    LessonComplete --> CourseDetail: "Continuer"
    LessonComplete --> NextLesson: "Leçon suivante"
    
    LessonDetail --> CourseDetail: Back
    CourseDetail --> CoursesList: Back
```

### 3.4 Flux Conversation

```mermaid
stateDiagram-v2
    [*] --> ConversationsHome
    ConversationsHome --> TopicSelection: "Nouvelle conversation"
    ConversationsHome --> ConversationHistory: "Historique"
    
    TopicSelection --> Matchmaking: Sélection topic + préférences
    
    state Matchmaking {
        [*] --> Searching
        Searching --> PartnerFound: Match trouvé
        Searching --> Timeout: 2 min sans match
        Timeout --> Searching: Retry
        Timeout --> TopicSelection: Cancel
    }
    
    Matchmaking --> PreparationRoom: Partner found
    PreparationRoom --> VideoCall: Les 2 prêts
    
    state VideoCall {
        [*] --> Active
        Active --> PromptDisplayed: Nouveau prompt
        PromptDisplayed --> Active: Timer
        Active --> EndedByUser: Clic "Terminer"
        Active --> EndedByPartner: Partner quitte
    }
    
    VideoCall --> PostConversation
    PostConversation --> RatePartner
    RatePartner --> FeedbackPending: "Attendre feedback IA"
    RatePartner --> ConversationsHome: "Retour"
    FeedbackPending --> FeedbackDetail: Feedback prêt
```

---

## 4. États de l'Application

### 4.1 États Globaux

```typescript
interface AppState {
  // Auth
  isAuthenticated: boolean;
  currentUser: User | null;
  accessToken: string | null;
  
  // Profil d'apprentissage actif
  activeLearningProfile: LearningProfile | null;
  
  // Gamification temps réel
  currentXP: number;
  currentLevel: number;
  currentStreak: number;
  
  // Notifications
  unreadNotifications: number;
  pendingFeedbacks: number;
  
  // UI
  sidebarOpen: boolean;
  currentTheme: 'light' | 'dark' | 'system';
  uiLanguage: string;
}
```

### 4.2 États par Module

```typescript
// Lesson Module State
interface LessonState {
  courses: Course[];
  currentCourse: Course | null;
  currentLesson: Lesson | null;
  currentExerciseIndex: number;
  sessionScore: number;
  sessionErrors: ExerciseError[];
  isLoading: boolean;
}

// Conversation Module State
interface ConversationState {
  availableTopics: ConversationTopic[];
  matchRequest: MatchRequest | null;
  matchStatus: 'idle' | 'searching' | 'found' | 'timeout';
  currentSession: ConversationSession | null;
  peerConnection: RTCPeerConnection | null;
  localStream: MediaStream | null;
  remoteStream: MediaStream | null;
  currentPrompt: string | null;
  isRecording: boolean;
}

// Feedback Module State
interface FeedbackState {
  feedbackList: FeedbackSummary[];
  currentFeedback: FeedbackReport | null;
  userStats: UserStats | null;
  progressHistory: ProgressDataPoint[];
}

// Gamification Module State
interface GamificationState {
  profile: GamificationProfile | null;
  badges: Badge[];
  earnedBadges: string[];
  activeChallenges: Challenge[];
  leaderboard: LeaderboardEntry[];
  recentXPGains: XPTransaction[];
}
```
