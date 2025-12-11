classDiagram
    %% =========================
    %% Domain classes
    %% =========================
    class User {
      +id: string
      +email: string
      +passwordHash: string
      +preferredLang: string
      +signUp()
      +login()
      +takeSTAI(responses)
      +startChat()
    }

    class Session {
      +sessionId: string
      +startedAt: DateTime
      +expiresAt: DateTime
      +usedMinutesToday: int
      +authenticate()
      +isExpired(): bool
      +terminate()
    }

    class TestSTAI {
      +score: int
      +evaluate(responses): int
      +getLevel(): string
    }

    class Chatbot {
      +chatbotId: string
      +modeInteraction: string
      +language: string
      +dailyLimitMinutes: int
      +receiveMessage(text)
      +receiveVoice(audio)
      +sendResponse()
      +applyUsageLimit(userId)
    }

    class PipelineConv {
      +systemPromptTemplate: string
      +contextWindow: List
      +buildPrompt(userMessage): string
      +invokeModel(prompt): string
      +appendToContext(turn)
    }

    class ModelLLM {
      +name: string
      +endpoint: string
      +modelType: string
      +healthCheck(): bool
      +infer(prompt): string
    }

    %% =========================
    %% Infrastructure / Services
    %% =========================
    class Frontend {
      +framework: string = "ReactJS"
      +renderUI()
      +captureInput()
    }

    class BackendAPI {
      +framework: string = "Node/Express"
      +handleAuth()
      +handleSTAI()
      +handleChat()
      +routeToModel()
      +enforcePolicies()
    }

    class Database {
      +usersTable
      +sessionsTable
      +auditLogs
      +saveUser(user)
      +saveSession(session)
      +appendAudit(record)
    }

    class AuthService {
      +issueJWT(user): string
      +verifyJWT(token): bool
      +refreshToken(token): string
    }

    class STTService {
      +transcribe(audio): string
    }

    class TTSService {
      +synthesize(text): Audio
    }

    class RateLimiter {
      +checkQuota(userId): bool
      +consume(userId, minutes)
    }

    class SecurityModule {
      +maskPII(text): string
      +encrypt(data): string
      +validateInput(payload): bool
    }

    class Monitoring {
      +collectMetrics()
      +reportLatency()
      +alertOnThresholds()
    }

    class Logger {
      +logInteraction(userId, payload)
      +retrieveLogs(filter): List
    }

    %% =========================
    %% Associations / Multiplicities
    %% =========================
    User "1" --> "0..*" Session : creates/uses
    User "1" --> "0..1" TestSTAI : takes
    TestSTAI "1" --> "0..1" Chatbot : activates_if_level_ge_moderate
    Session "1" --> "0..1" Chatbot : controls_access
    Chatbot "1" --> "1" PipelineConv : orchestrates
    PipelineConv "1" --> "1..*" ModelLLM : invokes_selected_model
    Frontend --> BackendAPI : "HTTP / WebSocket"
    BackendAPI --> AuthService : authenticates_via
    BackendAPI --> Database : persists_to
    BackendAPI --> Chatbot : orchestrates
    BackendAPI --> STTService : audio_to_text
    BackendAPI --> TTSService : text_to_audio
    BackendAPI --> RateLimiter : enforces_quota
    BackendAPI --> Logger : logs_via
    BackendAPI --> Monitoring : emits_metrics_to
    BackendAPI --> SecurityModule : sanitizes
    ModelLLM --> Logger : reports_usage
    Database --> Logger : stores_audit_logs

    %% =========================
    %% Notes (expliquation & recommandations)
    %% =========================
    note right of PipelineConv
      SystemPrompt template injected here.
      ContextWindow: last N turns (short-term).
      No long-term persistent memory in current design.
    end note

    note left of RateLimiter
      Daily quota enforcement: 30 minutes per user.
      Quota tracked in Session.usedMinutesToday.
    end note

    note right of SecurityModule
      Recommended: PII redaction, TLS for transport,
      AES-256 at rest, RGPD consent flows.
    end note

    note left of Monitoring
      Recommended metrics: p50/p95/p99 latency,
      tokens per request, error rate, hallucination rate.
    end note
