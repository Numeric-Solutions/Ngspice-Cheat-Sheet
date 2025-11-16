# Ngspice Netlist Cheat Sheet

Comprehensive reference for Ngspice netlist syntax, based on Ngspice User's Manual (version 43).

## Table of Contents

- [Basic Netlist Structure](#basic-netlist-structure)
- [Passive Components](#passive-components)
- [Independent Sources](#independent-sources)
- [Dependent Sources](#dependent-sources)
- [Semiconductor Devices](#semiconductor-devices)
- [Subcircuits](#subcircuits)
- [Models](#models)
- [Analysis Commands](#analysis-commands)
- [Control Commands](#control-commands)
- [Output and Plotting](#output-and-plotting)
- [Variables and Expressions](#variables-and-expressions)
- [Examples](#examples)

---

## Basic Netlist Structure

```spice
* Title line (first line, required)
* Comments start with *

* Component definitions
<component> <nodes...> [value] [parameters]

* Control block
.control
  <commands>
.endc

* Analysis statements
.<analysis_type> [parameters]

* End of netlist
.end
```

### General Rules
- First line is the **title** (cannot be empty)
- **Case-insensitive** (except file names on Unix)
- Node names: alphanumeric, can include `_`, `+`, `-`, `.`, `#`, `@`, `!`
- Node `0` or `gnd` is the **global ground**
- Comments: lines starting with `*` or inline with `;`
- Line continuation: use `+` at the beginning of the next line
- Units: SI prefixes supported (T, G, MEG, K, M, MIL, U, N, P, F)

---

## Passive Components

### Resistor
```spice
Rxxx n+ n- [value] [model] [parameters]

R1 in out 1k
R2 n1 n2 10meg tc=0.01
R3 node1 node2 r='100+temp*0.5'
```

**Parameters:**
- `tc=val1[,val2]` - Temperature coefficients
- `temp=val` - Instance temperature
- `dtemp=val` - Temperature difference
- `m=val` - Multiplier
- `scale=val` - Scale factor
- `ac=val` - AC resistance
- `l=val, w=val` - Length and width (with model)
- `noisy=0|1` - Thermal noise (default 1)

### Capacitor
```spice
Cxxx n+ n- [value] [model] [parameters]

C1 in out 10u
C2 node1 node2 100p ic=5
C3 n1 n2 c='1n*area'
```

**Parameters:**
- `ic=val` - Initial voltage condition
- `temp=val` - Instance temperature
- `dtemp=val` - Temperature difference
- `tc=val1[,val2]` - Temperature coefficients
- `m=val` - Multiplier
- `scale=val` - Scale factor
- `l=val, w=val` - Length and width (with model)

### Inductor
```spice
Lxxx n+ n- [value] [model] [parameters]

L1 in out 100u
L2 node1 node2 1m ic=0.5
L3 n1 n2 l='10u*ratio'
```

**Parameters:**
- `ic=val` - Initial current condition
- `temp=val` - Instance temperature
- `dtemp=val` - Temperature difference
- `tc=val1[,val2]` - Temperature coefficients
- `m=val` - Multiplier
- `scale=val` - Scale factor
- `nt=val` - Number of turns (with model)

### Coupled Inductors (Transformers)
```spice
Kxxx Lyyyy Lzzzz [value]

K1 L1 L2 0.99
K2 Lpri Lsec k='coupling'
```

**Value:** Coupling coefficient (0 < k ≤ 1)

---

## Independent Sources

### Voltage Source
```spice
Vxxx n+ n- [[DC] value] [AC magnitude phase] [transient]

V1 in 0 DC 5
V2 vdd 0 12
V3 input 0 DC 0 AC 1
V4 sig 0 PWL(0 0 1n 0 2n 5 10n 5)
```

### Current Source
```spice
Ixxx n+ n- [[DC] value] [AC magnitude phase] [transient]

I1 0 out DC 1m
I2 vdd load 100u
I3 in 0 DC 0 AC 1m 90
```

### Transient Source Types

**Pulse (PULSE)**
```spice
PULSE(v1 v2 td tr tf pw per [phase])

V1 in 0 PULSE(0 5 0 1n 1n 10n 20n)
```
- `v1` - Initial value
- `v2` - Pulsed value
- `td` - Delay time
- `tr` - Rise time
- `tf` - Fall time
- `pw` - Pulse width
- `per` - Period
- `phase` - Phase shift (optional)

**Sinusoidal (SIN)**
```spice
SIN(vo va freq td theta [phase])

V1 in 0 SIN(0 1 1MEG)
V2 ac 0 SIN(2.5 2.5 1k 0 0 90)
```
- `vo` - Offset voltage
- `va` - Amplitude
- `freq` - Frequency
- `td` - Delay
- `theta` - Damping factor
- `phase` - Phase (degrees)

**Exponential (EXP)**
```spice
EXP(v1 v2 td1 tau1 td2 tau2)

V1 in 0 EXP(0 5 1n 10n 30n 50n)
```

**Piecewise Linear (PWL)**
```spice
PWL(t1 v1 t2 v2 ... tn vn) [r=val | td=val]

V1 in 0 PWL(0 0 1u 0 1.1u 5 2u 5 2.1u 0)
```

**Single Frequency FM (SFFM)**
```spice
SFFM(vo va fc mdi fs [phase])

V1 sig 0 SFFM(0 1 10k 5 1k)
```

**Amplitude Modulated (AM)**
```spice
AM(va vo mf fc [td [phase]])

V1 sig 0 AM(1 0 0.5 10k)
```

**Transient Noise (TRNOISE)**
```spice
TRNOISE(na nt [td [alpha]])

V1 noise 0 TRNOISE(0.1 10n 0)
```

**Random Voltage (TRRANDOM)**
```spice
TRRANDOM(type td ts td2 [params])

V1 rand 0 TRRANDOM(1 0 0 0 1)
```
Types: 1=uniform, 2=Gaussian, 3=exponential, 4=Poisson

---

## Dependent Sources

### Voltage-Controlled Voltage Source (VCVS)
```spice
Exxx n+ n- nc+ nc- gain

E1 out 0 in 0 10
E2 vout gnd vin gnd 2.5
```

### Current-Controlled Current Source (CCCS)
```spice
Fxxx n+ n- vname gain

F1 out 0 Vsense 50
```
Note: `vname` is a voltage source through which control current flows

### Voltage-Controlled Current Source (VCCS)
```spice
Gxxx n+ n- nc+ nc- transconductance

G1 out 0 in 0 0.001
```

### Current-Controlled Voltage Source (CCVS)
```spice
Hxxx n+ n- vname transresistance

H1 out 0 Vsense 1k
```

### Nonlinear Dependent Sources

**Polynomial (POLY)**
```spice
Bxxx n+ n- v=expr [parameters]
Bxxx n+ n- i=expr [parameters]

B1 out 0 v='v(in)*v(in)'
B2 out 0 i='abs(v(vdd)-v(vss))*1m'
B3 vout gnd v='v(in)>2.5 ? 5 : 0'
```

**Expressions can include:**
- Arithmetic: `+`, `-`, `*`, `/`, `^`, `%`
- Functions: `abs()`, `sqrt()`, `exp()`, `ln()`, `log()`, `sin()`, `cos()`, `tan()`, etc.
- Conditionals: `? :`, `>`, `<`, `>=`, `<=`, `==`, `!=`
- Constants: `pi`, `e`, `c` (speed of light), `i` (imaginary unit)
- Circuit values: `v(node)`, `i(vname)`, `v(n1,n2)`, `time`, `temper`

---

## Semiconductor Devices

### Diode
```spice
Dxxx n+ n- modelname [area] [parameters]

D1 anode cathode DMOD
D2 in out 1N4148 area=2
.model DMOD D (is=1e-14 n=1.8)
```

**Parameters:**
- `area=val` - Area multiplier
- `pj=val` - Perimeter
- `temp=val` - Temperature
- `ic=val` - Initial voltage
- `m=val` - Multiplier

### BJT (Bipolar Junction Transistor)
```spice
Qxxx nc nb ne [ns] modelname [area] [parameters]

Q1 coll base emit QNPN
Q2 c b e substrate QPNP area=2
.model QNPN NPN (bf=100 is=1e-14)
.model QPNP PNP (bf=50)
```

**Nodes:** collector, base, emitter, [substrate]

**Parameters:**
- `area=val`, `areab=val`, `areac=val`
- `temp=val`, `dtemp=val`
- `ic=vbe,vce` - Initial conditions
- `m=val` - Multiplier

### MOSFET
```spice
Mxxx nd ng ns nb modelname [parameters]

M1 drain gate source bulk NMOS w=10u l=1u
M2 d g s b PMOS w=20u l=0.5u m=2
.model NMOS NMOS (level=1 vto=0.7)
.model PMOS PMOS (level=1 vto=-0.7)
```

**Nodes:** drain, gate, source, bulk

**Parameters:**
- `w=val` - Width
- `l=val` - Length
- `ad=val`, `as=val` - Drain/source area
- `pd=val`, `ps=val` - Drain/source perimeter
- `nrd=val`, `nrs=val` - Number of squares
- `temp=val`, `dtemp=val`
- `ic=vds,vgs,vbs` - Initial conditions
- `m=val` - Multiplier

### JFET
```spice
Jxxx nd ng ns modelname [area] [parameters]

J1 drain gate source JMOD
.model JMOD NJF (vto=-2 beta=1e-3)
.model JPMOD PJF (vto=2)
```

### MESFET
```spice
Zxxx nd ng ns modelname [area] [parameters]

Z1 drain gate source ZMOD
.model ZMOD NMF (vto=-2 beta=1e-3)
```

---

## Subcircuits

### Definition
```spice
.subckt subckt_name n1 n2 n3... [params: param1=val1 param2=val2]
  * Subcircuit components
.ends [subckt_name]

.subckt opamp inp inn out vdd vss
  * Internal circuit
  Rin inp inn 1Meg
  Rout out 0 50
  Eout out 0 inp inn 100k
.ends opamp
```

### Instantiation
```spice
Xxxx n1 n2 n3... subckt_name [params: param1=val1...]

X1 vin+ vin- vout vcc vee opamp
X2 in+ in- out 15 -15 opamp
```

### Hierarchical Subcircuits
Subcircuits can contain other subcircuit calls (nested subcircuits).

---

## Models

### Syntax
```spice
.model modelname type (param1=val1 param2=val2...)

.model DMOD D (is=1e-14 n=1.8 cjo=2p)
.model QMOD NPN (bf=100 is=1e-15 vaf=100)
.model NMOS NMOS (level=1 vto=0.7 kp=20u)
```

### Common Device Types
- **R** - Resistor
- **C** - Capacitor
- **L** - Inductor
- **D** - Diode
- **NPN/PNP** - BJT
- **NMOS/PMOS** - MOSFET
- **NJF/PJF** - JFET
- **NMF/PMF** - MESFET
- **SW** - Voltage-controlled switch
- **CSW** - Current-controlled switch

### MOSFET Levels
- `level=1` - Shichman-Hodges
- `level=2` - Grove-Frohman (geometric)
- `level=3` - Semi-empirical short channel
- `level=6` - MOS6
- `level=8` - BSIM1
- `level=9` - BSIM2
- `level=14` - BSIM3v3
- `level=49` - BSIM3v3 (same as 14)
- `level=54` - BSIM4
- `level=57` - BSIM3 SOI
- `level=58` - BSIM4 SOI
- `level=68` - BSIM6

---

## Analysis Commands

### Operating Point (DC)
```spice
.op

.op
```
Calculates DC operating point.

### DC Analysis (Sweep)
```spice
.dc srcname start stop step [src2 start2 stop2 step2]

.dc Vin 0 5 0.1
.dc Vdd 0 10 1 Temp -25 75 25
```

### AC Analysis
```spice
.ac dec|oct|lin points start_freq stop_freq

.ac dec 10 1 100MEG
.ac lin 100 1k 10k
.ac oct 8 1 100k
```

### Transient Analysis
```spice
.tran tstep tstop [tstart [tmax]] [uic]

.tran 1n 100n
.tran 0.1u 10u 0 0.01u
.tran 1n 1u uic
```
- `tstep` - Suggested time step
- `tstop` - Stop time
- `tstart` - Start time for plotting (default 0)
- `tmax` - Maximum time step
- `uic` - Use initial conditions (skip DC operating point)

### Fourier Analysis
```spice
.four freq v(node) | i(source)

.four 1k v(out)
```

### Noise Analysis
```spice
.noise v(output [,ref]) source (dec|oct|lin) points fstart fstop [pts_per_summary]

.noise v(out) Vin dec 10 1k 100MEG
```

### Distortion Analysis
```spice
.disto [rfreq [sfreq]]

.disto
```

### Transfer Function
```spice
.tf outvar insrc

.tf v(out) Vin
.tf i(Vload) Vin
```

### Sensitivity Analysis
```spice
.sens outvar [ac|dc]

.sens v(out)
.sens v(5,3) ac
```

### Pole-Zero Analysis
```spice
.pz node1 node2 node3 node4 cur|vol pol|zer|pz

.pz 1 0 3 0 vol pz
```

---

## Control Commands

### Control Block
```spice
.control
  <commands>
.endc
```

### Common Control Commands

**Run simulation:**
```spice
run
```

**Print values:**
```spice
print [all | var1 var2 ...]
print v(out) i(Vdd) v(in,out)
print all
```

**Plot:**
```spice
plot [var1 var2 ...]
plot v(out) v(in)
plot vdb(out)
plot vm(out) vp(out)
```

**Display available vectors:**
```spice
display
```

**Set options:**
```spice
set option=value
set temp=27
set hcopydevtype=postscript
```

**Let (define variable):**
```spice
let result = v(out) - v(in)
let gain = vdb(out) - vdb(in)
```

**Measurement:**
```spice
meas [tran|ac|dc] result_name meas_type args

meas tran trise TRIG v(out) VAL=0.5 RISE=1 TARG v(out) VAL=4.5 RISE=1
meas ac gain3db WHEN vdb(out)='vdb(out,start)-3'
meas tran avgpower AVG power FROM=0 TO=10n
```

**Loop:**
```spice
foreach var val1 val2 val3
  alter R1 = $var
  run
  print v(out)
end
```

**Conditional:**
```spice
if condition
  <commands>
else
  <commands>
end
```

**Save/Load:**
```spice
save var1 var2
save all
write filename.raw
load filename.raw
```

**Reset:**
```spice
reset
destroy all
```

---

## Output and Plotting

### Save Directive
```spice
.save [all] [vector1 vector2 ...]

.save v(out) i(Vdd)
.save all
```

### Print Directive
```spice
.print analysis_type [vector1 vector2 ...]

.print tran v(out) i(Vin)
.print ac vdb(out) vp(out)
.print dc v(3) v(4) v(5)
```

### Plot Functions
- `v(node)` - Voltage at node
- `v(n1,n2)` - Voltage between nodes
- `i(source)` - Current through source
- `vdb(node)` - Voltage in dB (20*log10)
- `vm(node)` - Voltage magnitude
- `vp(node)` - Voltage phase (degrees)
- `vr(node)` - Real part
- `vi(node)` - Imaginary part

### Mathematical Operations
```spice
plot v(out)*2
plot v(out)-v(in)
plot db(v(out)/v(in))
plot ph(v(out))
```

---

## Variables and Expressions

### Setting Variables
```spice
.param param_name = expression

.param vdd = 5
.param rload = 1k
.param gain = {vdd / 2.5}
.param width = {10u * scale_factor}
```

### Using Variables
```spice
V1 vdd 0 {vdd}
R1 out 0 {rload}
M1 d g s b nch w={width} l=1u
```

### Global Parameters
```spice
.global vdd gnd

.param vdd=5
V1 vdd 0 {vdd}
```

### Temperature
```spice
.temp temp1 [temp2 temp3 ...]

.temp 27
.temp 0 27 75 125
```

### Options
```spice
.option opt1 opt2=value

.option nopage
.option reltol=1e-4
.option tnom=27
.option method=gear
```

**Common Options:**
- `abstol=val` - Absolute current tolerance (default 1pA)
- `reltol=val` - Relative tolerance (default 0.001)
- `vntol=val` - Voltage tolerance (default 1uV)
- `tnom=val` - Nominal temperature (default 27°C)
- `temp=val` - Operating temperature
- `gmin=val` - Minimum conductance (default 1e-12)
- `method=trap|gear` - Integration method
- `maxord=val` - Maximum order for gear (2-6)
- `nopage` - Suppress paging
- `nomod` - Suppress model parameter printing

---

## Examples

### Simple Resistor Divider
```spice
* Resistor Divider
Vin in 0 DC 10
R1 in out 1k
R2 out 0 1k
.op
.print dc v(out)
.end
```

### RC Low-Pass Filter
```spice
* RC Low-Pass Filter
Vin in 0 DC 0 AC 1
R1 in out 1k
C1 out 0 1u
.ac dec 20 1 100k
.print ac vdb(out) vp(out)
.control
  run
  plot vdb(out) vp(out)
.endc
.end
```

### MOSFET Amplifier
```spice
* NMOS Common Source Amplifier
Vdd vdd 0 DC 5
Vin in 0 DC 2.5 AC 0.01
M1 out in 0 0 NMOS w=10u l=1u
Rd vdd out 10k
.model NMOS NMOS (vto=1 kp=100u)
.op
.ac dec 20 1 100MEG
.control
  run
  print v(out)
  plot vdb(out)
.endc
.end
```

### Transient Pulse Response
```spice
* Pulse Response
Vin in 0 PULSE(0 5 0 1n 1n 10n 20n)
R1 in out 1k
C1 out 0 1n
.tran 0.1n 50n
.control
  run
  plot v(in) v(out)
.endc
.end
```

### OpAmp Using Subcircuit
```spice
* OpAmp Circuit
.subckt opamp inp inn out vdd vss
  Rin inp inn 1Meg
  Eout out 0 inp inn 100k
  Rout out 0 50
.ends

Vdd vdd 0 15
Vss vss 0 -15
Vin inp 0 AC 1
Vref inn 0 0

X1 inp inn out vdd vss opamp
Rf out inn 100k
Ri inp inn 10k

.ac dec 20 1 1MEG
.control
  run
  plot vdb(out)
.endc
.end
```

### Parametric Sweep
```spice
* Parametric Component Sweep
.param rval = 1k

Vin in 0 DC 10
R1 in out {rval}
R2 out 0 1k

.control
  foreach resistance 500 1k 2k 5k 10k
    alter R1 = $resistance
    op
    print v(out)
  end
.endc
.end
```

### BJT Amplifier with DC Sweep
```spice
* BJT Amplifier DC Sweep
Vcc vcc 0 DC 12
Vin base 0 DC 0
Rc vcc coll 2k
Q1 coll base 0 QNPN
.model QNPN NPN (bf=100 is=1e-15)
.dc Vin 0 1 0.01
.control
  run
  plot v(coll)
.endc
.end
```

---

## Additional Resources

- **Ngspice Official Documentation:** [ngspice.sourceforge.net](http://ngspice.sourceforge.net/)
- **Ngspice User's Manual (v43):** Available from the official website
- **SPICE Model Libraries:** Check device manufacturer websites for SPICE models

---

## Quick Reference Card

| Component | Syntax | Example |
|-----------|--------|---------|
| Resistor | `Rxxx n+ n- value` | `R1 1 0 1k` |
| Capacitor | `Cxxx n+ n- value` | `C1 1 0 10u` |
| Inductor | `Lxxx n+ n- value` | `L1 1 0 1m` |
| V-Source | `Vxxx n+ n- value` | `V1 1 0 5` |
| I-Source | `Ixxx n+ n- value` | `I1 0 1 1m` |
| VCVS | `Exxx n+ n- nc+ nc- gain` | `E1 2 0 1 0 10` |
| VCCS | `Gxxx n+ n- nc+ nc- value` | `G1 2 0 1 0 0.01` |
| CCCS | `Fxxx n+ n- Vyyy value` | `F1 2 0 V1 10` |
| CCVS | `Hxxx n+ n- Vyyy value` | `H1 2 0 V1 100` |
| Diode | `Dxxx n+ n- model` | `D1 1 0 DMOD` |
| BJT | `Qxxx c b e model` | `Q1 2 1 0 QNPN` |
| MOSFET | `Mxxx d g s b model` | `M1 2 1 0 0 NMOS` |
| JFET | `Jxxx d g s model` | `J1 2 1 0 JMOD` |
| Subckt | `Xxxx nodes... subckt` | `X1 1 2 3 opamp` |

| Analysis | Syntax | Example |
|----------|--------|---------|
| OP | `.op` | `.op` |
| DC | `.dc src start stop step` | `.dc V1 0 5 0.1` |
| AC | `.ac type pts fstart fstop` | `.ac dec 10 1 100k` |
| TRAN | `.tran step stop` | `.tran 1n 100n` |

---

**Version:** Based on Ngspice User's Manual version 43  
**License:** This cheat sheet is provided as-is for educational and reference purposes.
