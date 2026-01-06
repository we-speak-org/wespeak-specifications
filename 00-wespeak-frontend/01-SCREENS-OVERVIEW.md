# WeSpeak Frontend - Vue d'Ensemble des Écrans

## 📋 Table des Matières

1. [Architecture de Navigation](#1-architecture-de-navigation)
2. [Flux Utilisateur Principal](#2-flux-utilisateur-principal)
3. [Écrans par Module](#3-écrans-par-module)
4. [Composants Partagés](#4-composants-partagés)

---

## 1. Architecture de Navigation

```mermaid
flowchart TB
    subgraph PUBLIC["🌐 Zone Publique"]
        LANDING["Landing Page"]
        LOGIN["Login"]
        REGISTER["Register"]
        FORGOT["Forgot Password"]
        RESET["Reset Password"]
    end

    subgraph ONBOARDING["🎯 Onboarding (1ère connexion)"]
        ON1["1. Choix langue cible"]
        ON2["2. Langue maternelle"]
        ON3["3. Niveau actuel"]
        ON4["4. Objectif"]
        ON5["5. Goal hebdo"]
        ON6["6. Test placement (optionnel)"]
    end

    subgraph MAIN["📱 Application Principale"]
        DASH["Dashboard"]
        
        subgraph LEARN["📚 Apprentissage"]
            COURSES["Courses List"]
            COURSE_DETAIL["Course Detail"]
            LESSON_PLAYER["Lesson Player"]
            EXERCISE["Exercise View"]
            LESSON_COMPLETE["Lesson Complete"]
        end
        
        subgraph CONV["💬 Conversations"]
            CONV_HOME["Conversations Home"]
            TOPIC_SELECT["Topic Selection"]
            MATCHMAKING["Matchmaking"]
            VIDEO_CALL["Video Call"]
            POST_CONV["Post Conversation"]
        end
        
        subgraph FEEDBACK["📊 Feedback & Progress"]
            FB_LIST["Feedback List"]
            FB_DETAIL["Feedback Detail"]
            PROGRESS["Progress Dashboard"]
            SKILLS_MAP["Skills Map"]
        end
        
        subgraph GAMIF["🏆 Gamification"]
            LEADERBOARD["Leaderboard"]
            BADGES["Badges"]
            CHALLENGES["Challenges"]
            ACHIEVEMENTS["Achievements"]
        end
        
        subgraph PROFILE["👤 Profile"]
            SETTINGS["Settings"]
            ACCOUNT["Account"]
            SUBSCRIPTION["Subscription"]
            LANG_PROFILES["Learning Profiles"]
        end
    end

    LANDING --> LOGIN
    LANDING --> REGISTER
    LOGIN --> DASH
    LOGIN --> ON1
    REGISTER --> ON1
    
    ON1 --> ON2 --> ON3 --> ON4 --> ON5 --> ON6 --> DASH
    
    DASH --> LEARN
    DASH --> CONV
    DASH --> FEEDBACK
    DASH --> GAMIF
    DASH --> PROFILE

    classDef public fill:#94a3b8,stroke:#64748b,color:#fff
    classDef onboard fill:#a78bfa,stroke:#8b5cf6,color:#fff
    classDef main fill:#22c55e,stroke:#16a34a,color:#fff
    classDef learn fill:#3b82f6,stroke:#2563eb,color:#fff
    classDef conv fill:#f59e0b,stroke:#d97706,color:#fff
    classDef feedback fill:#ec4899,stroke:#db2777,color:#fff
    
    class LANDING,LOGIN,REGISTER,FORGOT,RESET public
    class ON1,ON2,ON3,ON4,ON5,ON6 onboard
    class DASH main
