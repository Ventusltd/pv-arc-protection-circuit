Open Hardware Disclosure

Module Level DC Arc Suppression and Insulation Fault Disconnection Circuit for PV Systems

License: CERN OHL S v2
DOI: https://doi.org/10.5281/zenodo.15571844  
Published: 2025 06 02 (v1.3.2.1)
Version: 1.3.2.1
Author: Vikram (Ventus Ltd)
Date: 2025 05 29
License URL: https://ohwr.org/cern_ohl_s_v2.txt

 

1. Purpose

This open hardware design provides embedded, MCU free protection against DC arc faults, insulation leakage and cable or connector failures within photovoltaic (PV) modules. It is intended to achieve nuclear grade resilience through analog only fault logic that functions even in high radiation, EMI heavy or railway environments. 

The system is specifically designed to prevent rooftop fires, vegetation ignition and wildfires caused by undetected DC arcs in solar installations. Disconnection is gated by sunlight presence inferred from PV voltage and confirmed current flow, preventing false alarms without optical sensors or software.

It enables sub microsecond autonomous disconnection at module level to mitigate fire, electrocution and backfeed risks in real world solar deployments.

This PCB is designed to go inside the module’s junction box and function as a module level protection element, enabling system wide fire and fault risk reduction whether the site is 10 kW or 1000 MWp.

Designed with high noise, high security and high voltage environments in mind and intended to scale with modules across utility systems. It also serves domestic and commercial sectors with ease, offering high reliability and straightforward replacement of modules or components in the  event of failure.

We envision solar modules that protect themselves, disconnect in microseconds and rejoin the system without human intervention, once it’s safe to do so.
No software, no reboot, no delay.

Just by reading this release the user will obtain critical safety and PV Array design insights.

 

2. Problem Statement

Existing PV protection relies on string level detection or external rapid shutdown devices. These approaches often fail to prevent:
	•	Insulation breakdown to frame or ground
	•	Arc faults in connectors or junction boxes
	•	Sustained voltage after cable breaks
	•	Backfeed currents up to 40 A from paralleled strings (backfeed may be worse in some home run designs with insulation piercing connectors, which may reduce MPPT protection, it may not be abovr the fuse current snd it may cause massive DC leakage beyond what is normal leakage current i.e. in Amps not milliamps!)
	•	Unsafe voltage presence during maintenance or emergencies

Standard fuses and AFCIs often miss low current faults or trip too late. Battery cells are already protected individually. Solar modules must follow suit.

 

3. Solution Overview

Exeutive summary based on iteration 3
NOTE: MOSFET snd exisging diode heat dissipation against tinned copper bus bars or similar must be modelled at detailed design stage.

Dual 1700 V SiC MOSFETs in series for 2–3 kV withstand, <1 µs disconnection via analog comparator logic   no MCU, no reboot, no comms.
Arc fault detected through shunt + RC filter (30–100 kHz ripple), insulation leakage via 10 MΩ divider to frame, both feeding a diode OR comparator trigger.
Gate pull down by PNP transistor, protected with 18 V MOV clamp; surge absorbed via TVS + snubber (2.2–4.7 µF + 100 Ω).
Entire system self energised via PV voltage (>15 V = sun logic gate).
PCB fits 35 × 45 mm, <10 mm height, ~20 SMD components, designed for Stäubli/Amphenol j boxes at up to 17–20 A continuous, with 30–55 A backfeed possible in parallel string faults, 1500–2000 VDC and according to OEM datasheets! 

The proposed circuit integrates within the module junction box and includes:
	•	High speed N MOSFET disconnection (<1 µs)
	•	Passive snubber and TVS for arc energy absorption
	•	Analog fault detection:
	•	Voltage divider for insulation leakage (PV– to frame)
	•	Shunt resistor and RC integrator for arc precursors
	•	Cable break detection via sunlight + no current logic
	•	Failsafe design (opens on power loss)
	•	Integrated bypass diodes for current continuity

No firmware, signal bus, or central logic required.

 

4. Block Diagram (Text Format) Detailed iteration 2

PV+ ─────┐
         │
         ├──► [1] Bypass Diodes ──► OUT+
         │
     [2] PV Cells
         │
         ├──► [3] TVS Diode ───┐
         │                    │
         │               ┌────┴────┐
         │               │  Snubber │
         │               └────┬────┘
         │                    │
         ├──► [4] Q1 (MOSFET) ───┬──►
         │                      │
         └──► [5] Q1B (MOSFET) ──┘──► OUT–
                                │
                                ├──► [6] Bleeder Resistor
                                │
                                └──► Gate driven by Q2
                                      ▲
        ┌─────────────────────────────┴────────────────────────────┐
        │                                                          │
 [7] Comparator U1A ◄──── [9] Voltage Divider (R2/R3)     [8] Comparator U1B ◄──── [10] Shunt R4 + RC
        │                                                          │
        └──────────────► Logic OR ───────────────► [11] Q2 (PNP Gate Driver)
                                                             │
                                                 [12] MOV (Gate–Source Clamp)

