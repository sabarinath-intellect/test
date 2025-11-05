# Container Security & Zero Trust - Visual Diagrams

## 1. Overall Security Pipeline Architecture

```mermaid
graph LR
    A[Dev Repo] --> B[Jenkins CI/CD]
    B --> C{Security Gates}
    C -->|Scan| D[Trivy/Grype]
    C -->|Sign| E[Cosign]
    C -->|SBOM| F[Syft]
    D --> G{Pass?}
    E --> G
    F --> G
    G -->|Yes| H[DockerHub/ECR]
    G -->|No| I[❌ Block]
    H --> J[Kubernetes/Lambda]
    J --> K[Customer Environment]
    
    style A fill:#e1f5ff
    style B fill:#fff4e1
    style C fill:#ffe1e1
    style H fill:#e1ffe1
    style J fill:#f0e1ff
    style K fill:#ffe1f5
    style I fill:#ff0000,color:#fff
```

## 2. Zero Trust Security Layers

```mermaid
graph TB
    subgraph "Build Time"
        A1[Source Code] --> A2[Vulnerability Scan]
        A2 --> A3[Image Build]
        A3 --> A4[Sign with Cosign]
        A4 --> A5[Generate SBOM]
        A5 --> A6[in-toto Attestation]
    end
    
    subgraph "Registry"
        B1[ECR/DockerHub]
        B2[Image Scanning]
        B3[Lifecycle Policies]
        B4[Access Control]
    end
    
    subgraph "Deployment"
        C1[Signature Verification]
        C2[OPA Gatekeeper]
        C3[Admission Webhooks]
        C4[SBOM Check]
    end
    
    subgraph "Runtime"
        D1[Continuous Verification]
        D2[Network Policies]
        D3[Pod Security]
        D4[Falco Monitoring]
    end
    
    A6 --> B1
    B1 --> B2
    B2 --> B3
    B3 --> B4
    B4 --> C1
    C1 --> C2
    C2 --> C3
    C3 --> C4
    C4 --> D1
    D1 --> D2
    D2 --> D3
    D3 --> D4
    
    style A1 fill:#e3f2fd
    style A6 fill:#c8e6c9
    style B4 fill:#fff9c4
    style C4 fill:#f8bbd0
    style D4 fill:#d1c4e9
```

## 3. Implementation Roadmap Timeline

```mermaid
gantt
    title Container Security Implementation Roadmap (12 Weeks)
    dateFormat YYYY-MM-DD
    section Phase 1: Foundation
    Generate signing keys           :2025-01-01, 3d
    Implement Cosign signing        :2025-01-04, 4d
    Integrate Trivy scanning        :2025-01-04, 5d
    ECR lifecycle policies          :2025-01-08, 3d
    ECR scanning on push            :2025-01-10, 2d
    
    section Phase 2: Supply Chain
    Integrate Syft SBOM             :2025-01-15, 5d
    Attach SBOMs to images          :2025-01-20, 3d
    Implement in-toto               :2025-01-22, 5d
    SLSA provenance                 :2025-01-24, 3d
    
    section Phase 3: Deployment
    Sigstore Policy Controller      :2025-01-29, 4d
    ClusterImagePolicy config       :2025-02-02, 3d
    OPA Gatekeeper policies         :2025-02-05, 4d
    Admission webhooks              :2025-02-08, 3d
    Lambda verification script      :2025-02-10, 2d
    
    section Phase 4: Zero Trust Runtime
    Continuous verification         :2025-02-12, 5d
    Pod Security Standards          :2025-02-17, 4d
    Runtime init containers         :2025-02-20, 3d
    
    section Phase 5: Network & Identity
    Default-deny policies           :2025-02-24, 5d
    Service mesh mTLS               :2025-03-01, 5d
    SPIFFE/SPIRE                    :2025-03-04, 4d
    External Secrets                :2025-03-07, 3d
    
    section Phase 6: Monitoring
    Prometheus alerts               :2025-03-10, 4d
    Falco integration               :2025-03-13, 3d
    Security response operator      :2025-03-15, 4d
    Compliance dashboards           :2025-03-18, 3d
```

## 4. Security Decision Flow

