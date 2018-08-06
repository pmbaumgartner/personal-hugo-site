---
title: "Potential Issues with Criminal Justice Data"
date: 2018-08-06T12:40:29-04:00
draft: false
---

**Summary:** I wrote this document based on my experience validating a risk assessment instrument. These were some of the issues, rewritten for generalizability, that we encountered. Accessible, quality data is often a project bottleneck, and I've found these helpful to consider before working on a project with criminal justice data.

**Comments:** I wrote these notes in the context of assessing a risk assessment instrument, so "risk factors" are independent variables and "outcomes" are dependent variables.

### Potential Issues

- **Data System Diversity** – Risk factor information can require data from several systems, potentially including individual demographic data, pretrial systems data, criminal history data, arrest data, jail booking data, and court disposition data. 
- **Agency Diversity** – An individual jurisdiction can also have multiple courts, jails, and other agencies that maintain the same type of data but stored in different systems.
- **Data Encoding Diversity** – These data are often encoded in different ways across different systems, requiring reconciliation.
- **Insufficient Data Definition** – Jurisdictions or agencies might not define or capture specific data required for risk factors or outcomes if it’s not required for existing systems.
  - e.g. Felony/Misdemeanor offense information is commonly captured, but violent/non-violent offense information is often not. An accurate lookup of whether an offense is violent might not exist.
- **Definitional Diversity** – Definitions of criminal history factors and severity may differ across jurisdictions.
- **Intra-Agency System Changes** – Agencies adopt new systems for the same data over time.
  - This requires linking records for an individual across time.
  - Legacy systems may not have every piece of requisite data for an accurate criminal history.
  - Jurisdictions have different practices for archiving historical data from legacy data systems. For example, full criminal history information may be unavailable.
- **Individual Data Veracity** – The consistency of the data for any one individual varies across all of these systems.
  - Reconciling various demographic descriptors (such as Hispanicity) across these systems has implications for determining predictive bias.
- **Documentation Quality** – Documentation about the data in different systems is often incomplete, out-of-date, or incorrect.
- **Proprietary System Inconsistencies** – Proprietary systems can store data in different formats and have different options for exporting data.
- **Absence of New Data Storage** – Scores from risk assessment instruments, as well as decisions made using them, aren’t stored in existing systems. This requires historical analysis (i.e. revalidation) based off risk factors where a real-time analysis with calculated assessment scores would be more precise. 