Key: Iteration 2 
[1] Bypass Diodes – Allow current to bypass the module when it is disconnected. Prevents open string interruption.
[2] PV Cells – The standard photovoltaic cell string producing DC output.
[3] TVS Diode – Suppresses transient voltage spikes (e.g. from lightning) across PV+ and PV–.
[4] Q1 (MOSFET) – Main high speed N channel MOSFET switch. Disconnects the module under fault conditions.
[5] Q1B (MOSFET) – Second N MOSFET in series for redundancy. Ensures fail safe isolation if Q1 fails short.
[6] Bleeder Resistor – Discharges residual voltage on OUT– after MOSFETs open. Improves safety during maintenance.
[7] Comparator U1A – Monitors voltage divider to detect insulation leakage (PV– to frame).
[8] Comparator U1B – Monitors ripple on output current for arc fault detection.
[9] Voltage Divider (R2/R3) – High value resistor divider that scales leakage voltage for comparator input.
[10] Shunt Resistor + RC Filter – Detects high frequency ripple caused by arc faults on output current.
[11] Q2 (Gate Driver, PNP) – Pulls down MOSFET gate(s) to isolate the module when either comparator trips.
[12] MOV (Gate–Source Clamp) – Protects the MOSFET gates from voltage overshoot during switching events.

Iteration 2 of the module level DC arc suppression and insulation fault disconnection circuit introduces several key improvements to enhance safety, reliability and deployability in real world photovoltaic systems. The most critical enhancement is the addition of a second N channel MOSFET (Q1B) in series with the primary switch (Q1), which provides fail safe isolation in the event that Q1 fails short due to stress, aging, or surge. This dual MOSFET architecture mirrors best practices found in EV battery packs and other high reliability DC systems. To improve arc energy absorption, the original snubber capacitor has been upgraded from 2.2 µF film to a 4.7 µF X7R ceramic, enabling more robust suppression of inductive spikes when the MOSFETs disconnect rapidly under load.

Gate protection has also been strengthened with the addition of a MOV or TVS diode across each MOSFET’s gate source junction, preventing gate oxide damage from voltage overshoots caused by fast switching. To address the risk of nuisance tripping from electromagnetic interference (EMI), transient shading, or inverter switching noise, the TLV7012 comparator has been replaced with the TLV7032, which adds 6 mV of hysteresis—stabilizing trip thresholds and reducing false positives. The arc detection path now includes an explicit RC filter following the shunt resistor (R4), targeting the characteristic high frequency ripple of DC arcs while rejecting inverter harmonics or normal fluctuations.

In the insulation leakage detection circuit, the high resistance divider (R2/R3) has been reinforced with clear support for adding a low pass filter to suppress noise and ensure reliable triggering. A bleeder resistor has been added across the output (OUT–) to safely discharge residual voltage after the MOSFETs disconnect, improving safety during maintenance or emergency response. Logically, the comparator outputs (U1A for leakage and U1B for arc) now feed into a diode OR gate structure before driving Q2, ensuring that either fault type can independently initiate disconnection. The circuit is also now structured to support optional status signaling—such as a trip LED or remote flag output—to help installers and operators identify faulted modules without relying on central communication systems.

Collectively, these enhancements position the circuit for greater reliability, better immunity to environmental and electrical noise and improved compatibility with emerging standards like IEC 63027 and UL 1699B. Iteration 2 reflects a more mature, safety hardened design suitable for global PV deployment at scale.

Iteration 3 

PV+ ───┬─────────────────────────────────────────────────────┐
       │                                                     │
   [3] TVS Diode                                             │
       │                                                     │
   ┌───▼────┐                                                 │
   │Snubber│                                                 │
   └───┬────┘                                                 │
       │                                                     ▼
     [1] Bypass Diodes ───────────────────────────────────► OUT+
       │
   [2] PV Cells
       │
   ┌───▼────┐
   │ Q1     │──┐
   │ MOSFET │  │
   └───┬────┘  │
       │    ┌─▼───────┐
   [5] R1   │ Q1B      │
   10 MΩ    │ MOSFET   │─────► OUT–
       │    └────┬────┘
       │         │
       └─────────┘
                 │
             [6] Bleeder Resistor
                 │
                 ▼
              Discharge

