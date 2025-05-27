# picoCTF Challenge Write-Up: WebNet0
**By:** Monish Polimetla 
**Author;** Jason
**Category:** Forensics  
**Difficulty:** Hard  
**Challenge Type:** CTF Challenge I Solved  

---

## 📝 Description

> We found this packet capture and key. Recover the flag.  
> *Hint: Try using a tool like Wireshark. How can you decrypt the TLS stream?*

In this challenge, we're given a `.pcap` file and an RSA private key. The goal is to decrypt TLS traffic to find the flag.

---

## 🔍 Walkthrough

### 1. Download the files
- `capture.pcap`
- `picopico.key`

### 2. Open Wireshark
- File → Open → `capture.pcap`

### 3. Load the Private Key
- Edit → Preferences → Protocols → TLS
- Click **Edit** next to “RSA Keys List”
- Add:
  - **IP Address**: (leave blank)
  - **Port**: (leave blank)
  - **Protocol**: `http`
  - **Key File**: `picopico.key`
  - **Password**: (leave blank)

### 4. Decrypt the Traffic
- After adding the key, TLS sessions decrypt.
- Look for **HTTP traffic**.
- Right-click → Follow → TLS Stream

### 5. Find the Flag
- The decrypted stream reveals the flag in plain text.

---

## 🏁 Flag: picoCTF{nongshim.shrimp.crackers}

