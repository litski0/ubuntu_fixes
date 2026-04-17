# X11 Key Remapping: The Ergonomic Dual-Swap (Win -> Alt -> Space)

## The Logical Bottleneck (Chained Modifiers)
When remapping multiple modifier keys simultaneously, you cannot resolve them one at a time. The X11 kernel evaluates modifier arrays dynamically. If you change `Alt_L` to `space`, but fail to completely clear and rebuild the `mod1` (Alt) and `mod4` (Super) arrays in the correct sequence, the operating system will trigger ghost modifiers (e.g., typing a space will also trigger an invisible Alt command). 

The strict architectural equation for chained swaps is: **Clear ALL affected groups $\rightarrow$ Swap ALL Keysyms $\rightarrow$ Rebuild groups.**

---

## Deployment Steps

Execute this exact sequence in your terminal to safely execute the dual-swap in a single runtime environment.

### 1. Sever All Target Modifiers
Strip the special privileges from both the Alt and Windows keys simultaneously so they become blank hardware triggers:
```bash
xmodmap -e "clear mod1"
xmodmap -e "remove mod4 = Super_L"
```

### 2. Execute the Keysym Chain
Reassign the logical character outputs. 
*(Note: Order matters. We overwrite the original Alt key first, then we overwrite the Windows key to become the new Alt).*
```bash
xmodmap -e "keysym Alt_L = space"
xmodmap -e "keysym Super_L = Alt_L"
```

### 3. Rebuild the Modifier Arrays
Now that the physical keys have their new logical identities, we must tell the OS which key is legally allowed to act as the `Alt` trigger for system shortcuts. We add our *new* Left Alt (the physical Windows key) and restore the native Right Alt:
```bash
xmodmap -e "add mod1 = Alt_L Alt_R Meta_R"
```

*(Observation: Your physical Left Alt key now exclusively types spaces. Your physical Windows key now exclusively triggers Alt shortcuts).*

---

## Persistence (Surviving a Reboot)
`xmodmap` commands are volatile and die when the X server restarts. You must dump this new, combined active map to your configuration file to make the dual-swap permanent.

Overwrite your old configuration with the current state:
```bash
xmodmap -pke > ~/.Xmodmap
```
*(Note: GNOME usually sources `~/.Xmodmap` automatically on login. If it fails, explicitly append `xmodmap ~/.Xmodmap` to your `~/.bashrc` file).*

---

## Future Independence: Core Engineering Principles
When chaining key reassignments, never map a target to a key that is currently bound to an active modifier array. By running `clear mod1` at the absolute beginning, you destroy the array structure entirely, giving you a clean mathematical slate to manipulate the `keysym` values without the X server misinterpreting your inputs. Always rebuild the array (`add mod1 = ...`) as the absolute final step in the execution chain.
