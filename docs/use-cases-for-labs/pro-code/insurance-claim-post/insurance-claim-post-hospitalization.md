# Health Insurance - Post Hospitalization Claim 

Healthcare insurance claims involve multiple steps, data sources, and decision points, making it ideal for a multi-agent system.

Here's a scenario: **"Member Sarah submits a claim for a specialist visit and subsequent lab tests."**

We'll define a few agents, each with a specific role and toolset. There will also be a "Master Orchestrator Agent" (often implicit or part of the platform) that routes tasks between agents.

**The Agents & Their Tools:**

1.  **Frontend Conversational Agent (FCA)**
    *   **Role:** Interacts directly with Sarah (the member) via chat, voice, or a web portal. Gathers initial information, provides updates, and answers basic queries.
    *   **Tools:**
        *   **Natural Language Understanding (NLU) & Dialogue Management Engine:** To understand Sarah's requests and maintain conversational context.
        *   **Knowledge Base Access:** For FAQs (e.g., "How do I submit a claim?", "What's my deductible?").
        *   **Member Authentication API:** To verify Sarah's identity.
        *   **Basic Claim Form Interface:** A structured way to collect initial claim details if not purely conversational.

2.  **Claim Intake & Validation Agent (CIVA)**
    *   **Role:** Receives initial claim data from FCA. Validates completeness and basic accuracy of submitted information. Checks for immediate red flags.
    *   **Tools:**
        *   **Optical Character Recognition (OCR) Service:** If claim documents (e.g., superbill) are uploaded.
        *   **Data Extraction & Structuring Tools:** To pull relevant fields (CPT codes, ICD-10 codes, provider NPI, date of service, costs) from unstructured/semi-structured input.
        *   **Member Eligibility Verification API:** Connects to the core insurance system to confirm Sarah is an active member and eligible for benefits on the date of service.
        *   **Provider Network API:** Checks if the specialist and lab are in-network or out-of-network.
        *   **Basic Rules Engine:** For initial validation checks (e.g., "Is date of service in the past?", "Are cost fields numeric?").

3.  **Policy & Benefits Adjudication Agent (PBAA)**
    *   **Role:** Takes validated claim information. Determines coverage based on Sarah's specific policy, applies benefits (deductibles, co-pays, coinsurance), and checks for pre-authorization requirements.
    *   **Tools:**
        *   **Core Policy Management System API:** Accesses detailed policy terms, coverage limits, accumulator status (deductible met, out-of-pocket max).
        *   **Benefits Engine/Calculator:** Applies financial rules based on the policy.
        *   **Pre-authorization Database API:** Checks if the services required pre-authorization and if it was obtained.
        *   **Medical Necessity & Coding Guidelines Database (e.g., InterQual, MCG, NCD/LCD):** For initial cross-referencing of procedure codes against diagnosis codes for appropriateness (can be a lighter check than a dedicated clinical review agent).

4.  **Clinical Review & Fraud Detection Agent (CRFDA) - (Optional, for complex cases or high-value claims)**
    *   **Role:** Performs deeper clinical review for medical necessity, appropriateness of codes, and potential fraud, waste, or abuse. This agent might be triggered if certain thresholds or flags are met.
    *   **Tools:**
        *   **Advanced Medical Coding & Billing Edit Software (e.g., ClaimScrubber functionality):** Detects un-bundling, up-coding, incorrect modifiers.
        *   **Predictive Analytics & Machine Learning Models:** Trained on historical claim data to identify patterns indicative of fraud or abuse.
        *   **Clinical Expert System/Knowledge Base:** More in-depth medical necessity guidelines and treatment protocols.
        *   **Case Management System API:** To flag claims for human review if AI confidence is low or a significant issue is detected.