Gate Drive Control Path:
 ┌──────────────────────────────────────────────────────────────┐
 │                                                              │
 │ [7] Comparator U1A ◄──── [9] Voltage Divider (R2/R3)         │
 │                                                              │
 │ [8] Comparator U1B ◄──── [10] Shunt R4 + RC Filter           │
 │                                                              │
 └────────► Diode OR Logic ──────► [11] Q2 (PNP Gate Driver) ──►
                                                  │
                                         [12] MOV Gate Clamp

Iteration 3 

Bill of Materials (BOM)
					 
1.	Q1, Q1B – SiC N Channel MOSFETs
Infineon IMZA65R027M1H, 1700 V, 27 mΩ, TO 247
	2.	Q2 – PNP Gate Driver
BC327, 45 V, 800 mA, TO 92
	3.	TVS Diode (across PV+ to PV–)
Littelfuse SMCJ1000CA, 1000 V clamp, DO 214AB
	4.	MOV (Gate–Source Clamp)
Littelfuse V18ZA1 or 18 V bidirectional TVS, SMC package
	5.	Snubber Capacitor
WIMA MKP10, 2.2 µF, 1600 VDC, Film Cap
	6.	Snubber Resistor
100 Ω, 2 W, Metal Film (Vishay PR02)
	7.	Bleeder Resistors (across each MOSFET)
10 MΩ, 0.25 W, axial, 1% tolerance (x2)
	8.	Comparator – Dual Analog Comparator with Hysteresis
TLV7032, SOT 23 6, rail to rail, 6 mV hysteresis
	9.	Voltage Divider Resistors (Insulation Fault Detect)
R2: 10 MΩ, R3: 100 kΩ, both 0.5 W rated, 1% tolerance
	10.	Shunt Resistor (Arc Ripple Detect)
0.1 Ω, 1 W, 1% low inductance metal strip
	11.	RC Filter Components (post shunt)
1 kΩ resistor + 100 nF ceramic (X7R), 100 V rating
	12.	Diode OR (for comparator outputs)
1N4148 (x2), fast switching diodes, 75 V, SOD 123 or DO 35

Confirm that selected MOSFETs, snubber capacitor, and TVS/MOV components are suitable for 30 A RMS-rated current paths and peak surge currents 55 A. If not, propose candidate upgrades. Note during to changing module technology, backfeed, short circuit currents, bifaciality must be observed with care. 

Iteration 3 Summary (to compare with ealier concept)
	•	Q1 and Q1B are now explicitly shown in series for 2 kV withstand.
	•	Both gates are driven together by Q2 (PNP) after either comparator trips.
	•	Gate protection now includes MOV clamp and layout assumes full discharge path.
	•	TVS diode and snubber are upstream, correctly protecting against line surges and inductive spikes.
	•	Comparator outputs are fed into a diode OR, ensuring either arc or leakage detection activates disconnection.
	•	Bleeder and voltage sharing resistors are visible, enabling stable voltage division across MOSFETs.
	•	Overall structure now matches both schematic and physical layout intent.


5. Questions for prototype iterations 

While the design logic is sound, early real world testing could expose issues like false trips from humidity, EMI, or transient shading—especially without tuned hysteresis or filtering. There’s also risk that fast MOSFET switching, if not properly suppressed, could damage components or fail to interrupt high backfeed safely. Should we include minimal analog tuning (e.g. RC filters or comparator hysteresis) before testing to avoid early failures and reduce the need for multiple prototype iterations?

Solutions for robustness:
Hysteresis (TLV7032) – Already noted; strongly recommended.
RC Filter Tuning – Adjust time constants to avoid rapid fluctuations.
MOV on Gate Source – Prevents MOSFET damage from voltage spikes.

Confirm PV+ voltage threshold (~15 V) as a sufficient proxy for sunlight presence.
	•	Validate triple condition logic: PV+ high, current > 0, ripple sustained → required to trigger disconnect.
	•	Assess whether adding an optical sun sensor provides real benefit, or introduces unnecessary complexity.
	•	Verify immunity to EMI transients under IEC 61000 4 4 and  4 5 conditions using this analog gating.

For harsh environments such as nuclear facilities, industrial plants, or railway systems

Initial field validation should include EMI immunity testing to IEC 61000 4 6 (conducted disturbances) and thermal stress cycles per IEC 60068.

6. Benefits
	•	Ultra fast arc suppression (<1 µs)
	•	Withstands 40 A backfeed from parallel strings
	•	Fully autonomous protection logic
	•	Opens on power loss (failsafe)
	•	No external control, MCU, or communications
	•	Compatible with leading junction box formats
	•	Enhances safety for installers, first responders and operations teams


• Immune to nuisance tripping caused by EMI, inverter switching, or grid loss, due to PV voltage based gating and current presence confirmation
• Designed for extreme conditions: no MCU, no firmware, no optical sensors   only analog comparators and passive components
• Operational integrity maintained in radiation prone, high noise environments including rail and nuclear infrastructure
• All trip logic requires sunlight + current + arc signature, reducing false positives to near zero without digital filtering

