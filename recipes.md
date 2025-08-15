# Debug Recipes (Task‑oriented)


Short, repeatable checklists you can run during bring‑up or troubleshooting. Each includes a **CubeIDE** snippet.


---


## A) Verify encoder offset without spinning
**Goal:** Ensure `FOC.enc_offset` aligns θe correctly.


1. Terminal: `set uart_dreq 10` (lock rotor with Id)
2. Terminal: `set FOC_enc_ang 0`
3. Toggle a small `Iq` and check torque direction


**CubeIDE Debug tips**
- Live Expressions: add `FOC.enc_offset`, `FOC.enc_angle`, `FOC.Id_req`, `FOC.Iq`
- Watchpoint: `&FOC.enc_offset` (write) to confirm update
- SWV ITM log: emit `Id/Iq` per control tick (HCLK 168 MHz, SWO 2 MHz, port 0)


---


## B) Confirm phase order and sign
**Goal:** Ensure electrical zero and phase wiring yield forward torque with +Iq.


1. Lock with `Id` (e.g., `set uart_dreq 8`)
2. Apply small `Iq > 0`; observe intended direction
3. If reversed: swap any two motor phases **or** invert encoder polarity; re‑calibrate


**CubeIDE Debug tips**
- Live Expressions: `FOC.Iq`, `FOC.enc_angle`
- Peripherals → **TIM** PWM outputs and polarity
- Watchpoint on encoder counter register if using timer‑quadrature


---


## C) ADC scaling sanity check (bus & phase)
**Goal:** Verify ADC channels map and scale correctly.


1. Read raw/throttled values from terminal (`adc1`, `adc2`, etc.)
2. Compare to multimeter / bench supply
3. Adjust gain/offset constants if mismatched
## <Incremental encoder set up>


**Goal:** <Zero out encoder value.>
**Safety:** <Motor will spin, use low amperage values>


**Preconditions:**
- <Power supply connect>
- <STM32CubeIDE Connected in debugger>
- <Firmware state = MOTOR_TRACKING>
- <Required parameters known (pp_par, FOC_enc_pol)>


**Terminal commands (quick copy/paste):**
```text
# 1
<command 1, e.g., set uart_dreq 10>
# 2
<command 2, e.g., set FOC_enc_ang 0>
# 3
<command 3, optional>