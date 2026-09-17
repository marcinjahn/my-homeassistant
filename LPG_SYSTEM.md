# LPG in Home Assistant

## Purpose and where to find it

The LPG system estimates the amount of gas remaining in the tank from a pulse
counter attached to the gas meter. It is an estimate, not a direct tank-level
measurement: its accuracy depends on the pulse calibration and the tank level
entered manually.

Open **LPG** in the Home Assistant sidebar. There are two tabs:

- **Tank and consumption** (`/heating-dashboard/tank`) — everyday readings,
  calibration, tank-level corrections, and refills.
- **Diagnostics** (`/heating-dashboard/diagnostics`) — meter health and recovery
  after a counter reset or replacement.

The configured tank capacity is **2700 L**. The forecast uses a **270 L reserve**
(10% of capacity), rather than predicting when the tank will be completely empty.
The calibration was left at **400 pulses/L** on September 17, 2026; this is a
provisional value that still needs verification against the physical meter.

## The most important distinction

There are three separate operations:

| Operation                      | When to use it                                                  | Does it reset usage since refill?                                      |
| ------------------------------ | --------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Apply calibration**          | The conversion from pulses to litres is wrong.                  | **No.** It recalculates the litres represented by the recorded pulses. |
| **Correct current tank level** | You have a better measurement of the gas currently in the tank. | **No.** It changes the remaining-level reference only.                 |
| **Record a physical refill**   | Gas has actually been delivered into the tank.                  | **Yes.** It starts a new refill period.                                |

**Editing a number field alone does nothing to the accounting.** Each field
holds a proposed value. You must press its corresponding action button and
confirm before the change is applied. A value left in an input field is not
necessarily the currently active value or the latest tank estimate.

## Tank and consumption: what the readings mean

| Reading                         | Meaning                                                                                                                                                                                                                  |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Remaining (L)**               | The latest manually measured tank level minus calculated consumption since that measurement. This is the estimated amount currently in the tank.                                                                         |
| **Remaining (%)**               | Remaining litres divided by the configured 2700 L capacity, expressed as a percentage.                                                                                                                                   |
| **Used since refill (L)**       | All pulse-derived consumption since the last recorded physical refill. A manual level correction does not erase it. Calibration changes can change its litre value without resetting the recorded pulse interval.        |
| **Average consumption (L/day)** | Recent consumption expressed as an average daily rate, using a rolling window of up to seven days. It is not just today's consumption.                                                                                   |
| **Days to 270 L reserve**       | Estimated time until the tank reaches 270 L at the recent average rate. It is not days until empty or a guarantee of future usage. At or below the reserve, it is zero.                                                  |
| **Meter and accounting**        | Whether the meter data and saved accounting state are usable. Normally this reads **Ready**. See troubleshooting below for other states.                                                                                 |
| **Forecast confidence**         | Whether enough reliable data exists for a forecast, and whether the seven-day window is complete. This is a data-availability label, not a statistical confidence interval.                                              |
| **Tank estimate below zero**    | Warns that calculated consumption has exceeded the saved amount. Remaining litres are displayed as zero rather than a negative number, but this warning exposes the discrepancy. Check the actual level and calibration. |

### Daily LPG consumption chart

The chart shows daily consumption for the last 14 days, in litres. Each bar is
calculated from that day's recorded pulse increase using the **currently active
calibration**.

Consequently:

- Applying a new calibration changes the litre values shown for earlier days.
- The underlying pulse history is unchanged; calibration does not invent new
  pulses or create a consumption spike on the day you apply it.
- Correcting the current tank level does not change the consumption bars.
- Recording a refill does not erase earlier daily consumption.
- Today's bar represents the day so far, not a full-day forecast.
- Zero means no recorded increase. During a meter problem it is not proof that
  the house consumed no gas; check **Meter and accounting** alongside the chart.

The chart toolbar also provides export/download controls.

## Action 1: apply calibration

Use this when the pulse counter is working, but its readings convert into too
many or too few litres.

1. Look at **Active pulses/L** to see the factor actually in use.
2. Enter the proposed factor in **LPG new calibration (pulses/L)**.
3. Press **Apply** on the **Apply calibration** row.
4. Confirm the recalculation.
5. Check **Active pulses/L** and the recalculated readings. If the action fails,
   look at **LPG Last Action Status** on the Diagnostics tab.

The factor means **how many pulses represent one litre**, not litres per pulse:

| Change            | Effect for the same number of pulses                                       |
| ----------------- | -------------------------------------------------------------------------- |
| Increase pulses/L | Fewer calculated litres consumed; the tank estimate decreases more slowly. |
| Decrease pulses/L | More calculated litres consumed; the tank estimate decreases faster.       |

