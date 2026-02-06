# AWS Mainframe Modernization Learning Path

This guide provides a step-by-step plan to learn AWS mainframe modernization, including hands-on practice, certifications, and reference links. 

---

## 1. Understand Mainframe Modernization Concepts
- **Goal:** Learn the basics of mainframe systems and modernization strategies.
- **Resources:**
  - [AWS Mainframe Modernization Overview](https://aws.amazon.com/mainframe-modernization/)
  - [AWS Mainframe Modernization Blog](https://aws.amazon.com/blogs/mainframe/)
- **Outcome:** Understand why and how organizations modernize mainframes, and the AWS services involved.

---

## 2. Learn AWS Fundamentals
- **Goal:** Build foundational AWS knowledge.
- **Resources:**
  - [AWS Cloud Practitioner Essentials (Free Digital Training)](https://explore.skillbuilder.aws/learn/course/134/aws-cloud-practitioner-essentials)
  - [AWS Free Tier](https://aws.amazon.com/free/) (Practice with free services)
- **Outcome:** Navigate AWS, understand core services, and use the AWS Console.

---

## 3. Explore Mainframe Modernization Services
- **Goal:** Deep dive into AWS Mainframe Modernization service and related tools.
- **Resources:**
  - [AWS Mainframe Modernization Service Documentation](https://docs.aws.amazon.com/mainframe-modernization/index.html)
  - [AWS Mainframe Modernization Workshop (Hands-on Lab)](https://mainframe-modernization.workshop.aws/)
- **Outcome:** Learn about application migration, refactoring, and replatforming options.

---

## 4. Hands-On Practice: Migration & Modernization
- **Goal:** Practice migrating and modernizing mainframe workloads.
- **Resources:**
  - [AWS Mainframe Modernization Sample Projects](https://github.com/aws-samples/aws-mainframe-modernization-blueprints)
  - [AWS Mainframe Modernization Tutorials](https://docs.aws.amazon.com/mainframe-modernization/latest/userguide/tutorials.html)
  - [AWS Free Tier](https://aws.amazon.com/free/) (Use for hands-on labs)
- **Outcome:** Gain practical experience with migration tools, refactoring COBOL, and deploying on AWS.

---

## 5. Learn Integration & DevOps for Modernized Workloads
- **Goal:** Integrate modernized workloads with AWS services and CI/CD pipelines.
- **Resources:**
  - [AWS DevOps Essentials](https://explore.skillbuilder.aws/learn/course/57/aws-devops-essentials)
  - [AWS Lambda, API Gateway, and Step Functions Tutorials](https://aws.amazon.com/getting-started/hands-on/)
- **Outcome:** Connect modernized applications to cloud-native services and automate deployments.

---

## 6. Security, Monitoring, and Operations
- **Goal:** Secure, monitor, and operate mainframe workloads on AWS.
- **Resources:**
  - [AWS Security Fundamentals](https://explore.skillbuilder.aws/learn/course/158/aws-security-fundamentals)
  - [AWS CloudWatch and CloudTrail Tutorials](https://aws.amazon.com/getting-started/hands-on/)
- **Outcome:** Understand best practices for security, compliance, and monitoring in AWS.

---

## 7. Certification: AWS Mainframe Modernization Specialty
- **Goal:** Validate your skills with an AWS certification.
- **Resources:**
  - [AWS Certified Mainframe Modernization Specialist (Coming Soon)](https://aws.amazon.com/certification/)
  - [AWS Certification Preparation Resources](https://explore.skillbuilder.aws/learn/course/217/aws-certification-preparation)
- **Outcome:** Earn a credential recognized by employers and peers.

---

## 8. Community & Continuous Learning
- **Goal:** Stay updated and connect with experts.
- **Resources:**
  - [AWS Mainframe Modernization Community](https://repost.aws/tags/mainframe-modernization)
  - [AWS Events & Webinars](https://aws.amazon.com/events/)
- **Outcome:** Be part of a community, get support, and stay current with new features.

---

## Free Software & Practice Options
- **AWS Free Tier:** Practice most services for free.
- **COBOL Compilers:** [OpenCOBOL/GnuCOBOL](https://open-cobol.sourceforge.io/) for local COBOL practice.
- **Emulators:** [Hercules Mainframe Emulator](http://www.hercules-390.eu/) (for learning mainframe basics, not AWS-specific).

---

## Summary Table

| Step | What You’ll Achieve | Key Links |
|------|---------------------|-----------|
| 1    | Mainframe basics    | [Overview](https://aws.amazon.com/mainframe-modernization/) |
| 2    | AWS fundamentals    | [Cloud Practitioner](https://explore.skillbuilder.aws/learn/course/134/aws-cloud-practitioner-essentials) |
| 3    | Service deep dive   | [Docs](https://docs.aws.amazon.com/mainframe-modernization/index.html) |
| 4    | Hands-on migration  | [Workshop](https://mainframe-modernization.workshop.aws/) |
| 5    | Integration/DevOps  | [DevOps Essentials](https://explore.skillbuilder.aws/learn/course/57/aws-devops-essentials) |
| 6    | Security/Monitoring | [Security Fundamentals](https://explore.skillbuilder.aws/learn/course/158/aws-security-fundamentals) |
| 7    | Certification       | [Certification](https://aws.amazon.com/certification/) |
| 8    | Community           | [Community](https://repost.aws/tags/mainframe-modernization) |

---

## Tips
- Use [AWS Free Tier](https://aws.amazon.com/free/) for hands-on labs.
- Follow [AWS Mainframe Modernization Workshop](https://mainframe-modernization.workshop.aws/) for guided practice.
- Join AWS webinars and community for real-world insights.

---

*Last updated: February 6, 2026*

# GEMINI
Practitioner certification, you are perfectly positioned for this niche. You already know what you are modernizing (COBOL, JCL, CICS); now you just need to learn the mechanism of moving it to the Cloud.

There is no single "AWS Mainframe Modernization" exam. Instead, AWS uses a system of Accreditations (Levels 1-3) and Partner Competencies (primarily for Blu Age and Micro Focus/Rocket Software).

Below is your end-to-end expert learning plan, followed by the requested Markdown file.

Phase 1: The Strategic Fundamentals (Week 1)
Goal: Understand the "Two Paths" of AWS Mainframe Modernization.

AWS does not use a "one size fits all" approach. You must choose between these two primary patterns immediately:

Automated Refactoring (Blu Age): Converting COBOL/JCL into Java/Spring Boot. (You stop being a mainframe dev and become a Java dev).

Replatforming (Rocket Software/Micro Focus): Moving COBOL/JCL as-is to run on an emulator on EC2. (You remain a COBOL dev, but your infrastructure is AWS).

Action Items:

Study: Read the AWS Mainframe Modernization User Guide specifically the "Refactor vs. Replatform" section.

Training: Take the free digital course "Getting Started with AWS Mainframe Modernization Service" on AWS Skill Builder or Coursera.

Outcome: Ability to look at a legacy app and decide "Should we rewrite this (Refactor) or just move it (Replatform)?"

Phase 2: The Tooling Deep Dive (Weeks 2-3)
Goal: Master the specific software chains.

Since you are a developer, focus on the Refactoring track first, as this is where the industry is trending (modernizing to Java).

1. AWS Blu Age (Refactoring)

What it is: A tool that reads your mainframe code and generates modern Java code.

Tutorial: Workshop: Automated Refactoring with AWS Blu Age.

Key Concept: Learn to use "Blu Insights" (the analysis tool) to map dependencies between your COBOL programs.

2. Micro Focus / Rocket Software (Replatforming)

What it is: Enterprise Developer (an IDE based on Eclipse or VS Code) that compiles COBOL to run on Windows/Linux.

Tutorial: Workshop: Replatform with Micro Focus.

Key Concept: "Enterprise Analyzer" – mapping the inventory of your mainframe assets before moving them.

Phase 3: Hands-On & "Free" Practice
Goal: Build a Proof of Concept (PoC).

Can you practice for free?

The Software: Mostly No. The specific tools (Blu Age, Micro Focus Enterprise Developer) are enterprise-licensed.

The Workaround:

Visual Studio Code: Download the "AWS Blu Age" extension for VS Code. It is free to install and lets you see syntax highlighting and some transformation logic, though you cannot run the full engine without an AWS account.

Micro Focus Personal Edition: Search for "Micro Focus Visual COBOL Personal Edition". They essentially offer a free trial version for personal learning that runs on your local Windows PC. This allows you to practice compiling COBOL on a non-mainframe machine.

AWS Free Tier: The Mainframe Modernization Service is not free. It charges an hourly rate for the runtime instances. Warning: Be very careful spinning up "M2" (Mainframe Modernization) environments in your personal AWS account; they can be expensive. Always delete them immediately after finishing a tutorial.

Phase 4: Certification & Validation
Goal: Get credentialed.

Since there is no "AWS Certified Mainframe Specialist" exam yet, you should aim for these credentials to prove your worth to clients:

AWS Blu Age L1 Accreditation: This is often available to partners, but check AWS Skill Builder for "Blu Age Fundamentals".

AWS Certified Solutions Architect – Associate: If you don't have this, get it. You cannot design a modernization architecture (VPCs, Subnets, RDS for the new database) without it.

Badge: Complete the "Migrating to AWS" learning badge on AWS Skill Builder.

Your Learning Path Markdown File
Here is the structured file you can save as AWS_Mainframe_Modernization_Plan.md.

Markdown
# AWS Mainframe Modernization Expert Learning Path

**User Profile:** Mainframe Developer (10 YOE) + AWS Cloud Practitioner  
**Goal:** Master AWS Mainframe Modernization (M2) Service

## 🏆 Certifications & Badges to Target
1.  **Primary:** AWS Certified Solutions Architect – Associate (SAA-C03) - *Crucial for infrastructure.*
2.  **Specialized:** AWS Partner: Mainframe Modernization Accreditation (Level 1 - Business, Level 2 - Technical). *Note: Usually requires Partner Portal access.*
3.  **Public:** "AWS Mainframe Modernization - Essentials" (Digital Badge on Skill Builder).

---

## 📅 Month 1: The Foundation (Assess & Mobilize)

### Week 1: Concepts & Architecture
* **Theory:** Understand the "7 R's" of migration, focusing on **Refactor** vs. **Replatform**.
* **Service:** Learn the [AWS Mainframe Modernization (M2) Service](https://aws.amazon.com/mainframe-modernization/).
    * *Key components:* Managed Runtime, Blu Insights, Micro Focus Enterprise Analyzer.
* **Activity:** Read the [AWS Prescriptive Guidance for Mainframe Modernization](https://docs.aws.amazon.com/prescriptive-guidance/latest/strategies-mainframe-modernization/welcome.html).

### Week 2: Assessment Tools (The "Before" Picture)
* **Tool:** **AWS Blu Insights**.
* **Task:** Learn how to upload a zip of COBOL code to Blu Insights to generate a "call graph" (who calls whom).
* **Practice:** Use the "AWS Card Demo" sample code (available on AWS GitHub) to practice assessment.

---

## 📅 Month 2: The Execution (Refactor & Replatform)

### Week 3: Track A - Automated Refactoring (Blu Age)
* **Concept:** Converting COBOL/CICS/DB2 to Java/Tomcat/Aurora.
* **Tutorial:** [Refactor to Java with AWS Blu Age](https://catalog.workshops.aws/bluage-refactor/en-US).
* **Deep Dive:** Understand how JCL is converted to **Groovy** scripts or AWS Step Functions.

### Week 4: Track B - Replatforming (Micro Focus/Rocket)
* **Concept:** Recompiling COBOL to run on Linux EC2.
* **Tutorial:** [Replatform with Micro Focus Enterprise Server](https://catalog.workshops.aws/mainframe-replatform-microfocus/en-US).
* **Task:** Learn how to configure a "Listen" port in the cloud to replace the CICS TN3270 listener.

---

## 📅 Month 3: DevOps & Modernization
* **CI/CD:** Learn how to set up a CodePipeline that triggers a build when you push COBOL code.
* **Database:** Learn **CDC (Change Data Capture)**. How to sync a DB2 (Mainframe) database with an RDS (Cloud) database during the transition period.
* **Project:** Build a "Strangler Fig" pattern.
    1.  Deploy a modernized microservice on AWS.
    2.  Route 10% of traffic from the Mainframe to AWS using an API Gateway.

## 📚 Resource Bank
* **Official Documentation:** [AWS M2 User Guide](https://docs.aws.amazon.com/m2/latest/userguide/what-is-m2.html)
* **Sample Code:** [AWS Mainframe Modernization CardDemo](https://github.com/aws-samples/aws-mainframe-modernization-carddemo)
* **IDE Setup:** [VS Code for AWS Blu Age](https://marketplace.visualstudio.com/items?itemName=AmazonWebServices.aws-blu-age)