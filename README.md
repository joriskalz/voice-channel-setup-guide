# Setting Up the initial Demo Environment for Dynamics 365 Contact Center / Customer Service (Voice Channel)

This guide is a **practical checklist** for creating a **demo-ready environment** for Dynamics 365 Contact Center / Customer Service with the **Voice Channel**.

If everything is set up correctly, you can complete the initial configuration in about **one hour**. If you hit one of the common pitfalls (wrong region, missing roles, misconfigured ACS/Event Grid, etc.), you can easily lose **days**.  

Use this guide to **spot and avoid those traps early**.

**Scope of this guide**

- Focuses on the **initial Voice Channel setup** for Dynamics 365 Contact Center / Customer Service not a complete setup.
- Targets the areas where many people **struggle the first time** or where a wrong decision causes long delays.
- Acts as a **shortcut and sanity check**, helping you verify critical steps quickly.

**What this guide is *not***

- It does **not replace** the official Microsoft documentation.
- It is **not** a full production implementation guide.

Use it as **additional guidance** to set up the Voice Channel efficiently and avoid common errors. For full product information, always refer to:

* [Microsoft Dynamics 365 Contact Center Documentation](https://learn.microsoft.com/en-us/dynamics365/contact-center/)
* [Microsoft Dynamics 365 Customer Service Documentation](https://learn.microsoft.com/en-us/dynamics365/customer-service/)

If you prefer watching instead of reading, there is an excellent video training by MVP Peter (Principal Solution Architect / CTO at Capgemini), also known as **Power Pete**:

[Microsoft Dynamics 365 Contact Center Video Training](https://powerpete.com/microsoft-dynamics-365-contact-center-video-training/)


If you are a visual learner also checkout the following flow-diagram to understand the different options for connecting the PSTN voice channel with Dynamics 365:  [PSTN Voice Flow Diagram](https://voice-flow.demo-monkey.net/)

The latest version of this document is available at [GitHub](https://github.com/joriskalz/voice-channel-setup-guide)


## Common Pitfalls When Setting Up a Dynamics 365 Contact Center (Voice) Demo

Before you start, check this list of common pitfalls to avoid loosing time be going into the  the wrong direction.  Use this as a **checklist of what can go wrong** and how to avoid it.

---

### 1. Licensing & Environment Setup

* **Wrong region (Voice not supported)**
  *Pitfall:* Creating the environment in a region where Voice Channel is not available → Voice can’t be activated.
  *Avoid:* Always choose a **Voice-enabled region** (e.g., EMEA, North America) and verify in the documentation. Check youre region here: [Supported cloud locations for voice channel
](https://learn.microsoft.com/en-us/dynamics365/customer-service/administer/voice-channel-region-availability)


* **Early access features enabled**
  *Pitfall:* Turning on **“Get new features early”** during environment creation. Voice is **not supported** in early access environments.
  *Avoid:* Set **Get new features early = Disabled**.

---

### 2. Security Roles & User Access

* **Relying only on “System Administrator”**
  *Pitfall:* Assuming System Administrator automatically sees all Omnichannel features.
  *Avoid:* Explicitly assign:

  * **Basic User** plus
  * **Omnichannel Agent** or
  * **Omnichannel Administrator** or
  * **Omnichannel Supervisor**
  * plus **Customer Service Representative/Manager**, **Productivity Tools User**, **App Profile User** as required.

* **Roles missing for multi-role users**
  *Pitfall:* A single person acts as Admin, Agent, and Supervisor but only has one role assigned → features “disappear” in the UI.
  *Avoid:* Assign **all relevant roles** to multi-role demo users.

---

### 3. Channels & Phone Numbers

* **Not reloading the Copilot Service Admin Center**
  *Pitfall:* Channel activation done, but menus like **Phone Numbers** or advanced settings don’t appear.
  *Avoid:* After activating channels or completing key configurations, **fully reload the browser tab**.

* **Using the trial phone number in a full demo**
  *Pitfall:* Continuing to use the **trial phone number** (60 minutes limit) and treating it as production → calls suddenly stop working.
  *Avoid:* Under **Channels → Phone Numbers**, select **End Trial** and then configure your “real” telephony setup (ACS or Teams Phone).

---

### 4. ACS, App Registration & Tenant/Region Alignment

* **Region mismatch (ACS vs Dynamics environment)**
  *Pitfall:* Creating ACS in a different region from the Dynamics environment → unnecessary latency.
  *Avoid:* Create your **Resource Group and ACS** in the **same region** as the Dynamics environment.

* **Tenant mismatch**
  *Pitfall:* ACS or App Registration is in a different Azure tenant than Dynamics 365 → connection fails.
  *Avoid:* Ensure **Dynamics 365 environment, ACS, and App Registration** are all in the **same tenant**. Cross-tenant setups are not supported.

* **Using an Azure trial/credit subscription for phone numbers**
  *Pitfall:* Trying to acquire phone numbers on a **credit-based or trial subscription** → number purchase fails.
  *Avoid:* Use a **paid Azure subscription** for ACS telephony.

* **Not being Owner of the App Registration**
  *Pitfall:* The person configuring the integration is not an **Owner** of the App Registration → authentication errors and connection failures.
  *Avoid:* In the App Registration, make sure you (or the configurator) are added as **Owner**.

* **Wrong values in ACS configuration**
  *Pitfall:* Mixing up ACS Resource Name, Resource ID, Connection String, or entering the wrong **Event Grid App ID**.
  *Avoid:*

  * Copy **ACS Resource Name** and **Resource ID** from ACS → Settings → Properties.
  * Copy **Connection String** from ACS → Settings → Keys.
  * For **Event Grid App ID**, use the **Application (Client) ID** from the App Registration (not an Event Grid ID).

---

### 5. Event Grid & Webhooks (Signaling)

* **Event Grid not configured**
  *Pitfall:* Ignoring the warning:

  > “Your Event Grid is not configured for incoming calls or recording…”
  > Calls and recordings then simply don’t work.
  > *Avoid:* Create an **Event Grid System Topic** for ACS and configure **Event Subscriptions** for Incoming Calls and Recording.

* **Wrong webhook URLs**
  *Pitfall:* Using outdated or incorrect URLs for the Event Subscription endpoints.
  *Avoid:* Always copy the endpoints directly from
  **Copilot Service Admin Center → Channels → Phone Numbers → Advanced**.

* **Missing Microsoft Entra authentication in Event Subscriptions**
  *Pitfall:* Creating Event Subscriptions without enabling **Use Microsoft Entra Authentication** → Dynamics won’t accept the events.
  *Avoid:* In each Event Subscription, enable **Microsoft Entra Authentication** and enter:

  * **Tenant ID**
  * **Application (Client) ID**

* **Mixing Tenant ID and Application ID**
  *Pitfall:* Reversing or misplacing **Tenant ID** and **Application ID** in the Event Subscription form (order is different than in the App Registration UI).
  *Avoid:* Double-check:

  * First: **Tenant ID**
  * Second: **Application (Client) ID**

* **Not reloading after Event Grid setup**
  *Pitfall:* Event Subscriptions are created, but Dynamics still shows the warning.
  *Avoid:* After finishing Event Grid setup, **reload the Copilot Service Admin Center** and check **Phone Numbers → Advanced** again.

---

### 6. Workstreams, Queues & Routing

* **Agents not assigned to the queue**
  *Pitfall:* Creating a custom queue (or relying on fallback queue) but not assigning agents → agents never receive calls.
  *Avoid:* Ensure all demo agents are **assigned to the queue(s)** used by the Voice Workstream.

* **No fallback queue strategy**
  *Pitfall:* Complex routing rules without a proper fallback queue → calls can get stuck.
  *Avoid:* Verify that your **Fallback Queue** is configured and monitored.

---

### 7. Testing & Demo Execution

* **Using the wrong app for the agent**
  *Pitfall:* Logging in as agent in an app that doesn’t support Omnichannel (e.g., Customer Service Hub instead of **Copilot Service Workspace**) → no call pop, no session UI.
  *Avoid:* Use **Copilot Service Workspace** for voice demos.

* **Not waiting for provisioning/propagation**
  *Pitfall:* Testing immediately after major configuration changes (channel activation, phone number assignment, ACS connection) → intermittent failures.
  *Avoid:* Allow some **minutes for provisioning**, especially for:

  * Initial Voice channel activation
  * Phone number setup
  * ACS/Event Grid configuration

* **Too few demo users**
  *Pitfall:* Only one user configured → can’t show supervisor/agent scenarios, consult, transfer, etc.
  *Avoid:* Always have at least **one Supervisor and one additional Agent** user to demonstrate routing and collaboration features.

---




## 1. Partner Sandbox Licenses (Placeholder)

> **Note:** This chapter is reserved for a detailed description of how partners obtain and manage **sandbox licenses** (promo codes) for Dynamics 365 Customer Service / Contact Center.
> You will add the detailed content later.

For now, this guide assumes you **already have sandbox licenses available in your tenant** and continues with the steps **after** licenses are received.

---

## 2. After Receiving the Sandbox Licenses

### 2.1 Activate Promo Codes in Microsoft 365 Admin Center

1. Go to **Microsoft 365 Admin Center**.
2. Activate the **promo codes** for:

   * Dynamics 365 Customer Service
   * Contact Center licenses
3. After activation, go to **Billing → Licenses**.
4. Assign the licenses to the users you will use in your demos.

### 2.2 Plan Your Demo Users

For a realistic demo, create at least:

* **Supervisor** (can also act as an Agent)
* **Second Agent user**

This enables scenarios such as:

* Skill-based routing
* Supervisor joining or listening in on active calls
* Typical multi-user Contact Center demonstrations

---

## 3. Create Security Groups

### 3.1 Create a Security Group

In **Microsoft 365 Admin Center → Active Teams & Groups**:

* Create a new **Security Group**. (Optional)

Example name:

```text
SG-Dynamics-365-EMEA-Demo
```

### 3.2 Assign Members

Add to the Security Group:

* Both Contact Center demo users (Supervisor + Agent)

This group determines **who can access the Dataverse environment** later.

---

## 4. Create the Environment in Power Platform Admin Center

### 4.1 Important: Region Selection for Voice Channel

**Not all regions support Voice Channel.**
Your environment **must** be deployed in a region where Microsoft has enabled Voice Channel, for example:

* **Europe** – for EU demos
* **North America**
* (Other supported regions are listed in the official documentation)

> [Supported cloud locations for voice channel
](https://learn.microsoft.com/en-us/dynamics365/customer-service/administer/voice-channel-region-availability)

### 4.2 Create the Environment

Navigate to: **Power Platform Admin Center → Manage → Environments → New**

Configure:

* **Name:** e.g. `Contoso EMEA Dynamics 365 Demo`
* **Region:** Select a Voice-enabled region
* **Get new features early:** **Disable**

  * Early access is **not supported** for Voice Channel
* **Type:** Production or Sandbox, (Trial environments are not fully supported for the Voice Channel using a ACS or Teams Phone System)
* **Dataverse:** Must be enabled (add Dataverse storage)
* **Language:** English (recommended for demos), you can later add additional UI languages.
* **Currency:** Euro (€) for EMEA, or as required
* **Security Group:** Select the Security Group created earlier

### 4.3 Enable Dynamics 365 Apps

During environment creation, choose **which Dynamics 365 apps** should be provisioned:

#### Option A — Customer Service (Recommended)

* Recommended for most demos
* Provides the **full CRM + Contact Center experience**
* No external CRM required

#### Option B — Contact Center Only

Use only if your demo requires an **external CRM**, such as:

* Salesforce
* ServiceNow

Notes:

* Contact Center (standalone) does **not** include Microsoft’s Customer Service features like case management, knowledge management, and other Customer Service features.
* Customer Service and Contact Center **cannot** be enabled together in a single environment.
* You *can* create **multiple environments** with different app configurations.

### 4.4 Storage & Multi-Environment Strategy

* Licenses are **not environment-bound** — you can create multiple environments as long as you have sufficient **Dataverse storage**.
* Recommended setup:

  * **Stable Demo Environment** (Customer Service + Voice Channel)
  * **Experiment/Test Environment** (e.g. for Private Previews, different regions, or different CRM integrations)

### 4.5 Environment Provisioning Time

* Creating the environment typically takes **1–2 minutes**.

---

## 5. User Management After Environment Creation

### 5.1 Add Users (If Not Yet Created)

In **Microsoft 365 Admin Center**, ensure your Supervisor and Agent accounts exist in **Microsoft Entra ID**.

### 5.2 Assign Licenses

Make sure both users have:

* Dynamics 365 Customer Service and Contact Center license
* Any additional required add-ons (depending on your scenario)

### 5.3 Grant Access to the Environment

Because the environment uses a **Security Group**, access is controlled automatically:

* If a user is in the Security Group → they can access the environment.
* If not → access is denied.

---

## 6. Assigning Users and Roles in the Environment

### 6.1 Open the Environment

In the **Power Platform Admin Center**:

* Select the newly created environment.
* Go to **Settings → Users**.

Here you can:

* See existing users.
* Add new users (e.g., Agent, Supervisor, additional Admins).

### 6.2 Assign Required Security Roles

For each demo user, assign the following roles:

#### 6.2.1 Core Roles

* **Basic User**
* **Customer Service Representative** or
* **Customer Service Manager (CSM)** (if case management scenarios are required)

#### 6.2.2 Omnichannel Roles

These must be **explicitly assigned**, even if the user is a System Administrator:

* **Omnichannel Agent**
* **Omnichannel Administrator**
* **Omnichannel Supervisor**

> If one user acts in multiple capacities, assign all relevant roles.

#### 6.2.3 Additional Required Roles as required

* **Productivity Tools User**
* **App Profile User**

#### 6.2.4 Important Note

Even a **System Administrator** does *not* automatically see all Omnichannel features.

The Omnichannel roles must still be assigned because:

* Some surfaces are not displayed without the explicit role.
* The UI hides elements unless the user has the correct security role.

---

## 7. Activate Channels in Copilot Service Admin Center

### 7.1 Access the Copilot Service Admin Center

In the environment overview, a URL link to the **Copilot Service Admin Center** is shown.

### 7.2 Enable Required Channels

Go to: **Copilot Service Admin Center** → **Manage → Channels**

If channels are not yet activated, the menu will appear very limited.

Activate the channels you need for the demo, for example:

* Voice
* Chat
* Microsoft Teams
* Social (optional)
* SMS (optional)

For this setup, **only Voice is enabled** initially, because provisioning takes time.

#### 7.2.1 Provisioning Time

After saving:

* It can take **up to one hour** until the backend infrastructure for Voice is fully deployed and ready.

---

## 8. Post-Activation Steps for Channels

### 8.1 Reload the Copilot Service Admin Center

Once the channels have been activated:

* **Fully reload the browser tab** of the Copilot Service Admin Center.

This is recommended because:

* Additional configuration menus appear only **after** activation.
* The Channels menu expands (for example, the **Phone Numbers** section becomes visible).

---

## 9. Handling Trial Phone Numbers

### 9.1 Trial Phone Number Behavior

If a **trial phone number** is displayed under **Channels → Phone Numbers**:

* You must **end the trial first**.

A trial number is limited:

* Maximum **60 calling minutes**
* Intended for temporary testing
* Should be deactivated before configuring a full telephony setup

### 9.2 End the Trial

Steps:

1. Go to the **top-right corner** of the Copilot Service Admin Center.
2. Select **End Trial**.

Only after ending the trial can you continue with telephony configuration.

---

## 10. Configure the Telephony Channel in Dynamics 365

After removing the trial number, you can proceed to:

* Configure **Azure Communication Services (ACS)**, **or**
* Configure **Teams Phone** for the telephony channel inside Dynamics 365 Contact Center.

The following sections focus on **ACS** configuration.

---

## 11. Create Azure Resources for Telephony (ACS)

### 11.1 Create a Resource Group

In the **Azure Portal** (`portal.azure.com`):

1. Create a **Resource Group**.
2. The **Region must match** the Dynamics 365 environment region.

   * Example: **West Europe**

This region alignment avoids cross-region dependency issues for Voice Channel.

### 11.2 Create Azure Communication Services (ACS)

1. In the same Resource Group, create a new resource: **Azure Communication Services**.
2. Example name:

```text
ACS-Contoso-EMEA-ContactCenter-Demo
```

3. Provisioning takes only a few seconds.

---

## 12. Create App Registration (for Authentication between ACS and Dynamics)

### 12.1 Purpose

The **App Registration** connects:

* Dynamics 365 Contact Center
* Azure Communication Services

Authentication is handled through **Microsoft Entra ID**.

### 12.2 Create the App Registration

In the **Azure Portal → Microsoft Entra ID → App registrations**:

1. Select **New registration**.
2. Configure:

   * **Name:** recognizable (e.g., `D365-ACS-Connector-App`)
   * **Supported account types:** **Single tenant**
   * **Redirect URI:** leave empty (not required)
3. After creation, copy:

   * **Application (Client) ID**
   * **Tenant ID**

These values are required later during setup.

### 12.3 Critical Step: Become the Owner (ONA)

Within the App Registration:

* Add *yourself* as **Owner**.
* If you lack permissions, an admin must add you.

This is a **major source of configuration errors**.

Typical issue:

* The person configuring ACS in Dynamics ≠ person who created the App Registration.

Without being Owner:

* Authentication fails.
* Synchronization fails.
* Setup errors occur.

**Solution: always add yourself as Owner.**

---

## 13. Configure Telephony in Copilot Service Admin Center

Return to **Dynamics 365 → Copilot Service Admin Center**.

Go to: **Channels → Phone Numbers**

Here you complete the ACS configuration.

> Earlier, this was also the place where you disabled the trial phone number.

### 13.1 Start the Configuration

1. Select **Get Started**.
2. Choose between the two voice integration options:

#### Option A — Teams Phone System

* Easiest for customers already using Teams.
* ACS is managed by Microsoft (under the hood).
* No additional direct ACS cost.
* Requires the agent to have Teams Phone licenses.

#### Option B — Azure Communication Services (ACS)

* You manage ACS directly.
* Used for this demo scenario.
* Requires a **paid Azure subscription**

  * Credit-based or trial subscriptions **cannot** be used to obtain phone numbers.

For this guide, select: **Azure Communication Services (ACS)** (tab at the top).

---

## 14. Enter ACS Configuration Values

On the ACS configuration page in Dynamics, enter the following values.

### 14.1 From Azure Communication Services → Settings → Properties

* **ACS Resource Name**
* **ACS Resource ID**

### 14.2 From Azure Communication Services → Settings → Keys

* **ACS Connection String**

### 14.3 Event Grid App ID

This field is confusing:

* It is **NOT** an Event Grid ID.
* It **IS** the **Application (Client) ID** from the **App Registration**.
* The **Tenant ID** is already prefilled and correct (same tenant).

### 14.4 Tenant Alignment Requirement

* **Azure tenant** and **Dynamics 365 tenant** must be the **same tenant**.
* Cross-tenant setups are **not supported**.

---

## 15. Validate and Establish the Connection

After entering all values:

1. Confirm the configuration.
2. Dynamics 365 establishes the connection to ACS.

### 15.1 Common Errors

If an error appears, typical causes are:

* Incorrect ACS values (Name, ID, Connection String).
* Incorrect Application (Client) ID.
* Missing **Owner** role on the App Registration.

Fix the cause and retry.

---

## 16. Establish Signaling between Dynamics 365 and ACS (Event Grid Setup)

After ACS has been connected to Dynamics 365, the next step is to configure **signaling**.

Dynamics 365 uses **Azure Event Grid** to receive:

* Incoming call events
* Recording events
* (Optional) SMS events

These events are delivered via **webhooks**.

### 16.1 Warning in Copilot Service Admin Center

In **Copilot Service Admin Center → Channels → Phone Numbers**, you will see a warning:

> **"Your Event Grid is not configured for incoming calls or recording.
> No calls or recording on your Workstream will work without configuration."**

Under **Advanced**, you see three automatically generated webhooks:

1. **Incoming Call Webhook Endpoint**
2. **Recording Webhook Endpoint**
3. **SMS Webhook Endpoint** (optional, not required in this scenario)

These webhook URLs must be **registered in Azure Event Grid System Topic** as **Event Subscriptions** in the next step.

---

## 17. Create Event Grid System Topic for ACS

### 17.1 Navigate to Azure Portal

In the **Azure Portal**:

* Open the **Resource Group** in which you created ACS.

### 17.2 Create Event Grid System Topic

1. Select **Create → Event Grid System Topic**.
2. Set:

   * **Topic Type = Azure Communication Services**
3. Choose:

   * **Subscription**
   * **Resource Group**
   * **Resource:** the **Azure Communication Services resource** you created earlier
4. Assign a name, for example:

```text
eventgrid-st-emea-demo
```

After creation, open the new resource.

---

## 18. Create Event Subscriptions (Link ACS → Dynamics 365 Webhooks)

In the Event Grid System Topic, create the individual **Event Subscriptions**.

### 18.1 Subscription for Incoming Calls

Create a new **Event Subscription**:

* **Name:** `Incoming-Calls`
* **Event Type Filter:** `IncomingCall`
* **Endpoint Type:** `Webhook`
* **Endpoint URL:**

  * Copy the **Incoming Call Webhook Endpoint** from
    **Dynamics 365 → Copilot Service Admin Center → Channels → Phone Numbers → Advanced**

#### Additional Features (Very Important)

Under **Additional Features → Microsoft Entra Authentication**:

* Enable **Use Microsoft Entra Authentication**.
* Enter:

  * **Tenant ID**
  * **Application (Client) ID** from the App Registration

> ⚠️ **Important:**
> In the Event Subscription form, the order is **Tenant ID first, then Application ID**.
> This reversed order in comparison to the order in the App Registration is a common source of mistakes.

### 18.2 Subscription for Recording Events

Create a second Event Subscription:

* **Name:** `Recording-Updates`
* **Event Type Filter:** `RecordingFileStatusUpdated`
* **Endpoint Type:** `Webhook`
* **Endpoint URL:**

  * Copy the **Recording Webhook Endpoint** from the **Advanced** tab in Dynamics.

#### Additional Features

Again:

* Enable **Use Microsoft Entra Authentication**.
* Enter:

  * **Tenant ID**
  * **Application (Client) ID**

> The pattern is the same as for the Incoming Calls subscription.

### 18.3 (Optional) Subscription for SMS Events

If you plan to use SMS through ACS:

* Create a third subscription for SMS event types.
* Use the **SMS Webhook Endpoint** from Dynamics as the target.

---

## 19. Validate Event Grid Configuration

After creating the subscriptions:

1. Go back to **Dynamics 365 Copilot Service Admin Center**.
2. **Reload the browser completely.**
3. Under **Channels → Phone Numbers → Advanced**, verify:

* The warning message should be **gone**.
* Signaling is now functional.

### 19.1 Result

Dynamics 365 can now:

* Receive incoming call events (signaling via webhook).
* Process recording status updates (Transcription)
* Optionally handle SMS events.

These events drive, within Dynamics:

* Routing to agents.
* Queue routing and work distribution.
* Forwarding to Voice Bots (IVRe).
* Queuing, hold music, and call flow processes.

---

## 20. Configure the First Voice Channel

Now the actual **channel configuration** begins.

### 20.1 Add a New Channel

In the **Copilot Service Admin Center**:

* Go to **Channels → Add new channel**.
* Choose **Voice** (Voice is automatically preselected if it is the only active channel).

In the next step, you will be prompted to configure the Workstream.
---

## 21. Create and Configure a Workstream

The **Workstream** controls:

* Routing
* Queues
* Distribution of incoming calls
* Fallback mechanisms

### 21.1 Workstream Basics

When adding a Voice Channel, you are prompted to configure a Workstream in the next step

Provide a name for the Workstream, example names:

```text
voice-en
voice-de
```

#### Distribution Mode

* For Voice the mode is always **PUSH**:

  * Real-time channels must be delivered immediately.
  * A pull model is not supported for voice.

#### Fallback Queue

If no routing rule matches, the call is routed to the:

* **Fallback Queue**

You can:

* Use the automatically generated queue, or
* Create your **own queue**.

> If you create your own queue, make sure all agents that should receive calls are **assigned to that queue**. Otherwise, they will not receive any calls.

---

## 22. Assign Phone Numbers to the Channel / Workstream

In the next step:

* Select the **phone number** to assign.

Notes:

* A single Workstream can manage **multiple phone numbers**.
* The number is assigned to the **Channel**, and the Channel is linked to the **Workstream**.

For this setup:

* Select **one** phone number for simplicity. You can add additional channels later if required.

---

## 23. Configure Languages

Next:

* Choose the **primary language** for the channel (e.g., English).
* You can add **multiple languages** for multilingual scenarios.

The selected language affects:

* Announcements (greetings, hold messages, etc.).
* Voice Profiles / TTS voice.

> **Tip:** Choose a Voice Profile that uses the **latest TTS capabilities** to showcase modern speech output quality.

All of these settings can be changed later.

---

## 24. Configure Behaviors (Announcements, Hours, Recording, Automated Messages)

This section contains the **voice behavior settings**.

All options can be adjusted later. As of now leave it as is or choose the appropriate settings for your demo.
---

## 25. Finalize Workstream Creation

After completing all steps:

* The Workstream is fully created.
* The Voice Channel is properly linked.
* The phone number is active for the channel.

You can now:

* Place a **test call** to the configured number.
* Verify that at least one agent is **online**.
* Confirm that **routing** and work distribution function correctly.

### 25.1 Select the Correct Agent App

For agents, open the appropriate app:

* **Copilot Service Workspace**

If the number is not yet reachable:

* Wait a few minutes (propagation delay is possible).
* Normally, changes are active after a short moment, but initial provisioning can take longer.

---