5.  **Payment & Communication Agent (PCA)**
    *   **Role:** Processes the finalized claim (approved amount), generates Explanation of Benefits (EOB), and initiates communication to the member and provider.
    *   **Tools:**
        *   **Payment Processing Gateway API:** To schedule payments to providers or reimbursements to members.
        *   **Document Generation Service:** To create EOBs.
        *   **Communication Platform API (Email, SMS, Secure Portal Messaging):** To send notifications and EOBs.
        *   **Claim Status Update API:** To update the central claim record.

**Conversational Flow & Multi-Agent Interaction:**

**(Master Orchestrator Agent is implicitly routing tasks based on outcomes)**

**Phase 1: Claim Submission & Initial Validation**

1.  **Sarah (Member) -> Frontend Conversational Agent (FCA):**
    *   **Sarah:** "Hi, I need to submit a new claim for a doctor's visit."
    *   **FCA (using NLU):** "Hello Sarah! I can help with that. To start, can you please provide your Member ID and Date of Birth for verification?"
    *   *(Sarah provides details)*
    *   **FCA (using Member Authentication API):** "Thank you, you're verified. Was this for a specialist visit?"
    *   **Sarah:** "Yes, Dr. Emily Carter on October 26th, 2023. And I also had some lab tests done at Quest Diagnostics the same day."
    *   **FCA (using Claim Form Interface logic & NLU):** "Okay, Dr. Carter on 10/26/2023 and labs at Quest Diagnostics. Do you have the superbill or details of the services and costs? You can type them or upload a document."
    *   *(Sarah uploads a PDF superbill for the specialist and types info for the lab tests)*
    *   **FCA:** "Thanks! I've received the information. I'll pass this along for initial processing. Your reference ID for this submission is CLM12345."
        *   *FCA sends structured data + PDF to Orchestrator for routing.*

2.  **Orchestrator -> Claim Intake & Validation Agent (CIVA):**
    *   **CIVA (receives data):**
        *   *(Uses OCR Service on PDF):* Extracts Dr. Carter's NPI, CPT codes (e.g., 99214), ICD-10 codes (e.g., M54.5 - Low back pain), charges.
        *   *(Uses Data Extraction for typed lab info):* Extracts Quest NPI, CPT codes (e.g., 80053), charges.
        *   *(Uses Member Eligibility Verification API):* Confirms Sarah's policy was active on 10/26/2023. *Result: Active.*
        *   *(Uses Provider Network API):* Checks Dr. Carter (NPI) and Quest Diagnostics (NPI). *Result: Dr. Carter In-Network, Quest Diagnostics In-Network.*
        *   *(Uses Basic Rules Engine):* All mandatory fields present, dates valid. *Result: Basic validation passed.*
        *   *CIVA sends validated & structured claim data (now two separate line items: one for specialist, one for lab) to Orchestrator.*

**Phase 2: Adjudication**

