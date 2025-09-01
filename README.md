# SPWM-Generator
Using TI MSPM0 make SPWM wave

┌────────────────────────────────────────────────────────────────────────────┐<br>
│                      MSPM0G (SYSCLK 32 MHz / BUSCLK 4 MHz)                 │<br>
│                                                                            │<br>
│  ┌──────────────┐      200 kHz ZERO_EVENT IRQ      ┌──────────────────┐    │<br>
│  │  TIMA0       │ ─────────────────────────────────▶│   ISR (TIMA0)   │    │<br>
│  │  Periodic    │      (Period = 19 @ 4 MHz)       │  (5 µs 주기)     │    │<br>
│  └──────────────┘                                   └───────┬──────────┘    │<br>
│                                                             │               │<br>
│                                                             ▼               │<br>
│                                     ┌────────────────────────────────────┐  │<br>
│                                     │           SPWM Core               │  │<br>
│                                     │  (센터정렬 캐리어 + 3상 비교)     │  │<br>
│  ┌──────────────────────────────┐   │                                    │  │<br>
│  │  Phase Accumulator           │   │  ┌──────────────────────────────┐  │  │<br>
│  │  phase_acc += phase_inc      │   │  │ Triangle Generator           │  │  │<br>
│  │  (phase_inc = f_sine/ISR)    │   │  │ tri_val ∈ [−TRI_TOP,+TRI_TOP]│  │  │<br>
│  └──────────────┬───────────────┘   │  │ up/down, step=TRI_STEP       │  │  │<br>
│                 │ index             │  └──────────────┬───────────────┘  │  │<br>
│                 ▼                   │                 │                  │  │<br>
│     ┌──────────────────────────┐    │   ┌─────────────▼────────────┐    │  │<br>
│     │ 3-Phase Sine LUT         │    │   │ Comparator (per-phase)   │    │  │<br>
│     │ A,B,C = sin(θ 0/±120°)   │    │   │ A_high = (A > tri_val)   │    │  │<br>
│     └──────────────────────────┘    │   │ B_high, C_high 동일       │    │  │<br>
│                                     │   └─────────────┬────────────┘    │  │<br>
│                                     │                 │                  │  │<br>
│                                     │   ┌─────────────▼────────────┐    │  │<br>
│                                     │   │ Dead-time Inserter       │    │  │<br>
│                                     │   │ both OFF → wait → ON     │    │  │<br>
│                                     │   │ (DEADTIME_CYCLES @32MHz) │    │  │<br>
│                                     │   └─────────────┬────────────┘    │  │<br>
│                                     └─────────────────┼──────────────────┘  │<br>
│                                                       │                     │<br>
│                                                       ▼                     │<br>
│                          ┌──────────────────────────────────────────┐       │<br>
│                          │  GPIOB (한 포트 원샷)                    │       │<br>
│                          │  INHA=PB9, INHB=PB2, INHC=PB3           │       │<br>
│                          │  INLA=PB7, INLB=PB6, INLC=PB0           │       │<br>
│                          └───────────────┬──────────────────────────┘       │<br>
│                                          │                                  │<br>
│                                          ▼                                  │<br>
│                         ┌────────────────────────────────────┐               │<br>
│                         │  DRV8300                           │               │<br>
│                         │  INHA/INLA → Phase A gate          │               │<br>
│                         │  INHB/INLB → Phase B gate          │               │<br>
│                         │  INHC/INLC → Phase C gate          │               │<br>
│                         │  EN_GATE (MCU GPIO)                │               │<br>
│                         │  nFAULT  (MCU GPIO in)             │               │<br>
│                         └────────────────────────────────────┘               │<br>
│                                                                            │<br>
│                         (디버그 경로)                                       │<br>
│                         ┌────────────────────────────────────┐              │<br>
│                         │ DAC0 (PA15)                        │              │<br>
│                         │ tri_val → 12bit code → DAC OUT     │              │<br>
│                         │ (IOMUX = PF_DAC0_OUT, no pull)     │              │<br>
│                         └────────────────────────────────────┘              │<br>
└────────────────────────────────────────────────────────────────────────────┘