Fully testable at the analog level without debug tools   no code lockups, boot dependencies, or startup race conditions
 

7. Economic Justification

For a 660 Wp module priced at $0.08/Wp (~$52.80), this circuit adds approximately $1.80 in cost around 3.4%.

This is minor compared to the real world costs of:
	•	Structural or vegetation fires
	•	Electrocution injuries or fatalities
	•	Inverter or transformer destruction
	•	Long term degradation from galvanic corrosion
	•	Liability or compliance failures

In dry climates like California, Spain, Greece, or Portugal and during UK droughts with dry grass, even one DC arc can trigger a wildfire. Prevention at module level is the safest and most scalable strategy.

 

8. License Terms (CERN OHL S v2 Summary)
	•	You may use, study, modify and distribute this design
	•	You must release any modified version under the same license
	•	You may not impose further restrictions
	•	Attribution is encouraged but not legally required

The license ensures that all derivative safety improvements remain freely available and unencumbered by proprietary control.

 

9. Intent and Adoption Path

This release aims to establish an open, battery grade safety standard for PV modules and junction boxes. By embedding fault protection directly at the source, the risk of fire and electric shock is minimized without reliance on central systems.

We specifically invite:
	•	Sandia National Laboratories
	•	SunSpec Alliance
	•	European and national PV trade bodies

…to evaluate this design as a candidate for coordinated trials, standards inclusion and policy alignment. The fragmented nature of PV integration requires public institutions to lead adoption.


Collaborative scale up toward NREL’s 75 TWp net zero solar targets

10. Approximate size
Estimated: 35 × 45 mm PCB, < 10 mm height, 3oz copper, ~20 SMD components (dual MOSFETs, MOVs, RC filter, bleeder, gate pull down, etc.), 40 A rated, single sided, no MCU, low to moderate complexity, SMT reflow compatible, fits standard Staubli/Amphenol junction boxes.

11. Proof of concept from other industries

This circuit senses DC arc risk, insulation leakage, or abnormal current using passive analog components. It reacts in microseconds without software. This type of sensing is already trusted in high risk environments.

In electric vehicles it senses overcurrent or thermal imbalance in battery strings. Fast MOSFET disconnect protects against thermal runaway.

In satellites it senses voltage deviation or current ripple on DC buses. The analog design is used because it survives radiation and prevents electrical arcs in vacuum.

In railway DC systems it senses conductor faults or water ingress. Analog isolation circuits open before arcs escalate into fires.

In telecom base stations it senses insulation breakdown or line damage. Protection is autonomous because remote systems can’t wait for cloud logic.

In photovoltaic systems this is the first use of fast analog arc sensing directly inside a junction box. By sensing ripple, leakage, or loss of current while sunlight is present, the system prevents fire, shock, or backfeed. The circuit is simple enough to manufacture at scale and rugged enough to deploy globally.

This kind of sensing replaces expensive or complex solutions with something fail safe, proven in harsher sectors and now ready for solar.

12. Mission: Disconnection Under Load for Worker Safety

This circuit enables local disconnection under load, even during active power generation, to protect field technicians and reduce electrocution risk during testing, commissioning and fault handling of 1000–1500 VDC strings.

Unlike traditional systems that rely on inverter shutdowns or manual string isolators, this approach provides autonomous, module level fault isolation.

It parallels the fault response used in lithium battery packs and aerospace electronics, where local protection circuits detect fault conditions and disconnect the cell or load path instantly. Applying the same philosophy to PV modules allows for safer, decentralized DC architectures at scale.

This is a key step toward high voltage, touch safe DC systems for clean energy infrastructure. 

13. Module Level Overload Protection (Transformer & Inverter Safety)   Intelligent DC and AC load flow studies possibilities

Overview
This enhancement builds on the core arc suppression circuit by introducing a secondary function: proactive disconnection during transformer or inverter overload conditions. It uses the same high speed MOSFETs already present for safety switching.

Problem
In large scale PV plants, sudden irradiance changes or low grid demand can lead to thermal overload of inverters and magnetic saturation of MV transformers. Traditional systems respond too late or rely on inverter logic that may already be compromised.

Solution
This circuit allows for autonomous, module level output throttling or shutdown in response to local or external signals indicating overproduction risk. The goal is not energy optimisation, but grid conforming behavior and asset protection.

Key Differences from Voltage Optimisers
  Purpose: Protection, not yield gain
  Simplicity: No MCU, no tracking algorithm
  Trigger: Based on thermal stress, voltage rise, or grid signal—not mismatch
  Effect: Reduces DC side power delivery before inverter or transformer strain occurs

