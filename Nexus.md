**Nexus DashCam N1: an 18.0 cm (within 17.5–19 cm) forward-facing connected camera for Uber/Careem/inDrive vehicles in Egypt.**  
Architecture: **ESP32-S3‑WROOM‑1 (N16R8) + OV5640 (5 MP JPEG) + Quectel EC200U LTE Cat‑1/GNSS + BMI270 IMU + 64 GB industrial microSD + 12/24 V→5 V buck + 5.5 V supercapacitor backup.**  
Target BOM at 1 k units: **$28–32 (≈ EGP 1,350–1,550 at 48 EGP/USD)**; target retail **EGP 4,999–5,499** with optional cloud subscription **EGP 99/month**.  
Mandatory compliance: **NTRA type approval** for the LTE/Wi‑Fi/BT radio, **PDPL visual‑surveillance licence** (EGP 1,000 / 3 years for public‑place CCTV), consent + signage + 30‑day retention, and no cross‑border transfer without PDPC approval.  
Assumption: 48 EGP/USD; 1 k production run; Egypt‑focused first, US only after FCC/CE and state‑level two‑party consent review.

---

## 1. Product definition and mechanical envelope

| Parameter | Specification | Rationale |
|---|---|---|
| **Overall length** | **18.0 cm** | Inside 17.5–19 cm requirement |
| Width | 4.6 cm | Fits behind mirror / windshield top |
| Thickness | 3.2 cm | Clearance for lens + PCB + supercap |
| Housing material | PA12 SLS or PETG FDM, IP54 | Egypt summer cabin temps; 3D‑print pilot |
| Mount | 3M VHB + ball joint | No suction‑cup failure in heat |
| Lens | 6‑glass, f/1.8, 140° diagonal | Plate capture at 8–10 m |
| Operating temp | −10 °C to +70 °C | Cabin dashboard extremes |
| Input power | 9–30 V DC (12/24 V vehicle) | Covers passenger cars and light commercial |

**PCB stack:** 4‑layer, 1.6 mm FR4; main board 75 × 45 mm; camera daughter board 32 × 32 mm; LTE/GNSS module on 30 × 30 mm keep‑out.

---

## 2. System architecture

```
12/24 V ──► Buck (5 V/3 A) ──► Supercap 5.5 V/1 F ──► ESP32-S3
                                                     │
OV5640 (DVP 8-bit JPEG) ◄── DVP/LCD_CAM ────────────┤
BMI270 (I²C) ◄──────────────────────────────────────┤
microSD (SDIO 4-bit) ◄──────────────────────────────┤
EC200U (UART + USB) ◄── LTE Cat‑1 + GNSS ──────────┤
Wi‑Fi/BT ──► ESP32-S3 internal radio ───────────────┘
```

**Why ESP32‑S3, not STM32:** STM32F4 has DCMI but no integrated Wi‑Fi/BT; adding ESP32 or LTE module increases BOM and board area. ESP32‑S3 has DVP camera interface, Wi‑Fi/BT, sufficient RAM for JPEG double‑buffering, and mature open‑source MJPEG recording stacks. For a pure control‑only board STM32 is better; for a connected camera ESP32‑S3 is the optimal single‑chip choice. Alternative: STM32H7 + ESP32‑C6 co‑processor for dual‑chip redundancy, +$6 BOM, −30 % firmware complexity risk.

---

## 3. Bill of materials (1 k units, USD)

| Item | Part | Qty | Unit $ | Ext $ | Notes |
|---|---|---|---|---|---|
| MCU module | ESP32‑S3‑WROOM‑1‑N16R8 | 1 | 4.20 | 4.20 | 16 MB flash, 8 MB PSRAM |
| Camera | OV5640 DVP module | 1 | 5.50 | 5.50 | 5 MP, JPEG output, 140° lens |
| LTE/GNSS | Quectel EC200U‑CN/EU | 1 | 9.80 | 9.80 | Cat‑1, GNSS, Egypt bands |
| IMU | Bosch BMI270 | 1 | 1.60 | 1.60 | 6‑axis, low‑power |
| Storage | 64 GB industrial microSD | 1 | 3.80 | 3.80 | High‑endurance |
| Power | MP1584 buck + supercap + TVS | 1 set | 2.20 | 2.20 | 5 V/3 A, 5.5 V/1 F |
| PCB + assembly | 4‑layer, SMT | 1 | 4.50 | 4.50 | 1 k volume |
| Case + mount | PA12 SLS / PETG | 1 | 2.80 | 2.80 | 3D‑printed pilot; injection later |
| Harness + connectors | JST‑GH, automotive | 1 set | 1.50 | 1.50 | |
| **Total** | | | | **35.90** | Negotiated volume ≈ **$28–32** |

