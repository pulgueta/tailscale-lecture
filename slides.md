---
theme: apple-basic
title: "Tailscale: Redes Mesh Modernas con WireGuard"
info: |
  ## Tailscale & WireGuard
  Una inmersión profunda en cómo Tailscale revoluciona las redes privadas usando WireGuard.
class: text-center
drawings:
  persist: false
layout: center
transition: slide-left
colorSchema: dark
fonts:
  sans: Geist Mono
  serif: Geist Mono
  mono: "Geist Mono, monospace"
  fallbacks: false
htmlAttrs:
  lang: es
---

<div class="flex items-center gap-4 max-w-max mx-auto">
  <img src="./tailscale-logo.svg" alt="Tailscale" class="mx-auto mb-6 block size-16" />
  <h1>Tailscale</h1>
</div>

VPN peer-to-peer, cifrado E2E y zero-config

---

# Contenido

1. El problema de las VPN tradicionales
2. Wireguard
3. Arquitectura de Tailscale
4. Casos de uso
5.

---

# El problema de las VPN tradicionales

```mermaid {theme: 'dark', scale: 0.6}
graph TD
    subgraph "Modelo Hub-and-Spoke"
        C["Concentrador VPN - San Francisco"]
        A["Usuario A - Nueva York"] -->|"Túnel cifrado"| C
        B["Usuario B - Londres"] -->|"Túnel cifrado"| C
        D["Servidor 1 - Nueva York"] -->|"Túnel cifrado"| C
        E["Servidor 2 - Tokio"] -->|"Túnel cifrado"| C
        F["Usuario C - Berlín"] -->|"Túnel cifrado"| C
    end
    style C fill:#ef4444,stroke:#dc2626,color:#fff
    style A fill:#1e293b,stroke:#334155,color:#94a3b8
    style B fill:#1e293b,stroke:#334155,color:#94a3b8
    style D fill:#1e293b,stroke:#334155,color:#94a3b8
    style E fill:#1e293b,stroke:#334155,color:#94a3b8
    style F fill:#1e293b,stroke:#334155,color:#94a3b8
```

<v-clicks>

- Todo el tráfico pasa por un <span v-mark.red="1">único</span> punto central
- El usuario en NY accediendo al servidor en NY viaja a SF ida y vuelta
- Si el concentrador cae, **toda la red cae**

</v-clicks>

---

# VPN Mesh Completa: La Alternativa Costosa

```mermaid {theme: 'dark', scale: 0.75}
graph LR
    A["Nodo A"] <-->|"Túnel"| B["Nodo B"]
    A <-->|"Túnel"| C["Nodo C"]
    A <-->|"Túnel"| D["Nodo D"]
    A <-->|"Túnel"| E["Nodo E"]
    B <-->|"Túnel"| C
    B <-->|"Túnel"| D
    B <-->|"Túnel"| E
    C <-->|"Túnel"| D
    C <-->|"Túnel"| E
    D <-->|"Túnel"| E
    style A fill:#3b82f6,stroke:#2563eb,color:#fff
    style B fill:#8b5cf6,stroke:#7c3aed,color:#fff
    style C fill:#ec4899,stroke:#db2777,color:#fff
    style D fill:#10b981,stroke:#059669,color:#fff
    style E fill:#f59e0b,stroke:#d97706,color:#fff
```

<div class="grid grid-cols-2 gap-8 mt-4">
  <div v-click>
    <h3 class="text-red-400 mb-2">Problema de escala</h3>
    <div class="p-3 bg-gray-900/50 rounded border border-gray-800" >
      <div>10 nodos = <span class="text-red-400 font-bold">45 túneles</span></div>
      <div>100 nodos = <span class="text-red-400 font-bold">4950 túneles</span></div>
      <div class="text-gray-500 mt-1">Fórmula: n(n - 1) / 2</div>
    </div>
  </div>
  <div v-click>
    <h3 class="text-green-400 mb-2">Ventaja</h3>
    <div class="p-3 bg-gray-900/50 rounded border border-gray-800" >
      <div>Conexiones directas peer-to-peer</div>
      <div>Mínima latencia posible</div>
      <div class="text-gray-500 mt-1">Sin punto único de fallo</div>
    </div>
  </div>