Implementation Options
  Local analog sensing (e.g., string overvoltage)
  External enable/disable signal line from combiner or substation
  Optional future integration with SCADA alerts

Status
Prototype ready. Function builds on existing architecture with minimal hardware changes.

13.1 Surge Isolation: SPD Behavior Plus Active Disconnect and Future Expansion (to protect inverters or combiners against DC surges)

This design, while focused on DC arc suppression and insulation fault disconnection, also exhibits characteristics aligned with Type 2 DC surge protective devices (SPDs) as defined in IEC 61643 31.

Surge Protection Attributes:
	•	Sub microsecond response time through fast N MOSFET shutdown
	•	Integrated TVS diode and MOV clamping across PV output
	•	Snubber capacitor to absorb inductive kickback from cable or array transients
	•	Failsafe disconnection to fully isolate surge damaged segments from the array

Planned Enhancements (Optional for Future Versions):
	•	Addition of dedicated high energy MOV for lightning class events (8/20 µs waveform)
	•	Coordination with thermal fuse or thermal cut off for MOV end of life signaling
	•	Optional PE grounded surge path for PV+ or PV– via clamped diversion
	•	Analog surge sensing logic could evolve to trigger disconnect + clamp, offering both surge suppression and energy isolation—exceeding conventional SPD behavior

Novelty:
Unlike traditional SPDs, which remain electrically connected post surge, this design uniquely offers the ability to:

Suppress the surge, then open the circuit.

This provides superior safety in PV arrays, especially in 1000–1500 VDC systems deployed across long cable runs and lightning prone terrain.

This arc protection circuit may thus serve a dual role:
As a PV specific SPD analog and as a selective disconnector, representing a new class of hybrid protection.

13.2 Precedent from Battery Safety and the Escalation Toward 3000 VDC: Batteries Are Already Treated as Fire Hazards   PV Must Follow

In the energy industry, lithium ion battery systems   even at voltages below 60 VDC   are universally classified as fire sensitive components. As such, they are required to include:
	•	Per cell voltage, current and thermal protection
	•	Fast acting MOSFET based battery management systems (BMS)
	•	Embedded fault logic and internal disconnects
	•	Certification under safety frameworks such as UL9540A and IEC 62619

Despite their lower system voltages and integrated BMS, batteries are subject to stricter safety regulations than most PV systems.

By contrast, photovoltaic strings:
• Routinely operate at 600 to 1500 VDC  
• Contain no module level active fault protection  
• Remain energised at all times under sunlight  
• Rely on installer dependent external disconnect methods  
• Future PV systems are already under study at up to 3000 VDC, where arc risk escalates non linearly  

14. Implementation Responsibility and System Level Integration

While this open hardware design defines a fail safe module level protection circuit, it is strongly recommended that qualified PCB and power electronics engineers be consulted to validate implementation before any prototyping or manufacturing. The system’s electrical behavior must be assessed not only at the component level, but from the PV cell output through to the grid connection, including:
	•	DC and AC load flow studies across the full array, inverter, transformer and grid interface
	•	Insulation coordination analysis, considering worst case overvoltage, surge and creepage/clearance conditions
	•	Thermal modelling, especially under sustained arc fault or leakage conditions
	•	Mechanical integration, including connector integrity, enclosure ratings and environmental protection

Many historical PV system failures—including DC arcs, backfeed faults and fire incidents—have occurred not from a single component fault but from weak points in the full electrical chain. Therefore, this circuit should be integrated as part of a coordinated protection scheme that spans from cell level to transformer level, ensuring that interaction with inverter behavior, string layouts, bypass diodes and grounding methods is fully understood and optimized.

14.1 Standards Alignment and Regulatory Pressure

This open hardware design addresses critical safety risks in photovoltaic systems by providing local, module level protection against arc faults, insulation leakage and backfeed voltage. It is intended to support or exceed the safety objectives of current international and UK standards.
	1.	IEC 62548 – PV Array Design Requirements
Aligns with section 4.6: Protection against faults. Introduces early disconnection logic within the module rather than relying on distant string level detection.
	2.	IEC 60364 7 712 – Electrical Installations (PV Systems)
Supports automatic disconnection requirements in response to insulation or fault conditions, reducing risk of fire or electrocution during operation or maintenance.
	3.	IEC TS 62738 – Arc Fault Protection
While focused on string level detection, this design anticipates the necessary evolution toward source level intervention to contain arcs early and safely.
	4.	UL 1699B / NEC 690.11 – Arc Fault Circuit Interruption
Meets the intent of US standards for rooftop PV safety, but does so through simpler, faster and MCU free analog mechanisms.
	5.	UK Health and Safety Executive (HSE) – Electricity at Work Regulations 1989
