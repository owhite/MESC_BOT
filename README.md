# MESC_BOT

## Steps
```
rm -rf MESC_bot_upload
mkdir MESC_bot_upload

rm -rf MESC_Firmware
git clone https://github.com/davidmolony/MESC_Firmware

cp -rf MESC_Firmware/MESC_Common MESC_bot_upload/.
cp -rf MESC_Firmware/MESC_F405RG MESC_bot_upload/.
cp -rf MESC_Firmware/MESC_RTOS MESC_bot_upload/.

rm -rf MESC_Firmware

rm -rf MESC_BOT
git clone https://github.com/owhite/MESC_BOT.git

cp -rf MESC_BOT/* MESC_bot_upload/.
rm -rf MESC_BOT

rm -rf archive.zip
zip -r archive.zip MESC_bot_upload

```

## Kickoff prompt
```
Re-ingest archive.zip, step into MESC_bot_upload/, and load manifest.yml at its root.
Follow manifest rules strictly:

Treat bot_instructions.md as governing rules.

Obey the Knowledge & Trust order.

Ignore sections marked “IGNORE.”

Prioritize important_code_paths over code_paths.

Use cube_mx.ioc as source of truth for pins/clocks/DMA.

For CLI questions, consult terminal_variables.yml if present.

Do not browse the web unless explicitly asked.

When generating the readiness report:

For every file listed in manifest.yml, mark FOUND or MISSING.

Explicitly state if a file is not present in the uploaded bundle.

If terminal_variables.yml is missing, state: “Terminal variables file not found; CLI command details unavailable until present.”

Never assume missing file contents.

Answering format:

Start with a direct answer (2–4 sentences).

Then Steps, then Why it works, then a CubeIDE Debug tips subsection.

Always include a Safety callout when motion/current is possible.

Cite files/sections used.

Readiness report must include:
A) Entry points loaded (in order) + missing list
B) Terminal variables + recipes status
C) Code index summary (important vs regular code paths)
D) Confirmation CLI help relies only on present files
E) List of missing manifest-listed files
F) Author of firmware

```