</div>

---

# WireGuard

## Primitivas criptográficas:

<div class="grid grid-cols-2 gap-4 mt-4">
  <div>
    <v-clicks>
    <div class="mb-4 p-4 rounded-lg border border-gray-800 bg-gray-900/50">
      <div class="text-blue-400 font-bold mb-1">Noise Protocol Framework</div>
      <div class="text-sm text-gray-400 text-pretty">Handshake y establecimiento de claves. Perfect forward secrecy y autenticación mutua.</div>
    </div>
    <div class="mb-4 p-4 rounded-lg border border-gray-800 bg-gray-900/50">
      <div class="text-purple-400 font-bold mb-1">Curve25519</div>
      <div class="text-sm text-gray-400 text-pretty">Curva elíptica para intercambio de claves Diffie-Hellman.</div>
    </div>
    <div class="mb-4 p-4 rounded-lg border border-gray-800 bg-gray-900/50">
      <div class="text-pink-400 font-bold mb-1">ChaCha20-Poly1305</div>
      <div class="text-sm text-gray-400 text-pretty">Cifrado simétrico autenticado (AEAD). Más rápido que AES en CPUs sin AES-NI.</div>
    </div>
    </v-clicks>
  </div>
  <div>
    <v-clicks>
    <div class="mb-4 p-4 rounded-lg border border-gray-800 bg-gray-900/50">
      <div class="text-green-400 font-bold mb-1">BLAKE2s</div>
      <div class="text-sm text-gray-400 text-pretty">Función hash criptográfica. Más rápida que SHA-256 con el mismo nivel de seguridad.</div>
    </div>
    <div class="mb-4 p-4 rounded-lg border border-gray-800 bg-gray-900/50">
      <div class="text-yellow-400 font-bold mb-1">SipHash24</div>
      <div class="text-sm text-gray-400 text-pretty">Hash para tablas internas. Previene ataques de colisión en estructuras de datos.</div>
    </div>
    <div class="mb-4 p-4 rounded-lg border border-gray-800 bg-gray-900/50">
      <div class="text-cyan-400 font-bold mb-1">HKDF</div>
      <div class="text-sm text-gray-400 text-pretty">Derivación de claves. Expande el material del handshake en claves de sesión.</div>
    </div>
    </v-clicks>
  </div>
</div>

---

# Cryptokey Routing: El concepto clave

La idea central de WireGuard: cada clave pública se asocia con una lista de IPs permitidas.

```ini
# Configuración del Servidor WireGuard
[Interface]
PrivateKey = yAnz5TF+lXXJte14tji3zlMNq+hd2rYUIgJBgB3fBmk=
ListenPort = 51820

[Peer]
# Laptop
PublicKey  = xTIBA5rboUvnH4htodjb6e697QjLERt1NAB4mZqp8Dg=
AllowedIPs = 10.192.122.3/32

[Peer]
# Celular
PublicKey  = TrMvSoP4jYQlY6RIzBgbssQqY3vxI2piVFBs2LRkCwk=
AllowedIPs = 10.192.122.4/32, 192.168.0.0/16
```

<!-- <div class="mt-4 grid grid-cols-2 gap-4">
  <div v-click class="p-3 bg-green-900/20 border border-green-800 rounded" >
    <span class="text-green-400 font-bold">Enviar:</span> AllowedIPs actúa como <span class="text-green-300">tabla de rutas</span>
  </div>
  <div v-click class="p-3 bg-blue-900/20 border border-blue-800 rounded" >
    <span class="text-blue-400 font-bold">Recibir:</span> AllowedIPs actúa como <span class="text-blue-300">lista de control de acceso</span>
  </div>
</div> -->

---

# Flujo en Wireguard

```mermaid {theme: 'dark', scale: 0.45}
sequenceDiagram
    participant App as Aplicación
    participant WG as Interfaz WireGuard
    participant Net as Red UDP
    participant WG2 as WireGuard Remoto
    participant App2 as App Remota
    App->>WG: Paquete IP destino 10.0.0.2
    WG->>WG: 1. Buscar peer por AllowedIPs
    WG->>WG: 2. Cifrar con clave del peer
    WG->>Net: 3. Encapsular en UDP
    Net->>WG2: 4. Paquete UDP llega
    WG2->>WG2: 5. Descifrar y autenticar
    WG2->>WG2: 6. Verificar IP vs AllowedIPs
    WG2->>App2: 7. Entregar paquete descifrado
    Note over WG2,Net: Actualizar endpoint del peer desde IP origen
```

