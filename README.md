# ZSA Voyager — Yanko's Keymap

Personal QMK keymap for the ZSA Voyager with Navigator trackball.
Built on [zsa/qmk_firmware](https://github.com/zsa/qmk_firmware) `firmware25` branch.

---

## Features

- QWERTY base with home row mods (`LGUI/A`, `LALT/S`, `LCTL/D`, `LSFT/F` and mirrors on right)
- Colemak layer (layer 4)
- Symbols layer (layer 1)
- Macros + Mac shortcuts layer (layer 2)
- RGB + media + navigation layer (layer 3)
- Mouse + numpad layer (layer 5, activated by holding `W`)
- Navigator trackball with automouse layer (layer 6)
- Mouse jiggler — keeps screen awake, LED 13 glows bright green when active
- Secret macros `ST_MACRO_2` / `ST_MACRO_3` injected at compile time via env vars (never stored in repo)
- Chordal hold enabled

---

## Repo structure

This keymap lives as a standalone repo, separate from the QMK framework folder.

```
voyager-keymap/
└── keyboards/
    └── zsa/
        └── voyager/
            └── keymaps/
                └── yanko/
                    ├── keymap.c       <- layout + macro logic + RGB indicators
                    ├── keymap.json    <- QMK modules declaration
                    ├── config.h       <- feature overrides (automouse, navigator, etc.)
                    ├── rules.mk       <- feature flags + SECRET OPT_DEFS
                    ├── i18n.h         <- Mac key shortcut aliases
                    ├── .gitignore     <- excludes .env.sh
                    └── .env.sh        <- LOCAL ONLY, never committed — holds SECRET_1/SECRET_2
```

The folder structure mirrors `~/qmk_firmware` so QMK can find the keymap via the overlay config (`qmk config user.overlay_dir`).

---

## Platform setup

---

### macOS M4

#### 1. Install QMK CLI

```bash
pip3 install qmk
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

#### 2. ARM toolchain PATH fix (required on M4)

Homebrew installs the ARM compiler but does not auto-link it. Both `gcc` and `binutils` live in different paths and both are required:

```bash
echo 'export PATH="/opt/homebrew/opt/arm-none-eabi-gcc@8/bin:$PATH"' >> ~/.zshrc
echo 'export PATH="/opt/homebrew/opt/arm-none-eabi-binutils/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

#### 3. Clone the ZSA firmware25 repo

```bash
qmk setup zsa/qmk_firmware -b firmware25
```

Say yes to all prompts. Clones to `~/qmk_firmware`.

#### 4. Clone this keymap repo and configure overlay

```bash
git clone git@github.com:yourusername/voyager-keymap.git ~/voyager-keymap
qmk config user.overlay_dir="$HOME/voyager-keymap"
```

Verify:
```bash
qmk config user.overlay_dir
```

#### 5. Recreate `.env.sh` (never committed — do this on every new machine)

```bash
cat > ~/voyager-keymap/keyboards/zsa/voyager/keymaps/yanko/.env.sh << 'EOF'
export SECRET_1='your_secret_1'
export SECRET_2='your_secret_2'
EOF
```

#### 6. Build

```bash
cd ~/qmk_firmware

# With secrets via .env.sh (recommended)
source ~/voyager-keymap/keyboards/zsa/voyager/keymaps/yanko/.env.sh && make zsa/voyager:yanko

# With secrets inline
SECRET_1='your_secret_1' SECRET_2='your_secret_2' make zsa/voyager:yanko

# Without secrets (ST_MACRO_2 and ST_MACRO_3 will send empty strings)
make zsa/voyager:yanko
```

> Always use single quotes around secrets — bash misinterprets `!`, `@`, `$` inside double quotes.

Output `.bin`:
```
~/qmk_firmware/zsa_voyager_yanko.bin
```

#### 7. Flash

**Keymapp (GUI):**
1. Open Keymapp → Flash
2. Browse to `~/qmk_firmware/zsa_voyager_yanko.bin`
3. Press reset button on top edge of keyboard (near the `3` key)

**dfu-util CLI:**
```bash
# Put keyboard in reset mode first, then:
dfu-util -d 3297:1969 -a 0 -s 0x08000000:leave -D ~/qmk_firmware/zsa_voyager_yanko.bin
```

**Build + flash in one step:**
```bash
cd ~/qmk_firmware
source ~/voyager-keymap/keyboards/zsa/voyager/keymaps/yanko/.env.sh && qmk flash -kb zsa/voyager -km yanko
```

---

### Windows — WSL2 (Ubuntu 24.04)

#### 1. Install dependencies

```bash
sudo apt update && sudo apt install -y \
  gcc-arm-none-eabi binutils-arm-none-eabi \
  avr-libc binutils-avr gcc-avr \
  avrdude dfu-util dfu-programmer dos2unix

pip3 install qmk --break-system-packages
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

#### 2. Clone the ZSA firmware25 repo

```bash
qmk setup zsa/qmk_firmware -b firmware25
```

Say yes to all prompts. Clones to `~/qmk_firmware`.

> **Note on QMK Doctor warnings:** WSL will show warnings about AVR tools — irrelevant for Voyager. As long as you see `Found arm-none-eabi-gcc` and `Successfully compiled using arm-none-eabi-gcc` you are good.

#### 3. Clone this keymap repo and configure overlay

```bash
git clone git@github.com:yourusername/voyager-keymap.git ~/voyager-keymap
qmk config user.overlay_dir="$HOME/voyager-keymap"
```

Verify:
```bash
qmk config user.overlay_dir
```

#### 4. Recreate `.env.sh` (never committed — do this on every new machine)

```bash
cat > ~/voyager-keymap/keyboards/zsa/voyager/keymaps/yanko/.env.sh << 'EOF'
export SECRET_1='your_secret_1'
export SECRET_2='your_secret_2'
EOF
```

#### 5. Build

```bash
cd ~/qmk_firmware

# With secrets via .env.sh (recommended)
source ~/voyager-keymap/keyboards/zsa/voyager/keymaps/yanko/.env.sh && make zsa/voyager:yanko

# With secrets inline
SECRET_1='your_secret_1' SECRET_2='your_secret_2' make zsa/voyager:yanko

# Without secrets (ST_MACRO_2 and ST_MACRO_3 will send empty strings)
make zsa/voyager:yanko
```

> Always use single quotes around secrets — bash misinterprets `!`, `@`, `$` inside double quotes.

Output `.bin`:
```
~/qmk_firmware/zsa_voyager_yanko.bin
```

Access from Windows Explorer or Keymapp:
```
\\wsl$\Ubuntu\home\yanko\qmk_firmware\zsa_voyager_yanko.bin
```

#### 6. Flash

**Keymapp (GUI — recommended for WSL):**
1. Open Keymapp on Windows → Flash
2. Navigate to `\\wsl$\Ubuntu\home\yanko\qmk_firmware\zsa_voyager_yanko.bin`
3. Press reset button on top edge of keyboard (near the `3` key)

**dfu-util CLI in WSL:**
```bash
# Requires usbipd-win to attach USB device to WSL — Keymapp is easier
dfu-util -d 3297:1969 -a 0 -s 0x08000000:leave -D ~/qmk_firmware/zsa_voyager_yanko.bin
```

**Build + flash in one step:**
```bash
cd ~/qmk_firmware
source ~/voyager-keymap/keyboards/zsa/voyager/keymaps/yanko/.env.sh && qmk flash -kb zsa/voyager -km yanko
```

---

### Windows — QMK MSYS

Download and install QMK MSYS from [github.com/qmk/qmk_distro_msys](https://github.com/qmk/qmk_distro_msys/releases).

> **Important:** QMK MSYS does not support `make` directly. Always use `qmk compile` instead.

#### 1. Clone the ZSA firmware25 repo

Open QMK MSYS terminal, then:

```bash
qmk setup zsa/qmk_firmware -b firmware25
```

Say yes to all prompts. Clones to `C:/Users/yanko/qmk_firmware`.

> QMK Doctor will warn about avr-gcc version — irrelevant for Voyager. As long as `Found arm-none-eabi-gcc` appears you are good.

#### 2. Clone this keymap repo and configure overlay

```bash
git clone git@github.com:yourusername/voyager-keymap.git C:/Users/yanko/voyager-keymap
qmk config user.overlay_dir="C:/Users/yanko/voyager-keymap"
```

Verify:
```bash
qmk config user.overlay_dir
```

#### 3. Recreate `.env.sh` (never committed — do this on every new machine)

```bash
cat > C:/Users/yanko/voyager-keymap/keyboards/zsa/voyager/keymaps/yanko/.env.sh << 'EOF'
export SECRET_1='your_secret_1'
export SECRET_2='your_secret_2'
EOF
```

#### 4. Build

```bash
# With secrets via .env.sh (recommended)
source C:/Users/yanko/voyager-keymap/keyboards/zsa/voyager/keymaps/yanko/.env.sh
qmk compile -kb zsa/voyager -km yanko

# With secrets inline (single command)
SECRET_1='your_secret_1' SECRET_2='your_secret_2' qmk compile -kb zsa/voyager -km yanko

# Without secrets (ST_MACRO_2 and ST_MACRO_3 will send empty strings)
qmk compile -kb zsa/voyager -km yanko
```

> Always use single quotes around secrets — bash misinterprets `!`, `@`, `$` inside double quotes.

Output `.bin`:
```
C:/Users/yanko/qmk_firmware/zsa_voyager_yanko.bin
```

#### 5. Flash

**Keymapp (GUI — recommended):**
1. Open Keymapp → Flash
2. Browse to `C:/Users/yanko/qmk_firmware/zsa_voyager_yanko.bin`
3. Press reset button on top edge of keyboard (near the `3` key)

**dfu-util CLI:**
```bash
# Put keyboard in reset mode first, then:
dfu-util -d 3297:1969 -a 0 -s 0x08000000:leave -D C:/Users/yanko/qmk_firmware/zsa_voyager_yanko.bin
```

**Build + flash in one step:**
```bash
source C:/Users/yanko/voyager-keymap/keyboards/zsa/voyager/keymaps/yanko/.env.sh
qmk flash -kb zsa/voyager -km yanko
```

---

## Verify secrets baked into binary

After building, confirm the strings actually made it in before flashing:

```bash
# macOS / WSL
strings ~/qmk_firmware/zsa_voyager_yanko.bin | grep "your_secret"

# Windows MSYS
strings C:/Users/yanko/qmk_firmware/zsa_voyager_yanko.bin | grep "your_secret"
```

If nothing is returned the env vars did not reach the compiler — check that `rules.mk` has the `OPT_DEFS` lines (see Secret macros section below).

---

## Secret macros

Strings are injected at compile time via environment variables and never stored in the repo.

`rules.mk` passes them to the compiler:
```makefile
OPT_DEFS += -DSECRET_1="\"$(SECRET_1)\""
OPT_DEFS += -DSECRET_2="\"$(SECRET_2)\""
```

`keymap.c` uses them:
```c
case ST_MACRO_2:
    if (record->event.pressed) { SEND_STRING(SECRET_1); }
    break;
case ST_MACRO_3:
    if (record->event.pressed) { SEND_STRING(SECRET_2); }
    break;
```

`.env.sh` holds the values locally (gitignored):
```bash
export SECRET_1='your_secret_1'
export SECRET_2='your_secret_2'
```

> Firmware strings are readable by anyone with physical keyboard access via a hex editor.
> Use for emails, usernames, boilerplate. **Never use for passwords.**

---

## Mouse jiggler

Keeps screen awake and Teams status green. Toggle with `KC_MS_JIGGLER_TOGGLE` on layer 5.

**LED indicator:** LED 13 (the `A` key on the left half) glows bright green when active — visible on any layer.

The jiggler moves the cursor by a tiny amount at regular intervals — intentionally subtle. To confirm it is working, leave the machine idle past your normal sleep timeout and check if the screen stays awake.

---

## QMK modules (`keymap.json`)

```json
{
    "modules": [
        "zsa/oryx",
        "zsa/mousejiggler",
        "zsa/navigator_trackball",
        "zsa/automouse",
        "zsa/defaults"
    ]
}
```

---

## Navigator / automouse (`config.h`)

```c
#define AUTOMOUSE_LAYER 6
#define AUTOMOUSE_THRESHOLD 10
#define NAVIGATOR_SCROLL_DIVIDER 50
```

---

## New machine checklist

### macOS M4
- [ ] `pip3 install qmk` and add `~/.local/bin` to PATH in `~/.zshrc`
- [ ] Add both ARM toolchain paths to `~/.zshrc`
- [ ] `qmk setup zsa/qmk_firmware -b firmware25`
- [ ] `git clone ... ~/voyager-keymap`
- [ ] `qmk config user.overlay_dir="$HOME/voyager-keymap"`
- [ ] Recreate `.env.sh` with secrets
- [ ] `source .env.sh && make zsa/voyager:yanko`
- [ ] `strings` verify, then flash

### Windows WSL2
- [ ] `sudo apt install` ARM toolchain + avrdude + dfu-util + dos2unix
- [ ] `pip3 install qmk --break-system-packages` and add `~/.local/bin` to PATH in `~/.bashrc`
- [ ] `qmk setup zsa/qmk_firmware -b firmware25`
- [ ] `git clone ... ~/voyager-keymap`
- [ ] `qmk config user.overlay_dir="$HOME/voyager-keymap"`
- [ ] Recreate `.env.sh` with secrets
- [ ] `source .env.sh && make zsa/voyager:yanko`
- [ ] `strings` verify, flash via Keymapp using `\\wsl$\Ubuntu\...` path

### Windows MSYS
- [ ] Install QMK MSYS from github.com/qmk/qmk_distro_msys
- [ ] `qmk setup zsa/qmk_firmware -b firmware25`
- [ ] `git clone ... C:/Users/yanko/voyager-keymap`
- [ ] `qmk config user.overlay_dir="C:/Users/yanko/voyager-keymap"`
- [ ] Recreate `.env.sh` with secrets
- [ ] `source .env.sh && qmk compile -kb zsa/voyager -km yanko` (not `make`)
- [ ] `strings` verify, then flash via Keymapp