```mermaid
flowchart TD
    Start([Container Build Started]) --> Scan[Vulnerability Scan]
    Scan --> ScanCheck{Critical/High<br/>Vulnerabilities?}
    ScanCheck -->|Yes| Block1[❌ Block Build]
    ScanCheck -->|No| Sign[Sign with Cosign]
    
    Sign --> SBOM[Generate SBOM]
    SBOM --> Attest[Create in-toto Attestation]
    Attest --> Push[Push to Registry]
    
    Push --> Deploy[Deployment Request]
    Deploy --> VerifySig{Signature<br/>Valid?}
    VerifySig -->|No| Block2[❌ Block Deployment]
    VerifySig -->|Yes| VerifySBOM{SBOM<br/>Present?}
    
    VerifySBOM -->|No| Block3[❌ Block Deployment]
    VerifySBOM -->|Yes| Policy{OPA Policy<br/>Pass?}
    
    Policy -->|No| Block4[❌ Block Deployment]
    Policy -->|Yes| Runtime[Deploy to Runtime]
    
    Runtime --> Monitor[Continuous Monitoring]
    Monitor --> Anomaly{Anomaly<br/>Detected?}
    Anomaly -->|Yes| Alert[🚨 Alert & Remediate]
    Anomaly -->|No| Continue[✅ Continue Running]
    
    Alert --> Quarantine[Quarantine Pod]
    Continue --> Monitor
    
    style Start fill:#4caf50,color:#fff
    style Block1 fill:#f44336,color:#fff
    style Block2 fill:#f44336,color:#fff
    style Block3 fill:#f44336,color:#fff
    style Block4 fill:#f44336,color:#fff
    style Continue fill:#4caf50,color:#fff
    style Alert fill:#ff9800,color:#fff
```

## 5. Zero Trust Principles Matrix

```mermaid
graph TB
    subgraph "Zero Trust Principles"
        ZT1[Never Trust<br/>Always Verify]
        ZT2[Least Privilege<br/>Access]
        ZT3[Assume<br/>Breach]
        ZT4[Verify<br/>Explicitly]
        ZT5[Secure by<br/>Default]
    end
    
    subgraph "Implementation"
        I1[Image Signing<br/>& Verification]
        I2[RBAC & Network<br/>Policies]
        I3[Continuous<br/>Monitoring]
        I4[Multi-factor<br/>Authentication]
        I5[Default Deny<br/>All]
    end
    
    subgraph "Tools"
        T1[Cosign<br/>Sigstore]
        T2[OPA<br/>Gatekeeper]
        T3[Falco<br/>Prometheus]
        T4[SPIFFE<br/>SPIRE]
        T5[Network<br/>Policies]
    end
    
    ZT1 --> I1 --> T1
    ZT2 --> I2 --> T2
    ZT3 --> I3 --> T3
    ZT4 --> I4 --> T4
    ZT5 --> I5 --> T5
    
    style ZT1 fill:#e3f2fd
    style ZT2 fill:#f3e5f5
    style ZT3 fill:#fff3e0
    style ZT4 fill:#e8f5e9
    style ZT5 fill:#fce4ec
```

## 6. Supply Chain Security Flow

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Git as Git Repo
    participant CI as Jenkins CI
    participant Scan as Trivy Scanner
    participant Sign as Cosign
    participant SBOM as Syft SBOM
    participant Attest as in-toto
    participant Reg as ECR Registry
    participant K8s as Kubernetes
    
    Dev->>Git: Push Code
    Git->>CI: Trigger Build
    CI->>CI: Build Image
    CI->>Scan: Scan Image
    Scan-->>CI: Vulnerability Report
    
    alt Critical Vulnerabilities
        CI->>Dev: ❌ Build Failed
    else Pass
        CI->>Sign: Sign Image
        Sign-->>CI: Signature Created
        CI->>SBOM: Generate SBOM
        SBOM-->>CI: SBOM Created
        CI->>Attest: Create Attestation
        Attest-->>CI: Attestation Signed
        CI->>Reg: Push Image + Artifacts
        Reg->>K8s: Pull Request
        K8s->>Sign: Verify Signature
        Sign-->>K8s: ✅ Verified
        K8s->>SBOM: Verify SBOM
        SBOM-->>K8s: ✅ Verified
        K8s->>K8s: Deploy Container
    end
```

## 7. Runtime Security Architecture

```mermaid
graph TB
    subgraph "Application Pod"
        A1[App Container]
        A2[Sidecar Verifier]
        A3[Init Container]
    end
    
    subgraph "Security Controls"
        B1[Pod Security<br/>Standards]
        B2[Network<br/>Policies]
        B3[RBAC]
        B4[Secrets<br/>Management]
    end
    
    subgraph "Monitoring"
        C1[Falco<br/>Runtime Monitor]
        C2[Prometheus<br/>Alerts]
        C3[Tetragon<br/>eBPF]
    end
    
    subgraph "Response"
        D1[Alert<br/>Notification]
        D2[Quarantine<br/>Pod]
        D3[Auto<br/>Remediation]
    end
    
    A1 --> B1
    A2 --> B2
    A3 --> B3
    B4 --> A1
    
    B1 --> C1
    B2 --> C2
    B3 --> C3
    
    C1 --> D1
    C2 --> D2
    C3 --> D3
    
    style A1 fill:#bbdefb
    style C1 fill:#ffccbc
    style D1 fill:#f8bbd0
