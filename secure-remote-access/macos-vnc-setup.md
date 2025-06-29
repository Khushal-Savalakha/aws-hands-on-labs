# Access Your Mac Remotely from Windows Using TightVNC

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform](https://img.shields.io/badge/Platform-macOS%20%7C%20Windows-blue.svg)](https://github.com)
[![VNC](https://img.shields.io/badge/Protocol-VNC-green.svg)](https://en.wikipedia.org/wiki/Virtual_Network_Computing)

A comprehensive guide for setting up secure remote desktop access from Windows to macOS using TightVNC. This tutorial provides step-by-step instructions with visual aids to help you establish a reliable remote connection.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [System Architecture](#system-architecture)
- [Setup Instructions](#setup-instructions)
  - [Part 1: macOS Configuration](#part-1-macos-configuration)
  - [Part 2: Windows Client Setup](#part-2-windows-client-setup)
  - [Part 3: Establishing Connection](#part-3-establishing-connection)

## Overview

This guide demonstrates how to enable remote desktop access to your macOS machine from a Windows computer using TightVNC. The setup involves configuring macOS's built-in screen sharing feature and connecting through TightVNC Viewer on Windows.

## Prerequisites

**macOS Requirements:**
- macOS 10.14 (Mojave) or later
- Administrator privileges
- Network connectivity

**Windows Requirements:**
- Windows 10 or later
- TightVNC Viewer (provided in guide)
- Network access to the macOS machine

## System Architecture

```mermaid
flowchart TB
    subgraph "macOS Setup"
        A[System Settings] --> B[Users & Groups Verification]
        B --> C[Enable Screen Sharing]
        C --> D[Configure Access Password]
        D --> E[Note Local Hostname]
    end
    
    subgraph "Windows Setup"
        F[Download TightVNC] --> G[Custom Installation]
        G --> H[Remove Server Component]
        H --> I[Launch TightVNC Viewer]
    end
    
    subgraph "Connection Process"
        J[Enter macOS Hostname] --> K[Authenticate with Password]
        K --> L[Access macOS Desktop]
    end
    
    E --> J
    I --> J
    
    style A fill:#e1f5fe
    style F fill:#f3e5f5
    style J fill:#e8f5e8
```

## Setup Instructions

### Part 1: macOS Configuration

#### Step 1: Access System Settings

Navigate to **System Settings** from the Apple menu or Spotlight search.

![System Settings Access](https://github.com/user-attachments/assets/9fe4ecbf-dea6-404a-93ae-f14b96125957)

#### Step 2: Verify Current User Account

Open **Users & Groups** to confirm the current user account that will be used for remote access.

![Users & Groups Verification](https://github.com/user-attachments/assets/bf606406-aeb8-478b-a658-20e19a75261f)

#### Step 3: Navigate to Sharing Settings

Go to **General** → **Sharing** to access network sharing options.

![Sharing Settings Navigation](https://github.com/user-attachments/assets/f2ef454b-c3d7-41b8-b96b-e7136f844b53)

#### Step 4: Record Local Hostname

**Important:** Note the **Local Hostname** displayed at the bottom of the sharing panel. This identifier will be used to establish the connection from Windows.

![Local Hostname Identification](https://github.com/user-attachments/assets/85865350-3c79-41be-8986-4244edd72d33)

#### Step 5: Enable Screen Sharing

Toggle **Screen Sharing** to **ON** and set a secure password for VNC access. This password will be required when connecting from the Windows client.

![Screen Sharing Configuration](https://github.com/user-attachments/assets/03850f04-8d8b-46fc-86ae-2a77a526ca2f)

### Part 2: Windows Client Setup

#### Step 6: Download TightVNC

Search for **TightVNC** in your preferred web browser to access the official download page.

![TightVNC Search](https://github.com/user-attachments/assets/e07cf6ff-76d8-446d-b87d-00fed55310b1)

#### Step 7: Select Appropriate Version

Download the TightVNC installer that matches your Windows system architecture (32-bit or 64-bit).

![TightVNC Download Selection](https://github.com/user-attachments/assets/c98f422a-7100-4316-9ee6-56e63962dce3)

#### Step 8: Choose Custom Installation

During the installation process, select **Custom Setup** to customize the installation components.

![Custom Installation Selection](https://github.com/user-attachments/assets/def7f7dd-ffc5-479f-897b-ebde134e8b5e)

#### Step 9: Remove Server Component

**Important:** Remove the **TightVNC Server** feature from the installation. We only need the viewer component for connecting to macOS.

![Server Component Removal](https://github.com/user-attachments/assets/5f9fc88f-edf1-48e1-a679-0d5b2f73f251)

#### Step 10: Complete Installation

Verify the server component removal (indicated by the cross mark) and proceed with the installation.

![Installation Completion](https://github.com/user-attachments/assets/b31f27af-a518-4bbe-9f30-c5edfe977ba9)

### Part 3: Establishing Connection

#### Step 11: Launch TightVNC Viewer

Search for and launch **TightVNC Viewer** from the Windows Start menu.

![TightVNC Viewer Launch](https://github.com/user-attachments/assets/fcd9a468-fae8-422d-8db5-4b13726f52f2)

#### Step 12: Enter Connection Details

In the **Remote Host** field, enter the macOS local hostname noted in Step 4, then click **Connect**.

![Connection Configuration](https://github.com/user-attachments/assets/1924b298-9a08-4f00-aac2-3fe96d5d72b4)

#### Step 13: Authenticate

Enter the screen sharing password configured in Step 5.

![Authentication](https://github.com/user-attachments/assets/bb49e579-099a-4fba-9233-59efacb9b6ab)

#### Step 14: macOS Login

You will see the macOS login screen. Enter your macOS user account credentials.

![macOS Login Screen](https://github.com/user-attachments/assets/e33d6f16-15d8-4434-b44f-bea2e9a1bb88)

#### Step 15: Successful Connection

You now have remote access to your macOS desktop from Windows!

![Successful Remote Access](https://github.com/user-attachments/assets/c6973976-25a2-43fd-9bf3-521ecd7eae9f)

## License

This documentation is provided under the MIT License. See the LICENSE file for details.