This circuit directly supports employer duties under Regulation 4(1) and 8(1) to prevent electrical danger. It exposes the weakness of existing systems that leave high voltage DC cables energised under fault conditions.

Interim Risk Acknowledgement
Until such safety circuits are standardised and embedded in module designs, it is vital that PV system operators conduct regular insulation resistance (Riso) testing—especially in large scale arrays using 1000–1500 VDC strings. This is the only currently viable method to detect breakdowns that may otherwise go unnoticed until catastrophic failure occurs.

14.2 Fire Risk Context for Commercial Rooftop and Utility Scale PV Systems

Recent incidents such as the rooftop PV fire at a Hospital in Bristol demonstrate that commercial buildings are increasingly vulnerable to electrical fires linked to photovoltaic systems. Large utility scale solar farms often covering hundreds of hectares—also present significant wildfire hazards, particularly in drought prone areas like southern Europe, California and Australia. The 2024 Talaván PV fire in Spain burned over 800 hectares due to a fault in electrical conductors combined with dry vegetation, highlighting the systemic risk.

With over 2 terawatts of PV now installed globally and multi terawatt annual deployment expected per NREL forecasts, the scale of risk is increasing. High voltage string architectures (1000 to 1500 volts) and dense layouts raise both ignition potential and the challenge of fault isolation. This open hardware design addresses these risks by enabling autonomous, module level fault disconnection—particularly relevant for rooftops where delayed access and structural vulnerability add to the danger.

Given the industry size and risk exposure, proactive safety integration must become the norm. This design is released with no commercial restrictions, available for immediate adoption and further development by manufacturers, universities, national labs and standards committees.

14.3 Global Fire Risk Summary from DC PV Systems

Estimate global data on fires caused by DC side faults (arcing, insulation breakdown, connector failure) is limited due to inconsistent reporting. However, credible industry estimates show a clear and rising safety risk:

Fraunhofer ISE 2024
1 fire per 2.5 MW per year in older PV systems
Estimated 3 to 5 fires per day

Solarif Insurance 2023
35 percent of 1200 fire claims linked to DC faults
Estimated 1.2 fires per day

NREL 2025
0.016 percent annual fire rate across 2.3 TW installed PV capacity
Estimated 1 fire per day

Australia Clean Energy Council 2024
44 percent of solar fires from DC arcs based on field data
Estimated 0.8 fires per day

UK Fire Statistics 2024
42 solar related fires in 2023 mostly DC side
Estimated 0.1 fires per day

Consolidated Estimate: 2 to 4 preventable DC side fires occur daily worldwide.
This is likely underestimated due to underreporting!

Root Causes of DC Fires (industry averages)
	•	61%: Connector failure (mismatch, poor crimps, thermal cycling)
	•	24%: Insulation breakdown (UV, rodents, moisture ingress)
	•	15%: Junction box and cable arc faults

High Risk Regions
	•	Dry climates: California, Spain, Australia (wildfire ignition)
	•	Rooftop markets: UK, Germany, Japan (building integrated risk)

Outlook to 2030
With global PV expected to reach 8 TW by 2030, DC fires may exceed 10 per day unless protection is embedded at module level.

Why It Matters
	•	Each fire causes $250,000 to $2 million in damage (based on insurer claims)
	•	Electrocution risk is high at 1000–1500 VDC for firefighters and O&M crews
	•	Open source module level protection could prevent over 70% of incidents

Primary Sources
	•	Fraunhofer ISE: PV Fire Incident Review (2024)
	•	NREL: Global PV Reliability Report (TP 7A40 90582, 2025)
	•	Solarif: Solar Insurance Annual Report (2023)
	•	Australia Clean Energy Council: DC Arc Fault Field Study (2024)
	•	UK Home Office Fire Statistics (2024)

14.4 Recommendation
Use this evidence to support urgent updates to IEC 62548, UL 1699B, NEC 690.11 and rapid shutdown standards. Module level analog protection circuits offer a simple, reliable and scalable solution to one of the most persistent risks in global PV deployment.

Freedom to Operate Statement (Engineering Level, Non Legal)

This open hardware design—Module Level DC Arc Suppression and Insulation Fault Disconnection Circuit for PV Systems—is an independently developed, analog only solution released under the CERN OHL S v2 license.

The circuit architecture:
	•	Relies entirely on passive and comparator based analog logic
	•	Contains no software, firmware, microcontroller, or digital communication logic
	•	Is designed for integration inside the PV module junction box
	•	Enables autonomous, sub microsecond disconnection in response to arc or insulation faults

A review of current industry practices and public documentation indicates that no known commercial PV module in the 400–800 Wp class currently ships with built in arc suppression or insulation fault disconnection inside the junction box using analog only methods.

This disclosure is made in good faith, in the public interest and without the intention to infringe or compete with proprietary platforms that rely on software, central coordination, or digital control logic.