For example, 80 pulses represent 0.2 L at 400 pulses/L, or 0.4 L at 200 pulses/L.
Changing 400 to 200 doubles calculated consumption; it does not halve it.

### What is recalculated?

- **Used since refill** is recalculated for the entire existing refill period.
- **Remaining** is recalculated using consumption since the latest manually
  measured level.
- Historical daily litre bars and consumption-based forecasts use the new factor.

The operation does **not** record a refill, reset its pulse baseline, or change
the refill date. It also does not change the saved measured level or its date.

If you corrected the tank level yesterday, changing calibration today only
changes the consumption deducted from that observation since yesterday. Usage
since refill still covers the full refill period, which may be much longer.

Change calibration through this dashboard, **not through the Zigbee device's
multiplier, divisor, or unit controls**. The accounting expects a raw pulse count;
changing device scaling could change what those readings mean.

## Action 2: correct the current tank level

Use this when you have a more accurate measurement or estimate of what is in
the tank now. This is the appropriate action during initial calibration, or
when the calculated level has drifted from a trustworthy physical reading.

1. Enter the **current total amount in the tank** in **LPG measured current level**.
2. Press **Correct** on **Save measured current level**.
3. Confirm that this is a current measurement, not a refill.
4. Check **Remaining (L)** and **Last measured level**, which records the time of
   the observation.

Example: Home Assistant estimates 2450 L, but your measurement indicates 2400 L.
Enter **2400** and press **Correct**. The remaining estimate starts from 2400 L
at that moment, and future consumption is deducted from it.

If **Used since refill** was 120 L before the correction, it remains 120 L
afterward. The refill date is also unchanged.

A correction does not fix an incorrect calibration factor. If the estimate keeps
drifting, investigate calibration rather than repeatedly correcting the level.
After corrections, do not expect the original post-refill level minus usage
since refill to equal the remaining estimate: the more recent observation is
now the reference for remaining gas.

## Action 3: record a physical refill

Use this only after an actual gas delivery.

1. Determine the **total tank level after the delivery**.
2. Enter it in **LPG measured total after refill**.
3. Press **Refilled** on **Record completed delivery**.
4. Confirm that a physical refill occurred.
5. Check that **Used since refill** is zero, **Remaining** starts from the entered
   level, and **Last refill** shows the new refill time.

**Enter the total in the tank, not just the amount delivered.** For example,
if there were 500 L and 1500 L were delivered, enter **2000 L**, not 1500 L.

This action saves both a new refill baseline and a new measured-level observation.
It does not change the calibration factor or delete earlier daily history.

## Suggested workflow while calibrating

1. Compare the pulse-derived consumption with a reliable independent measurement.
   Remember that a coarse tank gauge may not resolve small amounts of consumption.
2. Use **Apply calibration** when you have a better pulses-per-litre factor.
3. If you also have a better reading of the amount currently in the tank, use
   **Correct** to save that current level after applying the calibration.
4. Continue observing usage. Do not press **Refilled** unless a delivery occurred.

A level correction and a calibration change solve different problems. You may
need both, but neither should be treated as a refill.

## Forecasting and temporary unavailability

The forecast needs at least **24 hours of reliable samples** and uses a rolling
window of up to **seven days**. It reflects recent usage; a change in weather or
heating habits can make future consumption very different.

**Forecast confidence** can display:

- **Collecting 24 hours of reliable data** — insufficient reliable observation
  time; wait rather than assuming the missing forecast means no consumption.
- **Collecting consumption data** — the consumption-rate calculation is not ready.
- **No consumption in observation window** — no usable positive consumption rate
  exists for predicting a date. The system does not invent an unlimited supply.
- **Partial 7-day window** — an estimate is available, but less than seven days
  are represented.
- **7-day estimate** — the rolling window is sufficiently populated.
- A meter/accounting problem — resolve that problem before trusting a forecast.

After a Home Assistant restart, the saved pulse total, calibration, and tank
references survive. However, dependent estimates may temporarily be unavailable
until the meter sends a new MQTT message. This is intentional: missing data must
not be shown as zero consumption or a full tank.

## Diagnostics: what the values mean

