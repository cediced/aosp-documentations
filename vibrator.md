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
### Detailed Layer Architecture

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
```

### AOSP Vibrator File Structure Diagram
```
aosp/
├── 📱 App Layer
│   └── packages/apps/YourApp/
│       └── src/main/java/com/example/yourapp/
│           └── MainActivity.java          # Your app code calling vibrator.vibrate()
│
├── 🏗️ Framework API Layer
│   └── frameworks/base/
│       ├── core/java/android/os/
│       │   └── Vibrator.java              # Public Vibrator API
│       └── core/java/android/os/
│           └── VibrationEffect.java       # Vibration effects (API 26+)
│
├── 🔧 System Service Layer  
│   └── frameworks/base/
│       └── services/
│           └── core/java/com/android/server/
│               ├── SystemServer.java      # ★ Main system service starter
│               └── VibratorService.java   # ★ Core vibrator service implementation
│
├── 🌉 JNI Bridge Layer
│   └── frameworks/base/
│       └── services/core/jni/
│           └── com_android_server_VibratorService.cpp  # ★ Java ↔ C++ bridge
│
├── 💽 Hardware Abstraction Layer (HAL)
│   ├── hardware/interfaces/
│   │   └── vibrator/
│   │       ├── 1.0/                       # HAL interface definitions
│   │       │   ├── IVibrator.hal
│   │       │   └── types.hal
│   │       └── 1.3/                       # Newer versions
│   └── hardware/libhardware/modules/vibrator/
│       └── vibrator.c                     # ★ Default HAL implementation
│
├── 🐧 Kernel Layer
│   └── drivers/input/misc/
│       ├── vibrator.c                     # ★ Generic kernel driver
│       ├── qcom-vibrator.c               # Qualcomm-specific driver
│       └── .../vendor-specific drivers
│
└── 🎯 Emulator-Specific
    ├── device/generic/goldfish/
    │   └── vibrator/
    │       └── goldfish_vibrator.c        # ★ Emulator vibrator driver
    └── external/qemu/
        └── android/
            └── vibrator/                  # QEMU vibrator implementation
```


| Interface | Method | Duration (500ms) | Package Name    |
|-----------|--------|------------------|-----------------|
| Token     | ID (0) | 0x00000000000001F4 | "com.example.app" |
