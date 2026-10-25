# 🌐 Enterprise Multi-Branch Network Design with High Availability and Comprehensive Security

> **Đồ án môn học:** Thiết kế mạng  
> **Sinh viên thực hiện:** Vạn Đức Công Minh — MSSV: `23162056`  
> **Trường:** Đại học Sư phạm Kỹ thuật TP.HCM (HCMUTE)  
> **Học kỳ:** Học kỳ 2 (2025 – 2026)  
> **Định hướng ứng dụng:** E-Commerce Enterprise Infrastructure / Network & Security Architecture

---

## 📌 Executive Summary (Tổng quan dự án)

Dự án tập trung nghiên cứu, hoạch định và thiết kế toàn diện hạ tầng mạng doanh nghiệp thương mại điện tử đa chi nhánh quy mô vừa, phục vụ **300 nhân sự nội bộ** (định hướng mở rộng đến **500 nhân sự**) và vận hành nền tảng E-Commerce phục vụ mục tiêu **1.000.000 khách hàng trực tuyến** trong vòng 3 năm.

Hệ thống được thiết kế theo các triết lý kỹ thuật chuẩn Enterprise:
- **High Availability (HA) & Fault Tolerance:** Cam kết SLA uptime **≥ 99.9%** (tối đa ≤ 8.76 giờ downtime/năm), loại bỏ hoàn toàn điểm lỗi đơn (**No Single Point of Failure - SPOF**) từ WAN Edge, Firewall, Core/Distribution Switch đến Server Farm và SAN Storage.
- **Defense-in-Depth (Phòng thủ đa tầng Multi-vendor):** 4 lớp Firewall chuyên biệt tại Trụ sở chính (Fortinet + Palo Alto Networks + FortiADC) kết hợp phân vùng an ninh (Security Zones) và Micro-segmentation.
- **Tiêu chuẩn tuân thủ quốc tế:** Đáp ứng các yêu cầu bảo mật khắt khe của **PCI-DSS** (bảo vệ môi trường dữ liệu thanh toán thẻ - CDE) và khuyến nghị quản lý an toàn thông tin **ISO 27001**.
- **Hiệu quả kinh tế & Khả năng mở rộng:** Tối ưu hóa chi phí đầu tư (TCO) bằng giải pháp cân bằng tải tích hợp WAF, ảo hóa VMware vSphere HA kết hợp bare-metal cho cụm Web DMZ.

---

## 🏗️ Quy mô và Mô hình Hoạt động

| Hạng mục | Trụ sở chính (HQ) | Chi nhánh 1 (BR1) | Chi nhánh 2 (BR2) | Tổng cộng |
|---|---|---|---|---|
| **Nhân sự** | 200 người (5 phòng ban) | 50 người (3 phòng ban) | 50 người (3 phòng ban) | **300 nhân sự** |
| **Phòng ban** | Ban Giám đốc, Kế toán - Tài chính, Kinh doanh - Marketing, Kỹ thuật - CNTT, Nhân sự - Hành chính | GĐ Chi nhánh, Sale, Chăm sóc khách hàng (CSKH) | GĐ Chi nhánh, Sale, Chăm sóc khách hàng (CSKH) | **11 đơn vị vận hành** |
| **Mô hình mạng** | 3 tầng phân cấp: Core – Distribution – Access | 2 tầng rút gọn: Collapsed Core/Dist – Access | 2 tầng rút gọn: Collapsed Core/Dist – Access | Phân cấp chuẩn Cisco |
| **Hạ tầng máy chủ** | Integrated Data Center (3 Bare-metal + 3 ESXi Host + SAN) | 1 Branch Server (Dell PowerEdge R350) | 1 Branch Server (Dell PowerEdge R350) | Hybrid Virtualization |
| **Đường truyền WAN** | Triple WAN (DIA 1Gbps BGP + FTTH 1Gbps + MPLS 100Mbps) | Dual WAN (FTTH 500Mbps + MPLS 50Mbps) | Dual WAN (FTTH 500Mbps + MPLS 50Mbps) | Hybrid SD-WAN Overlay |

---

## 🛡️ Điểm nhấn Kiến trúc Bảo mật 4 Lớp (Defense-in-Depth)

