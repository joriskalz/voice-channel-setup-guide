# Setting Up the Initial Demo Environment for Dynamics 365 Contact Center / Customer Service (Voice Channel)

This guide is a **practical checklist** for creating a **demo-ready environment** for Dynamics 365 Contact Center / Customer Service with the **Voice Channel**.

If everything is set up correctly, you can complete the initial configuration in about **one hour**. If you hit one of the common pitfalls (wrong region, missing roles, misconfigured ACS/Event Grid, etc.), you can easily lose **days**.

Use this guide to **spot and avoid those traps early**.

> [!TIP]
> **Scope of this guide**
> *   Focuses on the **initial Voice Channel setup** for Dynamics 365 Contact Center / Customer Service (not a complete setup).
> *   Targets the areas where many people **struggle the first time** or where a wrong decision causes long delays.
> *   Acts as a **shortcut and sanity check**, helping you verify critical steps quickly.

**What this guide is *not***
*   It does **not replace** the official Microsoft documentation.
*   It is **not** a full production implementation guide.

Use it as **additional guidance** to set up the Voice Channel efficiently and avoid common errors. For full product information, always refer to:
*   [Microsoft Dynamics 365 Contact Center Documentation](https://learn.microsoft.com/en-us/dynamics365/contact-center/)
*   [Microsoft Dynamics 365 Customer Service Documentation](https://learn.microsoft.com/en-us/dynamics365/customer-service/)

**Additional Resources:**
*   🎥 ** Awesome Video Training:** [Microsoft Dynamics 365 Contact Center Video Training](https://powerpete.com/microsoft-dynamics-365-contact-center-video-training/) by MVP Peter Ruiter (aka Power Pete).
*   📊 **Visual Diagram:** [PSTN Voice Flow Diagram](https://voice-flow.demo-monkey.net/) to understand options for connecting the PSTN voice channel. Hover over the items for more details and explanations.
*   📂 **Latest Version:** Available at [GitHub](https://github.com/joriskalz/voice-channel-setup-guide).

---

## Table of Contents

- [Common Pitfalls When Setting Up a Dynamics 365 Contact Center (Voice) Demo](#common-pitfalls-when-setting-up-a-dynamics-365-contact-center-voice-demo)
- [1. Partner Sandbox Licenses (Placeholder)](#1-partner-sandbox-licenses-placeholder)
- [2. After Receiving the Sandbox Licenses](#2-after-receiving-the-sandbox-licenses)
- [3. Create Security Groups](#3-create-security-groups)
- [4. Create the Environment in Power Platform Admin Center](#4-create-the-environment-in-power-platform-admin-center)
- [5. User Management After Environment Creation](#5-user-management-after-environment-creation)
- [6. Assigning Users and Roles in the Environment](#6-assigning-users-and-roles-in-the-environment)
- [7. Activate Channels in Copilot Service Admin Center](#7-activate-channels-in-copilot-service-admin-center)
- [8. Post-Activation Steps for Channels](#8-post-activation-steps-for-channels)
- [9. Handling Trial Phone Numbers](#9-handling-trial-phone-numbers)
- [10. Configure the Telephony Channel in Dynamics 365](#10-configure-the-telephony-channel-in-dynamics-365)
- [11. Create Azure Resources for Telephony (ACS)](#11-create-azure-resources-for-telephony-acs)
- [12. Create App Registration (for Authentication between ACS and Dynamics)](#12-create-app-registration-for-authentication-between-acs-and-dynamics)
- [13. Configure Telephony in Copilot Service Admin Center](#13-configure-telephony-in-copilot-service-admin-center)
- [14. Enter ACS Configuration Values](#14-enter-acs-configuration-values)
- [15. Validate and Establish the Connection](#15-validate-and-establish-the-connection)
- [16. Establish Signaling between Dynamics 365 and ACS (Event Grid Setup)](#16-establish-signaling-between-dynamics-365-and-acs-event-grid-setup)
- [17. Create Event Grid System Topic for ACS](#17-create-event-grid-system-topic-for-acs)
- [18. Create Event Subscriptions (Link ACS → Dynamics 365 Webhooks)](#18-create-event-subscriptions-link-acs--dynamics-365-webhooks)
- [19. Validate Event Grid Configuration](#19-validate-event-grid-configuration)
- [20. Configure the First Voice Channel](#20-configure-the-first-voice-channel)
- [21. Create and Configure a Workstream](#21-create-and-configure-a-workstream)
- [22. Assign Phone Numbers to the Channel / Workstream](#22-assign-phone-numbers-to-the-channel--workstream)
- [23. Configure Languages](#23-configure-languages)
- [24. Configure Behaviors](#24-configure-behaviors-announcements-hours-recording-automated-messages)
- [25. Finalize Workstream Creation](#25-finalize-workstream-creation)

---

## Common Pitfalls When Setting Up a Dynamics 365 Contact Center (Voice) Demo

Before you start, check this list to avoid losing time going in the wrong direction.

### 1. Licensing & Environment Setup

*   **Wrong region (Voice not supported)**
    *   ❌ **Pitfall:** Creating the environment in a region where Voice Channel is not available → Voice can’t be activated.
    *   ✅ **Avoid:** Always choose a **Voice-enabled region** (e.g., EMEA, North America) and verify in the documentation here: [Supported cloud locations for voice channel](https://learn.microsoft.com/en-us/dynamics365/customer-service/administer/voice-channel-region-availability).

*   **Early access features enabled**
    *   ❌ **Pitfall:** Turning on **“Get new features early”** during environment creation. Voice is **not supported** in early access environments.
    *   ✅ **Avoid:** Set **Get new features early = Disabled**.

### 2. Security Roles & User Access

*   **Relying only on “System Administrator”**
    *   ❌ **Pitfall:** Assuming System Administrator automatically sees all Omnichannel features.
    *   ✅ **Avoid:** Explicitly assign:
        *   **Basic User** plus
        *   **Omnichannel Agent** / **Administrator** / **Supervisor**
        *   plus **Customer Service Representative/Manager**, **Productivity Tools User**, **App Profile User** as required.

*   **Roles missing for multi-role users**
    *   ❌ **Pitfall:** A single person acts as Admin, Agent, and Supervisor but only has one role assigned → features “disappear” in the UI.
    *   ✅ **Avoid:** Assign **all relevant roles** to multi-role demo users.

### 3. Channels & Phone Numbers

*   **Not reloading the Copilot Service Admin Center**
    *   ❌ **Pitfall:** Channel activation done, but menus like **Phone Numbers** or advanced settings don’t appear.
    *   ✅ **Avoid:** After activating channels or completing key configurations, **fully reload the browser tab**.

*   **Using the trial phone number in a full demo**
    *   ❌ **Pitfall:** Continuing to use the **trial phone number** (60 minutes limit) and treating it as production → calls suddenly stop working.
    *   ✅ **Avoid:** Under **Channels → Phone Numbers**, select **End Trial** and then configure your “real” telephony setup (ACS or Teams Phone).

### 4. ACS, App Registration & Tenant/Region Alignment

*   **Region mismatch (ACS vs Dynamics environment)**
    *   ❌ **Pitfall:** Creating ACS in a different region from the Dynamics environment → unnecessary latency.
    *   ✅ **Avoid:** Create your **Resource Group and ACS** in the **same region** as the Dynamics environment.

*   **Tenant mismatch**
    *   ❌ **Pitfall:** ACS or App Registration is in a different Azure tenant than Dynamics 365 → connection fails.
    *   ✅ **Avoid:** Ensure **Dynamics 365 environment, ACS, and App Registration** are all in the **same tenant**. Cross-tenant setups are not supported.

*   **Using an Azure trial/credit subscription for phone numbers**
    *   ❌ **Pitfall:** Trying to acquire phone numbers on a **credit-based or trial subscription** → number purchase fails.
    *   ✅ **Avoid:** Use a **paid Azure subscription** for ACS telephony.

*   **Not being Owner of the App Registration**
    *   ❌ **Pitfall:** The person configuring the integration is not an **Owner** of the App Registration → authentication errors and connection failures.
    *   ✅ **Avoid:** In the App Registration, make sure you (or the configurator) are added as **Owner**.

*   **Wrong values in ACS configuration**
    *   ❌ **Pitfall:** Mixing up ACS Resource Name, Resource ID, Connection String, or entering the wrong **Event Grid App ID**.
    *   ✅ **Avoid:**
        *   Copy **ACS Resource Name** and **Resource ID** from ACS → Settings → Properties.
        *   Copy **Connection String** from ACS → Settings → Keys.
        *   For **Event Grid App ID**, use the **Application (Client) ID** from the App Registration (not an Event Grid ID).

### 5. Event Grid & Webhooks (Signaling)

*   **Event Grid not configured**
    *   ❌ **Pitfall:** Ignoring the warning: *"Your Event Grid is not configured for incoming calls or recording…"* Calls and recordings then simply don’t work.
    *   ✅ **Avoid:** Create an **Event Grid System Topic** for ACS and configure **Event Subscriptions** for Incoming Calls and Recording.

*   **Wrong webhook URLs**
    *   ❌ **Pitfall:** Using outdated or incorrect URLs for the Event Subscription endpoints.
    *   ✅ **Avoid:** Always copy the endpoints directly from **Copilot Service Admin Center → Channels → Phone Numbers → Advanced**.

*   **Missing Microsoft Entra authentication in Event Subscriptions**
    *   ❌ **Pitfall:** Creating Event Subscriptions without enabling **Use Microsoft Entra Authentication** → Dynamics won’t accept the events.
    *   ✅ **Avoid:** In each Event Subscription, enable **Microsoft Entra Authentication** and enter the **Tenant ID** and **Application (Client) ID**.

*   **Mixing Tenant ID and Application ID**
    *   ❌ **Pitfall:** Reversing or misplacing **Tenant ID** and **Application ID** in the Event Subscription form (order is different than in the App Registration UI).
    *   ✅ **Avoid:** Double-check the order:
        1.  **Tenant ID**
        2.  **Application (Client) ID**

*   **Not reloading after Event Grid setup**
    *   ❌ **Pitfall:** Event Subscriptions are created, but Dynamics still shows the warning.
    *   ✅ **Avoid:** After finishing Event Grid setup, **reload the Copilot Service Admin Center** and check **Phone Numbers → Advanced** again.

### 6. Workstreams, Queues & Routing

*   **Agents not assigned to the queue**
    *   ❌ **Pitfall:** Creating a custom queue (or relying on fallback queue) but not assigning agents → agents never receive calls.
    *   ✅ **Avoid:** Ensure all demo agents are **assigned to the queue(s)** used by the Voice Workstream.

*   **No fallback queue strategy**
    *   ❌ **Pitfall:** Complex routing rules without a proper fallback queue → calls can get stuck.
    *   ✅ **Avoid:** Verify that your **Fallback Queue** is configured and monitored.

### 7. Testing & Demo Execution

*   **Using the wrong app for the agent**
    *   ❌ **Pitfall:** Logging in as agent in an app that doesn’t support Omnichannel (e.g., Customer Service Hub instead of **Copilot Service Workspace**) → no call pop, no session UI.
    *   ✅ **Avoid:** Use **Copilot Service Workspace** for voice demos.

*   **Not waiting for provisioning/propagation**
    *   ❌ **Pitfall:** Testing immediately after major configuration changes (channel activation, phone number assignment, ACS connection) → intermittent failures.
    *   ✅ **Avoid:** Allow some **minutes for provisioning**, especially for initial Voice channel activation and Phone number setup.

*   **Too few demo users**
    *   ❌ **Pitfall:** Only one user configured → can’t show supervisor/agent scenarios, consult, transfer, etc.
    *   ✅ **Avoid:** Always have at least **one Supervisor and one additional Agent** user to demonstrate routing and collaboration features.

---

## 1. Partner Sandbox Licenses (Placeholder)

## 1. Get Partner Sandbox Licenses

### 1.1 Summary – What & Why

As a Microsoft partner or consultant, you should always use **partner sandbox licenses** for Dynamics 365 when learning, building demos, or testing voice scenarios.

These licenses:

* Give you **non-production** access to Dynamics 365 apps in **your own tenant** or a **custom tenant**.
* Let you build and refine **voice channel** scenarios end-to-end without touching a customer’s environment.
* Are provided at **low or no cost** to qualifying Business Applications partners.

Environment creation comes later – this chapter is only about **getting the licenses** you will need.

---

### 1.2 Steps to Request Sandbox Licenses

1. **Verify partner eligibility and access**

   * Make sure your organization is enrolled as a **Microsoft Business Applications partner**.
   * Access to sandbox licensing is managed via the **partner network administrator** in your company.
   * Ask them to confirm you have access to the **Sandbox Licensing** page:
     [https://dynamicspartners.transform.microsoft.com/benefits-skilling/sandbox-licenses](https://dynamicspartners.transform.microsoft.com/benefits-skilling/sandbox-licenses)

2. **Review available sandbox SKUs**

   On the sandbox licensing page, look for these Dynamics 365 partner sandbox offers:

   * **Dynamics36 Sales,Field Service,and Customer Service PartnerSandbox 25 users**

     * If available, choose the **free** variant.
     * Includes: Sales, Field Service, Customer Service – ideal if you plan to use **Dynamics 365 Customer Service** for voice.
   * **Dynamics 365 Contact Center Partner Sandbox**, this one is user based order at least two in order to demo agent & supervisor scenarios. 

     * Use this if you specifically want to build and demo **Dynamics 365 Contact Center** scenarios.

3. **Request licenses**

   * Go to the license request portal:
     [https://experience.dynamics.com/requestlicense/](https://experience.dynamics.com/requestlicense/)
   

### 1.3 **Docs**

  * [Partner sandbox licenses](https://dynamicspartners.transform.microsoft.com/benefits-skilling/sandbox-licenses)
  * [Request licenses](https://experience.dynamics.com/requestlicense/)

---

## 2. After Receiving the Sandbox Licenses

### 2.1 Activate Promo Codes in Microsoft 365 Admin Center
1.  Go to **Microsoft 365 Admin Center**.
2.  Activate the **promo codes** for:
    *   Dynamics 365 Customer Service
    *   Contact Center licenses
3.  After activation, go to **Billing → Licenses**.
4.  Assign the licenses to the users you will use in your demos.

### 2.2 Plan Your Demo Users
For a realistic demo, create at least:
*   **Supervisor** (can also act as an Agent)
*   **Second Agent user**

This enables scenarios such as Skill-based routing, Supervisor joining/listening, and typical multi-user Contact Center demonstrations.

---

## 3. Create Security Groups

### 3.1 Create a Security Group
In **Microsoft 365 Admin Center → Active Teams & Groups**:
*   Create a new **Security Group**. (Optional)
    *   Example name: `SG-Dynamics-365-EMEA-Demo`

### 3.2 Assign Members
Add to the Security Group:
*   Both Contact Center demo users (Supervisor + Agent)

This group determines **who can access the Dataverse environment** later.

---

## 4. Create the Environment in Power Platform Admin Center

### 4.1 Important: Region Selection for Voice Channel
**Not all regions support Voice Channel.** Your environment **must** be deployed in a region where Microsoft has enabled Voice Channel (e.g., Europe, North America).

> Check availability: [Supported cloud locations for voice channel](https://learn.microsoft.com/en-us/dynamics365/customer-service/administer/voice-channel-region-availability)

### 4.2 Create the Environment
Navigate to: **Power Platform Admin Center → Manage → Environments → New**

Configure:
*   **Name:** e.g., `Contoso EMEA Dynamics 365 Demo`
*   **Region:** Select a Voice-enabled region
*   **Get new features early:** **Disable** (Early access is **not supported** for Voice Channel)
*   **Type:** Production or Sandbox (Trial environments are not fully supported for Voice Channel using ACS/Teams Phone)
*   **Dataverse:** Must be enabled (add Dataverse storage)
*   **Language:** English (recommended for demos; add others later)
*   **Currency:** Euro (€) for EMEA, or as required
*   **Security Group:** Select the Security Group created earlier

### 4.3 Enable Dynamics 365 Apps
Choose **which Dynamics 365 apps** should be provisioned:

**Option A — Customer Service (Recommended)**
*   Recommended for most demos.
*   Provides the **full CRM + Contact Center experience**.
*   No external CRM required.

**Option B — Contact Center Only**
*   Use only if your demo requires an **external CRM** (e.g., Salesforce, ServiceNow).
*   *Note: Contact Center (standalone) does **not** include Case management.*

### 4.4 Storage & Multi-Environment Strategy
*   Licenses are **not environment-bound** — you can create multiple environments as long as you have sufficient **Dataverse storage**.
*   Recommended setup:
    *   **Stable Demo Environment** (Customer Service + Voice Channel)
    *   **Experiment/Test Environment** (e.g., for Private Previews)

### 4.5 Environment Provisioning Time
*   Creating the environment typically takes **1–2 minutes**.

---

## 5. User Management After Environment Creation

### 5.1 Add Users (If Not Yet Created)
In **Microsoft 365 Admin Center**, ensure your Supervisor and Agent accounts exist in **Microsoft Entra ID**.

### 5.2 Assign Licenses
Make sure both users have:
*   Dynamics 365 Customer Service and Contact Center license
*   Any additional required add-ons

### 5.3 Grant Access to the Environment
Because the environment uses a **Security Group**, access is controlled automatically.

---

## 6. Assigning Users and Roles in the Environment

### 6.1 Open the Environment
In the **Power Platform Admin Center**:
1.  Select the newly created environment.
2.  Go to **Settings → Users**.
3.  Add new users (e.g., Agent, Supervisor, additional Admins).

### 6.2 Assign Required Security Roles
For each demo user, assign the following roles:

#### 6.2.1 Core Roles
*   **Basic User**
*   **Customer Service Representative** or **Customer Service Manager (CSM)** (if case management scenarios are required)

#### 6.2.2 Omnichannel Roles
These must be **explicitly assigned**, even if the user is a System Administrator:
*   **Omnichannel Agent**
*   **Omnichannel Administrator**
*   **Omnichannel Supervisor**

> If one user acts in multiple capacities, assign all relevant roles.

#### 6.2.3 Additional Required Roles (as needed)
*   **Productivity Tools User**
*   **App Profile User**

> [!IMPORTANT]
> Even a **System Administrator** does *not* automatically see all Omnichannel features. The Omnichannel roles must be explicitly assigned.

---

## 7. Activate Channels in Copilot Service Admin Center

### 7.1 Access the Copilot Service Admin Center
In the environment overview, click the URL link to the **Copilot Service Admin Center**.

### 7.2 Enable Required Channels
Go to: **Copilot Service Admin Center** → **Manage → Channels**

If channels are not yet activated, the menu will appear very limited. Activate the channels you need (Voice, Chat, Teams, etc.). For this setup, **only Voice is enabled** initially.

#### 7.2.1 Provisioning Time
*   After saving, it can take **up to one hour** until the backend infrastructure for Voice is fully deployed.

---

## 8. Post-Activation Steps for Channels

### 8.1 Reload the Copilot Service Admin Center
Once the channels have been activated:
*   **Fully reload the browser tab**.
*   This ensures additional configuration menus (like **Phone Numbers**) appear.

---

## 9. Handling Trial Phone Numbers

### 9.1 Trial Phone Number Behavior
If a **trial phone number** is displayed under **Channels → Phone Numbers**:
*   You must **end the trial first**.
*   Trial numbers have a 60-minute limit and should be deactivated before configuring a full telephony setup.

### 9.2 End the Trial
1.  Go to the **top-right corner** of the Copilot Service Admin Center.
2.  Select **End Trial**.

---

## 10. Configure the Telephony Channel in Dynamics 365

After removing the trial number, you can proceed to configure **Azure Communication Services (ACS)** or **Teams Phone**.

The following sections focus on **ACS** configuration.

---

## 11. Create Azure Resources for Telephony (ACS)

### 11.1 Create a Resource Group
In the **Azure Portal** (`portal.azure.com`):
1.  Create a **Resource Group**.
2.  **Important:** The **Region must match** the Dynamics 365 environment region (e.g., West Europe).

### 11.2 Create Azure Communication Services (ACS)
1.  In the same Resource Group, create a new resource: **Azure Communication Services**.
2.  Example name: `ACS-Contoso-EMEA-ContactCenter-Demo`

---

## 12. Create App Registration (for Authentication between ACS and Dynamics)

### 12.1 Purpose
The **App Registration** connects Dynamics 365 Contact Center and Azure Communication Services via **Microsoft Entra ID**.

### 12.2 Create the App Registration
In the **Azure Portal → Microsoft Entra ID → App registrations**:
1.  Select **New registration**.
2.  **Name:** e.g., `D365-ACS-Connector-App`
3.  **Supported account types:** **Single tenant**
4.  **Redirect URI:** Leave empty.
5.  After creation, copy:
    *   **Application (Client) ID**
    *   **Tenant ID**

### 12.3 Critical Step: Become the Owner (ONA)
> [!WARNING]
> **This is a major source of configuration errors.**

Within the App Registration:
*   Add *yourself* as **Owner**.
*   If the person configuring ACS in Dynamics is not the Owner, authentication and setup will fail.

---

## 13. Configure Telephony in Copilot Service Admin Center

Return to **Dynamics 365 → Copilot Service Admin Center**.
Go to: **Channels → Phone Numbers** (select **Get Started**).

### 13.1 Choose Voice Integration Option
*   **Teams Phone System:** Easiest for customers already using Teams.
*   **Azure Communication Services (ACS):** Used for this demo scenario.

Select: **Azure Communication Services (ACS)** (tab at the top).
*   *Note: Requires a **paid Azure subscription**.*

---

## 14. Enter ACS Configuration Values

On the ACS configuration page in Dynamics, enter the following values:

### 14.1 From Azure Communication Services → Settings → Properties
*   **ACS Resource Name**
*   **ACS Resource ID**

### 14.2 From Azure Communication Services → Settings → Keys
*   **ACS Connection String**

### 14.3 Event Grid App ID
*   **Value:** Enter the **Application (Client) ID** from the **App Registration**.
*   *Note: This field name is confusing. It is **NOT** an Event Grid ID.*

### 14.4 Tenant Alignment Requirement
*   **Azure tenant** and **Dynamics 365 tenant** must be the **same tenant**.

---

## 15. Validate and Establish the Connection

1.  Confirm the configuration.
2.  Dynamics 365 establishes the connection to ACS.

**Common Errors:** Incorrect ACS values, incorrect Application ID, or missing **Owner** role on the App Registration.

---

## 16. Establish Signaling between Dynamics 365 and ACS (Event Grid Setup)

After connecting ACS, you must configure **signaling** via **Azure Event Grid**.

### 16.1 Warning in Copilot Service Admin Center
In **Channels → Phone Numbers**, you will see a warning:
> "Your Event Grid is not configured for incoming calls or recording. No calls or recording on your Workstream will work without configuration."

Under **Advanced**, find the automatically generated webhooks:
1.  **Incoming Call Webhook Endpoint**
2.  **Recording Webhook Endpoint**
3.  **SMS Webhook Endpoint** (Optional)

These URLS must be registered in Azure in the next step.

---

## 17. Create Event Grid System Topic for ACS

### 17.1 Navigate to Azure Portal
Open the **Resource Group** where you created ACS.

### 17.2 Create Event Grid System Topic
1.  Select **Create → Event Grid System Topic**.
2.  **Topic Type:** `Azure Communication Services`
3.  **Subscription/Resource Group:** Select yours.
4.  **Resource:** Select the ACS resource created earlier.
5.  **Name:** e.g., `eventgrid-st-emea-demo`

After creation, open the new resource.

---

## 18. Create Event Subscriptions (Link ACS → Dynamics 365 Webhooks)

In the Event Grid System Topic, create the individual **Event Subscriptions**.

### 18.1 Subscription for Incoming Calls
*   **Name:** `Incoming-Calls`
*   **Event Type Filter:** `IncomingCall`
*   **Endpoint Type:** `Webhook`
*   **Endpoint URL:** Copy the **Incoming Call Webhook Endpoint** from Dynamics 365 (Phone Numbers → Advanced).

#### Additional Features (Very Important)
Under **Additional Features → Microsoft Entra Authentication**:
1.  Enable **Use Microsoft Entra Authentication**.
2.  **Tenant ID:** Enter Tenant ID.
3.  **Application (Client) ID:** Enter App Registration ID.

> [!CAUTION]
> In the Event Subscription form, the order is **Tenant ID first**, then **Application ID**. This is often reversed compared to other UIs. Check carefully.

### 18.2 Subscription for Recording Events
*   **Name:** `Recording-Updates`
*   **Event Type Filter:** `RecordingFileStatusUpdated`
*   **Endpoint Type:** `Webhook`
*   **Endpoint URL:** Copy the **Recording Webhook Endpoint** from Dynamics 365.
*   **Auth:** Enable Microsoft Entra Authentication (same as above).

### 18.3 (Optional) Subscription for SMS Events
*   Create a third subscription for SMS event types if required, using the **SMS Webhook Endpoint**.

---

## 19. Validate Event Grid Configuration

1.  Go back to **Dynamics 365 Copilot Service Admin Center**.
2.  **Reload the browser completely.**
3.  Under **Channels → Phone Numbers → Advanced**, the warning message should be **gone**.

---

## 20. Configure the First Voice Channel

1.  Go to **Channels → Add new channel**.
2.  Choose **Voice**.

---

## 21. Create and Configure a Workstream

The **Workstream** controls routing, queues, and distribution.

### 21.1 Workstream Basics
*   **Name:** e.g., `voice-en`
*   **Distribution Mode:** **PUSH** (Voice always uses Push).

### 21.2 Fallback Queue
*   Use the automatically generated queue or create your own.
*   *Requirement:* Ensure agents are **assigned to the queue** used.

---

## 22. Assign Phone Numbers to the Channel / Workstream

Select the **phone number** to assign to this channel.
*   *Note: A single Workstream can manage multiple phone numbers.*

---

## 23. Configure Languages

*   Choose the **primary language** (e.g., English).
*   Select a **Voice Profile** (TTS voice).
    *   *Tip: Choose a profile with the latest TTS capabilities for better demo quality.*

---

## 24. Configure Behaviors (Announcements, Hours, Recording, Automated Messages)

Configure hold music, greetings, and recording settings. These can be adjusted later.

---

## 25. Finalize Workstream Creation

After completing all steps, the Voice Channel is linked, and the phone number is active.

**Validation:**
1.  Place a **test call** to the number.
2.  Ensure at least one agent is logged into **Copilot Service Workspace**.
3.  Verify the call rings and routing works.

> [!NOTE]
> If the number is not immediately reachable, wait a few minutes for propagation.
