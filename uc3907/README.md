# Simplified UC3907 load-share model

The existing `UC3907.sub` now provides a five-terminal behavioral model.
`UC3907.asy` was updated in place to match. Terminal order is:

```text
CS+  CS-  SHARE  ADJ  GND
```

```spice
.include UC3907.sub
XU cs_plus cs_minus share adj 0 UC3907 VCC_MODEL=12 C_ADJ=1u
```

CS+ and CS- sense the shunt voltage. SHARE connects between modules.
ADJ is a buffered voltage from 0 to 100 mV relative to GND, representing
an increase in the power module's voltage-loop reference. Increasing ADJ
must increase that module's delivered current. The module's feedback model
must implement this connection; ADJ alone does not supply power.

ADJ is an abstract reference-increase command, not the physical UC3907
pin-14 voltage or a UCC39002-style current sink. A leader or standalone
module produces approximately zero adjustment.

`VCC_MODEL` supplies an assumed always-on supply voltage and output headroom.
`C_ADJ` sets the internal adjust-amplifier compensation capacitor. Do not
add a capacitor at the ideal buffered ADJ output to tune the sharing loop;
change C_ADJ instead.

The model retains current amplification, a source-only share bus, the adjust
amplifier's offset/current limits, and bounded reference adjustment. The
voltage feedback amplifier, optocoupler driver, status, UVLO, protection,
temperature/tolerances, input loading, and detailed slew/compliance are omitted.
The independently authored equations are based on
[TI SLUS165C](https://www.ti.com/lit/ds/symlink/uc3907.pdf); this is not a
manufacturer-qualified model.

## Run the existing example

```bash
cd ~/uc3907-model
ngspice -b -o share-demo.log share-demo.cir
```

The existing example was updated for five terminals. In ngspice 47, the
60 ms transient completed and numeric checks passed: the leader produces
0 V adjustment, the follower saturates at 0.100 V with externally fixed
currents, and their roles exchange when the sensed currents exchange.
Standalone and grounded-bus operation both produce 0 V adjustment.

An additional temporary test with two synthetic averaged power modules
confirmed parameter overrides and closed-loop response. Currents were
9.38396/9.13538 A before a nominal setpoint change and 9.09673/9.34566 A
afterward. The approximately 0.249 A difference is consistent with this
model's intentional sharing deadband; perfect sharing is not claimed.
That temporary test does not model a specific physical converter.

Previously wired 16-terminal instances and symbols are incompatible with
this interface and must be rewired. Older mixed-controller examples using
the full model also need migration before reuse. Hardware accuracy and
real converter stability remain unvalidated; LTspice was not run here.
