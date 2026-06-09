# Be Connect+

![Language](https://img.shields.io/badge/language-C%23-239120)
![Framework](https://img.shields.io/badge/.NET%20Framework-4.5-512BD4)
![UI](https://img.shields.io/badge/UI-Windows%20Forms-blue)
![Platform](https://img.shields.io/badge/platform-Windows-0078D6)

**Be Connect+** is a Windows desktop application for peer-to-peer interaction between computers on a local area network (LAN). From a single Material-styled interface it lets you discover other machines on your network and then chat, transfer files, or open a remote desktop session with any of them.

## Features

| Feature | Description | Transport |
| --- | --- | --- |
| 🖥️ **Network Discovery** | Lists the computers currently reachable on your LAN, with hostname and IP address. | ARP table + DNS |
| 💬 **Chat** | Real-time text chat between two computers. | UDP |
| 📁 **File Sharing** | Browse your local drives and send a file to the selected peer, with a progress bar and resumable transfer. | TCP |
| 🔗 **Remote Access** | Open a Remote Desktop (RDP) session to another computer. | RDP ActiveX |

## Tech Stack

- **Language:** C#
- **Framework:** .NET Framework 4.5
- **UI:** Windows Forms, themed with [MaterialSkin](https://github.com/IgnaceMaes/MaterialSkin) 0.2.1 (NuGet)
- **Login database:** Microsoft Access (`.accdb`) accessed via OLE DB (`Microsoft.ACE.OLEDB.12.0`)
- **Remote Desktop:** Microsoft Terminal Services client COM control (`MSTSCLib` / `AxMSTSCLib`)
- **Tooling:** Visual Studio (2012-era solution) / MSBuild

## How It Works

The application launches at the **login screen** (`frm_Login`), which validates credentials against the `tbl_Login` table in the Access database. New users can register from the same screen.

After login, **`Frm_Main0`** acts as the main shell. It hosts each feature as a child form inside a single window, switching between **Home**, **Chat**, **File Transfer**, **Remote Desktop**, and **About** via the side navigation.

The **Home** screen (`frm_Home`) discovers peers by reading the local ARP table (`arp -a`) and resolving hostnames via DNS. Selecting a computer there sets the "active peer" that the Chat, File Transfer, and Remote Desktop features then act on.

| Feature | Default port |
| --- | --- |
| Chat (UDP) | 80 |
| File Transfer (TCP) | 6868 |

## Prerequisites

- **Windows** (the app relies on Windows-only networking and the RDP ActiveX control)
- **.NET Framework 4.5** or later runtime
- **Microsoft Access Database Engine** (provides the `Microsoft.ACE.OLEDB.12.0` provider used for login) — install the bitness that matches how the app is built
- **Visual Studio** (2012 or newer) or the **MSBuild** toolchain, to build from source
- A **LAN with "Network Discovery" enabled** on participating machines

## Getting Started

### Build with Visual Studio

1. Open `Be Connect+.sln`.
2. Restore NuGet packages (Visual Studio does this automatically; the only package is MaterialSkin).
3. Build the solution (**Build ▸ Build Solution**) and run (**F5**).

### Build from the command line (PowerShell)

```powershell
# Restore the MaterialSkin NuGet package (packages.config style)
nuget restore "Be Connect+.sln"

# Build (Debug)
msbuild "Be Connect+.sln" /p:Configuration=Debug

# Run
& "Be Connect+\bin\Debug\Be Connect+.exe"
```

## Configuration

The login database connection string is currently defined in **`Be Connect+/frm_Login.cs`** and points to an absolute path:

```csharp
string PATH = @"Provider=Microsoft.ACE.OLEDB.12.0;Data Source=F:\...\Database\db_Login.accdb";
```

Before running on a new machine, update this to point at the database shipped with the repository:

```
Database/db_Login.accdb
```

## Project Structure

```
Be-Connect/
├─ Be Connect+.sln              # Visual Studio solution
├─ Be Connect+/                 # Main application project
│  ├─ Program.cs                # Entry point → launches frm_Login
│  ├─ frm_Login.cs              # Authentication & user registration
│  ├─ frm_Main_0.cs             # Main application shell (hosts feature forms)
│  ├─ frm_Home.cs               # Network/peer discovery
│  ├─ frm_Chat.cs               # UDP chat
│  ├─ frm_FileTrans.cs          # TCP file transfer (client + embedded server)
│  ├─ frm_RDCInput.cs           # Remote desktop credentials
│  ├─ frm_RemoteDesktop.cs      # Remote desktop (RDP ActiveX) host
│  ├─ frm_InputForm.cs          # Small reusable input dialog
│  └─ frm_About.cs              # About screen
├─ Database/                    # Microsoft Access login database
├─ Icons/ , PNGs/               # Application icons and image assets
└─ packages/                    # NuGet packages (MaterialSkin)
```
