# Blue Team Labs Online (BTLO) — Phishing Analysis Walkthrough

Welcome to the ultimate step-by-step write-up for the **Phishing Analysis** challenge on [Blue Team Labs Online (BTLO)](https://blueteamlabs.online/). This guide is designed to help security enthusiasts of all levels understand how to dissect a raw email (`.eml`) file, perform header analysis, investigate mail server bounces, and extract indicators of compromise (IOCs).

![Challenge Completed](./certificate.jpg)

---

## 📊 Challenge Overview

*   **Platform:** Blue Team Labs Online (BTLO)
*   **Difficulty:** Easy
*   **Category:** Security Operations (SO)
*   **Task Type:** Phishing Investigation & Header Analysis
*   **Points:** 10 Points

---

## 📧 Scenario & Email Flow Diagram

In this challenge, we are handed a suspicious `.eml` file. Before we look at the raw code, let's understand the flow of what actually happened. 

A spammer used a website contact form to send a spam message. The server attempted to send a copy of this message to the user-supplied email address, but because that address was disabled, a bounce message (undeliverable notification) was generated and eventually forwarded to the site owner/analyst.

```mermaid
graph TD
    A[Spammer / Bot <br> IP: 91.90.123.43] -->|Fills out Contact Form| B[Web Server <br> markgardner.com.au]
    B -->|Sends Confirmation Email| C(kinnar1975@yahoo.co.uk)
    C -->|SMTP Error: Mailbox Disabled 554| D[Yahoo DNS / Mail Server]
    D -->|Generates Bounce Message| E[Mail Delivery System <br> Mailer-Daemon@se7-syd.hostedmail.net.au]
    E -->|Delivers Failure Log to Administrator| F[johnsmith123@gmail.com]
```

---

## 🛠️ Phase 1: Setup & Environment Prep

### Step 1.1: Download the Lab Material
1. Log in to [Blue Team Labs Online](https://blueteamlabs.online/).
2. Start the **Phishing Analysis** challenge and download the provided `.zip` archive.
3. Extract the archive using the default platform password: `btlo`.
4. You will get a file named `Website contact form submission.eml`.

### Step 1.2: Choose Your Tools safely
> [!WARNING]
> Never open phishing files or click URLs inside them on your host machine directly. Always perform analysis in a secure text editor or a sandboxed environment.

We will use:
*   **Visual Studio Code (VS Code)** or **Notepad++** to inspect the raw headers.
*   **CyberChef** (optional) for URL decoding.
*   **URL2PNG** or **Browserling** to safely render the malicious link.

---

## 🔍 Phase 2: Step-by-Step Investigation & Answers

### Question 1: Who is the primary recipient of this email?
*   **How to find:** Open the `.eml` file in a text editor. Look at the outer structure of the bounce message. On **Line 105**, under the `Mailer-Daemon` section, you will see the intended recipient of the original message:
    ```text
    To: kinnar1975@yahoo.co.uk <kinnar1975@yahoo.co.uk>
    ```
*   **Why:** Even though the email was eventually delivered to `johnsmith123@gmail.com` (as seen in the `Delivered-To` header at the top), the primary recipient of the failed message was Yahoo mail.
*   **Answer:** `kinnar1975@yahoo.co.uk`

---

### Question 2: What is the subject of this email?
*   **How to find:** Look at the `Subject` header of the undeliverable bounce message.
    *   On **Line 106** (Plain Text) and **Line 143** (HTML View):
        ```text
        Subject: Undeliverable: Website contact form submission
        ```
*   **Answer:** `Undeliverable: Website contact form submission`

---

### Question 3: What is the date and time the email was sent?
*   **How to find:** Locate the timestamp when the bounce message was sent by the mail server.
    *   On **Line 104**:
        ```text
        Sent: 18 March 2021 04:14
        ```
*   **Format Required:** `DD MonthName YYYY HH:MM`
*   **Answer:** `18 March 2021 04:14`

---

### Question 4: What is the Originating IP?
*   **How to find:** Scroll down to the attached email headers (the nested email inside the attachment, starting around **Line 170**). 
    *   On **Line 202**, look for the custom IP header:
        ```text
        X-Originating-IP: 103.9.171.10
        ```
*   **Answer:** `103.9.171.10`

---

### Question 5: Perform reverse DNS on this IP address, what is the resolved host?
*   **How to find:** You can perform a reverse lookup using `whois.domaintools.com` or look directly at the `Received` header block in the attachment (**Line 176**):
    ```text
    Received: from c5s2-1e-syd.hosting-services.net.au ([103.9.171.10])
    ```
*   **Answer:** `c5s2-1e-syd.hosting-services.net.au`

---

### Question 6: What is the name of the attached file?
*   **How to find:** Look at the attachment metadata header. Under `Content-Disposition: attachment;` (starting at **Line 171**), the attachment is a nested email container. In email clients, this displays as the attached email message.
*   **Answer:** `Website contact form submission.eml`

---

### Question 7: What is the URL found inside the attachment?
*   **How to find:** Look at the body of the attached email (**Line 245-247**). You will see the message contents containing two concatenated URLs pointing to Blogspot pages:
    ```text
    https://35000usdperwwekpodf.blogspot.sg?p=9swghttps://35000usdperwwekpodf.blogspot.co.il?o=0hnd
    ```
*   **Answer:** `https://35000usdperwwekpodf.blogspot.sg?p=9swghttps://35000usdperwwekpodf.blogspot.co.il?o=0hnd`

---

### Question 8: What service is this webpage hosted on?
*   **How to find:** Examine the domain name in the URL (`blogspot.sg` and `blogspot.co.il`). Both belong to Google's blog hosting platform.
*   **Answer:** `blogspot` (or `Blogger`)

---

### Question 9: Using URL2PNG, what is the heading text on this page?
*   **How to find:** If you query the URL on a safe screenshot generator tool like **URL2PNG**, you will see a preview indicating the blog has been taken down due to violating terms of service.
*   **Answer:** `Blog has been removed`

---

## 🏆 Summary of Challenge Answers

Below is the consolidated list of answers to submit in the BTLO portal:

| Question | Answer | Format |
| :--- | :--- | :--- |
| **Recipient** | `kinnar1975@yahoo.co.uk` | `mailbox@domain.tld` |
| **Subject** | `Undeliverable: Website contact form submission` | Text |
| **Date & Time** | `18 March 2021 04:14` | `DD MonthName YYYY HH:MM` |
| **Originating IP** | `103.9.171.10` | `X.X.X.X` |
| **Reverse DNS** | `c5s2-1e-syd.hosting-services.net.au` | Domain Name |
| **Attached File** | `Website contact form submission.eml` | `filename.extension` |
| **Malicious URL** | `https://35000usdperwwekpodf.blogspot.sg?p=9swghttps://35000usdperwwekpodf.blogspot.co.il?o=0hnd` | Full URL |
| **Hosting Service** | `blogspot` | Service Name |
| **Heading Text** | `Blog has been removed` | Page Heading |

---

## 💡 Key Takeaways for Analysts
1.  **Look for Backscatter/Bounce Abuse:** Spammers often abuse website contact forms to send unsolicited emails. Since the target mailbox is disabled, the bounce notification backscatter reaches the administrator.
2.  **Inspect Nested RFC822 Attachments:** When analyzing bounce emails, the core indicators (Originating IP, original sender, payload URLs) are almost always nested inside the attached original message block.
3.  **Perform OSINT Safely:** Do not interact with live spam links. Utilize passive lookup tools, reverse DNS, and online screenshot tools (like URL2PNG) to conduct safe investigations.
