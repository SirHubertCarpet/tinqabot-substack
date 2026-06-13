# Ossyrix, The Thinking Skull : Pseudocode

The core of Ossyrix is a single cooperative loop on a one-thread microcontroller.
The rule that governs everything: NOTHING BLOCKS. Every slow job is a state machine
that the loop advances one small step per pass. The animation update runs on every
pass, so the face is always alive.

## The main loop

```
EVERY pass of the main loop (thousands of times per second):
    advanceWifi()            # one tiny step of the connection state machine
    runWatchdog_ifDue()      # periodic health check, non-blocking
    updateAnimation()        # ALWAYS runs: keeps the face moving
    updateTracking()         # may yield to an active exhibition scene
    sampleCurrent()          # overcurrent guard
    serviceWebRequests()     # handle any pending HTTP command, then return
    # never sleep, never busy-wait, always fall through to the next pass
```

## Animation: keyframes with easing

```
STRUCTURE Keyframe:
    targetAngles[4]      # jaw, headTilt, headTurn, eyes
    transitionTime       # seconds to glide into the pose
    holdTime             # seconds to stay
    easing               # SINE, CUBIC, ... shapes the motion curve
    personality          # small idle motions during the hold

FUNCTION updateAnimation():
    IF no active scene: RETURN
    kf = currentKeyframe
    elapsed = now - keyframeStartTime

    IF elapsed < kf.transitionTime:                  # gliding into pose
        progress = elapsed / kf.transitionTime
        eased = applyEasing(progress, kf.easing)
        FOR each servo s:
            angle = startAngle[s] + (kf.targetAngles[s] - startAngle[s]) * eased
            writeServo(s, angle)                     # via hysteresis below
    ELSE:                                            # holding
        applyPersonalityMotion(kf.personality)
        IF elapsed >= kf.transitionTime + kf.holdTime:
            advanceToNextKeyframe()

FUNCTION writeServo(s, angle):
    pulse = angleToPulse(angle)
    IF abs(pulse - lastPulse[s]) < HYSTERESIS:       # suppress micro-jitter
        RETURN
    setPWM(s, pulse); lastPulse[s] = pulse
```

## Non-blocking wifi state machine

```
STATES: IDLE, SCANNING, ATTEMPTING, CONNECTED, FAILED

FUNCTION advanceWifi():
    SWITCH wifiState:

      CASE IDLE:
          startScan()                                # async, returns immediately
          wifiState = SCANNING

      CASE SCANNING:
          IF scanNotFinished(): RETURN               # do NOT wait, just come back
          visible = scanResults()
          target = firstCachedNetworkAlsoIn(visible) # look before you leap
          IF target == none:
              target = nextHardcodedFallback()       # only if nothing cached is on air
          IF target == none:
              wifiState = FAILED; RETURN
          ap = strongestAccessPointFor(target)       # pin the best radio
          beginConnect(target, ap)
          attemptStart = now
          wifiState = ATTEMPTING

      CASE ATTEMPTING:
          IF isConnected():
              wifiState = CONNECTED; RETURN
          IF accessPointNotFound():                  # let it roam next time
              clearPinnedAccessPoint()
              wifiState = FAILED; RETURN
          IF now - attemptStart > ATTEMPT_TIMEOUT:   # ~45s, but non-blocking
              wifiState = FAILED
          # else: still trying, fall through and let the face keep moving

      CASE FAILED:
          wifiState = IDLE                            # loop back round and retry

      CASE CONNECTED:
          pass                                        # watchdog handles loss
```

## Watchdog

```
FUNCTION runWatchdog_ifDue():
    IF now - lastWatchdog < WATCHDOG_INTERVAL: RETURN
    lastWatchdog = now
    IF wifiState == CONNECTED and not isConnected():
        markConnectionLost(now)
    IF connectionLostFor > RECOVERY_TIMEOUT and now - lastRecovery > RECOVERY_COOLDOWN:
        radioDown(); radioUp()                        # bounce it, then cooldown
        lastRecovery = now
        wifiState = IDLE
```

## Exhibition vs tracking arbitration

```
# Exhibition scenes always win. Tracking only fills the idle gaps.
FUNCTION updateTracking():
    IF activeScene != none AND NOT activeScene.isIdleScene:
        RETURN                                        # a real performance is running: hands off
    blob = warmestThermalBlob()
    IF blob exists:
        steerHeadToward(blob)                         # only touches servos when no scene owns them
```

## Overcurrent interlock (the pain reflex)

```
FUNCTION sampleCurrent():
    amps = readServoRailCurrent()                     # real-time, from the current sensor
    IF amps > OVERCURRENT_THRESHOLD:
        disableServoOutput()                          # cut power before anything cooks
        raiseInterlock("overcurrent")
        # stays latched until a human clears it

FUNCTION clearInterlocks():                           # one button / one command
    resetOvercurrentState()
    resetFaults()
    enableServoOutput()
```

## Safe boot (no violent lurch at power-on)

```
FUNCTION safeBoot():
    disableServoOutput()                              # phase 1: outputs off
    FOR each servo s: setCenter(s)                    # phase 2: aim at center while dead
    enableServoOutput()                               # phase 3: glide gently to center
```

The shape of the whole thing: one loop, no blocking, every slow job a state machine,
and a strict pecking order whenever two impulses reach for the same servo.