**Egypt landed cost additions:** camera import duty 5 % + 14 % VAT; electronics accessories 20 % duty in some categories; NTRA type‑approval test fees; PDPL licence EGP 1,000/3 years. 3D‑printing pilot cost in Egypt: **3–6 EGP/gram** for PLA/PETG, SLS higher; a 120 g case ≈ **EGP 360–720** at pilot volumes.

---

## 4. Mathematical validation: power, thermal, storage, bandwidth

### 4.1 Power budget

| Rail | Load | Current | Power |
|---|---|---|---|
| 5 V | ESP32‑S3 (active, Wi‑Fi off) | 180 mA | 0.90 W |
| 5 V | OV5640 (1080p30 JPEG) | 120 mA | 0.60 W |
| 5 V | EC200U (LTE idle + GNSS) | 250 mA avg | 1.25 W |
| 5 V | microSD write | 80 mA | 0.40 W |
| 5 V | IMU + misc | 10 mA | 0.05 W |
| **Total** | | **640 mA** | **3.20 W** |

Buck efficiency 88 % → input power ≈ **3.64 W** → at 12 V, **303 mA**; at 24 V, **152 mA**. Supercap 5.5 V/1 F holds enough energy to flush the current file: \(E = \frac{1}{2} C V^2 = 0.5 × 1 × 5.5^2 ≈ 15.1 J\); at 3.2 W write load, hold‑up ≈ **4.7 s**, enough to close a 60 s segment metadata write.

### 4.2 Thermal

Cabin dashboard can reach **70 °C**. Junction temperature of ESP32‑S3 at 3.2 W with θJA ≈ 40 °C/W in still air: \(T_j = T_a + P × θ_{JA} = 70 + 3.2 × 40 = 198 °C\) → **unacceptable** without thermal path. Required: aluminium heat‑spreader (20 × 20 × 2 mm) + vented case; target θJA ≤ 15 °C/W → \(T_j ≈ 118 °C\), still marginal. **Mitigation:** duty‑cycle LTE (upload only on event), reduce camera to 720p when parked, and specify automotive‑grade components rated 85 °C. **Do not claim continuous 4K in Egyptian summer without active cooling.**

### 4.3 Storage

1080p30 MJPEG at ~8 Mbps = **1 MB/s**; 60 s segment = **60 MB**. Loop recording with 30‑day retention at 2 h driving/day: \(60 MB/min × 120 min/day × 30 = 216 GB\) → **64 GB card holds ≈ 8.9 days** at 2 h/day. To meet 30 days, either **720p (4 Mbps → 32 GB for 30 days at 2 h/day)** or **event‑triggered recording + 7‑day rolling buffer**. Recommendation: 720p default, 1080p event clips retained 30 days, 64 GB card.

### 4.4 Upload bandwidth

Event clip 30 s at 720p ≈ 15 MB. LTE Cat‑1 uplink in Egypt typically **5–10 Mbps** → 15 MB upload ≈ **12–24 s**. Acceptable for post‑incident upload; not for continuous live streaming. Continuous live view would require Cat‑4 and 50+ MB/min data cost — **reject for unit economics**.

---

## 5. Firmware skeleton (ESP‑IDF, C)