<div v-click class="mt-4 p-3 bg-purple-900/20 border border-purple-800 rounded text-sm">
  <span class="text-purple-400 font-bold">Roaming integrado:</span> WireGuard actualiza el endpoint de cada peer al recibir un paquete autenticado. Ambos lados pueden cambiar de IP sin reconfiguración.
</div>

---

# WireGuard: Por qué es tan rápido

```mermaid {theme: 'dark', scale: 0.7}
graph TD
    subgraph "OpenVPN - userspace"
        A1["Paquete"] --> A2["Kernel"]
        A2 --> A3["Userspace OpenVPN"]
        A3 --> A4["Kernel"]
        A4 --> A5["Red"]
    end
    style A3 fill:#ef4444,stroke:#dc2626,color:#fff
```

```mermaid {theme: 'dark', scale: 0.7}
graph TD
    subgraph "WireGuard - kernel"
        B1["Paquete"] --> B2["Kernel WireGuard"]
        B2 --> B3["Red"]
    end
    style B2 fill:#22c55e,stroke:#16a34a,color:#fff
```

<v-clicks>

- **Sin context switches**: todo ocurre dentro del kernel, sin copiar datos al userspace
- **Primitivas rápidas**: ChaCha20 y Poly1305 son eficientes en CPUs ARM y x86 sin AES-NI
- **Interfaz de red nativa**: WireGuard se presenta como `wg0`, una interfaz estándar del sistema operativo

</v-clicks>

---

# Arquitectura de Tailscale

## Control Plane vs Data Plane

```mermaid {theme: 'dark', scale: 0.5}
graph TB
    subgraph CP["PLANO DE CONTROL - Centralizado"]
        CS["Servidor de Coordinación"]
        Auth["Proveedor OAuth2/OIDC"]
        ACL["Políticas ACL"]
        Auth --> CS
        ACL --> CS
    end
    subgraph DP["PLANO DE DATOS - Mesh Distribuido"]
        N1["Nodo A - 10.100.0.1"]
        N2["Nodo B - 10.100.0.2"]
        N3["Nodo C - 10.100.0.3"]
        N1 <-->|"WireGuard P2P"| N2
        N2 <-->|"WireGuard P2P"| N3
        N1 <-->|"WireGuard P2P"| N3
    end
    CS -.->|"Claves + ACLs"| N1
    CS -.->|"Claves + ACLs"| N2
    CS -.->|"Claves + ACLs"| N3
    style CP fill:none,stroke:#3b82f6,stroke-width:2
    style DP fill:none,stroke:#22c55e,stroke-width:2
    style CS fill:#1e40af,stroke:#3b82f6,color:#fff
    style Auth fill:#7c3aed,stroke:#8b5cf6,color:#fff
    style ACL fill:#7c3aed,stroke:#8b5cf6,color:#fff
    style N1 fill:#166534,stroke:#22c55e,color:#fff
    style N2 fill:#166534,stroke:#22c55e,color:#fff
    style N3 fill:#166534,stroke:#22c55e,color:#fff
```

<div class="grid grid-cols-2 gap-4 mt-2">
  <div v-click class="p-3 bg-blue-900/20 border border-blue-800 rounded" style="font-family: 'Geist Mono', monospace; font-size: 0.75rem;">
    Solo distribuye claves públicas y políticas. Nunca toca los datos. Si cae, las conexiones existentes siguen funcionando.
  </div>
  <div v-click class="p-3 bg-green-900/20 border border-green-800 rounded" style="font-family: 'Geist Mono', monospace; font-size: 0.75rem;">
    <span class="text-green-400 font-bold">Plano de Datos:</span> Conexiones WireGuard directas peer-to-peer. El tráfico nunca pasa por Tailscale. Cifrado extremo a extremo.
  </div>
</div>

---

# Intercambio de claves: El buzón compartido

