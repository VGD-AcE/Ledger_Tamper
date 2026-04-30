
# 🛡️ **Ledger_Tamper**

## 🏗️ **Project Structure**

This project contains the following components:

- **attack.py**: Attack script that patches the binary to bypass password verification.
- **demo.c**: Target C program with password verification logic.
- **Ledger_cfi.py**: GDB plugin that enforces runtime CFI checks using the ledger.
- **Ledger_generate.py**: Generates a ledger for validating the binary at runtime.

## ⚙️ **Requirements**

- **Operating System**: Ubuntu 20.04 (tested)
- **GDB**: GNU Debugger, to run the target program and inject defense logic.
- **Python 3**: Required to run the automation and attack scripts.
- **32-bit GCC toolchain**: Used to build the target program for 32-bit architecture.

## 🛠️ **Environment Setup**

On a fresh Ubuntu 20.04 system, follow these steps to set up the environment:

### 1️⃣ **Clone the repository**:
```bash
git clone https://github.com/hailin-creat/Ledger_Tamper.git
cd Ledger_Tamper
```

### 2️⃣ **Update your system**:
```bash
sudo apt update
sudo apt upgrade
```

### 3️⃣ **Install build tools**:
```bash
sudo apt install autoconf automake libtool
sudo apt install gcc python3-pip
```

### 4️⃣ **Install Python dependencies**:
```bash
pip3 install lief
```

## 🔐 **How It Works**

### 1️⃣ **Target Program (demo.c)**

The **`demo.c`** program checks for a hardcoded password (**123456**). If the password is incorrect, it refuses to perform a sensitive operation.

### 2️⃣ **Attack (attack.py)**

The **`attack.py`** script:

- Locates the **JE/JNE** conditional jump after the password check.
- Patches it with two **NOP** instructions (`0x90 0x90`).
- This bypasses the password verification, allowing any input to succeed.

### 3️⃣ **Defense (Ledger-based Defense)**

The ledger-based defense mechanism works by:

- **Ledger Generation (Ledger_generate.py)**: Creates a list of valid instruction signatures (ledger) for the binary.
- **Runtime Verification (Ledger_cfi.py)**: A GDB script that monitors instruction flow and verifies return addresses and instruction signatures against the ledger.
- If tampering is detected, the program terminates immediately.

## 🚀 **Usage**

### 1️⃣ **Build the target binary**
```bash
gcc -o demo demo.c -no-pie -fno-stack-protector -O0
```

### 2️⃣ **Run the target normally**
```bash
./demo
```
- Enter the correct password (**123456**) to execute the sensitive operation.
- Any other password will be rejected.

### 3️⃣ **Perform the attack**
```bash
python3 attack.py
```
- Now, any password will bypass verification:
```bash
./demo
```

### 4️⃣ **Enable the defense mechanism**
#### (1) Recompile the target binary:
```bash
gcc -o demo demo.c -no-pie -fno-stack-protector -O0
```

#### (2) Generate the ledger:
```bash
python3 Ledger_generate.py ./demo
```

#### (3) Try the attack again:
```bash
python3 attack.py
```

#### (4) Run with ledger verification:
```bash
gdb -q -ex "source Ledger_cfi.py" -ex "ledgercfi" --args demo
```
- The ledger-based defense will detect the tampering and terminate the program.

## ⚠️ **Notes**

- The defense uses the **lightweight PRESENT block cipher** to generate cryptographic instruction tags.
- This project is for educational and research purposes only. **Do not use it for unauthorized tampering.**

## 📜 **License**

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.
