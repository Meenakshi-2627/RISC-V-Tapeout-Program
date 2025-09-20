RISC-V-Tapeout-Program
---

## 🖥️ Tools Installation  

### **System Requirements**
- 6 GB RAM  
- 50 GB HDD  
- Ubuntu 20.04 or higher  
- 4 vCPU  

---

### **Merging with the sudo user group**
```bash
$ sudo usermod -aG sudo 

```

### **Yosys Installation**
```bash
$ sudo apt-get update
$ git clone https://github.com/YosysHQ/yosys.git
$ cd yosys
$ sudo apt install make
$ sudo apt-get install build-essential clang bison flex \
    libreadline-dev gawk tcl-dev libffi-dev git \
    graphviz xdot pkg-config python3 libboost-system-dev \
    libboost-python-dev libboost-filesystem-dev zlib1g-dev
$ make config-gcc
$ git submodule update --init --recursive
$ sudo apt install yosys
```


### **IVerilog Installation**
```bash
$ sudo apt-get update
$ sudo apt install iverilog
```


### **Gtkwave Installation**
```bash
$ sudo apt-get update
$ sudo apt-get install gtkwave
```