```mermaid {theme: 'dark', scale: 0.5}
sequenceDiagram
    participant A as Nodo A
    participant CS as Servidor Coordinación
    participant B as Nodo B
    Note over A: Genera par de claves
    A->>CS: 1. Sube clave pública + IP actual
    Note over CS: Almacena como buzón compartido
    B->>CS: 2. Sube clave pública + IP actual
    CS->>A: 3. Entrega clave pública de B + endpoint
    CS->>B: 3. Entrega clave pública de A + endpoint
    Note over A,B: Las claves privadas NUNCA salen del nodo
    A->>B: 4. Túnel WireGuard directo P2P
    Note over A,B: Cifrado E2E - el coordinador no puede descifrar
```

<div v-click class="mt-4 p-3 bg-yellow-900/20 border border-yellow-800 rounded text-sm">
  <span class="text-yellow-400 font-bold">Principio clave:</span> Tailscale externaliza la autenticación a proveedores OAuth2/OIDC (Google, Microsoft, GitHub). Esto minimiza la información personal que almacena.
</div>

---

# NAT Traversal: Conectando lo imposible

```mermaid {theme: 'dark', scale: 0.65}
graph LR
    subgraph "Red A - NAT"
        NA["Nodo A - 192.168.1.10"]
    end
    subgraph "Internet"
        STUN["Servidor STUN"]
        DERP["Servidor DERP - Relay"]
    end
    subgraph "Red B - NAT"
        NB["Nodo B - 10.0.0.5"]
    end
    NA -->|"1. ¿Qué IP tengo?"| STUN
    NB -->|"1. ¿Qué IP tengo?"| STUN
    STUN -.->|"2. 203.0.113.5:4820"| NA
    STUN -.->|"2. 198.51.100.3:9281"| NB
    NA <-->|"3. Conexión directa UDP - hole punching"| NB
    style STUN fill:#3b82f6,stroke:#2563eb,color:#fff
    style DERP fill:#f59e0b,stroke:#d97706,color:#fff
    style NA fill:#166534,stroke:#22c55e,color:#fff
    style NB fill:#166534,stroke:#22c55e,color:#fff
```

<div class="grid grid-cols-3 gap-3 mt-4 text-sm">
  <div v-click class="p-3 bg-blue-900/20 border border-blue-800 rounded">
    <div class="text-blue-400 font-bold mb-1">Paso 1: STUN</div>
    Cada nodo pregunta a un servidor STUN su IP pública y puerto.
  </div>
  <div v-click class="p-3 bg-green-900/20 border border-green-800 rounded">
    <div class="text-green-400 font-bold mb-1">Paso 2: Hole Punching</div>
    Ambos envían paquetes UDP simultáneamente, perforando sus NATs.
  </div>
  <div v-click class="p-3 bg-yellow-900/20 border border-yellow-800 rounded">
    <div class="text-yellow-400 font-bold mb-1">Paso 3: DERP</div>
    Si UDP está bloqueado, servidores DERP retransmiten tráfico cifrado.
  </div>
</div>

---

# DERP: Designated Encrypted Relay for Packets

```mermaid {theme: 'dark', scale: 0.5}
graph TD
    subgraph "Red corporativa - UDP bloqueado"
        A["Nodo A"]
    end
    subgraph "Servidores DERP globales"
        D1["DERP NYC"]
        D2["DERP SFO"]
        D3["DERP FRA"]
    end
    subgraph "Red doméstica"
        B["Nodo B"]
    end
    A -->|"HTTPS - TCP"| D1
    D1 -->|"UDP"| B
    style A fill:#ef4444,stroke:#dc2626,color:#fff
    style B fill:#22c55e,stroke:#16a34a,color:#fff
    style D1 fill:#f59e0b,stroke:#d97706,color:#fff
    style D2 fill:#6b7280,stroke:#4b5563,color:#9ca3af
    style D3 fill:#6b7280,stroke:#4b5563,color:#9ca3af
```

<v-clicks>

- **¿Cuándo se usa?** Redes restrictivas que bloquean UDP. DERP usa HTTPS (TCP) como transporte
- **¿Es seguro?** Sí. Las claves privadas nunca salen del nodo.
- **Enrutamiento inteligente** Los paquetes van al DERP más cercano al destinatario, no al emisor

</v-clicks>

---

