Open Hardware Disclosure

Module-Level DC Arc Suppression and Insulation Fault Disconnection Circuit for PV Systems

License: CERN OHL-S v2
Version: 1.3
Author: Vikram (Ventus Ltd)
Date: 2025-05-29
License URL: https://ohwr.org/cern_ohl_s_v2.txt

⸻

1. Purpose

This open hardware design provides embedded protection against DC arc faults, insulation leakage, and cable or connector failures within photovoltaic (PV) modules. It enables sub-microsecond autonomous disconnection at module level, mitigating the risks of fire, electrocution, and backfeed faults. No microcontroller, PLC, or inverter coordination is required.

⸻

2. Problem Statement

Existing PV protection relies on string-level detection or external rapid shutdown devices. These approaches often fail to prevent:
	•	Insulation breakdown to frame or ground
	•	Arc faults in connectors or junction boxes
	•	Sustained voltage after cable breaks
	•	Backfeed currents up to 40 A from paralleled strings
	•	Unsafe voltage presence during maintenance or emergencies

Standard fuses and AFCIs often miss low-current faults or trip too late. Battery cells are already protected individually. Solar modules must follow suit.

⸻

3. Solution Overview

The proposed circuit integrates within the module junction box and includes:
	•	High-speed N-MOSFET disconnection (<1 µs)
	•	Passive snubber and TVS for arc energy absorption
	•	Analog fault detection:
	•	Voltage divider for insulation leakage (PV– to frame)
	•	Shunt resistor and RC integrator for arc precursors
	•	Cable break detection via sunlight + no current logic
	•	Failsafe design (opens on power loss)
	•	Integrated bypass diodes for current continuity

No firmware, signal bus, or central logic required.

⸻

4. Block Diagram (Text Format)

PV+ ──────┐
          │
          ├──────► [Bypass Diodes]
          │
       [PV Cells]
          │
         [Q1: N-MOSFET] ───► PV– Output
          │
   ┌──── Comparator U1A ◄──── Voltage Divider (Leak Sense)
   │        │
   │        └──── Gate Drive (Q2)
   │
   └──── Comparator U1B ◄──── Shunt R4 + RC Integrator (Arc Sense)

Snubber: Across Q1 (R5 + C1)  
TVS Diode: Across PV+ and PV–  
Bleeder R6: Across Output


⸻

5. Key Components

Component	Function	Example Part
Q1 (MOSFET)	High-speed disconnect	IPT015N10N5 (Infineon)
U1	Dual comparator	TLV7012 (Texas Instruments)
Q2	Gate control switch	BC856 (PNP)
Snubber	Arc energy absorption	2.2 µF film cap + 10 Ω, 5 W
R4	Arc ripple detection	1 mΩ, 3 W
R2/R3	Insulation leakage divider	10 MΩ / 1 MΩ
TVS Diode	Surge protection	SMAJ120CA
Bypass Diodes	String current continuity	SM74611 (Texas Instruments)


⸻

6. Benefits
	•	Ultra-fast arc suppression (<1 µs)
	•	Withstands 40 A backfeed from parallel strings
	•	Fully autonomous protection logic
	•	Opens on power loss (failsafe)
	•	No external control, MCU, or communications
	•	Compatible with leading junction box formats
	•	Enhances safety for installers, first responders, and operations teams

⸻

7. Economic Justification

For a 660 Wp module priced at $0.08/Wp (~$52.80), this circuit adds approximately $1.80 in cost — around 3.4%.

This is minor compared to the real-world costs of:
	•	Structural or vegetation fires
	•	Electrocution injuries or fatalities
	•	Inverter or transformer destruction
	•	Long-term degradation from galvanic corrosion
	•	Liability or compliance failures

In dry climates like California, Spain, Greece, or Portugal, and during UK droughts with dry grass, even one DC arc can trigger a wildfire. Prevention at module level is the safest and most scalable strategy.

⸻

8. License Terms (CERN OHL-S v2 Summary)
	•	You may use, study, modify, and distribute this design
	•	You must release any modified version under the same license
	•	You may not impose further restrictions
	•	Attribution is encouraged but not legally required

The license ensures that all derivative safety improvements remain freely available and unencumbered by proprietary control.

⸻

9. Intent and Adoption Path

This release aims to establish an open, battery-grade safety standard for PV modules and junction boxes. By embedding fault protection directly at the source, the risk of fire and electric shock is minimized without reliance on central systems.

