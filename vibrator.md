# Complete Guide: How an App Call Reaches Hardware in Android
## Global Overview with Architecture Diagrams
### High-Level Architecture Flow

┌─────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│    App      │    │   Framework  │    │  System      │    │   Hardware   │
│             │    │              │    │   Services   │    │              │
│ vibrate(500)│ →  │ getSystem()  │ →  │ Vibrator     │ →  │ Vibration    │
│             │    │              │    │ Service      │    │ Motor        │
└─────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
       │                    │                   │                  │
       │ Java Method Call   │    Binder IPC     │   JNI → HAL      │ Electrical
       │                    │    (Process       │   → Kernel       │ Signal
       └────────────────────┘    Boundary)      └──────────────────┘

| Interface | Method | Duration (500ms) | Package Name    |
|-----------|--------|------------------|-----------------|
| Token     | ID (0) | 0x00000000000001F4 | "com.example.app" |