Tại Trụ sở chính (HQ), luồng lưu lượng truy cập từ Internet vào Data Center hoặc kết nối liên chi nhánh được kiểm soát qua kiến trúc **4 lớp tường lửa phân tầng chuyên sâu đa hãng (Multi-vendor Security)** nhằm triệt tiêu nguy cơ lỗ hổng Zero-day xuyên chuỗi:

```
                  [ Internet / Remote Users / Branches ]
                                   │
                                   ▼
┌──────────────────────────────────────────────────────────────────────┐
│  Lớp 0: Edge Anti-DDoS & Caching (Cloudflare CDN / Anti-DDoS)        │
└──────────────────────────────────┬───────────────────────────────────┘
                                   │ (DIA 1G / FTTH 1G / MPLS 100M)
                                   ▼
┌──────────────────────────────────────────────────────────────────────┐
│  Lớp 1: Edge Firewall HA (2× FortiGate 200F - Active/Standby)        │
│  - SD-WAN Hub thông minh điều phối 3 đường WAN                       │
│  - Stateful Inspection, NAT (Static/PAT), IPsec VPN & SSL-VPN Remote │
└──────────────────────────────────┬───────────────────────────────────┘
                                   │ (10G SFP+ Link)
                                   ▼
┌──────────────────────────────────────────────────────────────────────┐
│  Lớp 2: Advanced NGFW HA (2× Palo Alto PA-850 - Multi-vendor)        │
│  - Kiểm soát ứng dụng App-ID, IPS chuyên sâu                         │
│  - Cloud-based Sandbox (WildFire) dò quét & chặn Zero-day Malware    │
└──────────────┬───────────────────────────────────────┬───────────────┘
               │ (Traffic e-Commerce HTTPS)            │ (Internal LAN)
               ▼                                       ▼
┌────────────────────────────────────────┐ ┌───────────────────────────┐
│ Lớp 3: WAF & Load Balancer HA          │ │ Lớp 4: Internal FW HA     │
│ (2× FortiADC 400F - Reverse Proxy)     │ │ (2× FortiGate 90G HA)     │
│ - Chặn OWASP Top 10 (SQLi, XSS, CSRF)  │ │ - Micro-segmentation LAN  │
│ - L7 Load Balancing cho 3 Web R650     │ │ - Kiểm soát chặt truy cập │
│ - SSL Offloading (TLS 1.2/1.3)         │ │   vào Server Farm (DB/AD) │
└──────────────────┬─────────────────────┘ └─────────────┬─────────────┘
                   ▼                                     ▼
           [ Vùng Mạng DMZ ]                    [ Server Farm & LAN ]
       3× Web Server Magento 2               PostgreSQL Patroni Cluster,
      (Dell PowerEdge R650 Bare-metal)        AD-DS, ERP, CRM, SIEM, NMS
```

### Chi tiết các lớp bảo vệ:
1. **Lớp 0 — Anti-DDoS & CDN:** Cloudflare bảo vệ lớp ngoài cùng, hấp thụ các cuộc tấn công DDoS volumetric (SYN Flood, UDP Reflection, HTTP Flood), đồng thời cache nội dung tĩnh giảm tải cho Web Server.
2. **Lớp 1 — Perimeter Edge Firewall (FortiGate 200F HA Active/Standby):** Đóng vai trò SD-WAN Hub điều phối 3 luồng mạng WAN, quản lý kết nối Site-to-Site VPN với các chi nhánh, cấp phát SSL-VPN cho người dùng từ xa (kết hợp 2FA) và thực thi chính sách NAT/PAT.
3. **Lớp 2 — Advanced NGFW (Palo Alto Networks PA-850 HA):** Sử dụng thiết bị khác hãng (Palo Alto vs Fortinet) để chống bypass Zero-day. Thực hiện giải mã SSL Inbound, nhận diện ứng dụng tầng 7 (App-ID), quét lỗ hổng IPS và phân tích tệp tin độc hại qua đám mây Palo Alto WildFire.
4. **Lớp 3 — Web Application Firewall & ADC (FortiADC 400F HA):** Triển khai mô hình Reverse Proxy combo WAF + L7 Load Balancing giúp tối ưu thiết bị. Bảo vệ ứng dụng web theo chuẩn **PCI-DSS Requirement 6.6** (chống SQLi, XSS, Command Injection, Botnet), đồng thời giảm tải tính toán mã hóa thông qua SSL Offload.
5. **Lớp 4 — Internal Segmentation Firewall (FortiGate 90G HA):** Đặt tại ranh giới Core Switch và Data Center Switch, thực thi chính sách Zero-Trust cục bộ, ngăn chặn lây nhiễm mã độc ngang (Lateral Movement) giữa mạng người dùng văn phòng và CDE Database.