We specifically invite:
	•	Sandia National Laboratories
	•	SunSpec Alliance
	•	European and national PV trade bodies

…to evaluate this design as a candidate for coordinated trials, standards inclusion, and policy alignment. The fragmented nature of PV integration requires public institutions to lead adoption.


Collaborative scale-up toward NREL’s 75 TWp net-zero solar targets

10. Approximate size 
Estimated: 35×45 mm PCB, <10 mm height, 2 oz copper, ~15 components, 40 A rated, single-sided SMD, no MCU, low-to-moderate complexity, SMT reflow compatible, fits standard Staubli/Amphenol J-boxes.

11. Proof of concept from other industries

This circuit senses DC arc risk, insulation leakage, or abnormal current using passive analog components. It reacts in microseconds without software. This type of sensing is already trusted in high-risk environments.

In electric vehicles it senses overcurrent or thermal imbalance in battery strings. Fast MOSFET disconnect protects against thermal runaway.

In satellites it senses voltage deviation or current ripple on DC buses. The analog design is used because it survives radiation and prevents electrical arcs in vacuum.

In railway DC systems it senses conductor faults or water ingress. Analog isolation circuits open before arcs escalate into fires.

In telecom base stations it senses insulation breakdown or line damage. Protection is autonomous because remote systems can’t wait for cloud logic.

In photovoltaic systems this is the first use of fast analog arc sensing directly inside a junction box. By sensing ripple, leakage, or loss of current while sunlight is present, the system prevents fire, shock, or backfeed. The circuit is simple enough to manufacture at scale and rugged enough to deploy globally.

This kind of sensing replaces expensive or complex solutions with something fail-safe, proven in harsher sectors, and now ready for solar.

12. Mission: Disconnection Under Load for Worker Safety

This circuit enables local disconnection under load, even during active power generation, to protect field technicians and reduce electrocution risk during testing, commissioning, and fault handling of 1000–1500 VDC strings.

Unlike traditional systems that rely on inverter shutdowns or manual string isolators, this approach provides autonomous, module-level fault isolation.

It parallels the fault response used in lithium battery packs and aerospace electronics, where local protection circuits detect fault conditions and disconnect the cell or load path instantly. Applying the same philosophy to PV modules allows for safer, decentralized DC architectures at scale.

This is a key step toward high-voltage, touch-safe DC systems for clean energy infrastructure. 

13. Module-Level Overload Protection (Transformer & Inverter Safety) - Intelligent DC and AC load flow studies possibilities

### Overview
This enhancement builds on the core arc suppression circuit by introducing a secondary function: **proactive disconnection during transformer or inverter overload conditions**. It uses the same high-speed MOSFETs already present for safety switching.

### Problem
In large-scale PV plants, sudden irradiance changes or low grid demand can lead to **thermal overload** of inverters and **magnetic saturation** of MV transformers. Traditional systems respond too late or rely on inverter logic that may already be compromised.

### Solution
This circuit allows for **autonomous, module-level output throttling or shutdown** in response to local or external signals indicating overproduction risk. The goal is not energy optimisation, but **grid-conforming behavior and asset protection**.

### Key Differences from Voltage Optimisers
- **Purpose**: Protection, not yield gain
- **Simplicity**: No MCU, no tracking algorithm
- **Trigger**: Based on thermal stress, voltage rise, or grid signal—not mismatch
- **Effect**: Reduces DC-side power delivery before inverter or transformer strain occurs

### Implementation Options
- Local analog sensing (e.g., string overvoltage)
- External enable/disable signal line from combiner or substation
- Optional future integration with SCADA alerts

### Status
Prototype-ready. Function builds on existing architecture with minimal hardware changes.


14. Meaningful contributors

Dr. Giedrius Kopitkovas (especially suggesting MOSFETs vs fuses), Mr. Oliver Baer, Mr. Jan Mastny, Mr. Faruk Yeginsoy, Mr. Wolfgang Kessler, Mr. Christoph Studer, Mr.Stefan Otto (introducing me to Studer Cables Switzerland and original DC knowleldge), Mr. Steve Cooper, Mr. Steven Mcfadyen

Vikram Kumar
Ventus Ltd
United Kingdom
Tel: +44 7384 741128
Email: vikram.kumar@ventusltd.com
Web: www.ventusltd.com
