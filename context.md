Below is a concise, LLM‑ready summary of **XYCLE**—what it is, who it’s for, and the landscape it operates in. I’ve anchored language and facts to your Messaging, Feature List, Product Updates, and the NetApp follow‑up/demo transcript; I also call out the exact UI terms for consistency with your Instructions.     

---

## Product: What XYCLE is

**One‑liner.** *Advanced LCA software for data‑driven sustainability and compliance* that connects data, teams, and insights in a single workspace—turning complex supply‑chain information into transparent, actionable models and reports. 

**Core idea.** XYCLE replaces fragmented tools and spreadsheets with a unified canvas that links **modelling**, **data**, and **results**, making high‑quality LCAs faster to build, easier to understand, and simpler to share across technical and non‑technical stakeholders. (See *Overview* and short descriptions.) 

**Key capabilities (high‑level).**

* **Flow‑Model:** Interactive process blocks and system diagram for building and refining inventories. 
* **Data Navigator:** End‑to‑end data visibility (from sources to assumptions) to trace how results are derived. 
* **Reporting Tool & flexible charting:** Stakeholder‑friendly visuals; grouping by process, material, component, scope, item type (see screenshot and notes on *page 12* for grouping defaults). 
* **Primary + background data together:** Import supplier/ERP spreadsheets at scale; map to **ecoinvent 3.10**, Minviro’s Battery DB, and (new) Carbon Minds for regionalised chemicals/plastics data. (*Messaging* p.3 for ecoinvent/BMD; *Product Updates* for Carbon Minds).  
* **Collaboration & roles:** Multi‑user modelling, read‑only reviews, and **Viewer** licenses for safe, drill‑down consumption (see *page 3* for explicit viewer permissions). 
* **Compliance methods baked in:** EF **Data Quality** (TeR/GeR/TiR/Precision), ISO 14067 climate split, **AWARE** water scarcity, and full **CFF** (input‑side + end‑of‑life) for EU Battery Regulation/PEF/OEF. 

**Recent additions (selected).**

* **Unified “My Database” table** consolidating supplier products, project products, shared datasets, and imports (see *page 1*). 
* **Bulk Mapping + Mapping Presets** to speed repeat mappings (see *pages 1–2*). 
* **Manage Functional Units** centrally and apply across flows (see *page 2*). 
* **Editable background systems** and **Projects‑as‑background** for sensitivity and reuse (see *page 9–10*). 
* **Databricks integration** to generate modelled LCIs from data lakes (see *page 4*). 
* **Terminology alignment in UI**: *Inbound/Process/Outbound →* **Intermediate inputs / Inputs / Products**; **Characterisation → Dataset**; plus **Dataset unit** and **Dataset region** (see *page 4*). Use these exact strings in prompts and tagging. 

**Security & enterprise fit.** MFA today (OTP), SSO/SAML planned; designed for enterprise IT requirements (noted in the demo).  

---

## Industry: LCA for the Energy Transition

**Context.** Organisations across batteries, EV/ESS, electronics, and materials are moving from spreadsheet LCAs to model‑based workflows to meet reporting, design, and procurement needs. XYCLE’s vision is to make LCA as indispensable as financial models, with values of **Transparency, Efficiency, Collaboration** shaping both product and messaging. 

**Regulatory drivers & methods.**

* **EU Battery Regulation / PEF/OEF** → implemented via **Circular Footprint Formula (CFF)** at inventory level (including end‑of‑life) so circularity is reflected directly in LCIA. (See *pages 5–7*). 
* **ISO 14067 (PCF)** → climate category split; warnings when sub‑scores are absent; export aligned to reporting needs (see *page 7*). 
* **EF Data Quality (DQI)** and **AWARE** water scarcity → model TeR/GeR/TiR/Precision and apply WSF factors for credible, audit‑ready outputs (see *pages 8 & 10*). 

**Data foundations.**

* **Minviro Battery Materials DB** (~70% cathode/anode coverage; region/route detail) for battery supply chains. 
* **Carbon Minds** for regionalised chemicals/plastics with supplier‑specific factors supporting market‑based Scope 3. (See *pages 6–7*). 

---

## Market: Who buys, why now, and how XYCLE positions

**Primary users & stakeholders.**

* **LCA practitioners** seeking speed, accuracy, and transparent audit trails (Flow‑Model, DQI, CFF). 
* **Engineers & product teams** needing scenarios and sensitivity (editable background, flexible grouping). 
* **Sustainability/reporting teams** delivering ISO/PEF/CSRD‑aligned outputs and supplier engagement. 
* **Executives & non‑experts** consuming results via read‑only **Viewer** flows with drill‑down (explicitly described in the demo and *page 3* of Product Updates).  

**Why teams switch.**
Legacy tools are powerful but often siloed and hard to scale; XYCLE emphasises a **single interface** (see *Feature List—Core/Functionality sections*) that scales from spreadsheet import to data‑lake automation and collaborative review, reducing friction from setup to reporting. 

**Differentiators.**

* **Speed to credible models:** Bulk mapping, presets, FU management, and performance improvements for large inventories (see *pages 1–4*). 
* **Transparent traceability:** Data Navigator, EF DQIs, and method‑level alignment (ISO 14067, CFF, AWARE) for regulator‑ready evidence.  
* **Collaboration footprint:** Multi‑user projects, governed reuse (Projects‑as‑background, Products Manager), and explicit viewer controls (*pages 9–11 & 3*). 
* **Enterprise extensibility:** Databricks integration and spreadsheet‑exact exports for downstream BI/reporting. (*pages 4 & 7; Export and Report in Feature List*).  

**Language to standardise in prompts & tagging.**
Use XYCLE’s current UI terms: **Intermediate inputs / Inputs / Products / Dataset**, plus **Dataset unit** and **Dataset region** (see *page 4*, *Product Updates*). This ensures your LLM matches the product’s vocabulary; apply taxonomy levels and confidence gates from your Instructions when auto‑tagging knowledge.  

---

**At‑a‑glance (for LLM memory)**

* **Category:** Life Cycle Assessment (LCA) platform for the energy transition. 
* **Pillars:** Transparency, Efficiency, Collaboration. 
* **Data:** ecoinvent 3.10; Minviro Battery DB; Carbon Minds (regionalised).   
* **Methods:** CFF (incl. EoL), EF DQI, ISO 14067, AWARE. 
* **Collab & roles:** Multi‑user; **Viewer** read‑only with defined restrictions (see *page 3*). 
* **Integrations:** Databricks (data‑lake to model). 

This summary is designed to be dropped into your custom LLM’s knowledge so it can use the **same terms, screenshots, and methods** your product and teams use today.
