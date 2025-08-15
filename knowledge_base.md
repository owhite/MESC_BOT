## Incremental encoder behavior & bootup problem, Role of the Z/index pulse
- A/B channels from the MT6701 without Z input causes loss of absolute position after reboot.
- CNT always starts at 0 on boot with no Z signal, so FOC_enc_oset can’t be applied meaningfully.
- One full manual rotation after boot lets MESC track position during that run but doesn’t persist across power cycles.
- the MT6701 can output a Z pulse once per revolution.
- MESC uses Z if connected and configured.
- MESC captures the Z pulse in TIM3 CCR3 to correct alignment after boot.

## Hardware pin assignments for encoder signals of the MP2 controller
- PC6 → Encoder A (TIM3_CH1)
- PC7 → Encoder B (TIM3_CH2)
- PC8 TIM3_CH3 input for Z on STM32F405.

## .ioc files
- so far chatgpt is not able to create functional .ioc files. 

