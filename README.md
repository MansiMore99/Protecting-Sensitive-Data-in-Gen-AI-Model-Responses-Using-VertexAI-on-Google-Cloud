## ☕ Protecting-Sensitive-Data-in-Gen-AI-Model-Responses-Using-VertexAI-on-Google-Cloud

<img width="1600" height="871" alt="image" src="https://github.com/user-attachments/assets/0861c348-5ecd-4a7e-8ab6-8ac4d3985e04" />

<div align="center">

[![Google Cloud](https://img.shields.io/badge/Google_Cloud-4285F4?logo=googlecloud&logoColor=white)](https://cloud.google.com/)
[![Vertex AI](https://img.shields.io/badge/Vertex_AI-34A853?logo=googlecloud&logoColor=white)](https://cloud.google.com/vertex-ai)
[![Cloud Run](https://img.shields.io/badge/Cloud_Run-4285F4?logo=googlecloud&logoColor=white)](https://cloud.google.com/run)

</div>

This project leverages the Cloud Data Loss Prevention (DLP) API to identify, classify, and remediate sensitive data within Gemini model responses. The mission is to transform a raw AI pipeline into a secure, enterprise-grade ecosystem through three core capabilities:

• **Sensitive Data Discovery:** Automated profiling to identify the presence of high-risk data types across the AI ecosystem. You cannot protect what you cannot see; discovery is the foundation of the security lifecycle.

• **De-identification and Redaction:** A surgical approach to data privacy. By masking specific infoTypes, we preserve the utility and context of the Gemini response while neutralizing the risk of PII or financial data exposure.

• **Response Blocking:** A preventative "fail-safe" mechanism. For high-value assets like intellectual property or source code, the architecture implements a complete suppression of the response to prevent exfiltration.
This two-tiered strategy—balancing data utility with rigid IP protection—is essential for moving Generative AI workloads from experimental sandboxes to regulated production environments.

--------------------------------------------------------------------------------
### Environment Setup and Prerequisites:

**Implementation Prerequisites**
• **Google Cloud Services:** Active Project with Vertex AI and Cloud Data Loss Prevention (DLP) API enabled.
• [ ] **Knowledge Areas:** Proficiency in Vertex AI prompting workflows and a foundational understanding of DLP inspection templates and infoTypes.
• [ ] **Tooling:** A Vertex AI Workbench instance configured with a Python 3 kernel.
• [ ] **Configuration:** Ensure your Project ID and Location are pre-configured within your notebook environment as per the lab requirements.

**Quick Start:** Dependency Installation The following Python packages are required to interface with the Vertex AI platform and the Sensitive Data Protection service:

### Install the Google Cloud SDKs for Vertex AI and DLP
```
!pip install google-cloud-aiplatform google-cloud-dlp --upgrade
```
With the environment successfully provisioned, we can proceed to implement automated redaction controls.

--------------------------------------------------------------------------------
### Architecture:

<img width="697" height="361" alt="image" src="https://github.com/user-attachments/assets/2c7a0f51-23e3-48a2-b44f-afd9fc14fa55" />

--------------------------------------------------------------------------------

### Implementation: Redacting Sensitive Data (PII & Financials)

Unlike a total block, redaction replaces specific sensitive strings (identified by the DLP API as infoTypes) with placeholders. This is the preferred method for handling PII and financial data where the user still requires the non-sensitive context of the output.

<img width="964" height="539" alt="image" src="https://github.com/user-attachments/assets/71b20246-bb7a-4744-98d8-4d0357245539" />

--------------------------------------------------------------------------------

De-identification Implementation The following function serves as the primary interface for redacting model outputs. It calls the DLP API to inspect the string for specified global infoTypes and applies a masking transformation.

```
from google.cloud import dlp_v2

def deidentify_gemini_response(project_id, input_text):
    dlp = dlp_v2.DlpServiceClient()
    parent = f"projects/{project_id}/locations/global"

    # Define the Global infoTypes to redact
    info_types = [
        {"name": "PERSON_NAME"},
        {"name": "DATE_OF_BIRTH"},
        {"name": "CREDIT_CARD_NUMBER"}
    ]

    # Construct the de-identification config (Redaction)
    inspect_config = {"info_types": info_types}
    deidentify_config = {
        "info_type_transformations": {
            "transformations": [{"primitive_transformation": {"replace_with_info_type_value": {}}}]
        }
    }

    # API Call to de-identify
    item = {"value": input_text}
    response = dlp.deidentify_content(
        request={
            "parent": parent,
            "deidentify_config": deidentify_config,
            "inspect_config": inspect_config,
            "item": item,
        }
    )
    return response.item.value
```

--------------------------------------------------------------------------------
### Advanced Protection: Blocking Responses Based on Document Types:

In scenarios involving intellectual property, proprietary source code, or legal patents, redaction is insufficient. To prevent the exfiltration of high-value corporate assets, we implement a "block-all" strategy. If the DLP API classifies a Gemini response as a specific sensitive document type, the system intercepts the response entirely, returning a security violation notice instead of the model output.

<img width="1047" height="586" alt="image" src="https://github.com/user-attachments/assets/6095356b-7eff-43dd-87ad-d3c5d46e4ee8" />


Document-Level Protection Targets This implementation utilizes specialized Document infoTypes to detect and block:

• **Source Code:** Built-in detection for programming logic and proprietary code snippets.
• **Patents:** Detection of sensitive intellectual property and patent filings.

```
def secure_document_gatekeeper(project_id, input_text):
    # Call DLP API to inspect for document-level infoTypes
    # Target infoTypes: ['SOURCE_CODE', 'PATENT']
    
    # Logic Flow:
    # 1. Initialize dlp.inspect_content
    # 2. Set info_types = [{"name": "SOURCE_CODE"}, {"name": "PATENT"}]
    # 3. inspection_result = dlp_client.inspect_content(...)
    
    # If a finding exists, trigger a security block
    if inspection_result.findings:
        return "[SECURITY BLOCK]: Response suppressed due to sensitive document classification (IP Protection)."
    
    return input_text
```

This tiered architecture—redacting PII for utility while blocking IP for security—provides a comprehensive safety net for enterprise AI.

--------------------------------------------------------------------------------

### Technical Validation and Verification:

To ensure the reliability of these security controls, we enforce invariable testing. In Gemini model configurations, the temperature must be set to 0. This eliminates stochasticity, ensuring that model outputs are consistent across test iterations, which is critical for reproducible security audits and validating DLP detection accuracy.

Success Criteria for Security Audits
**1. PII Generation:** Confirm the Gemini model can successfully generate text containing sample PII for testing.
**2. API Integration:** Verify the Python function correctly authenticates and communicates with the DLP API using Global infoTypes.
**3. Financial Redaction:** Confirm that CREDIT_CARD_NUMBER infoTypes are successfully identified and masked.
**4. IP Intervention:** Validate that any Gemini response containing SOURCE_CODE or PATENT infoTypes is successfully intercepted and blocked.

```
[!IMPORTANT] Security Requirement: To prevent credential leakage and environment contamination, always conduct testing in an Incognito/Private browser window using temporary project-specific credentials. Ensure all local environment variables are cleared after the session.
```
<img width="1409" height="747" alt="image" src="https://github.com/user-attachments/assets/52151ab6-8fcb-4250-9c5e-133b75e923c4" />

--------------------------------------------------------------------------------

### Resources and Further Documentation:

For detailed technical specifications and advanced de-identification techniques, consult the following:
• ![Cloud Data Loss Prevention (DLP) API Documentation](https://docs.cloud.google.com/sensitive-data-protection/docs)
• ![Redacting Sensitive Data from Text](https://docs.cloud.google.com/sensitive-data-protection/docs/redacting-sensitive-data)
Securing Generative AI is an iterative process. By leveraging the DLP API as security middleware, organizations can realize the benefits of Gemini's intelligence while maintaining an uncompromising security posture.

---

#### 📬 Let’s Connect
Have feedback, questions, or want to contribute? Feel free to reach out or fork the project!
Feel free to reach out and follow me on social media:

<p align="center">
  <a href="https://www.linkedin.com/in/mansimore9/">
    <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn" />
  </a>
  <a href="https://github.com/MansiMore99">
    <img src="https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub" />
  </a>
  <a href="https://medium.com/@mansi.more943">
    <img src="https://img.shields.io/badge/Medium-000000?style=for-the-badge&logo=medium&logoColor=white" alt="Medium" />
  </a>
  <a href="https://x.com/MansiMore99">
    <img src="https://img.shields.io/badge/X-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white" alt="X (Twitter)" />
  </a>
  <a href="https://www.youtube.com/@tech_girl-m9">
    <img src="https://img.shields.io/badge/YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white" alt="YouTube" />
  </a>
</p>

<sub>MIT License © 2025 Mansi</sub>
