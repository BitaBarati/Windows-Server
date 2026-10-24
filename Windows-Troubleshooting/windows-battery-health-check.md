
# Windows Battery Health Check

## 1. Generate a Battery Report

Windows provides the built-in `powercfg` utility to generate an HTML report containing battery capacity, usage history, and estimated battery life.

### 1.1. Open Command Prompt (CMD)

1. Press `Win + R`.
2. Type `cmd`.
3. Press Enter.

Alternatively, open Start, search for Command Prompt, and select it.

### 1.2. Generate the Report

Run the following command:

```cmd
powercfg /batteryreport
```

Windows generates an HTML file and displays its location in the Command Prompt window.

To save the report to a specific path:

```cmd
powercfg /batteryreport /output "%USERPROFILE%\Desktop\battery-report.html"
```

### 1.3. Open the Report

1. Open File Explorer.
2. Navigate to the file location displayed in CMD.
3. Double-click `battery-report.html`.
4. The report opens in your default web browser.

**Note:** Administrator privileges are generally not required to generate a battery report.

---

## 2. Battery Report Sections

### 2.1. System Information

Displays information about the computer and operating system.

- **Computer Name:** Name of the computer.
- **System Product Name:** Device model, if reported.
- **BIOS:** BIOS version and date.
- **OS Build:** Windows build information.
- **Report Time:** Date and time when the report was generated.

### 2.2. Installed Batteries

This is the most important section for estimating battery health.

| Field | Description |
|---|---|
| Name | Battery name or identifier reported by the system |
| Manufacturer | Battery manufacturer, if available |
| Serial Number | Battery serial number, if available |
| Chemistry | Battery technology, such as Lithium-ion |
| Design Capacity | Original rated battery capacity |
| Full Charge Capacity | Estimated maximum capacity the battery can currently hold when fully charged |
| Cycle Count | Number of charge cycles reported by the battery, if supported |

**Important:** Capacity values are usually reported in milliwatt-hours (mWh). Some devices may report incomplete information or omit certain fields.

### 2.3. Recent Usage

Shows recent battery and power activity.

- **Start Time:** When the recorded activity began.
- **State:** Whether the device was active, suspended, or connected to AC power.
- **Source:** Battery or AC power.
- **Capacity Remaining:** Estimated remaining battery capacity at that time.

Use this section to review recent charging, discharging, and power-state activity.

### 2.4. Battery Usage

Shows recorded battery discharge over recent periods.

- **Start Time:** Beginning of a recorded usage period.
- **State:** Power state during the period.
- **Duration:** Length of the recorded period.
- **Energy Drained:** Estimated amount of battery capacity consumed.

This section helps identify periods of battery usage and unusually high discharge.

### 2.5. Usage History

Displays historical usage patterns over time.

- **Period:** Recorded day or week.
- **Battery Duration:** Time spent using battery power.
- **AC Duration:** Time spent connected to external power.

Use this section to compare battery-powered usage with plugged-in usage.

### 2.6. Battery Capacity History

Shows how the battery's reported full-charge capacity has changed over time.

- **Period:** Date or period of the record.
- **Full Charge Capacity:** Estimated maximum capacity during that period.
- **Design Capacity:** Original rated capacity used as the reference.

A declining Full Charge Capacity generally indicates battery wear.

### 2.7. Battery Life Estimates

Shows Windows' estimates of battery runtime based on recorded usage and available capacity information.

- **At Full Charge:** Estimated runtime based on the reported full-charge capacity.
- **At Design Capacity:** Estimated runtime based on the original design capacity.

These are estimates, not guaranteed runtimes. Actual battery life depends on workload, display brightness, power settings, temperature, and other factors.

---

## 3. Calculate Battery Health

### 3.1. Find the Required Values

In the **Installed Batteries** section, locate:

- **Design Capacity:** Original battery capacity.
- **Full Charge Capacity:** Current estimated maximum capacity when fully charged.

### 3.2. Battery Health Formula

Battery Health (%) = (Full Charge Capacity / Design Capacity) × 100

### 3.3. Example Calculation

Assume the report shows:

- Design Capacity: `50,000 mWh`
- Full Charge Capacity: `40,000 mWh`

Calculation:

`(40,000 / 50,000) × 100 = 80%`

**Estimated Battery Health: 80%**

This means the battery's reported full-charge capacity is approximately 80% of its original rated capacity.

### 3.4. Calculate Battery Wear

To estimate the percentage of capacity lost:

Battery Wear (%) = ((Design Capacity - Full Charge Capacity) / Design Capacity) × 100

Using the previous example:

`((50,000 - 40,000) / 50,000) × 100 = 20%`

**Estimated Battery Wear: 20%**

Battery health and battery wear are complementary estimates:

Battery Health (%) + Battery Wear (%) = 100%

This relationship applies when both values are calculated from the same valid capacity readings.

---

## 4. Interpret the Result

| Estimated Health | Interpretation |
|---|---|
| Close to 100% | Full-charge capacity is close to the original rated capacity |
| 80%–89% | Some capacity loss is present |
| 60%–79% | Noticeable capacity loss |
| Below 60% | Substantial capacity loss |

These ranges are general guidelines, not universal manufacturer replacement thresholds. Consider actual runtime, unexpected shutdowns, battery warnings, and the device manufacturer's recommendations.

---

## 5. Important Notes

- Battery health is not the current charge percentage. A battery can be at 100% charge while its health is 80%.
- A lower Full Charge Capacity than Design Capacity is normal as a rechargeable battery ages.
- If either capacity value is missing, zero, or unreliable, do not calculate a health percentage from it.
- A battery report estimates capacity health; it does not fully diagnose battery safety or every possible hardware fault.
- If the battery is swollen or the device enclosure is bulging, stop using the device and seek qualified service.
