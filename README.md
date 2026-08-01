<div align="center">
  <img src="assets/images/logo.png" width="108" alt="BlueWhaleX logo">
  <h1>BlueWhaleX</h1>
  <p><strong>Technical Operations · Public-Service Systems · Emerging Technology</strong></p>
  <p>
    A bilingual portfolio by <strong>Kittiphum Prasomsap</strong>, documenting systems built for real operational work.
  </p>
  <p lang="th">
    พอร์ตโฟลิโอสองภาษาโดย <strong>กิตติภูมิ ประสมทรัพย์</strong> รวบรวมระบบที่พัฒนาขึ้นเพื่อแก้ปัญหาการทำงานจริง
  </p>
  <p>
    <img src="https://img.shields.io/badge/release-v0.5-0b0b0b?style=flat-square" alt="Release v0.5">
    <img src="https://img.shields.io/badge/site-static-0b0b0b?style=flat-square" alt="Static website">
    <img src="https://img.shields.io/badge/language-EN%20%2F%20TH-0b0b0b?style=flat-square" alt="English and Thai">
    <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-0b0b0b?style=flat-square" alt="MIT License"></a>
  </p>
  <p>
    <a href="#project-portfolio">Projects</a> ·
    <a href="#portfolio-experience">Experience</a> ·
    <a href="#local-development">Development</a> ·
    <a href="#contact">Contact</a>
  </p>
</div>

---

## Overview

BlueWhaleX is a single-page portfolio for operational software, interactive visualizations, and developer tooling. It focuses on the work behind each interface: the problem being solved, the people and workflow involved, the system architecture, the technology choices, and the current delivery status.

The experience uses a monochrome flight-deck design language and organizes work into **Mega**, **Advanced**, and **Mini** project tiers. The implementation remains intentionally lightweight—semantic HTML, CSS, and vanilla JavaScript with no framework or build step.

## Project portfolio

| # | Project | Category / Status | Core technology | Project links |
| :-: | --- | --- | --- | --- |
| 01 | **NACC Overnight Parking Log**<br><sub>Records and reviews vehicles parked overnight across NACC buildings.</sub> | Operational system<br>**In service** | Python · Streamlit · Google Sheets · ReportLab | [Source](https://github.com/captwcan/naccparking) · [Live](https://naccparking.streamlit.app/) |
| 02 | **NACC Official-Letter Parking Request & Allocation**<br><sub>Manages requests, allocation, field execution, evidence review, and closure.</sub> | Workflow platform<br>**In service** | TypeScript · Next.js · React · Supabase | [Source](https://github.com/Sleepingknight0/Parking_request) · [Live](https://nacc-parking-user.vercel.app) |
| 03 | **Starlink Mission Control — 3D Explainer**<br><sub>Explains satellite-network paths through a bilingual 24-hour simulation.</sub> | Interactive visualization<br>**Live demo** | JavaScript · Three.js · WebGL · Geobuf | [Source](https://github.com/Sleepingknight0/BWX-STARLINK) · [Demo](https://starlink-plum.vercel.app) |
| 04 | **Chess Vision Assistant**<br><sub>Combines calibrated screen capture, legal position tracking, and Stockfish guidance.</sub> | Windows application<br>**Desktop build** | Python · PySide6 · OpenCV · Stockfish | [Source](https://github.com/Sleepingknight0/chess-vision-assistant) |
| 05 | **Universal AI CLI Launcher**<br><sub>Provides isolated provider and account workflows for installed terminal AI tools.</sub> | Developer tool<br>**Local Windows utility** | PowerShell · JSON · Windows Terminal | [Case study](projects/universal-ai-cli-launcher/README.md) |

## Selected project visuals

<table>
  <tr>
    <td width="50%">
      <img src="assets/images/projects/parking-request-allocation-concept.webp" alt="Parking request and allocation system">
    </td>
    <td width="50%">
      <img src="assets/images/projects/overnight-parking-log-concept.webp" alt="Overnight parking log system">
    </td>
  </tr>
  <tr>
    <td align="center">
      <strong>Parking Request &amp; Allocation</strong><br>
      <sub>End-to-end operational workflow</sub>
    </td>
    <td align="center">
      <strong>Overnight Parking Log</strong><br>
      <sub>Field capture and review dashboard</sub>
    </td>
  </tr>
</table>

## Portfolio experience

- **Bilingual throughout** — English and Thai content share stable translation keys and equivalent interface states.
- **Project registry** — work is classified by scale, system type, technology, and operational status.
- **Search and filters** — visitors can locate projects by keyword, class, stack, or status.
- **Project dossiers** — detailed views present screenshots, workflow, architecture, stack, highlights, and external links.
- **Responsive interaction** — layouts and controls adapt across desktop, tablet, mobile, mouse, and touch input.
- **Accessible behavior** — keyboard navigation, visible focus, managed dialogs, and reduced-motion preferences are supported.

## Technical profile

| Area | Approach |
| --- | --- |
| Front end | Semantic HTML5, CSS custom properties, and vanilla JavaScript |
| Content model | JavaScript project registry with structured English and Thai content |
| Internationalization | Stable `data-i18n` attributes backed by the `TH` dictionary |
| Responsive design | Fluid layouts, mobile navigation, touch targets, and capability detection |
| Motion | Progressive enhancement gated by `RM` and `TOUCH` capability flags |
| Dependencies | Google Fonts is the only external runtime dependency |
| Delivery | Static hosting; no package installation, compilation, or server runtime |

## Local development

Clone the repository and serve its root directory with Python 3:

```bash
git clone https://github.com/Sleepingknight0/bluewhalex-portfolio-v05.git
cd bluewhalex-portfolio-v05
python -m http.server 8000
```

Open [http://localhost:8000/index.html](http://localhost:8000/index.html).

### Repository layout

```text
.
├── index.html                       # Canonical source and site entry point
├── bluewhalex-portfolio.html        # Synchronized distribution copy
├── assets/
│   ├── images/                      # Brand, background, and project media
│   └── videos/                      # Optimized production video
├── projects/
│   └── universal-ai-cli-launcher/   # Public project case study
├── AGENTS.md                        # Repository conventions
└── LICENSE                          # MIT license
```

Make normal changes in `index.html`. New translated elements require a stable `data-i18n` key and a matching entry in the `TH` dictionary.

After an approved change, synchronize the distribution copy:

```powershell
Copy-Item index.html bluewhalex-portfolio.html
```

Confirm both files are identical:

```bash
git diff --no-index index.html bluewhalex-portfolio.html
```

No output means the files match.

## Release checks

Before publishing, verify desktop and mobile layouts, English/Thai switching, keyboard navigation, drawer and dossier focus behavior, reduced-motion handling, media loading, and the browser console.

Project screenshots may originate from public production interfaces. Do not commit credentials, personal records, vehicle registrations, or other operationally sensitive information.

## Contact

- **Kittiphum Prasomsap**
- **GitHub:** [@Sleepingknight0](https://github.com/Sleepingknight0)
- **Email:** [bwcodex@gmail.com](mailto:bwcodex@gmail.com)
- **Location:** Nonthaburi, Thailand · UTC+7

## License

Released under the [MIT License](LICENSE). Copyright © 2026 Kittiphum Prasomsap.
