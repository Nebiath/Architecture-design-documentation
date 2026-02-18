# NTP Configuration — Time Synchronization

> **Propietario:** @network-team @lan-lead

## 1. NTP Hierarchy

### 1.1 Stratum Levels

```mermaid

graph TB
    subgraph STRATUM0["Stratum 0 (Reference Clock)"]
        EXT["time.google.com<br/>pool.ntp.org<br/>External NTP Servers"]
    end
    
    subgraph STRATUM1["Stratum 1 - Core Switches"]
        C1["CORE-SW-01<br/>10.10.1.2<br/>ntp master 1"]
        C2["CORE-SW-02<br/>10.10.1.3<br/>ntp master 1"]
    end
    
    subgraph STRATUM2["Stratum 2 - Distribution Switches"]
        D1["DIST-F1-SW-01<br/>10.10.1.10"]
        D2["DIST-F1-SW-02<br/>10.10.1.11"]
        D3["DIST-F2-SW-01"]
        D4["DIST-F2-SW-02"]
    end
    
    subgraph STRATUM3["Stratum 3 - Access Switches"]
        A1["ACC-F1-SW-01<br/>10.10.1.20"]
        A2["ACC-F1-SW-02<br/>10.10.1.21"]
        A3["ACC-F1-SW-03"]
        A4["ACC-F2-SW-01"]
    end
    
    subgraph SERVERS["Stratum 3 - Servers & Devices"]
        S1["Windows Servers<br/>w32time sync"]
        S2["Linux Servers<br/>ntpd/chrony"]
        S3["Network Devices<br/>APs, Firewalls"]
    end
    
    EXT -.->|"NTP Sync<br/>Internet"| C1
    EXT -.->|"NTP Sync<br/>Internet"| C2
    
    C1 ==>|"ntp server<br/>10.10.1.2"| D1
    C1 ==>|"ntp server<br/>10.10.1.2"| D2
    C1 ==>|"ntp server<br/>10.10.1.2"| D3
    C1 ==>|"ntp server<br/>10.10.1.2"| D4
    
    C2 ==>|"ntp server<br/>10.10.1.3"| D1
    C2 ==>|"ntp server<br/>10.10.1.3"| D2
    C2 ==>|"ntp server<br/>10.10.1.3"| D3
    C2 ==>|"ntp server<br/>10.10.1.3"| D4
    
    D1 -->|"ntp server<br/>10.10.1.10"| A1
    D2 -->|"ntp server<br/>10.10.1.11"| A2
    D1 -->|"ntp server<br/>10.10.1.10"| A3
    D3 -->|"ntp server"| A4
    
    D1 -->|"NTP"| S1
    D2 -->|"NTP"| S2
    D1 -->|"NTP"| S3
    
    classDef stratum0Style fill:#e63946,stroke:#9d0208,stroke-width:3px,color:#fff
    classDef stratum1Style fill:#ff6b6b,stroke:#c92a2a,stroke-width:3px,color:#fff
    classDef stratum2Style fill:#4ecdc4,stroke:#087f5b,stroke-width:2px,color:#fff
    classDef stratum3Style fill:#95e1d3,stroke:#0c8599,stroke-width:2px,color:#000
    classDef serverStyle fill:#f9f9f9,stroke:#868e96,stroke-width:1px,color:#000
    
    class EXT stratum0Style
    class C1,C2 stratum1Style
    class D1,D2,D3,D4 stratum2Style
    class A1,A2,A3,A4 stratum3Style
    class S1,S2,S3 serverStyle
```
🎨 Características de los diagramas
Colores diferenciados por capa:

🔴 Core: Rojo (#ff6b6b)

🔵 Distribution: Cyan (#4ecdc4)

🟢 Access: Verde claro (#95e1d3)

🟡 VLANs/SSIDs: Amarillo (#ffd43b)

🟣 Controllers: Púrpura (#845ef7)

⚪ End devices: Gris claro (#f9f9f9)


---

## 2. NTP Configuration on Switches

### 2.1 Core Switches (Stratum 1)
```cisco
! Sincronizar con fuente externa
ntp server time.google.com prefer
ntp server pool.ntp.org

! Permitir que otros switches sincronicen
ntp master 1
```

### 2.2 Distribution/Access (Stratum 2/3)
```cisco
! Sincronizar con core switches
ntp server 10.10.1.2
ntp server 10.10.1.3
```

### 2.3 Verificación
```cisco
show ntp status
show ntp associations
```

---

## Changelog
| Fecha | Autor | Cambio |
|-------|-------|--------|
| YYYY-MM-DD | @lan-lead | Creación inicial |