---

## ⚡ Thiết kế Hạ tầng WAN Độc lập (Triệt tiêu SPOF)

Hệ thống WAN tại Trụ sở chính được xây dựng với kiến trúc **Triple WAN - Triple Router**:
- **3 Router Cisco ISR 4331 riêng biệt**:
  - `Router 1`: Kết nối kênh truyền **DIA 1 Gbps** đối xứng, chạy BGP Peering (AS65001) với ISP, cam kết SLA chất lượng cao dành riêng cho dịch vụ E-Commerce.
  - `Router 2`: Kết nối kênh **FTTH Doanh nghiệp 1 Gbps** (Static default route) phục vụ nhu cầu lướt web, làm việc hàng ngày của khối văn phòng.
  - `Router 3`: Kết nối mạng riêng ảo **MPLS VPN 100 Mbps** (CE-PE BGP) kết nối trực tiếp đến các chi nhánh cho các dịch vụ nhạy cảm về độ trễ (VoIP, ERP/CRM, Database Replication).
- **Cơ chế Failover:** Cả 3 router chuyển tiếp lưu lượng qua các transit link độc lập tới FortiGate 200F. FortiGate SD-WAN liên tục giám sát chất lượng đường truyền (Jitter, Latency, Packet Loss) và tự động chuyển đổi đường truyền trong **< 1 giây** khi có sự cố ISP.

---

## 🏛️ Kiến trúc Phân tầng Mạng Nội bộ (Hierarchical Design)

### 1. Trụ sở chính (HQ - 3-Tier Enterprise):
- **Core Layer:** Cặp **Cisco Catalyst C9500-16X-A** chạy công nghệ **StackWise Virtual (SVL)**, liên kết bằng 2 đường 40G QSFP+, cung cấp băng thông chuyển mạch tốc độ cao, chạy định tuyến nội bộ **OSPF Area 0 Backbone**.
- **Distribution Layer:** 2 cụm Stack **Cisco Catalyst C9300L-24T-4X-E** (StackWise-320), đóng vai trò Default Gateway (SVI) cho toàn bộ các VLAN khối phòng ban, hỗ trợ DHCP Relay về Active Directory Server.
- **Access Layer:** 7 Switch **Cisco Catalyst C9200L-24P/48P** cấp nguồn PoE+ cho IP Phone và Access Point, áp dụng các tính năng bảo mật Port Security, DHCP Snooping, Dynamic ARP Inspection (DAI), 802.1X Network Access Control.
- **Data Center & DMZ Switching:** Sử dụng các cặp Cisco C9300L Stack riêng biệt cách ly vùng máy chủ nội bộ và vùng DMZ công cộng.

### 2. Chi nhánh (Branch Offices - Collapsed Core):
- Tối ưu không gian và chi phí với mô hình **Collapsed Core/Distribution**:
  - Cụm Stack 2x **Cisco C9300L-24T-4X-E** đóng vai trò Core/Dist gom mạch.
  - Các Switch Access **Cisco C9200L** kết nối Uplink 10G LACP về Stack.
- **Firewall WAN Edge:** Cặp **FortiGate 60F HA** tích hợp đa năng: Router biên, SD-WAN Spoke, VPN IPSec, DHCP Server có đồng bộ trạng thái cấp phát (HA Lease Sync), Firewall UTM (AV, IPS, Web Filtering).

---

## 💾 Hạ tầng Ảo hóa & Dịch vụ Data Center (Mô hình 3-3-3)

Hệ thống Data Center tại Trụ sở chính được thiết kế theo cấu trúc module dự phòng cao:

```
[ FortiADC 400F HA ] ─── DMZ Switch (C9300L Stack)
                               │
       ┌───────────────────────┼───────────────────────┐
       ▼                       ▼                       ▼
  Web Server 01           Web Server 02           Web Server 03
(Dell R650 Bare-metal)  (Dell R650 Bare-metal)  (Dell R650 Bare-metal)
[Ubuntu 22.04/Magento]  [Ubuntu 22.04/Magento]  [Ubuntu 22.04/Magento]
```
```
[ FortiGate 90G HA ] ─── DC Switch (C9300L Stack) ─── [ SAN ME5024 Dual Ctrl ]
                               │                               ▲
       ┌───────────────────────┼───────────────────────┐       │ (Dual 10G iSCSI)
       ▼                       ▼                       ▼       │
   ESXi Host 01            ESXi Host 02            ESXi Host 03┘
(Dell PowerEdge R750xs) (Dell PowerEdge R750xs) (Dell PowerEdge R750xs)
 19 Máy ảo trên VMware vSphere HA Cluster (Anti-affinity Rules)
 ├── DB-01 (Patroni Master) ── DB-02 (Read Replica 1) ── DB-03 (Read Replica 2)
 ├── APP-01 (Odoo/SAP ERP)  ── APP-02 (ERP/CRM Cluster)── APP-03 (CRM Service)
 └── Infrastructure VMs: AD-DC01, AD-DC02, RADIUS NPS, SIEM, Zabbix, Veeam Backup...
```

### Điểm nổi bật:
- **Tối ưu Web DMZ:** 3 máy chủ **Dell PowerEdge R650 chạy Bare-metal** (không chạy qua ảo hóa) đặt trong DMZ để tối đa hóa hiệu năng I/O cho web e-commerce và tiết kiệm chi phí bản quyền VMware.
- **Cụm Ảo hóa Doanh nghiệp (vSphere HA):** 3 máy chủ **Dell PowerEdge R750xs** (Dual Intel Xeon Gold 5317, 128GB RAM) kết nối hệ thống lưu trữ **SAN Dell PowerVault ME5024** qua 2 đường quang iSCSI 10Gbps độc lập (MPIO). Cụm cluster đảm bảo tự động phục hồi máy ảo (Auto-restart) khi có sự cố phần cứng.
- **Kiến trúc Cơ sở dữ liệu Chống chịu lỗi cao (Database HA Quorum):** Cụm 3 máy ảo PostgreSQL chạy cơ chế **Patroni + etcd** phân tán trên 3 host ESXi vật lý khác nhau (Anti-affinity rules):
  - `DB-01`: Master đảm nhận tác vụ Read/Write.
  - `DB-02 & DB-03`: 2 Read Replicas đồng bộ dữ liệu liên tục (Streaming Replication), vừa chia sẻ tải đọc khổng lồ cho 1 triệu khách hàng trực tuyến, vừa đảm bảo tính sẵn sàng biểu quyết quorum tự động bầu chọn Master mới trong vòng **< 10 giây**.
- **Tính tự lập tại Chi nhánh (Branch Autonomy):** Mỗi chi nhánh được trang bị 1 máy chủ **Dell PowerEdge R350** chạy **Read-Only Domain Controller (RODC)** và DNS Cache. Khi sự cố đứt kết nối WAN xảy ra, nhân viên chi nhánh vẫn đăng nhập máy tính và xác thực nội bộ bình thường.

---

## 📊 Quy hoạch Địa chỉ IP & Phân vùng VLAN

### 1. Quy ước dải địa chỉ:
- **Mạng văn phòng (Office Access):** `192.168.x.0/24` (HQ: `192.168.10-99.x`, Chi nhánh 1: `192.168.110-199.x`, Chi nhánh 2: `192.168.210-249.x`).
- **Liên kết hạ tầng Point-to-Point (Core/Dist/WAN Links):** `10.10.x.x/30`.
- **Khu vực Trung tâm Dữ liệu (Server Farm & DMZ):** `172.16.x.0/24`.
- **VPN Remote Access Pool:** `172.16.40.0/23`.

### 2. Danh mục VLAN tiêu biểu tại Trụ sở chính:

| VLAN ID | Tên VLAN | Subnet | Gateway (SVI) | Mục đích / Chức năng |
|:---:|:---|:---|:---|:---|
| **10** | `HQ_BGD` | `192.168.10.0/24` | `192.168.10.1` | Ban Giám đốc |
| **20** | `HQ_KTTC` | `192.168.20.0/24` | `192.168.20.1` | Phòng Kế toán – Tài chính |
| **30** | `HQ_KDMK` | `192.168.30.0/24` | `192.168.30.1` | Phòng Kinh doanh – Marketing |
| **40** | `HQ_KTIT` | `192.168.40.0/24` | `192.168.40.1` | Phòng Kỹ thuật – CNTT |
| **50** | `HQ_NSHC` | `192.168.50.0/24` | `192.168.50.1` | Phòng Nhân sự – Hành chính |
| **60** | `HQ_VOIP` | `192.168.60.0/24` | `192.168.60.1` | Thoại IP (~70 Deskphone 3CX + Softphone) |
| **70** | `HQ_CAM` | `192.168.70.0/24` | `192.168.70.1` | Hệ thống IP Camera giám sát an ninh |
| **80** | `HQ_GUEST` | `192.168.80.0/24` | `192.168.80.1` | Mạng Wi-Fi Khách (cách ly hoàn toàn) |
| **99** | `HQ_MGMT` | `192.168.99.0/24` | `192.168.99.1` | Quản trị thiết bị mạng (OOB / In-band an toàn) |
| — | `DMZ_WEB` | `172.16.10.0/24` | `172.16.10.1` | Cụm máy chủ Web E-Commerce |
| — | `SRV_APP` | `172.16.20.0/24` | `172.16.20.1` | Cụm máy chủ ứng dụng ERP, CRM, AD-DS |
| — | `SRV_DB` | `172.16.21.0/24` | `172.16.21.1` | Cụm Cơ sở dữ liệu (PCI-DSS CDE) |

### 3. Phân vùng Định tuyến OSPF Đa Vùng (Multi-Area OSPF):
- **Area 0 (Backbone):** Kết nối liên vùng giữa Core Switch C9500, Edge Firewall FortiGate 200F, Internal Firewall FortiGate 90G và 3 WAN Routers.
- **Area 1:** Vùng phân phối và người dùng Trụ sở chính (Distribution & Access Switches).
- **Area 2:** Phân vùng Data Center nội bộ.
- **Area 10 & 20:** Vùng định tuyến nội bộ tại Chi nhánh 1 và Chi nhánh 2.

---

## 📶 Giải pháp Không dây Doanh nghiệp (Enterprise Wireless)

- **Hệ thống Wi-Fi 6:** Triển khai **18 Access Point Aruba AP-535** (10 AP tại HQ, 4 AP tại mỗi Chi nhánh), cấp nguồn trực tiếp qua cáp PoE+ từ Access Switch.
- **Quản lý tập trung qua WAN:** Toàn bộ AP được quản lý thống nhất bởi **Aruba Mobility Controller 7010** đặt tại HQ thông qua đường hầm mã hóa bảo mật.
- **Phân tách sóng SSID chuyên biệt:**
  - `CORP-WIFI`: Xác thực cấp doanh nghiệp **WPA3-Enterprise / 802.1X (EAP-TLS)** qua máy chủ RADIUS (Windows Server NPS), tự động gán VLAN tương ứng với phòng ban của nhân viên.
  - `GUEST-WIFI`: Cổng Captive Portal xác thực khách, cô lập mạng (Client Isolation), chỉ cho phép đi Internet qua cổng HTTP/HTTPS, chặn toàn bộ kết nối vào mạng nội bộ.

---

## 📋 Danh mục Thiết bị & Dự toán Đầu tư (BOM & Budget)

### 1. Bảng thiết bị chính (Bill of Materials):