Any overlap with preexisting commercial systems—if asserted—will be considered carefully and transparently. However, this design was created from first principles, based on decades of high voltage industrial safety experience and is fundamentally distinct in method and intent.

We assert the right to publish, share, manufacture and refine this design under the terms of the CERN OHL S license and welcome collaborative engagement from all sectors—including industry, research and standards bodies.

16 Disclaimer 

The author makes no warranty, express or implied, regarding safety, fitness for purpose, or compliance with local electrical regulations. Field use requires insulation coordination, surge and arc destruction testing and environmental validation in accordance with applicable standards (e.g. IEC 62548, IEC 61730, UL 1699B).

Misuse or modification of this design without testing may result in serious failure or fire. Use at your own risk.

⚠️ This is a safety critical design. Use only under the direction of a Chartered Electronics Engineer or equivalent professional.

17 Meaningful contributors

Dr. Giedrius Kopitkovas (especially suggesting MOSFETs vs fuses), Mr. Oliver Baer, Mr. Jan Mastny, Mr. Faruk Yeginsoy, Mr. Wolfgang Kessler, Mr. Christoph Studer, Mr.Stefan Otto (introducing me to Studer Cables Switzerland and original DC knowledge), Mr. Steve Cooper, Mr. Steven Mcfadyen, Mr. Liam Hicks, Mr. Dathan Eldridge, Ms. Anjali Kumar (early logic checking), Mr. Marc Scambler, Mr. Matthew Xenakis, Mr. Joern Hackbarth, Mr. Jason Daniels

Media Coverage

