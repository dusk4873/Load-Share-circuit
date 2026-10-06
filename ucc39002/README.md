# Simplified seven-terminal UCC39002 model

No official downloadable SPICE model was found on the
[TI product page](https://www.ti.com/product/UCC39002).
A [TI support reply](https://e2e.ti.com/administrators1/f/1/t/1046828)
states that a UCC39002 simulation model was not created. That reply is
historical; it does not establish that no third-party model exists.

`UCC39002.lib` is independently authored using
[SLUS495J](https://www.ti.com/lit/ds/symlink/ucc39002.pdf).
It is a preliminary approximation, not a manufacturer-qualified device model.

```bash
cd ~/ucc39002-model
ngspice -b -o share-demo.log share-demo.cir
```

The existing file was replaced in place. Logical terminal order is
`CS+ CS- CSO SHARE EAO ADJ GND`; this is not the physical IC pin order:

```spice
.include UCC39002.lib
XU cs_plus cs_minus cso share eao adj gnd UCC39002 VCC_MODEL=12
```

Use external feedback around the current amplifier; the example uses
1k/39k resistors to set its gain. Connect EAO compensation externally and
connect ADJ to the module's sense adjustment network. SHARE connects between
controllers. ADJ is a current sink with a required compliance voltage,
which the simplified sink does not enforce.

VDD is omitted from this interface: the model assumes an always-powered
controller. `VCC_MODEL` is a constant parameter for amplifier/bus headroom,
not a simulated supply node. Supply startup, undervoltage shutdown and
supply current are omitted. CSO and EAO remain accessible for external gain
feedback and compensation. Old eight-terminal instances must be rewired.
The original mixed-controller examples also require migration before reuse.
The simplified UC3907's ADJ is a voltage command; it cannot be connected
directly to this model's ADJ current sink as if they were identical outputs.

The bench uses two synthetic averaged modules. It checks sharing under a
load change and a change of leader. It does not model a particular converter,
its switching behavior, or its remote-sense circuit. A successful run is
evidence of this example's operation only.

## Measured results with ngspice 47

Both benches completed with exit code zero. The share bench ran for 80 ms:

| Condition | Module A | Module B |
| --- | --- | --- |
| Initial load, 15 ms | 4.85051 A | 4.78695 A |
| Increased load, 35 ms | 9.29898 A | 9.23453 A |
| Leader changed, 75 ms | 9.19607 A | 9.26051 A |

In `block-check.cir`, a 50 mV input produced 1.99747 V at CSO with the
example's gain-setting network. The fitted ADJ current was 3.950 mA at
EAO=2 V, and 5.950 mA in fast engagement. External disable reduced ADJ current
to zero. With VCC_MODEL=6, the saturated CSO output was 4.99910 V, confirming
that the parameter sets headroom. Numeric checks also confirmed a current
difference below 70 mA at the three sampled times and the expected change
of which module's adjustment current is zero.

```bash
ngspice -b -o block-check.log block-check.cir
```

Logs are saved alongside the netlists. These are functional regression
checks of this approximation, not comparisons with a physical UCC39002.

Approximation limits include omitted LS-bus fault handling, supply/bias dynamics,
VDD clamp, temperature spread, input bias and common-mode errors, detailed
startup/recovery and extra amplifier poles. Sink-current limits and intrinsic
capacitances that are not specified are assumptions. Logic thresholds use
smooth transitions for numerical convergence. The ADJ transfer curve is fitted
to a typical operating point; supply-current accounting is omitted.
These limitations prevent using this model to validate protection, real
startup behavior, stability margins, or guaranteed sharing accuracy.
