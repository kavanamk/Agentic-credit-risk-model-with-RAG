# Credit Risk RAG Agent — README

An automated loan decision pipeline that scores credit risk, detects fraud, retrieves policy context via RAG, and sends personalised emails to applicants — fully automated, one-shot, with a human-in-the-loop escalation path for suspicious cases.

---

## Table of contents

1. [What this project does](#what-this-project-does)
2. [How it works — pipeline walkthrough](#how-it-works)
3. [Architecture diagram explained](#architecture-diagram-explained)
4. [Fraud detection](#fraud-detection)
5. [RAG — how policy documents power the emails](#rag)
6. [Email outputs](#email-outputs)
7. [Human-in-the-loop escalation](#human-in-the-loop-escalation)
8. [Tech stack](#tech-stack)
9. [Project structure](#project-structure)
10. [Getting started](#getting-started)

---

## What this project does

Most credit risk systems stop at the decision: approved or rejected. This project extends that decision into a full communication workflow — automatically generating and sending a personalised email to every applicant based on their outcome, grounded in real company policy data.

![IMG_2292](https://github.com/user-attachments/assets/6d1b933c-f6f4-4c2f-8334-7a158a3e3c02)

The pipeline handles three scenarios:

- **Approved** — sends a tailored approval email with the applicant's interest rate, loan terms, and next steps, all pulled from policy documents
- **Rejected** — sends an empathetic rejection email with an alternative product offer or incentive, also pulled from policy documents
- **Suspicious / flagged** — holds the application, logs the reason, and waits for a human to review before any email is sent

No emails are ever sent blindly. The RAG layer ensures every email is grounded in accurate, up-to-date policy content rather than hallucinated figures.

---

## How it works

The full data flow from application to inbox:

<svg width="100%" viewBox="0 0 680 820" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  <mask id="imagine-text-gaps-m9fdoj" maskUnits="userSpaceOnUse"><rect x="0" y="0" width="680" height="820" fill="white"/><rect x="316.0685729980469" y="8.425444602966309" width="47.862892150878906" height="18.46819496154785" fill="black" rx="2"/><rect x="265.4524841308594" y="39.80135726928711" width="149.0950469970703" height="20.397287368774414" fill="black" rx="2"/><rect x="254.13418579101562" y="56.765907287597656" width="171.7317352294922" height="18.46819496154785" fill="black" rx="2"/><rect x="267.4117431640625" y="86.42545318603516" width="145.17657470703125" height="18.46819496154785" fill="black" rx="2"/><rect x="183.6870574951172" y="121.8013687133789" width="120.6258544921875" height="20.397287368774414" fill="black" rx="2"/><rect x="177.76414489746094" y="140.7659149169922" width="132.47168731689453" height="18.46819496154785" fill="black" rx="2"/><rect x="368.1741638183594" y="121.8013687133789" width="135.65167999267578" height="20.397287368774414" fill="black" rx="2"/><rect x="375.12945556640625" y="140.7659149169922" width="121.7411117553711" height="18.46819496154785" fill="black" rx="2"/><rect x="307.6287841796875" y="233.80136108398438" width="64.74245071411133" height="20.397302627563477" fill="black" rx="2"/><rect x="288.03643798828125" y="248.7659149169922" width="103.9271469116211" height="18.46819496154785" fill="black" rx="2"/><rect x="186.7591094970703" y="226.4254608154297" width="62.48179626464844" height="18.46819496154785" fill="black" rx="2"/><rect x="433.1629638671875" y="226.4254608154297" width="57.674137115478516" height="18.46819496154785" fill="black" rx="2"/><rect x="354.0000305175781" y="283.4254455566406" width="69.27883529663086" height="18.46819496154785" fill="black" rx="2"/><rect x="291.7740478515625" y="338.4254455566406" width="96.45191192626953" height="18.46819496154785" fill="black" rx="2"/><rect x="253.4107666015625" y="315.8013610839844" width="173.17855834960938" height="20.397287368774414" fill="black" rx="2"/><rect x="272.49066162109375" y="332.76593017578125" width="135.01869201660156" height="18.46819496154785" fill="black" rx="2"/><rect x="86.03199005126953" y="371.8013916015625" width="107.93604278564453" height="20.397287368774414" fill="black" rx="2"/><rect x="59.09251022338867" y="388.76593017578125" width="161.81500244140625" height="18.46819496154785" fill="black" rx="2"/><rect x="484.58514404296875" y="371.8013916015625" width="110.82968139648438" height="20.397287368774414" fill="black" rx="2"/><rect x="454.6842346191406" y="388.76593017578125" width="170.6315460205078" height="18.46819496154785" fill="black" rx="2"/><rect x="282.55059814453125" y="371.8013916015625" width="114.89885711669922" height="20.397287368774414" fill="black" rx="2"/><rect x="265.1887512207031" y="388.76593017578125" width="149.62252807617188" height="18.46819496154785" fill="black" rx="2"/><rect x="275.896728515625" y="432.4254455566406" width="128.2065887451172" height="18.46819496154785" fill="black" rx="2"/><rect x="253.02645874023438" y="465.8013610839844" width="173.94717407226562" height="20.397287368774414" fill="black" rx="2"/><rect x="191.46127319335938" y="482.76593017578125" width="297.0775451660156" height="18.468165397644043" fill="black" rx="2"/><rect x="309.000244140625" y="524.4254760742188" width="61.9995231628418" height="18.46819496154785" fill="black" rx="2"/><rect x="267.72821044921875" y="557.8013916015625" width="144.54359436035156" height="20.397287368774414" fill="black" rx="2"/><rect x="219.5762939453125" y="574.7659301757812" width="240.84750366210938" height="18.46819496154785" fill="black" rx="2"/><rect x="293.6278076171875" y="641.8013305664062" width="92.74443817138672" height="20.397287368774414" fill="black" rx="2"/><rect x="277.2681884765625" y="656.7659301757812" width="125.46365356445312" height="18.46819496154785" fill="black" rx="2"/><rect x="284.969482421875" y="713.8013916015625" width="110.0610580444336" height="20.397287368774414" fill="black" rx="2"/><rect x="254.81991577148438" y="730.7659301757812" width="170.3602752685547" height="18.46819496154785" fill="black" rx="2"/><rect x="570.037841796875" y="510.42547607421875" width="95.92442321777344" height="18.46819496154785" fill="black" rx="2"/></mask></defs>

  <!-- LAYER 1: Input -->
  <text x="340" y="22" text-anchor="middle" style="fill:var(--color-text-tertiary);font-weight:500;letter-spacing:.05em;fill:rgb(156, 154, 146);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:500;text-anchor:middle;dominant-baseline:auto">INPUT</text>
  <g onclick="sendPrompt('What data does the credit risk notebook take as input?')" style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="200" y="32" width="280" height="44" rx="8" stroke-width="0.5" style="fill:rgb(68, 68, 65);stroke:rgb(180, 178, 169);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="340" y="50" text-anchor="middle" dominant-baseline="central" style="fill:rgb(211, 209, 199);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Loan application data</text>
    <text x="340" y="66" text-anchor="middle" dominant-baseline="central" style="fill:rgb(180, 178, 169);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Applicant features, financials</text>
  </g>

  <!-- Arrow down -->
  <line x1="340" y1="76" x2="340" y2="108" marker-end="url(#arrow)" mask="url(#imagine-text-gaps-m9fdoj)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- LAYER 2: Existing ML pipeline -->
  <text x="340" y="100" text-anchor="middle" style="fill:var(--color-text-tertiary);font-weight:500;letter-spacing:.05em;fill:rgb(156, 154, 146);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:500;text-anchor:middle;dominant-baseline:auto">EXISTING NOTEBOOK</text>
  <!-- Credit Risk Model box -->
  <g onclick="sendPrompt('What model is used for credit risk scoring?')" style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="160" y="112" width="168" height="56" rx="8" stroke-width="0.5" style="fill:rgb(60, 52, 137);stroke:rgb(175, 169, 236);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="244" y="132" text-anchor="middle" dominant-baseline="central" style="fill:rgb(206, 203, 246);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Credit risk model</text>
    <text x="244" y="150" text-anchor="middle" dominant-baseline="central" style="fill:rgb(175, 169, 236);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Approve / reject score</text>
  </g>
  <!-- OpenAI categorizer -->
  <g onclick="sendPrompt('How does the OpenAI API categorize loan purposes?')" style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="352" y="112" width="168" height="56" rx="8" stroke-width="0.5" style="fill:rgb(60, 52, 137);stroke:rgb(175, 169, 236);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="436" y="132" text-anchor="middle" dominant-baseline="central" style="fill:rgb(206, 203, 246);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">OpenAI categorizer</text>
    <text x="436" y="150" text-anchor="middle" dominant-baseline="central" style="fill:rgb(175, 169, 236);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Loan purpose labels</text>
  </g>
  <!-- connecting arrows to decision node -->
  <line x1="244" y1="168" x2="244" y2="194" marker-end="url(#arrow)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <line x1="436" y1="168" x2="436" y2="194" marker-end="url(#arrow)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <line x1="244" y1="194" x2="436" y2="194" stroke="var(--color-border-secondary)" marker-end="none" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <line x1="340" y1="194" x2="340" y2="218" marker-end="url(#arrow)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- Decision diamond -->
  <polygon points="340,218 400,248 340,278 280,248" fill="var(--color-background-secondary)" stroke="var(--color-border-primary)" stroke-width="0.5" style="fill:rgb(38, 38, 36);stroke:rgba(222, 220, 209, 0.4);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <text x="340" y="244" text-anchor="middle" dominant-baseline="central" style="fill:rgb(250, 249, 245);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Decision</text>
  <text x="340" y="258" text-anchor="middle" dominant-baseline="central" style="fill:rgb(194, 192, 182);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Approve / reject?</text>

  <!-- Branch left: Approved -->
  <line x1="280" y1="248" x2="160" y2="248" marker-end="url(#arrow)" stroke="#1D9E75" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <text x="218" y="240" text-anchor="middle" style="fill:#1D9E75;fill:rgb(29, 158, 117);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:auto">Approved</text>

  <!-- Branch right: Rejected -->
  <line x1="400" y1="248" x2="520" y2="248" marker-end="url(#arrow)" stroke="#D85A30" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <text x="462" y="240" text-anchor="middle" style="fill:#D85A30;fill:rgb(216, 90, 48);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:auto">Rejected</text>

  <!-- Fraud check branch (down from decision) -->
  <line x1="340" y1="278" x2="340" y2="308" marker-end="url(#arrow)" stroke="var(--color-text-warning)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <text x="358" y="297" style="fill:var(--color-text-warning);fill:rgb(209, 160, 65);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:start;dominant-baseline:auto">Suspicious</text>

  <!-- LAYER 3: Agent -->
  <text x="340" y="352" text-anchor="middle" style="fill:var(--color-text-tertiary);font-weight:500;letter-spacing:.05em;fill:rgb(156, 154, 146);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:500;text-anchor:middle;dominant-baseline:auto">AGENT LAYER</text>

  <!-- Fraud escalation box (center below decision) -->
  <g onclick="sendPrompt('What triggers a fraud escalation to a human?')" style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="234" y="308" width="212" height="44" rx="8" stroke-width="0.5" style="fill:rgb(99, 56, 6);stroke:rgb(239, 159, 39);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="340" y="326" text-anchor="middle" dominant-baseline="central" style="fill:rgb(250, 199, 117);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Fraud / anomaly detector</text>
    <text x="340" y="342" text-anchor="middle" dominant-baseline="central" style="fill:rgb(239, 159, 39);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Flag misclassifications</text>
  </g>

  <!-- Approved RAG agent (left) -->
  <g onclick="sendPrompt('What does the RAG agent do for approved loans?')" style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="40" y="362" width="200" height="56" rx="8" stroke-width="0.5" style="fill:rgb(8, 80, 65);stroke:rgb(93, 202, 165);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="140" y="382" text-anchor="middle" dominant-baseline="central" style="fill:rgb(159, 225, 203);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Approval agent</text>
    <text x="140" y="398" text-anchor="middle" dominant-baseline="central" style="fill:rgb(93, 202, 165);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">RAG → craft approval email</text>
  </g>

  <!-- Rejected RAG agent (right) -->
  <g onclick="sendPrompt('What does the RAG agent do for rejected applicants?')" style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="440" y="362" width="200" height="56" rx="8" stroke-width="0.5" style="fill:rgb(113, 43, 19);stroke:rgb(240, 153, 123);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="540" y="382" text-anchor="middle" dominant-baseline="central" style="fill:rgb(245, 196, 179);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Rejection agent</text>
    <text x="540" y="398" text-anchor="middle" dominant-baseline="central" style="fill:rgb(240, 153, 123);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">RAG → craft rejection + offer</text>
  </g>

  <!-- Human in the loop (center) -->
  <g onclick="sendPrompt('What does the human reviewer do in this pipeline?')" style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="234" y="362" width="212" height="56" rx="8" stroke-width="0.5" style="fill:rgb(99, 56, 6);stroke:rgb(239, 159, 39);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="340" y="382" text-anchor="middle" dominant-baseline="central" style="fill:rgb(250, 199, 117);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Human reviewer</text>
    <text x="340" y="398" text-anchor="middle" dominant-baseline="central" style="fill:rgb(239, 159, 39);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Manual review + override</text>
  </g>

  <!-- Arrows from decision branches to agents -->
  <line x1="160" y1="248" x2="140" y2="362" marker-end="url(#arrow)" stroke="#1D9E75" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <line x1="520" y1="248" x2="540" y2="362" marker-end="url(#arrow)" stroke="#D85A30" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <line x1="340" y1="352" x2="340" y2="362" marker-end="url(#arrow)" stroke="var(--color-text-warning)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- LAYER 4: RAG Knowledge Base -->
  <text x="340" y="446" text-anchor="middle" style="fill:var(--color-text-tertiary);font-weight:500;letter-spacing:.05em;fill:rgb(156, 154, 146);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:500;text-anchor:middle;dominant-baseline:auto">KNOWLEDGE BASE</text>

  <!-- RAG doc box -->
  <g onclick="sendPrompt('What policy documents go into the RAG knowledge base?')" style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="150" y="456" width="380" height="56" rx="8" stroke-width="0.5" style="fill:rgb(12, 68, 124);stroke:rgb(133, 183, 235);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="340" y="476" text-anchor="middle" dominant-baseline="central" style="fill:rgb(181, 212, 244);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Policy vector store (RAG)</text>
    <text x="340" y="492" text-anchor="middle" dominant-baseline="central" style="fill:rgb(133, 183, 235);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Interest rates, offers, legal disclaimers, alternatives</text>
  </g>

  <!-- Arrows from agents to RAG -->
  <line x1="140" y1="418" x2="250" y2="456" marker-end="url(#arrow)" stroke="#1D9E75" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <line x1="540" y1="418" x2="430" y2="456" marker-end="url(#arrow)" stroke="#D85A30" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- RAG feeds back to agents -->
  <path d="M150 484 Q110 484 110 418 Q110 390 140 390" fill="none" stroke="var(--color-border-secondary)" stroke-width="1" stroke-dasharray="4 3" marker-end="url(#arrow)" mask="url(#imagine-text-gaps-m9fdoj)" style="fill:none;stroke:rgba(222, 220, 209, 0.3);color:rgb(255, 255, 255);stroke-width:1px;stroke-dasharray:4px, 3px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <path d="M530 484 Q570 484 570 418 Q570 390 540 390" fill="none" stroke="var(--color-border-secondary)" stroke-width="1" stroke-dasharray="4 3" marker-end="url(#arrow)" mask="url(#imagine-text-gaps-m9fdoj)" style="fill:none;stroke:rgba(222, 220, 209, 0.3);color:rgb(255, 255, 255);stroke-width:1px;stroke-dasharray:4px, 3px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- Arrow from RAG down -->
  <line x1="340" y1="512" x2="340" y2="544" marker-end="url(#arrow)" mask="url(#imagine-text-gaps-m9fdoj)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- LAYER 5: Email composer -->
  <text x="340" y="538" text-anchor="middle" style="fill:var(--color-text-tertiary);font-weight:500;letter-spacing:.05em;fill:rgb(156, 154, 146);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:500;text-anchor:middle;dominant-baseline:auto">OUTPUT</text>

  <g onclick="sendPrompt('How does the LLM compose the final email?')" style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="150" y="548" width="380" height="56" rx="8" stroke-width="0.5" style="fill:rgb(68, 68, 65);stroke:rgb(180, 178, 169);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="340" y="568" text-anchor="middle" dominant-baseline="central" style="fill:rgb(211, 209, 199);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">LLM email composer</text>
    <text x="340" y="584" text-anchor="middle" dominant-baseline="central" style="fill:rgb(180, 178, 169);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Grounded in RAG context + decision data</text>
  </g>

  <line x1="340" y1="604" x2="340" y2="634" marker-end="url(#arrow)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- Email send -->
  <g onclick="sendPrompt('Which email API would work best for sending automated loan emails?')" style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="200" y="634" width="280" height="44" rx="8" stroke-width="0.5" style="fill:rgb(68, 68, 65);stroke:rgb(180, 178, 169);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="340" y="652" text-anchor="middle" dominant-baseline="central" style="fill:rgb(211, 209, 199);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Email sender</text>
    <text x="340" y="666" text-anchor="middle" dominant-baseline="central" style="fill:rgb(180, 178, 169);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">SendGrid / Gmail API</text>
  </g>

  <line x1="340" y1="678" x2="340" y2="706" marker-end="url(#arrow)" style="fill:none;stroke:rgb(156, 154, 146);color:rgb(255, 255, 255);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- Applicant inbox -->
  <g onclick="sendPrompt('What does the applicant receive in the final email?')" style="fill:rgb(0, 0, 0);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="200" y="706" width="280" height="44" rx="8" stroke-width="0.5" style="fill:rgb(8, 80, 65);stroke:rgb(93, 202, 165);color:rgb(255, 255, 255);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="340" y="724" text-anchor="middle" dominant-baseline="central" style="fill:rgb(159, 225, 203);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Applicant inbox</text>
    <text x="340" y="740" text-anchor="middle" dominant-baseline="central" style="fill:rgb(93, 202, 165);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">One-shot personalised email</text>
  </g>

  <!-- Human override path from human reviewer down to email -->
  <path d="M446 390 Q610 390 610 634 Q610 656 480 656" fill="none" stroke="var(--color-text-warning)" stroke-width="1" stroke-dasharray="4 3" marker-end="url(#arrow)" mask="url(#imagine-text-gaps-m9fdoj)" style="fill:none;stroke:rgb(209, 160, 65);color:rgb(255, 255, 255);stroke-width:1px;stroke-dasharray:4px, 3px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <text x="618" y="524" text-anchor="middle" style="fill:var(--color-text-warning);fill:rgb(209, 160, 65);stroke:none;color:rgb(255, 255, 255);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, &quot;system-ui&quot;, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:auto" transform="rotate(90,618,524)">human override</text>

</svg>

Each step is a discrete module — they can be tested and debugged independently before wiring together in `main.py`.

---

## Architecture diagram explained

The architecture is split into eight layers, each responsible for one stage of the pipeline.

### Layer 1 — Input

Raw loan application data comes in as a CSV row (or database record). This contains all the applicant features your ML model expects: income, employment length, credit history, loan amount, loan purpose description, and so on.

### Layer 2 — Existing notebook (ML + categorizer)

This is your existing Jupyter notebook work, migrated into `pipeline/model.py`. Two things happen here in parallel:

**Credit risk model** — your trained ML classifier (scikit-learn) scores the application and produces an approve/reject decision along with a confidence probability score. This probability score is important — it feeds directly into fraud detection.

**OpenAI loan categorizer** — a GPT call that reads the applicant's free-text loan purpose description and maps it to a structured category (e.g. "personal expense", "home improvement", "debt consolidation"). This enriches the downstream email with accurate context about why the loan was sought.

Neither of these components changes from what you already built. The new pipeline layers sit on top of them.

### Layer 3 — Decision gate

The combined output of the ML model and categorizer produces a decision: **Approve**, **Reject**, or **Suspicious**. The decision branches the pipeline into three separate paths.

The key point here is that **Suspicious is not a third outcome** — it's an interruption of the normal flow. A suspicious flag means the automated pipeline stops and a human takes over.

### Layer 4 — Fraud / anomaly detector

Before any agent fires, every application passes through `fraud_check.py`. This is a rule-based function (not another ML model) that checks for signals that the input data or the model's decision cannot be trusted.

It returns one of:

| Status | Meaning | What happens |
|--------|---------|-------------|
| `CLEAN` | No flags raised | Pass to approval or rejection agent |
| `REVIEW` | Something looks off | Hold application, alert human, no email sent |
| `AUTO_REJECT` | Extreme red flag | Hard reject, log it, no email sent |

See the [Fraud detection](#fraud-detection) section for the full list of triggers.

### Layer 5 — Agent layer

Two agents handle the two clean outcomes. Both follow the same pattern:

1. Query the RAG vector store with a relevant prompt
2. Receive the top 3 most relevant policy document chunks
3. Pass those chunks + the applicant's data to the LLM
4. Receive a complete, personalised email draft

**Approval agent** — retrieves interest rate tables, loan term details, and next-steps information from the policy store. Produces an email that tells the applicant what rate they qualified for, their monthly payment estimate, and what happens next.

**Rejection agent** — retrieves alternative product offers, discount incentives, and legal disclaimer language. Produces an email that acknowledges the rejection clearly, offers the applicant something useful (a secured loan option, a credit-builder product, a rate discount on a smaller loan), and satisfies any legal communication requirements.

The human reviewer's decision (for flagged cases) also feeds into this layer via the dashed override path — the same email composer fires, but triggered by a human rather than the model.

### Layer 6 — RAG knowledge base (ChromaDB)

The policy vector store is the heart of what makes this pipeline accurate. Instead of letting the LLM invent interest rates or offers, every email is grounded in real documents you provide.

The store is built once from files in your `policy_docs/` folder. Documents are chunked into ~500-token segments, embedded using OpenAI embeddings, and stored locally in ChromaDB. On subsequent runs the store loads from disk — no rebuild needed.

**What to put in your policy documents:**

| File | Contents |
|------|---------|
| `interest_rates.txt` | Rate tables by loan type, credit score band, and loan amount |
| `rejection_offers.txt` | Alternative products, discounts, and retention offers for rejected applicants |
| `legal_disclaimers.txt` | Required legal language for both approval and rejection communications |
| `next_steps.txt` | Post-approval instructions, required documents, processing timelines |

### Layer 7 — LLM email composer

A single OpenAI GPT-4o-mini call that receives:

- A system prompt defining tone, format, and length
- The applicant's name, loan amount, loan purpose category, and decision
- The RAG context chunks retrieved in the previous step

It outputs a plain-text email body ready to send. GPT-4o-mini is used here rather than GPT-4o because it is significantly cheaper (~$0.01 per email) and handles structured email generation very well.

### Layer 8 — Email sender + applicant inbox

The composed email is sent via the Gmail API using OAuth2 authentication. After a one-time browser login on first run, all subsequent sends are fully automated. SendGrid is an alternative if you prefer a dedicated email service.

The applicant receives one email — one-shot, no back-and-forth. The email is personalised to their specific application outcome and grounded in accurate policy data.

---

## Fraud detection

The fraud checker (`pipeline/fraud_check.py`) runs before any agent or email fires. It is intentionally rule-based rather than ML-based — rules are transparent, auditable, and easy to adjust without retraining.

**Triggers for `REVIEW`:**

- Model confidence score is in the grey zone (probability between 0.40 and 0.60 — the model isn't sure)
- Loan amount is more than 10× the applicant's stated annual income
- Implausible field values (negative age, employment tenure longer than the applicant's age, etc.)
- The same email address appears on multiple recent applications with different names or financial details
- OpenAI categorizer returned `unknown` or flagged low confidence on the loan purpose

**Triggers for `AUTO_REJECT`:**

Reserved for extreme cases — use sparingly. Designed for situations where the data is so clearly fraudulent that no human review is needed.

**When a flag is raised:**

- The application is logged with the flag reason
- No email is sent
- A human reviewer is alerted (console output in v1, can be extended to Slack or a dashboard)
- The pipeline moves on to the next application

---

## RAG

RAG (Retrieval-Augmented Generation) is the technique that keeps the LLM's email output accurate and grounded. Without it, GPT-4o-mini would invent plausible-sounding interest rates, offers, and legal language — which is dangerous in a lending context.

**How it works:**

1. At setup time, your policy documents are split into chunks and stored in a local ChromaDB vector database
2. At runtime, each agent queries the store with a natural language prompt relevant to the application (e.g. "interest rate for personal loan, credit score 680, $8,000")
3. ChromaDB returns the 3 most semantically similar chunks from your policy docs
4. Those chunks are injected into the LLM prompt as context
5. The LLM writes the email using only the provided context — it cannot invent figures that aren't in the retrieved chunks

**Why ChromaDB:**

It runs entirely locally on your Mac — no cloud account, no API key, no cost. The vector store is persisted to disk after the first build, so subsequent pipeline runs load instantly.

---

## Email outputs

### Approval email

Contains:
- Personalised greeting by name
- Approval confirmation with the approved loan amount
- Interest rate for their specific loan type and credit score band (from RAG)
- Estimated monthly payment and loan term
- Next steps — what documents to submit, processing timeline (from RAG)
- Contact information

### Rejection email

Contains:
- Empathetic opening
- Clear rejection statement with a general reason
- An alternative product offer or incentive (from RAG) — e.g. a secured loan option, a smaller personal loan at a higher rate, a credit-builder product
- Any applicable discount or retention offer (from RAG)
- Required legal disclaimer language (from RAG)
- Encouragement to reapply in the future

---

## Human-in-the-loop escalation

For flagged applications, the pipeline does not send any email. Instead:

1. The application is logged with the fraud flag reason
2. A human reviewer is notified
3. The reviewer examines the raw application data and the flag reason
4. They make one of three calls: **approve**, **reject**, or **discard**
5. Their decision is fed back into the email composer — the same RAG-grounded email is generated and sent, but triggered by a human decision rather than the model's

In version 1, the human input is a manual step (editing a flag in a file or running a script). This can be extended to a simple web form, a Slack workflow, or an internal dashboard later without changing the underlying pipeline.

The dashed amber line on the right side of the architecture diagram represents this override path — bypassing the automated decision entirely and flowing straight from human reviewer to email composer.

---

## Tech stack

| Component | Technology | Why |
|-----------|-----------|-----|
| Credit scoring | scikit-learn (existing) | Already built |
| Loan categorization | OpenAI GPT-4o (existing) | Already built |
| Fraud detection | Python rule-based | Transparent, auditable |
| Vector store | ChromaDB (local) | Free, no cloud needed, Mac-compatible |
| Embeddings | OpenAI text-embedding-3-small | Fast, cheap, high quality |
| Email composition | OpenAI GPT-4o-mini | ~$0.01/email, great at structured text |
| Email sending | Gmail API (OAuth2) | Free, easy to set up |
| Environment | Python 3.10+, virtualenv | Standard, lightweight |

---

## Project structure

```
credit-risk-pipeline/
│
├── .env                         # API keys — never commit
├── .gitignore
├── credentials.json             # Gmail OAuth — never commit
├── main.py                      # Runs the full pipeline
│
├── data/
│   └── loan_applications.csv    # Input application data
│
├── policy_docs/                 # Source documents for RAG
│   ├── interest_rates.txt
│   ├── rejection_offers.txt
│   ├── legal_disclaimers.txt
│   └── next_steps.txt
│
├── notebooks/
│   └── CreditRisk.ipynb         # Original ML notebook
│
└── pipeline/
    ├── model.py                 # ML scoring logic (from notebook)
    ├── fraud_check.py           # Rule-based fraud detector
    ├── rag.py                   # ChromaDB setup and retrieval
    ├── email_composer.py        # LLM email generation
    └── email_sender.py          # Gmail API integration
```

---

## Getting started

### Prerequisites

- macOS (2017 MacBook or newer)
- Python 3.10+
- An OpenAI API key
- A Google account (for Gmail API)

### Installation

```bash
# Clone the repo
git clone <your-repo-url>
cd credit-risk-pipeline

# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install openai langchain langchain-openai langchain-community \
  chromadb pandas scikit-learn google-auth google-auth-oauthlib \
  google-api-python-client python-dotenv jupyter tiktoken

# Add your OpenAI key
echo "OPENAI_API_KEY=sk-your-key-here" > .env
```

### Gmail API setup

1. Go to https://console.cloud.google.com
2. Create a project → enable **Gmail API**
3. Create an **OAuth client ID** (Desktop app type)
4. Download `credentials.json` and place it in the project root
5. On first run, a browser window opens for one-time authentication

### Running the pipeline

```bash
source venv/bin/activate
python main.py
```

On first run, ChromaDB builds the vector store from your policy docs. All subsequent runs load from disk.

### Testing individual modules

```bash
python -m pipeline.fraud_check     # test fraud detection logic
python -m pipeline.rag             # test RAG retrieval
python -m pipeline.email_composer  # test email generation
python -m pipeline.email_sender    # test Gmail sending
```

---

## Build order (recommended)

| Step | Module | Why this order |
|------|--------|---------------|
| 1 | `fraud_check.py` | No dependencies, easiest to test in isolation |
| 2 | `rag.py` | Needed before email composer |
| 3 | `email_composer.py` | Depends on RAG |
| 4 | `email_sender.py` | Test with your own inbox first |
| 5 | `main.py` | Wire everything together |

Test each module independently before connecting them. Run the full pipeline on 3–5 test rows before using real applicant data.

---

## Key design decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Email style | One-shot, no back-and-forth | Simpler, safer, no state management needed |
| Vector store | ChromaDB local | No cloud dependency, free, Mac-compatible |
| LLM for email | GPT-4o-mini | Cheap, fast, great at structured generation |
| Fraud detection | Rule-based, not ML | Transparent, auditable, easy to tune |
| Human escalation | Hold + log (no auto-email) | Agent never sends for flagged cases — safety first |
| RAG chunking | ~500 tokens per chunk | Balances context richness with retrieval precision |
