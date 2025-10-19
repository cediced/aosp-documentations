# Complete Guide: How an App Call Reaches Hardware in Android
## Global Overview with Architecture Diagrams
### High-Level Architecture Flow
```
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
```

┌─────────────────────────────────────────────────────────────────────────┐
│                            APPLICATION LAYER                            │
│  YourApp.java → Vibrator.vibrate(500)                                   │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │ (getSystemService + Binder IPC)
┌───────────────────────────────▼─────────────────────────────────────────┐
│                          FRAMEWORK LAYER                                │
│  Vibrator.java (Proxy) → Binder.transact()                              │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │ (Binder Driver in Linux Kernel)
┌───────────────────────────────▼─────────────────────────────────────────┐
│                         SYSTEM SERVER PROCESS                           │
│  VibratorService.onTransact() → startVibrationLocked()                  │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │ (JNI Call)
┌───────────────────────────────▼─────────────────────────────────────────┐
│                          NATIVE LAYER                                   │
│  com_android_server_VibratorService.cpp → vibrator.c (HAL)              │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │ (System Calls / Device Files)
┌───────────────────────────────▼─────────────────────────────────────────┐
│                          KERNEL LAYER                                   │
│  vibrator.ko driver → goldfish_vibrator.c (emulator)                    │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │ (Hardware Registers / GPIO/PWM)
┌───────────────────────────────▼─────────────────────────────────────────┐
│                          HARDWARE LAYER                                 │
│  Motor Driver IC → Vibration Motor                                      │
└─────────────────────────────────────────────────────────────────────────┘

| Interface | Method | Duration (500ms) | Package Name    |
|-----------|--------|------------------|-----------------|
| Token     | ID (0) | 0x00000000000001F4 | "com.example.app" |