```c
// main.c — Nexus N1 core firmware skeleton
#include "esp_camera.h"
#include "esp_http_client.h"
#include "driver/i2c.h"
#include "sdmmc_cmd.h"
#include "driver/sdmmc_host.h"
#include "bmi270.h"          // abstraction
#include "ec200u.h"          // UART AT driver
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

#define RECORD_RES   FRAMESIZE_HD   // 720p default; FRAMESIZE_FHD on event
#define SEGMENT_SEC  60
#define EVENT_G      2.5f           // g threshold

static camera_config_t cam_cfg = {
    .pin_pwdn = -1, .pin_reset = -1,
    .pin_xclk = 15, .pin_sccb_sda = 4, .pin_sccb_scl = 5,
    .pin_d7 = 16, .pin_d6 = 17, .pin_d5 = 18, .pin_d4 = 12,
    .pin_d3 = 10, .pin_d2 = 8,  .pin_d1 = 9,  .pin_d0 = 11,
    .pin_vsync = 6, .pin_href = 7, .pin_pclk = 13,
    .xclk_freq_hz = 20000000,
    .pixel_format = PIXFORMAT_JPEG,
    .frame_size = RECORD_RES,
    .jpeg_quality = 12,
    .fb_count = 2,
    .fb_location = CAMERA_FB_IN_PSRAM,
    .grab_mode = CAMERA_GRAB_WHEN_EMPTY,
};

void app_main(void) {
    ESP_ERROR_CHECK(esp_camera_init(&cam_cfg));
    sdmmc_card_t *card;
    sdmmc_host_t host = SDMMC_HOST_DEFAULT();
    sdmmc_slot_config_t slot = SDMMC_SLOT_CONFIG_DEFAULT();
    ESP_ERROR_CHECK(sdmmc_host_init_slot(SDMMC_HOST_SLOT_1, &slot));
    ESP_ERROR_CHECK(sdmmc_card_init(&host, &card));

    i2c_master_init();                 // BMI270
    bmi270_init();
    ec200u_init(UART_NUM_1, 115200);   // AT + QGPS=1
    ec200u_connect_lte("internet", "", ""); // Egypt APN: etisalat / vodafone

    xTaskCreatePinnedToCore(record_task, "rec", 8192, NULL, 5, NULL, 1);
    xTaskCreatePinnedToCore(event_task, "evt", 4096, NULL, 6, NULL, 1);
}

void record_task(void *arg) {
    while (1) {
        camera_fb_t *fb = esp_camera_fb_get();
        if (fb) {
            write_avi_segment(fb->buf, fb->len, SEGMENT_SEC);
            esp_camera_fb_return(fb);
        }
        vTaskDelay(pdMS_TO_TICKS(1000 / 30));
    }
}

void event_task(void *arg) {
    float ax, ay, az;
    while (1) {
        bmi270_read_accel(&ax, &ay, &az);
        float g = sqrtf(ax*ax + ay*ay + az*az);
        if (fabsf(g - 1.0f) > EVENT_G) {
            lock_current_segment();
            set_resolution(FRAMESIZE_FHD);
            upload_event_clip_async();
        }
        vTaskDelay(pdMS_TO_TICKS(20));
    }
}
```

**Production additions:** TLS 1.3 mutual auth to Nexus cloud, OTA partition rollback, secure boot + flash encryption, watchdog, LTE PSM sleep between uploads, GNSS geofence to disable recording where legally prohibited (e.g., private property with posted notice).

---

## 6. 3D case design parameters (for Hamza)

| Feature | Dimension | Note |
|---|---|---|
| Main body | 180 × 46 × 32 mm | Length exactly 18.0 cm |
| Lens barrel | Ø 16 mm, 8 mm protrusion | 140° lens |
| Vent pattern | 12 × 2 mm slots on top/side | Convection; dust mesh |
| PCB standoffs | M2 × 4 | Vibration |
| microSD access | Side slot with silicone plug | IP54 |
| Connector | 4‑pin JST‑GH power/data | Locking |
| Mount interface | 20 mm ball joint | 3M VHB pad |
| Material | PA12 SLS (pilot) / PC‑ABS injection (volume) | UV‑stabilised |
| Wall thickness | 1.8 mm | Impact + heat |

---

## 7. Legal and regulatory compliance (Egypt — mandatory before sale)

### 7.1 Licences and approvals

| Requirement | Authority | Cost / timeline | Consequence if missing |
|---|---|---|---|
| **NTRA type approval** for LTE/Wi‑Fi/BT device | NTRA | Test fees + sample; 4–8 weeks | Import blocked; device illegal to operate |
| **PDPL visual‑surveillance licence** (public‑place CCTV) | PDPC (MCIT) | **EGP 1,000 / 3 years** (licence) or EGP 500/year (permit) | Fines up to EGP 5 M; criminal liability |
| **PDPL controller/processor licence** if Nexus operates cloud | PDPC | Tiered by record count; up to EGP 666,666/year | Same as above |
| **Ride‑Hailing Act 2180/2019** compliance | Ministry of Transport / Governorate | Vehicle inspection integration | Driver licence non‑renewal |
| **Data Protection Officer** registration | PDPC | Examination + registration | Non‑compliance penalty |

