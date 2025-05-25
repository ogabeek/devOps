# DevOps class

## Installation

### 1. Install Go

```bash
sudo apt-get update
sudo apt-get install -y golang
```
---

## Building

From the project root:

```bash
go build -o bin/main main.go
```

This will produce an executable at `./bin/main`.

---

## SSH / DevOps

### 1. Generate a New SSH Key

```bash
ssh-keygen -f ~/.ssh/mykey -C "your_email@example.com"
```

By default, this creates:

* Private key: `~/.ssh/mykey`
* Public key:  `~/.ssh/mykey.pub`

---

### 2. Copy Public Key to Target Server

On **your local machine**, display the public key:

```bash
cat ~/.ssh/mykey.pub
```

On the **target server**, ensure the `.ssh` directory and `authorized_keys` file exist and have proper permissions:

```bash
# Create .ssh dir if needed
mkdir -p ~/.ssh
chmod 600 ~/.ssh

# Create or append your public key
cat >> ~/.ssh/authorized_keys << 'EOF'
<paste your ~/.ssh/mykey.pub contents here>
EOF

# Secure the file
chmod 600 ~/.ssh/authorized_keys
```

---

### 3. Connect Using Your Private Key

Ensure your private key is secure:

```bash
chmod 600 ~/.ssh/mykey
```

Then connect:

```bash
ssh -i ~/.ssh/mykey user@target-host
```

