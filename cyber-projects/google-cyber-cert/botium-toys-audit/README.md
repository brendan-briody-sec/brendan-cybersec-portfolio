# Botium Toys Security Audit — Risk Review & Recommendations

## 📌 Overview  
This project is part of the Google Cybersecurity Certificate.  
The objective was to review Botium Toys current security posture, identify risks, and recommend improvements based on industry best practices.

## 🎯 Scope  
The audit reviewed:
- Internal systems & employee devices  
- Internal network & internet access  
- Data storage & handling practices  
- Security tools, controls, and policies  
- Compliance considerations (PCI DSS, data privacy)  
- Business continuity readiness  

## 🛑 Key Risks Identified  

| Category | Issue | Risk Level |
|---|---|---|
| Asset Management | No full asset inventory or classification | High |
| Access Control | Excessive permissions, no least-privilege | High |
| Data Security | No encryption for payment & customer data | High |
| Password Security | Weak policy, no centralized management | Medium–High |
| Threat Detection | No IDS deployed | Medium–High |
| Continuity | No backup or disaster recovery plan | High |
| Legacy Systems | No scheduled maintenance plan | Medium |

## ✅ Recommendations  

Based on the risks identified, I recommend Botium Toys implement the following security improvements to strengthen its security posture:

1. **Develop a complete asset inventory and classification system**  
   Ensure all devices, systems, and data are tracked and protected properly.

2. **Introduce least-privilege access controls**  
   Restrict employees to only the data required for their role to reduce accidental or unauthorized access to sensitive information.

3. **Enable encryption for customer and payment information**  
   Encrypt sensitive data both in transit and at rest to protect confidentiality and support regulatory compliance (e.g., PCI DSS).

4. **Strengthen password requirements and implement a centralized password manager**  
   Improve authentication security and reduce risks associated with weak or reused passwords.

5. **Install an Intrusion Detection System (IDS)**  
   Detect and alert on unusual or potentially malicious activity as early as possible.

6. **Create regular data backup procedures and a disaster recovery plan**  
   Ensure critical data can be restored and operations can continue after an incident.

7. **Schedule consistent maintenance for legacy systems**  
   Reduce security vulnerabilities caused by outdated technology.

These improvements will help protect customer data, reduce overall security risk, and move the organization closer to compliance with security best practices.


## 🧠 Skills Demonstrated
- Risk assessment & security analysis  
- Understanding of compliance & data protection needs  
- NIST CSF fundamentals  
- Clear security documentation & communication  

## 📂 Supporting Work
I manually completed the controls and compliance checklist and documented recommendations for improving the company's security posture. Supporting files have been added to this folder.

## 📎 Supporting Files
- [Botium Toys Scope, Goals, and Risk Assessment Document](./botium-toys-scope-goals-and-risk-assessment-report.pdf)
- [My Completed Controls & Compliance Checklist + Recommendations](./compliance-checklist.pdf)

## 📁 Certificate Track
✅ Google Cybersecurity Certificate  
📍 Module: Security Frameworks and Controls

## 🚀 Personal Reflection
For this project, I focused on applying foundational security principles and best practices. This project strengthened my understanding of risk management, security controls, and how to communicate findings clearly and professionally. 
This project allowed me to actively demonstrate my ability to select controls that Botium Toys does or does not need to implement and compliance best practices that the company needs to adhere to in order to mitigate risks and avoid fines.