Our work on PV module-level DC arc suppression was featured in Forbes:
[DC Cable: The Overlooked Risk Of The $2 Trillion Solar Sector](https://www.forbes.com/councils/forbestechcouncil/2025/07/24/dc-cable-the-overlooked-risk-of-the-2-trillion-solar-sector/) (Forbes Tech Council, July 2025))

AI Technical Assistance
ChatGPT (OpenAI) was used for deep comparative searches, structural clarity and logic refinement. Confirmed: No commercially available PV module currently ships with embedded analog arc or insulation protection. This remains an unmet and urgent industry need.


18 AI Ethics Authorship Integrity and Grid Alignment Statement

This open hardware disclosure is the result of 13 years of direct solar field experience through VENTUS Ltd and over 20 years of cable and component system knowledge spanning roles at Cable and Wireless, Lapp, Leoni and client engagements with Studer Cables Switzerland. These insights form the technical and application specific foundation of the design shaped by field level failures electrical constraints and persistent safety challenges observed in real world photovoltaic systems.

VENTUS Ltd trading as VENTUS Cables and Connectivity is not new to the field. Since 2012 the company has focused deeply on solar direct current string cable layout and design across the United Kingdom and Europe delivering over 2.5 gigawatt peak of photovoltaic systems and advising on numerous bankable engineering procurement and construction projects. VENTUS benefits from an enduring agency relationship with Studer Cables Switzerland a manufacturer with a 100 gigawatt peak global delivery record who owns their own electron beam accelerator and in house compound development capability for crosslinked polymer insulation. This partnership enables unmatched technical depth in electron beam crosslinked direct current cables specifically tailored for high voltage utility scale solar applications.

VENTUS has made significant long term investment into application level knowledge of direct current string cables including insulation behaviour, fault energy absorption and inductive layout risks across dense array architectures. This gives the company unique insight into the physical realities of fire propagation backfeed currents and surge exposure within photovoltaic systems which most circuit designers do not encounter directly.

The current circuit design has been developed with the support of mentors, field colleagues and early stage contributors all credited in good faith. AI tools such as ChatGPT OpenAI were used under a paid subscription to assist with logic structuring technical language and comparative searches. All engineering decisions design architecture and deployment strategy remain entirely human originated and field driven.

This open hardware release is intended to:
	•	Enable funding consideration
	•	Support electronics and electrical engineering modelling
	•	Guide prototype development
	•	Initiate field level destructive testing in lightning and short circuit simulation environments

 AI Ethics and Responsible Use Statement

This disclosure includes input developed using ChatGPT OpenAI under a paid professional subscription £200 per month over. The AI was used strictly as an engineering support tool to assist with:
	•	Structuring circuit logic
	•	Clarifying component behaviour
	•	Formatting technical explanations
	•	Comparing international standards
	•	Error checking given volume of considerations 

All engineering decisions safety logic and circuit architecture remain fully human originated grounded in extensive solar field and cable engineering experience. The AI’s role is comparable to that of a field applications engineer supporting component assessment and language articulation under the full direction of the lead designer.

No content has been generated or claimed as original by AI. All logic and safety pathways have been verified against real system behaviour failure conditions and protection requirements.

This approach upholds transparency authorship integrity and verifiable engineering input. AI was not used to replace engineering judgement invent topologies or obscure authorship. It was used to support responsible engineering not to simulate it.

19 Electrical Modelling Requirements and System Competence

This design requires full system electrical modelling that combines electronics and electrical engineering competence with direct current string layout and array topology awareness. This includes:
	•	Capacitance discharge paths
	•	Inductive loop behaviour across dense field layouts
	•	Electromagnetic coupling from up to 160 parallel conductors in systems with 80 strings
	•	Arc propagation and residual energy conditions in long exposed cable runs


19.1 Capacitance and Inductance modeling of complete PV Array 

The engineering models must pay close attention to inductive and capacitative discharge behavious of dense string designs for example 24 strings in Sungrow 350kW string inverters or 20x4 = 80 strings in 1.1MW central inverter topologies. The system may behave as massive capacitor. Induced switching or transients or reverse current faults continue to manifest as dangeorus and often hiden arcs in the field. This behaviour may often be invisible to the inverter. 
Twisted or Quad string designs may be considered and have been delivered. 

The mission is to build self healing solar modules that autonomously disconnect under dangerous conditions reducing risks of fire, electrocution, inverter failure or transformer overload and enhancing the safety of photovoltaic systems at every scale.


19.2 Thermal and Integration Considerations

• The circuit does not involve switching losses under normal operation, but the main MOSFETs conduct continuously during daylight hours.
• Thermal dissipation per MOSFET must be modelled under typical PV string currents, with higher loads possible during backfeed events. Continuous conduction currents may reach 20–25 A in modern 550–660 Wp module strings, and that thermal dissipation and copper
• Existing tinned copper busbars inside the junction box may serve as primary thermal sinks.
• MOSFETs should be thermally bonded to these conductors using isolated thermal pads or coupled via PCB copper planes, ensuring proper electrical insulation.
• Ambient temperatures inside sealed junction boxes may exceed 70 degrees Celsius in high irradiance or rooftop installations, requiring accurate thermal simulation.
• The entire PCB, including MOSFET thermal paths and copper geometries, must be modelled in collaboration with the junction box manufacturer or tooling supplier.
• All layout decisions must respect Class II insulation requirements for high voltage DC, including appropriate creepage and clearance between conductors, copper planes and enclosure features.
• Any use of the module frame as a thermal path must preserve dielectric separation and meet IEC 61730 requirements for long term material integrity.
• This protection circuit must not be treated as a stand alone safety component. It requires joint integration between the junction box supplier on the hardware side and the client side power systems engineer or PV integrator on the deployment side.
• Proper system level modelling is essential to ensure coordination with PV array architecture, inverter protection logic, insulation behaviour and overall fault handling performance


20 Forward Looking System Architecture and Grid Control Impact

While this design remains a fully autonomous analog only safety system, future versions may integrate optional non intrusive software overlays to simulate faults or enable array level disconnection as part of grid responsive behaviour. This would allow safe coordination with inverters batteries and wind systems improving capacity factor and system resilience. It also opens the door for proactive disconnection during overproduction, voltage instability or thermal stress without compromising the reliability of the core analog safety logic.

This approach positions the photovoltaic module as a flexible responsive asset in distributed energy networks enabling grid operators to manage safety and system balance from the source not just at the inverter or substation. ALLOWING meaningul solar panel de energisation, NOT leaving dangerous live DC cables and wires on inverter shutdown WITHOUT use of external rapid shutdown devices that create high voltage (100V to 1500V, soon 2000V) switching risks and exposed to reverse currents up to 55A potentially. 

VENTUS Ltd is particularly well suited to this challenge due to its decade long focus on direct current string cable behaviour since 2012 and its active collaboration with Studer Cables Switzerland on EBXL solar cable systems that are already field proven across global gigawatt scale deployments.

Stakeholder Invitation

VENTUS Ltd invites evaluation and collaboration from grid authorities utilities and standards bodies including:
	•	National Grid ESO United Kingdom
	•	NESO 
	•	TenneT Netherlands and Germany where VENTUS has delivered 110 kilovolt solar transmission circuits
	•	TERNA Italy
	•	UK Power Networks and other UK Grid operators Northern Power Grid, Electricity North West, Scottish Power and SSE
	•	ENEL Group and its global infrastructure divisions

This open hardware design supports global decarbonisation goals and addresses a critical safety and fire risk gap in current photovoltaic deployment. It is now ready for stakeholder review, electrical simulation and field validation in partnership with public institutions and industry leadership.

Vikram Kumar
Ventus Ltd
United Kingdom
Tel: +44 7384 741128
Email: vikram.kumar@ventusltd.com
Web: www.ventusltd.com
