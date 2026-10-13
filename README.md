# Sysmon Integration with Wazuh

### Advanced Endpoint Monitoring and Threat Detection in a SOC Lab

## 📌 Project Overview

This project focuses on integrating **Microsoft Sysmon with Wazuh SIEM** to improve endpoint visibility and identify potentially suspicious activity on a Windows system.

Sysmon provides detailed system telemetry, including process creation, file creation, network connections, and module loading. Wazuh collects these events from the Windows endpoint and makes them available for centralized monitoring, alert investigation, and MITRE ATT&CK analysis.

The project was implemented in a home SOC lab using a Windows 11 endpoint running both the Wazuh agent and Sysmon. A controlled PowerShell command was used to validate the monitoring workflow, and the resulting events were investigated through the Wazuh dashboard.

The goal was to understand how endpoint telemetry can help SOC analysts investigate suspicious behavior and gain better visibility into activity that may not be obvious from basic Windows logging alone.

## 🎯 Project Objectives

* Install and configure Sysmon on a Windows endpoint.

* Apply a community-maintained Sysmon configuration.

* Verify that Sysmon is generating events in Windows Event Viewer.

* Configure the Wazuh agent to collect Sysmon Operational logs.

* Generate test activity to validate event collection and detection.

* Investigate alerts through the Wazuh dashboard.

* Explore MITRE ATT&CK technique and tactic mappings.

* Analyze process activity, file creation events, and associated event details.

## 🏗️ Lab Architecture

```
┌──────────────────────────────────┐
│       Windows 11 Endpoint        │
│                                  │
│  ┌────────────────────────────┐  │
│  │          Sysmon            │  │
│  │ Process and File Events    │  │
│  │ Module Load Telemetry      │  │
│  └─────────────┬──────────────┘  │
│                │                 │
│                ▼                 │
│  ┌────────────────────────────┐  │
│  │        Windows Event Log   │  │
│  │ Sysmon/Operational Channel │  │
│  └─────────────┬──────────────┘  │
│                │                 │
│                ▼                 │
│  ┌────────────────────────────┐  │
│  │         Wazuh Agent        │  │
│  └─────────────┬──────────────┘  │
└────────────────┼─────────────────┘
                 │
                 │ Event Collection
                 ▼
┌──────────────────────────────────┐
│         Wazuh Manager            │
│                                  │
│  Event Processing and Alerting   │
│  Security Event Analysis         │
│  MITRE ATT&CK Mapping            │
└────────────────┬─────────────────┘
                 │
                 ▼
┌──────────────────────────────────┐
│        Wazuh Dashboard           │
│                                  │
│  Security Alerts                 │
│  Threat Hunting                  │
│  Event Investigation             │
│  MITRE ATT&CK Visualization      │
└──────────────────────────────────┘
```

## 🛠️ Tools and Technologies

| Component            | Description                                                          |
| -------------------- | -------------------------------------------------------------------- |
| Wazuh                | SIEM platform for centralized security monitoring and alert analysis |
| Microsoft Sysmon     | Detailed Windows system activity monitoring                          |
| Windows 11           | Monitored endpoint operating system                                  |
| Sysmon Modular       | Community-maintained Sysmon configuration                            |
| Windows Event Viewer | Verification of Sysmon event collection                              |
| MITRE ATT\&CK        | Framework for understanding adversary tactics and techniques         |
| PowerShell           | Used to generate controlled test activity                            |

### Lab Environment

* **SIEM:** Wazuh Manager and Dashboard

* **Endpoint:** Windows 11

* **Endpoint agent:** `window_agent`

* **Sysmon version:** 15.22

* **Sysmon configuration:** Sysmon Modular, Balanced profile

* **Event channel:** `Microsoft-Windows-Sysmon/Operational`

* **Wazuh agent configuration:** `agent.conf`, applied to the `window_Lab` group

## ⚙️ Implementation

### 1. Install Sysmon

Download Sysmon from the official Microsoft Sysinternals website:

[https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
<img width="814" height="333" alt="Screenshot 2026-10-01 155710" src="https://github.com/user-attachments/assets/4f9e4bc2-64e1-4bec-bdfc-fd58f560937c" />


Extract the downloaded archive to a directory such as `C:\sysmon`.

Open PowerShell as Administrator and navigate to the installation directory.

### 2. Apply the Sysmon Configuration

This project uses the Balanced profile from the community-maintained Sysmon Modular project.

Repository:

[https://github.com/olafhartong/sysmon-modular](https://github.com/olafhartong/sysmon-modular)

Download the appropriate generated configuration file and place it alongside the Sysmon executable.

Install Sysmon with the configuration:

```
.\Sysmon64.exe -accepteula -i sysmonconfig.xml
```
<img width="975" height="510" alt="Screenshot 2026-10-01 160244" src="https://github.com/user-attachments/assets/57d5d128-e124-45a8-bc9b-bd23380a5aa1" />

The configuration determines which events Sysmon records and helps balance useful security visibility against excessive event volume.

### 3. Verify Sysmon Events

Open Windows Event Viewer and navigate to:

```
Applications and Services Logs
  └── Microsoft
      └── Windows
          └── Sysmon
              └── Operational
```

Verify that events are being generated. Depending on the configuration and activity on the endpoint, these can include process creation, file creation, and module-load events.

### 4. Configure Wazuh Log Collection

The Wazuh agent must collect events from the Sysmon Operational channel.

In this project, the configuration was applied through the `window_Lab` agent group in the Wazuh manager.

Example configuration:

```
<agent_config os="windows">

  <!-- Collect Sysmon Operational events -->
  <localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
  </localfile>

</agent_config>
```
<img width="1335" height="296" alt="Screenshot 2026-10-01 155602" src="https://github.com/user-attachments/assets/1d278702-3bc9-4c77-bfc4-8348f95d04e4" />

Add this block to the appropriate agent group configuration in the Wazuh manager. The exact group configuration file and deployment method depend on your Wazuh setup.

Apply the updated configuration and verify that the agent receives it successfully.

### 5. Validate Event Collection

Once Sysmon and Wazuh are configured, generate controlled test activity on the Windows endpoint.

The objective is to confirm that:

1. Sysmon records the activity.

2. The event appears in the Windows Sysmon Operational log.

3. The Wazuh agent collects the event.

4. The corresponding event or alert becomes visible in the Wazuh dashboard.

## 🧪 Detection Testing

To validate the monitoring pipeline, the project used a controlled PowerShell command:

```
powershell.exe -nop -w hidden -c "Write-Host 'Testing Fileless Detection'"
```
<img width="725" height="60" alt="Screenshot 2026-10-01 160326" src="https://github.com/user-attachments/assets/2d3a39d0-ca93-4a0e-813f-e26d36c8b26e" />

This command launches PowerShell with no profile and a hidden window, then prints a test message. It is a benign demonstration command, not a real fileless malware sample.

The purpose was to observe how Sysmon records the process activity and how Wazuh handles the resulting telemetry.

The project also examined routine endpoint activity, including browser-related scripting file creation and other process and module-load events.

**Important:** A logged event does not automatically mean a successful malicious execution or a confirmed compromise. Alerts must be investigated in context.

## 🔎 Alert Investigation

After configuring log collection, the generated events were reviewed in the Wazuh dashboard.

The investigation focused on:

* Process and parent-process information

* File creation activity

* Target file paths

* Sysmon event IDs

* Rule descriptions and classifications

* MITRE ATT&CK mappings

* Relevant user and endpoint details
<img width="1352" height="570" alt="Screenshot 2026-10-01 155358" src="https://github.com/user-attachments/assets/c8297220-bd29-4be9-adb5-e1e91133e0b9" />

One documented alert involved Firefox creating a JavaScript file in a user-profile directory. The event was classified by the configured ruleset under MITRE ATT&CK technique **T1059.007 — JavaScript**.

The report also documented a module-load event involving `taskschd.dll`. Such activity may warrant investigation depending on its context, but the event alone does not establish malicious behavior.

## 🗺️ MITRE ATT&CK Mapping

The Wazuh dashboard provides MITRE ATT&CK mappings for applicable alerts, helping analysts relate observed endpoint activity to known adversary techniques.

The project report documented mappings including:

* **T1059.007:** Command and Scripting Interpreter — JavaScript

* **T1059:** Command and Scripting Interpreter

* **T1105:** Ingress Tool Transfer
<img width="1362" height="570" alt="Screenshot 2026-10-01 161112" src="https://github.com/user-attachments/assets/13279921-2c52-4595-b974-1749e545462a" />

These mappings provide investigative context. They should not be interpreted as proof that an attacker successfully performed the corresponding technique.

## 📊 Key Findings

The project demonstrated the following:

* Sysmon provided detailed endpoint telemetry for process creation, file creation, and module-load activity.

* Wazuh successfully collected Sysmon Operational events through the Windows Event Channel.

* The configured ruleset generated alerts for selected endpoint activities, including scripting-file creation and the controlled PowerShell test.

* Relevant alerts included MITRE ATT&CK technique and tactic information.

* Event details preserved useful investigation context, such as process identifiers, parent-process information, file paths, and user details.

* The Wazuh dashboard provided a centralized location for reviewing endpoint events and investigating alerts.


## 👨‍💻 Author

**Muhammad Khubab**

**Focus:** SIEM Administration and Endpoint Threat Detection

* **GitHub:** [mkhubab11](https://github.com/mkhubab11)

* **LinkedIn:** [Muhammad Khubab](https://www.linkedin.com/in/muhammad-khubab-475046204)