**Critical legal distinction:** The Egyptian PDPL applies to **personal data**, and dashcam footage of identifiable passengers is personal data. Egypt’s executive regulations (Decree 816/2025) explicitly regulate **CCTV and visual surveillance systems**, require consent, signage, retention limits, and restrict cross‑border transfer. A car used for ride‑hailing is treated in Egyptian legal commentary as a **public‑place equivalent** during service, which is why the PDPC CCTV licence applies. Do **not** rely on “private vehicle” exemption — the vehicle is commercial during the trip.

### 7.2 Data protection by design (PDPL‑aligned)

| Control | Implementation |
|---|---|
| **Lawful basis** | Explicit consent via in‑app notice before first trip; passenger sees sticker + LED recording indicator |
| **Notice** | Arabic + English sticker: “This vehicle records video for safety. Footage retained 30 days. Data controller: Nexus. DPO: dpo@nexus.eg” |
| **Purpose limitation** | Recording only for safety/incident evidence; no marketing, no facial recognition (PDPL restricts biometrics) |
| **Retention** | Rolling 7 days at 720p; event clips 30 days; automatic purge |
| **Access** | Driver cannot export raw footage; only Nexus cloud can release to insurer/police with lawful request |
| **Data subject rights** | In‑app request form; 6‑business‑day response; fee capped at EGP 20,000 for complex requests |
| **Breach notification** | PDPC within **72 hours**; data subjects within **3 business days** |
| **Cross‑border transfer** | **No transfer outside Egypt without PDPC approval**; cloud region in Egypt (e.g., Cairo DC) |
| **DPO** | Registered with PDPC; contact published |

### 7.3 Uber / Careem / inDrive platform terms

Uber’s Egypt rider terms expressly acknowledge that drivers may record audio/video via in‑car dashcam, and that **the driver is responsible for obtaining passenger consent** under applicable law. Careem and inDrive operate similar safety‑camera policies but do not indemnify the driver for privacy violations. **Nexus’s product must shift the consent workflow to the platform/device:** passenger consent is captured in the ride‑hailing app before the trip starts, and the device only records when the platform signals an active trip. This is the only scalable legal model. A device that records continuously without trip‑state integration creates **joint liability** for Nexus.

### 7.4 Terms of Service (Nexus N1 — summary enforceable text)

1. **Device use.** N1 is a safety camera for commercial ride‑hailing. You may not use it to record private conversations unrelated to a trip, in private residences, or in any area where a reasonable expectation of privacy exists and no notice is posted.
2. **Consent.** You must display the supplied notice sticker and ensure the platform’s passenger consent flow is completed before recording. Recording without consent is your responsibility and may violate Egyptian Law 151/2020 and Penal Code privacy provisions.
3. **Data ownership.** Raw footage is owned by the vehicle owner; Nexus processes it as a data processor under a data‑processing agreement. Nexus may access footage only for support, safety investigation, or lawful request.
4. **Retention.** Default 7‑day rolling buffer and 30‑day event retention. You may not attempt to disable automatic deletion.
5. **Prohibited use.** No facial recognition, no continuous live streaming to third parties, no resale of footage, no use for marketing.
6. **Warranty.** 12 months hardware; excludes heat damage from operating above 70 °C, water ingress, unauthorised firmware.
7. **Liability.** Nexus liability capped at device purchase price; not liable for platform deactivation, traffic fines, or third‑party privacy claims arising from your failure to obtain consent.
8. **Governing law.** Egyptian law; disputes in Cairo courts.

### 7.5 Privacy Policy (Nexus N1 — key clauses)

**What we collect:** video (720p/1080p), audio if enabled, GPS location, accelerometer events, device ID, LTE signal metadata.  
**Why:** safety incident reconstruction, insurance claims, platform compliance, firmware diagnostics.  
**Legal basis:** explicit consent; legitimate interest in safety (where consent is not required by law).  
**Who we share with:** the platform (only when you submit an incident), insurer/police with lawful order, cloud provider in Egypt under DPA. **We do not sell footage.**  
**Retention:** rolling 7 days; event clips 30 days; GPS metadata 30 days; then automatic deletion.  
**Your rights:** access, correction, deletion, restriction, objection, withdraw consent; contact DPO at dpo@nexus.eg; response within 6 business days.  
**Security:** TLS 1.3, AES‑256 at rest, secure boot, access logging, role‑based access.  
**Breach:** PDPC notified within 72 hours; affected users within 3 business days.  
**Children:** device not intended for trips with unaccompanied minors; if a minor is recorded, guardian consent required.  
**Cross‑border:** no transfer outside Egypt without PDPC approval.