# ACLs: Seguridad Distribuida

<div class="grid grid-cols-2 gap-4 lg:gap-6 mt-4 w-full max-w-full items-start text-left">

<div class="min-w-0 flex flex-col gap-2">

<h3 class="text-sm font-semibold text-gray-100 shrink-0">Modelo tradicional</h3>

```mermaid {theme: 'dark', scale: 0.5}
graph TD
    FW["Firewall Central"] --> S1["Servidor 1"]
    FW --> S2["Servidor 2"]
    FW --> S3["Servidor 3"]
    U1["Usuario"] --> FW
    U2["Usuario"] --> FW
    style FW fill:#ef4444,stroke:#dc2626,color:#fff
```

<p class="mt-1 p-2 bg-red-900/20 border border-red-800 rounded text-xs leading-snug max-w-max">
Si el firewall cae, todo queda expuesto.
</p>

</div>

<div class="min-w-0 flex flex-col gap-2">

<h3 class="text-sm font-semibold text-gray-100 shrink-0">Modelo Tailscale</h3>

```mermaid {theme: 'dark', scale: 0.5}
graph TD
    CS["Coordinador"] -.->|"Políticas"| N1
    CS -.->|"Políticas"| N2
    CS -.->|"Políticas"| N3
    N1["Nodo 1 - Filtra"] <--> N2["Nodo 2 - Filtra"]
    N2 <--> N3["Nodo 3 - Filtra"]
    style CS fill:#3b82f6,stroke:#2563eb,color:#fff
    style N1 fill:#166534,stroke:#22c55e,color:#fff
    style N2 fill:#166534,stroke:#22c55e,color:#fff
    style N3 fill:#166534,stroke:#22c55e,color:#fff
```

<p class="mt-1 p-2 bg-green-900/20 border border-green-800 rounded text-xs leading-snug max-w-max">
Cada nodo aplica reglas al descifrar. Sin la regla "accept", se rechaza.
</p>

</div>

</div>

---

# MagicDNS y Tailnet

<div class="grid grid-cols-2 gap-8 mt-6">
  <div>
    <v-clicks>
    <div class="mb-4 p-4 rounded-lg border border-gray-800 bg-gray-900/50">
      <div class="text-blue-400 font-bold mb-2">MagicDNS</div>
      <div class="text-sm text-gray-400">
        En vez de recordar <code>100.64.0.3</code>, simplemente se usa <code>mi-laptop</code> o <code>mi-laptop.tail12345.ts.net</code>.
      </div>
    </div>
    <div class="mb-4 p-4 rounded-lg border border-gray-800 bg-gray-900/50">
      <div class="text-purple-400 font-bold mb-2">Tailnet</div>
      <div class="text-sm text-gray-400">
        Nuestra red privada virtual. Cada cuenta tiene un dominio único <code>*.ts.net</code>.
      </div>
    </div>
    <div class="mb-4 p-4 rounded-lg border border-gray-800 bg-gray-900/50">
      <div class="text-green-400 font-bold mb-2">HTTPS automático</div>
      <div class="text-sm text-gray-400">
        Certificados TLS para dominios <code>*.ts.net</code> sin configuración manual.
      </div>
    </div>
    </v-clicks>
  </div>
  <div v-click>
    <div class="p-4 rounded-lg border border-gray-800 bg-gray-900/80" >
      <div class="text-gray-500 mb-2"># Acceder a servicios</div>
      <div class="mb-3">
        <span class="text-gray-500">$</span> <span class="text-green-400">ssh</span> mi-servidor
      </div>
      <div class="mb-3">
        <span class="text-gray-500">$</span> <span class="text-green-400">curl</span> http://pi-hole:8080/admin
      </div>
      <div class="mb-3">
        <span class="text-gray-500">$</span> <span class="text-green-400">psql</span> -h db-prod postgres
      </div>
      <div class="text-gray-600 mt-4 border-t border-gray-700 pt-2">
        Sin IPs, sin puertos públicos, sin DNS manual
      </div>
    </div>
  </div>
</div>

---

# Casos de Uso Reales

## Caso 1: Homelab accesible desde cualquier lugar