| Reading                               | Meaning                                                                                                                                                                          |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Meter and accounting**              | Overall readiness of the accounting and meter input.                                                                                                                             |
| **LPG Last Action Status**            | Whether the last requested operation was applied or rejected. Merely entering a proposed value is not an applied operation.                                                      |
| **LPG Logical Pulses**                | The persistent, accumulated pulse total used for accounting. It is kept separate from the physical counter so accepted resets or replacements do not erase previous consumption. |
| **Raw counter (pulses)**              | The current number reported by the physical meter counter. This may reset or start from a different value after replacement.                                                     |
| **Last MQTT message received**        | When Home Assistant received a message from this device. This is receipt time, not necessarily the time the physical measurement was taken.                                      |
| **Battery / temperature**             | Diagnostics reported by the counter device. They are not the tank level or LPG temperature used in a conversion.                                                                 |
| **Seven-day pulse rate**              | The pulse-based rate behind the consumption forecast, before conversion with the calibration factor.                                                                             |
| **Last measured level / Last refill** | Dates of the latest saved tank observation and physical-refill baseline respectively. A level correction updates the former, not the latter.                                     |

The **Canonical daily pulses** chart shows daily pulse increases without
converting them into litres. It therefore does not change when you recalibrate.

Raw and logical totals do not have to be equal. For example, the recovered setup
on September 17, 2026 had raw count 118 and logical count 137 because the logical
history also included 19 pulses recorded before an earlier counter reset.
This difference alone is not a fault.

### Accounting status and what to do

| Status                             | What it means / what to do                                                                                                                                                                    |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Ready**                          | Data passes the accounting checks. Normal actions can be applied. This does not independently prove the physical calibration is accurate.                                                     |
| **Meter messages stale (3 hours)** | No device message has arrived for three hours. Check device power/battery, Zigbee connectivity, and Zigbee2MQTT. Do not use a refill or correction to conceal the problem.                    |
| **Meter unavailable**              | The current raw counter value is unavailable. This can happen during startup; wait for a message, or investigate if it persists.                                                              |
| **Invalid pulse reading**          | The source is not a valid non-negative integer pulse count. Investigate the device/integration and any scaling changes.                                                                       |
| **Counter reset needs review**     | The raw count decreased. Accounting is held to prevent an unexplained decrease from adding gas back or causing double-counting. Investigate before using either recovery button below.        |
| **Needs initialization**           | There is no valid saved accounting snapshot. This is not a normal restart state. Recover the saved state or investigate the configuration; do not initialize to zero or record a fake refill. |

If a temporary low reading returns to the previously accepted count, accounting
can resume without double-counting. A genuine reset requires a deliberate
recovery decision. Normal calibration, correction, and refill actions are
rejected while the accounting is not ready.

## Diagnostic recovery buttons — not everyday controls

Both buttons require a pending counter decrease and a fresh, valid reading.
Neither records a refill or changes the saved tank/refill references. They differ
in whether the current raw reading represents new consumption.

### Confirm counter reset to zero

Use this only when the physical counter genuinely reset to zero and its current
reading counts pulses accumulated since that reset.

Example: the counter reset and now reads **3**. Confirming adds those **3 pulses**
to the preserved logical total and resumes accounting from raw count 3. This
also affects calculated consumption and remaining gas normally.

Any unrecorded pulses lost before the reset cannot be recovered automatically.
Do not use this for an unexplained starting number on a replacement device.

### Rebaseline replaced counter

Use this when a replaced or changed counter starts from a reading that should
be accepted as a baseline, not charged as additional consumption.

Example: the replacement reads **50**. Rebaselining preserves the logical total
and adds **no pulses** for that reading. If it later reads 53, the increase of
**3 pulses** is counted normally.

If you cannot tell which situation occurred, leave the warning in place and
investigate rather than guessing.

## Reference: calculations and configuration

The accounting saves two independent pulse references:

- A **refill reference**, used to measure consumption since the last delivery.
- A **measured-level reference**, used to estimate the current remaining amount.

With logical pulse total `P`, refill reference `R`, measured-level reference `O`,
measured tank level `L`, and calibration `F` in pulses/L:

```text
Used since refill = (P - R) / F
Remaining estimate = max(0, L - (P - O) / F)
```

A correction changes `L` and `O`. A refill changes `L`, `O`, and `R`. Calibration
changes only `F`.

For maintenance, the live configuration is on the Home Assistant instance:

- SSH: `ssh homeassisstant`
- Configuration: `/homeassistant/packages/lpg.yaml`
- Accounting logic: `/homeassistant/custom_templates/lpg.jinja`
- Raw meter entity: `sensor.lpg_gas_meter_pulses_counter_energy`
- Persistent accounting entity: `sensor.lpg_logical_pulses`

The September 17, 2026 repair removed startup defaults that reset the tank to
2700 L and eliminated zero fallbacks for unavailable meter readings. A confirmed
0.28 L of phantom consumption was corrected, while old incorrect tank-state
history was preserved rather than silently rewritten. The pre-repair backup is
`/homeassistant/backups/lpg-20260917T185206Z`.
