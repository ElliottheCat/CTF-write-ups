# BYUCTF 2025 – Enabled 

---

## Challenge Overview

| Field       | Value                                     |
|------------|--------------------------------------------|
| Name        | Enabled                                   |
| Category    | Pwn / Bash / Sandbox Escaping             |
| Points      | 100                                       |
| Difficulty  | Medium                                    |
| Server      | `nc enabled.chal.cyberjousting.com 1352`  |

---

## Challenge Source

```bash
#!/bin/bash

unset PATH
enable -n exec
enable -n command
enable -n type
enable -n hash
enable -n cd
enable -n enable
set +x

echo "Welcome to my new bash, sbash, the Safe Bourne Again Shell! There's no exploiting this system"

while true; do
    read -p "safe_bash> " user_input

    [[ -z "$user_input" ]] && continue

    case "$user_input" in 
        *">"*|*"<"*|*"/"*|*";"*|*"&"*|*"$"*|*"("*|*"\`"*) echo "No special characters, those are unsafe!" && continue;;
    esac

    eval "$user_input"
done
```

This is a restricted shell that:
- Disables key bash builtins like exec, cd, command, and enable.
- Blocks dangerous characters like /, &, ;, and others through pattern matching.
- Relies on eval to execute your input, but only allows what it considers "safe".
Our goal is to bypass the character filter and regain shell control to extract the flag.


## Attacking strategy

### 1.Understand the Filter

This line filters out characters commonly used for command execution:

```bash
case "$user_input" in 
    *">"*|*"<"*|*"/"*|*";"*|*"&"*|*"$"*|*"("*|*"\`"*) ...
```
The following characters are banned:
> < / ; & $ ( `

So any payload must avoid all of those.

Despite the restrictions, the shell still allows:
```text
 Alphanumerics
 Brackets [ and ]
 Bash builtins (but only the enabled ones)
    The eval function for interpreting raw commands
    The enable builtin is disabled (enable -n enable), but he pushd and popd builtins are not disabled.
```

### 2. Abuse pushd and popd for directory stack


pushd and popd are directory stack builtins. If we can traverse directories and find the flag, we can read it using echo * once inside its directory (globbing is allowed!).

```bash
nc enabled.chal.cyberjousting.com 1352
Welcome to my new bash, sbash, the Safe Bourne Again Shell! There's no exploiting this system
>> pushd ..
/ /app
>> echo *
app bin boot dev etc flag home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var
>> pushd flag
/flag / /app
>> echo *
flag.txt
>> cat flag.txt                                             (cat don't work)
/app/run: line 25: .flag.txt: No such file or directory
>> head -n 10 flag.txt                                      (head don't work)
/app/run: line 25: head: No such file or directory
>> eval flag.txt                                            (eval works!)
flag.txt: line 1: byuctf{enable_can_do_some_funky_stuff_huh?_488h33d}: command not found
history -r flag.txt
```

### Flag:
```text
byuctf{bash_builtins_are_a_double_edged_sword}
```