---

## 8. Business model and go‑to‑market (Egypt → US)

| Element | Egypt (Year 1) | US (Year 2–3, after certification) |
|---|---|---|
| Target segment | Uber/Careem/inDrive drivers in Cairo, Giza, Alexandria | Rideshare + small fleets in NY, NJ, CA |
| Addressable drivers | ~700 k–800 k ride‑hailing workforce; ~90 k Uber active historically | ~1.5 M rideshare drivers |
| Device price | **EGP 4,999** | **$129–$149** |
| BOM | $30 | $32 (FCC/CE variants) |
| Gross margin | ~55 % at 1 k units | ~45 % after certification |
| Recurring | Cloud incident vault **EGP 99/month**; fleet dashboard EGP 49/vehicle/month | $4.99/month |
| Channel | Direct via driver Facebook/Telegram groups; installers in Cairo; platform partnership | Amazon + fleet distributors |
| Moat | PDPL/NTRA‑certified, Arabic consent flow, local support, 30‑day retention | Same + US state consent configurability |
| Key risk | Regulatory delay; driver price sensitivity | Two‑party consent states; product liability |

**Unit economics at 1 k units:** Revenue EGP 4.999 M; COGS EGP 1.55 M; gross profit EGP 3.45 M; minus certification, support, warranty ≈ EGP 0.8 M → **contribution ≈ EGP 2.65 M**. Break‑even at ~350 units if fixed costs EGP 1.2 M. Subscription attach rate 25 % adds EGP 25 k/month recurring at 1 k installed base.

---

## 9. Team structure and responsibilities

| Member | Role | Deliverables | RACI |
|---|---|---|---|
| **Anas Samir** | Lead Developer / Principal Researcher | Firmware architecture, LTE/GNSS integration, cloud API, heavy research on PDPL/NTRA, code review | **A/R** for firmware; **R** for research |
| **Mohamed Khaled** | Developer | Camera pipeline, SD card file system, OTA, test harness | **R** for camera/storage; **C** for architecture |
| **Malak Mohamed** | Primary Researcher | Regulatory research, competitor teardown, BOM sourcing, user interviews | **R** for research; **C** for product |
| **Mariam Mohamed** | Primary Tester | Test plans, thermal/vibration/long‑run tests, UX acceptance, bug triage | **A/R** for QA |
| **Hamza Ahmed** | Designer | 3D case CAD, packaging, icon, UI/UX, quality measurement fixtures | **A/R** for industrial design |

**Cadence:** weekly hardware/firmware sync; biweekly legal review; monthly pilot review. Anas and Malak jointly own the research backlog; Mariam has veto on release quality; Hamza has veto on mechanical fit.

---

## 10. Software Requirements Specification (SRS) — condensed

| ID | Requirement | Priority |
|---|---|---|
| FR‑01 | Record 720p30 MJPEG continuously in 60 s segments to microSD | Must |
| FR‑02 | On IMU event >2.5 g, lock current segment and record 30 s pre‑event + 60 s post‑event at 1080p | Must |
| FR‑03 | Upload event clip via LTE to Nexus cloud within 60 s of event | Must |
| FR‑04 | GNSS log every 5 s; embed coordinates in clip metadata | Must |
| FR‑05 | Wi‑Fi AP mode for local configuration and clip download | Must |
| FR‑06 | BLE for provisioning only; no continuous BLE streaming | Should |
| FR‑07 | OTA firmware update with rollback | Must |
| FR‑08 | LED recording indicator and audio consent chime | Must |
| NFR‑01 | Power ≤4 W average at 12 V | Must |
| NFR‑02 | Operating temp −10 to +70 °C; no thermal shutdown below 70 °C ambient | Must |
| NFR‑03 | 30‑day event retention; automatic purge | Must |
| NFR‑04 | TLS 1.3, secure boot, flash encryption | Must |
| NFR‑05 | Arabic + English UI/notices | Must |
| NFR‑06 | MTBF ≥20,000 h at 40 °C | Should |
| CR‑01 | PDPL consent recorded before first frame | Must |
| CR‑02 | No cross‑border data transfer without PDPC approval | Must |
| CR‑03 | NTRA‑approved radio module only | Must |