```mermaid {theme: 'dark', scale: 0.7}
graph TD
    subgraph "Casa"
        HL["Homelab Proxmox/Docker"]
        NAS["NAS"]
        PI["Raspberry Pi"]
    end
    subgraph "Tailnet"
        T["Tailscale Mesh"]
    end
    subgraph "Exterior"
        LAP["Laptop en cafetería"]
        MOB["Teléfono en 4G"]
    end
    HL --- T
    NAS --- T
    PI --- T
    LAP --- T
    MOB --- T
    style T fill:#3b82f6,stroke:#2563eb,color:#fff
    style HL fill:#166534,stroke:#22c55e,color:#fff
    style NAS fill:#166534,stroke:#22c55e,color:#fff
    style PI fill:#166534,stroke:#22c55e,color:#fff
    style LAP fill:#7c3aed,stroke:#8b5cf6,color:#fff
    style MOB fill:#7c3aed,stroke:#8b5cf6,color:#fff
```

<v-clicks>

- **Sin abrir puertos**: no se necesita port forwarding. NAT traversal se encarga de eso
- **Acceso seguro al NAS**: archivos, fotos y backups cifrados E2E desde cualquier lugar
- **AdBlocker en todos lados**: bloqueo de publicidad usando Pi-hole/Adguard Home a nivel de red, no de navegador

</v-clicks>

---

# Caso 2: Desarrollo Multi-Cloud

```mermaid {theme: 'dark', scale: 0.8}
graph LR
    subgraph AWS["AWS us-east-1"]
        A1["API Gateway"]
        A2["Lambda"]
    end
    subgraph GCP["GCP europe-west1"]
        G1["Cloud Run"]
        G2["Cloud SQL"]
    end
    subgraph Home["Local"]
        D["Dev Laptop"]
    end
    subgraph Azure["Azure"]
        AZ1["AKS Cluster"]
    end
    A1 <-->|"Tailscale"| G1
    G1 <-->|"Tailscale"| G2
    A2 <-->|"Tailscale"| AZ1
    D <-->|"Tailscale"| A1
    D <-->|"Tailscale"| G1
    D <-->|"Tailscale"| AZ1
    style A1 fill:#f59e0b,stroke:#d97706,color:#000
    style A2 fill:#f59e0b,stroke:#d97706,color:#000
    style G1 fill:#3b82f6,stroke:#2563eb,color:#fff
    style G2 fill:#3b82f6,stroke:#2563eb,color:#fff
    style AZ1 fill:#06b6d4,stroke:#0891b2,color:#fff
    style D fill:#22c55e,stroke:#16a34a,color:#fff
```

---

# Caso 3: IoT y Dispositivos Embebidos

```mermaid {theme: 'dark', scale: 0.9}
graph TB
    subgraph "Oficina Central"
        Dashboard["Dashboard Monitoreo"]
    end
    subgraph "Campo / Fábrica"
        RPi1["Raspberry Pi - Sensor Temp"]
        RPi2["Raspberry Pi - Cámara"]
        Arduino["ESP32 - Sensor Humedad"]
    end
    subgraph "Tailnet Segura"
        T["Red Mesh Tailscale"]
    end
    Dashboard --- T
    RPi1 --- T
    RPi2 --- T
    Arduino -.->|"via subnet router"| RPi1
    style T fill:#3b82f6,stroke:#2563eb,color:#fff
    style Dashboard fill:#7c3aed,stroke:#8b5cf6,color:#fff
    style RPi1 fill:#166534,stroke:#22c55e,color:#fff
    style RPi2 fill:#166534,stroke:#22c55e,color:#fff
    style Arduino fill:#f59e0b,stroke:#d97706,color:#000
```

---

# Recursos

- Presentación:
  - Web: https://tailscale-udi.pulgueta.com
  - GitHub: https://github.com/pulgueta/tailscale-lecture
- Tailscale:
  - Web: https://tailscale.com
  - GitHub: https://github.com/tailscale/tailscale
- Wireguard:
  - Web: https://www.wireguard.com
  - GitHub (mirrors): https://github.com/orgs/WireGuard/repositories
- https://www.paloaltonetworks.com/cyberpedia/what-is-a-vpn-concentrator
- https://tailscale.com/learn/understanding-mesh-vpns
- https://tailscale.com/blog/how-tailscale-works