| STT | Phân loại | Thiết bị / Model | SL | Vai trò kiến trúc |
|:---:|---|---|:---:|---|
| 1 | **WAN Router** | Cisco ISR 4331/K9 | 3 | Terminate 3 đường WAN độc lập (DIA, FTTH, MPLS) |
| 2 | **Edge Firewall** | FortiGate 200F (HA Pair) | 2 | Perimeter Defense, SD-WAN Hub, IPSec VPN Site-to-Site |
| 3 | **NGFW** | Palo Alto PA-850 (HA Pair) | 2 | Deep Packet Inspection, IPS, App-ID, WildFire Sandbox |
| 4 | **WAF / LB** | FortiADC 400F (HA Pair) | 2 | L7 Web Application Firewall, Reverse Proxy, SSL Offloading |
| 5 | **Internal Firewall**| FortiGate 90G (HA Pair) | 2 | Micro-segmentation Server Farm & LAN Management |
| 6 | **Branch Firewall**  | FortiGate 60F (HA Pair ×2) | 4 | WAN Edge, SD-WAN Spoke, UTM, DHCP HA Sync tại 2 chi nhánh |
| 7 | **Core Switch** | Cisco Catalyst C9500-16X-A | 2 | StackWise Virtual (SVL), L3 Routing OSPF Area 0 |
| 8 | **Dist / DC Switch**| Cisco Catalyst C9300L-24T-4X-E | 12 | StackWise-320 cho Dist (4 con), DC (2 con), DMZ (2 con), CN (4 con) |
| 9 | **Access Switch** | Cisco Catalyst C9200L-24P/48P | 13 | 7 Switch tại HQ + 6 Switch tại 2 Chi nhánh (PoE+, 802.1X) |
| 10 | **Wireless AP** | Aruba AP-535 Wi-Fi 6 | 18 | 10 AP tại HQ + 8 AP tại 2 Chi nhánh |
| 11 | **WLC** | Aruba Mobility Controller 7010 | 1 | Quản lý tập trung 18 AP toàn hệ thống |
| 12 | **Web Server** | Dell PowerEdge R650 | 3 | Bare-metal Ubuntu Server chạy Magento 2 trong DMZ |
| 13 | **Virtualization** | Dell PowerEdge R750xs | 3 | VMware vSphere HA 3-node cluster chứa 19 VMs |
| 14 | **SAN Storage** | Dell PowerVault ME5024 | 1 | Dual Controller Active/Active, 12×1.92TB SAS SSD (~11.5TB usable) |
| 15 | **Branch Server** | Dell PowerEdge R350 | 2 | Read-Only Domain Controller (RODC) + File/Print tại 2 Chi nhánh |
| 16 | **Nguồn & Tủ Rack**| APC Smart-UPS + Máy phát 30kVA | 8/1 | Lưu điện Online + Máy phát điện tự động chuyển nguồn ATS |

### 2. Tổng hợp Dự toán Dự án:

| Hạng mục đầu tư | Thành tiền (VNĐ) | Tỷ trọng |
|---|---:|:---:|
| **1. Thiết bị phần cứng toàn hệ thống** (HQ & 2 Chi nhánh) | 7.919.162.000 | 64.7% |
| **2. Dịch vụ đường truyền & Bản quyền phần mềm (Năm đầu)** | 2.290.449.000 | 18.7% |
| **3. Chi phí triển khai, lắp đặt cáp, đo kiểm & đào tạo** | 921.445.000 | 7.5% |
| **4. Chi phí dự phòng phát sinh (10%)** | 1.113.106.000 | 9.1% |
| **TỔNG MỨC ĐẦU TƯ TOÀN HỆ THỐNG** | **12.244.162.000 VNĐ** | **100%** |

*(Tương đương khoảng ~490.000 USD tại thời điểm thiết kế).*

---

## 👨‍💻 Tác giả & Thông tin Liên hệ

- **Tác giả:** Vạn Đức Công Minh
- **Mã số sinh viên:** `23162056`
- **Chuyên ngành:** Kỹ thuật Mạng & An toàn Thông tin / Công nghệ Thông tin
- **Trường:** Trường Đại học Sư phạm Kỹ thuật TP. Hồ Chí Minh (HCMUTE)
- **Email:** `23162056@student.hcmute.edu.vn` (hoặc liên hệ qua GitHub Profile)

---
*Dự án được thực hiện với tinh thần thiết kế chuyên nghiệp, bài bản, tiệm cận thực tế doanh nghiệp và sẵn sàng cho môi trường triển khai thực địa.*