---

## 11. Empathy map (primary persona: Cairo Uber driver)

| Quadrant | Content |
|---|---|
| **Says** | “I need proof if a passenger lies.” “I can’t afford a $150 camera.” “Will Uber deactivate me for recording?” |
| **Thinks** | “The camera must not invade privacy or I lose ratings.” “Installation must be quick.” “If it breaks, support must answer in Arabic.” |
| **Does** | Uses phone mount already; buys electronics from local market; relies on Telegram groups for tips; drives 10–14 h/day. |
| **Feels** | Anxious about false accusations; price‑sensitive; distrustful of foreign products; proud of providing safe service. |
| **Pains** | High upfront cost; unclear legality; heat damage; platform policy ambiguity. |
| **Gains** | Lower insurance disputes; platform trust badge; Arabic support; visible safety signal to passengers. |

**Passenger persona:** wants to feel safe but fears being recorded without consent; needs visible notice and assurance that footage is not shared. **Fleet operator:** needs 30‑day retention, GPS, and inspection‑passing documentation, not consumer features.

---

## 12. Portfolio of project artifacts

1. **Product requirements document** (this document, expanded).
2. **Hardware schematic + PCB layout** (KiCad, 4‑layer).
3. **3D CAD files** (STEP + STL, 18.0 cm case).
4. **Firmware repository** (ESP‑IDF, MIT licence for pilot; proprietary for production).
5. **Cloud API spec** (OpenAPI 3.1: `/events`, `/clips`, `/consent`, `/devices`).
6. **Test report** (thermal chamber 70 °C, vibration 5–500 Hz, 500 h endurance).
7. **Legal pack** (NTRA application, PDPL licence application, ToS, Privacy Policy, DPA template, consent UI flow).
8. **Pilot report** (50 drivers, 4 weeks, incident capture rate, false positive rate).
9. **Unit economics model** (Excel/Sheets with BOM, FX, volume tiers).
10. **Brand assets** (icon, sticker, packaging).

---

## 13. Icon design brief (for Hamza)

**Concept:** a bold “N” formed by two overlapping camera‑lens arcs; the centre negative space is a lens aperture. Monochrome deep navy (#0B1F3A) on white; accent red recording dot (#E03A3A) optional. SVG 512 × 512; works at 16 px favicon. No text, no gradients, no shadows. The right vertical stroke of the N is a stylised lens barrel; the diagonal is the road. Deliverable: `nexus-icon.svg`, `nexus-icon-1024.png`, `nexus-sticker-60x40mm.pdf` with Arabic/English consent text.

---

## 14. Critical risks and mitigations

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| PDPL licence delay | Medium | High | File early; pilot under research exemption? **No** — do not ship without licence. Use contract manufacturer’s existing CCTV licence? Not transferable. Budget 8–12 weeks. |
| NTRA type approval fails LTE band | Medium | High | Use pre‑certified Quectel module with Egypt band support; test on Etisalat/Vodafone networks. |
| Thermal shutdown in summer | High | Medium | Aluminium heat spreader, 720p default, LTE duty cycle, derating spec. |
| Driver refuses price | High | Medium | Offer instalments (EGP 450/month × 12) via partner; bundle with platform onboarding. |
| Platform policy change | Medium | High | Contract with platforms; make device trip‑state‑aware; never record outside active trip. |
| Data breach | Low | Very high | No raw footage on device longer than 7 days; AES‑256; DPO; 72 h breach playbook. |
| “Not legal” use by driver (continuous recording) | Medium | High | Firmware locks continuous mode unless platform trip signal; audit logs; remote disable. |

---

## 15. Next steps (30/60/90 days)

**0–30 days:** freeze BOM; order 10 EVT boards; Hamza delivers CAD rev A; Malak files NTRA type approval and PDPL CCTV licence; Anas ports camera + LTE firmware to EVT.  
**30–60 days:** 50‑unit pilot with Cairo Uber drivers; Mariam runs thermal/vibration/long‑run tests; legal pack reviewed by Egyptian counsel; refine consent flow with platform.  
**60–90 days:** DVT build 200 units; certification closure; pricing and instalment plan; pilot report; decide injection‑mould tooling (requires ~3 k unit forecast).  

**Do not scale beyond pilot until PDPL licence and NTRA approval are in hand.**