3.  **Orchestrator -> Policy & Benefits Adjudication Agent (PBAA):**
    *   **PBAA (receives validated claim data for Dr. Carter's visit):**
        *   *(Uses Core Policy Management System API):* Retrieves Sarah's plan details: $500 deductible (status: $200 met), 20% coinsurance for specialist after deductible, $30 copay for labs (deductible waived for labs).
        *   *(Uses Pre-authorization Database API):* Checks if CPT 99214 requires pre-auth for diagnosis M54.5. *Result: No pre-auth required.*
        *   *(Uses Benefits Engine):*
            *   Specialist charge: $250.
            *   Remaining deductible: $500 - $200 = $300.
            *   Amount applied to deductible: $250 (since charge < remaining deductible).
            *   Patient responsibility for specialist: $250 (goes to deductible).
            *   Insurance Payout for specialist: $0.
            *   Deductible now met: $200 + $250 = $450.
    *   **PBAA (receives validated claim data for Quest Labs):**
        *   *(Uses Core Policy Management System API):* Confirms lab benefits.
        *   *(Uses Pre-authorization Database API):* Checks if CPT 80053 requires pre-auth. *Result: No pre-auth required.*
        *   *(Uses Benefits Engine):*
            *   Lab charge: $120.
            *   Copay: $30.
            *   Patient responsibility for labs: $30.
            *   Insurance Payout for labs: $120 (allowed) - $30 (copay) = $90.
        *   *PBAA flags that the claim is relatively straightforward and doesn't meet thresholds for CRFDA. Sends adjudicated claim data (member responsibility, insurance payout for each line) to Orchestrator.*

**Phase 3: Finalization & Communication**

4.  **Orchestrator -> Payment & Communication Agent (PCA):**
    *   **PCA (receives adjudicated claim data):**
        *   *(Uses Document Generation Service):* Creates EOB for Sarah detailing:
            *   Dr. Carter: $250 charged, $250 applied to deductible, $0 paid by insurance.
            *   Quest Labs: $120 charged, $30 member copay, $90 paid by insurance.
            *   Updated deductible status: $450 met of $500.
        *   *(Uses Payment Processing Gateway API):* Schedules $90 payment to Quest Diagnostics.
        *   *(Uses Communication Platform API):* Sends email to Sarah: "Your claim CLM12345 has been processed. Your EOB is available in your secure portal. Click here to view." Also sends notification to secure portal.
        *   *(Uses Claim Status Update API):* Marks claim CLM12345 as "Processed."
        *   *PCA notifies Orchestrator of completion.*

5.  **Orchestrator -> Frontend Conversational Agent (FCA) (Optional update):**
    *   *(If Sarah is still in an active chat or checks back)*
    *   **FCA:** "Good news, Sarah! Your claim CLM12345 has been processed. You can find the Explanation of Benefits in your secure member portal. For the specialist visit, $250 was applied to your deductible. For the lab tests, your copay is $30, and we've paid $90 to Quest Diagnostics."

**If CRFDA Agent was Involved (Example):**

Suppose the specialist claim was for a very expensive, unusual procedure.
*   **PBAA** would flag it after initial benefit check and send it to Orchestrator.
*   **Orchestrator -> Clinical Review & Fraud Detection Agent (CRFDA):**
    *   **CRFDA:**
        *   *(Uses Advanced Medical Coding & Billing Edit Software):* Checks for bundling, modifier accuracy. *Result: Codes appear correct.*
        *   *(Uses Predictive Analytics & ML Models):* Assesses fraud risk. *Result: Low risk.*
        *   *(Uses Clinical Expert System):* Verifies medical necessity for the procedure against diagnosis. *Result: Consistent with guidelines for M54.5 after conservative treatment failure (assuming this info was also gathered or inferred).*
        *   *CRFDA sends "Approved from clinical standpoint" back to Orchestrator, who then routes it back to PBAA to finalize financial adjudication if not already done, or directly to PCA.*

**Key Benefits of this Agentic Approach:**

1.  **Specialization:** Each agent focuses on what it does best with its specific tools.
2.  **Scalability:** Individual agents can be scaled independently based on load.
3.  **Maintainability:** Easier to update or replace one agent's tools or logic without affecting the entire system.
4.  **Flexibility:** New agents (e.g., a "Prior Authorization Specialist Agent") can be added to the workflow.
5.  **Efficiency:** Parallel processing can occur (e.g., CIVA validating two parts of a claim simultaneously).
6.  **Improved User Experience:** The FCA provides a single, intelligent point of contact, shielding the user from complex backend processes.

This multi-agent system allows for a sophisticated, automated, yet adaptable claims processing workflow.

**Important things to consider**

- Make sure you go through the Agents and Tools files from the downloaded artifacts
- This use case is built using all native Agents and tools in Watsonx Orchestrate
- Agents and respective tools are deployed within wxO instance and AgentOps is taken care of.
- For adding Observability, you can integrate it with Langfuse and that would show all the traces, sessions, observations and more from all Agents, LLMs, Thinking process and much more on the dashboard.
- For further Agentic AI Governance, this can be integrated with IBM Watsonx Governance.

