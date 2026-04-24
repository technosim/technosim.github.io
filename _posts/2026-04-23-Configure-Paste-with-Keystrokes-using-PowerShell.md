---
layout: post
title: Configure Paste with Keystrokes using PowerShell
date: 2026-04-23
categories: Tools
---
Enable paste-like behavior for systems that do not support clipboard paste but accept keyboard input such as Hypervisor console, Remote KVM etc.


## Approach
- Copy text to clipboard
- Trigger PowerShell via hotkey
- 3 second delay to activate desired paste field user context
- Script reads clipboard and types characters using SendKeys
- Clipboard is cleared after use

---

## Requirements
- Microsoft PowerToys (running)
- PowerShell script saved locally, e.g.
  C:\Scripts\PasteClipboardSendKeys.ps1  
```
{% raw %}

 function ConvertTo-SendKeysLiteralChar {
    param([char]$c)
    switch ($c) {
        '{' { return '{{}' }   # literal {
        '}' { return '{}}' }   # literal }
        '+' { return '{+}' }
        '^' { return '{^}' }
        '%' { return '{%}' }
        '~' { return '{~}' }
        '(' { return '{(}' }
        ')' { return '{)}' }
        default { return [string]$c }
    }
}
try {
    Add-Type -AssemblyName System.Windows.Forms
    # Give yourself time to click into the VMware console field
    Start-Sleep -Seconds 3
    $text = Get-Clipboard -Raw
    foreach ($c in $text.ToCharArray()) {
        [System.Windows.Forms.SendKeys]::SendWait((ConvertTo-SendKeysLiteralChar $c))
        Start-Sleep -Milliseconds 10   # tweak up if the console drops chars
    }
}
finally {
    Add-Type -AssemblyName System.Windows.Forms
    [System.Windows.Forms.Clipboard]::Clear()
}

{% endraw %}
```
---

## PowerToys Setup (Keyboard Manager)

1. Open **PowerToys** > **Keyboard Manager**
2. Select **Remap a shortcut** > **Add shortcut**
3. **Physical shortcut**: choose a hotkey
   - Example: Ctrl + Alt + V
4. **Mapped to**: Run program

### Program
```
C:\Windows\System32\WindowsPowerShell1.0\powershell.exe
```

### Arguments
```
-NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -File "C:\Scripts\PasteClipboardSendKeys.ps1"
```

### Start in (recommended)
```
C:\Scripts
```

### Options
- If running: **Start another**
- Window style: **Hidden**

Save the mapping.

---

## Usage
1. Copy text to clipboard
2. Focus the console input field
3. Press the configured hotkey
4. Script types clipboard contents via keystrokes

---

## Tuning
- Reduce initial delay if using hotkey (e.g. 500 ms)
- Increase per-character delay if keystrokes drop

---

## Notes
- PowerToys must be running
- Runs in user context
- Works where clipboard paste is blocked but typing is allowed