```

## 8. Security Metrics Dashboard

```mermaid
graph LR
    subgraph "Security Metrics"
        M1[Image Signing Rate<br/>Target: 100%]
        M2[SBOM Coverage<br/>Target: 100%]
        M3[Vuln Scan Pass<br/>Target: >95%]
        M4[Policy Compliance<br/>Target: >98%]
    end
    
    subgraph "Performance Metrics"
        P1[MTTD<br/>Target: <5 min]
        P2[MTTR<br/>Target: <15 min]
        P3[Critical Vulns<br/>Target: 0]
        P4[Runtime Anomalies<br/>Track]
    end
    
    subgraph "Dashboards"
        D1[Security Overview]
        D2[Supply Chain]
        D3[Runtime Security]
    end
    
    M1 --> D1
    M2 --> D2
    M3 --> D1
    M4 --> D3
    P1 --> D1
    P2 --> D3
    P3 --> D1
    P4 --> D3
    
    style M1 fill:#c8e6c9
    style M2 fill:#c8e6c9
    style M3 fill:#fff9c4
    style M4 fill:#c8e6c9
    style P1 fill:#b3e5fc
    style P2 fill:#b3e5fc
```

## 9. Deployment Verification Process

```mermaid
stateDiagram-v2
    [*] --> ImageBuilt
    ImageBuilt --> ScanningImage
    ScanningImage --> VulnCheck
    
    VulnCheck --> SigningImage: Pass
    VulnCheck --> BuildFailed: Critical/High Found
    
    SigningImage --> GenerateSBOM
    GenerateSBOM --> CreateAttestation
    CreateAttestation --> PushRegistry
    
    PushRegistry --> DeployRequest
    DeployRequest --> VerifySignature
    
    VerifySignature --> VerifySBOM: Valid
    VerifySignature --> DeployBlocked: Invalid
    
    VerifySBOM --> CheckPolicy: Valid
    VerifySBOM --> DeployBlocked: Invalid
    
    CheckPolicy --> RuntimeDeploy: Pass
    CheckPolicy --> DeployBlocked: Fail
    
    RuntimeDeploy --> ContinuousMonitor
    ContinuousMonitor --> Running: No Anomaly
    ContinuousMonitor --> AlertResponse: Anomaly Detected
    
    AlertResponse --> Quarantine
    Running --> ContinuousMonitor
    
    BuildFailed --> [*]
    DeployBlocked --> [*]
    Quarantine --> [*]
```

## 10. Network Security Layers

```mermaid
graph TB
    subgraph "External Access"
        A[Internet]
    end
    
    subgraph "Ingress Layer"
        B1[Load Balancer]
        B2[WAF]
        B3[TLS Termination]
    end
    
    subgraph "Service Mesh"
        C1[Istio/Linkerd]
        C2[mTLS]
        C3[Service Auth]
    end
    
    subgraph "Network Policies"
        D1[Default Deny All]
        D2[Allow Frontend→Backend]
        D3[Allow DNS Only]
        D4[L7 Filtering]
    end
    
    subgraph "Application Pods"
        E1[Frontend Pod]
        E2[Backend Pod]
        E3[Database Pod]
    end
    
    A --> B1
    B1 --> B2
    B2 --> B3
    B3 --> C1
    C1 --> C2
    C2 --> C3
    C3 --> D1
    D1 --> D2
    D2 --> D3
    D3 --> D4
    D4 --> E1
    E1 -.->|Allowed| E2
    E2 -.->|Allowed| E3
    E1 -.->|Blocked| E3
    
    style D1 fill:#ffcdd2
    style C2 fill:#c8e6c9
    style E1 fill:#bbdefb
    style E2 fill:#bbdefb
    style E3 fill:#bbdefb
```

---

## How to Use These Diagrams

### Viewing Diagrams:
1. **GitHub/GitLab**: These Mermaid diagrams render automatically
2. **VS Code**: Install "Markdown Preview Mermaid Support" extension
3. **Online**: Use [Mermaid Live Editor](https://mermaid.live/)
4. **Documentation Tools**: Most modern tools support Mermaid (Confluence, Notion, etc.)

### Exporting Diagrams:
```bash
# Install mermaid-cli
npm install -g @mermaid-js/mermaid-cli

# Export to PNG
mmdc -i SECURITY-DIAGRAMS.md -o diagram1.png

# Export to SVG
mmdc -i SECURITY-DIAGRAMS.md -o diagram1.svg

# Export to PDF
mmdc -i SECURITY-DIAGRAMS.md -o diagram1.pdf
```

### Integration with Documentation:
- Copy individual diagrams into presentation slides
- Embed in Confluence pages
- Include in architecture documents
- Use in training materials

