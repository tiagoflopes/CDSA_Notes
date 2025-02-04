## What Is A SIEM Use Case?

A SIEM Use Case enables the effective identification and detection of potential security incidents.
For instance, consider a situation where a user named Rob experiences 10 consecutive failed authentication attempts. These events could originate from the actual user or from a malicious actor trying to brute force their way into the account. In either case, these events are sent to the SIEM system, which then correlates them into a single event and triggers an alert to the SOC team under the "brute force" use case category.

## SIEM Use Case Development Lifecycle

Consider the following when developing any use case:
![[usecase_lifecycle.webp]]

1. Requirements - Comprehend the purpose or necessity of the use case. These can be proposed by customers, analysts or employees.
2. Data Points - Identify all data points within the network where a user account can be used to log in.
3. Log Validation - Verify and validate logs, ensuring they contain all crucial information such as user, timestamp, source, destination, machine name and application name.
4. Design and Implementation - Begin designing the use case by defining the conditions under which an alert should be triggered. Consider: **Condition**, **Aggregation** and **Priority**.
5. Documentation - Standard Operating Procedures (SOP) detail the standard processes analysts must follow when working on alerts. This includes information about other teams to which analysts need to report.
6. Onboarding - Start with the development stage before moving the alert into the production environment. Identify and address any gaps to reduce false positives then proceed to production.
7. Periodic Update/Fine-tuning - Obtain regular feedback from analysts and maintain up-to-date correlation rules by whitelisting. Continually refine and optimize the use case.
## How To Build SIEM Use Cases

- Comprehend your needs, risks and establish alerts for monitoring all necessary systems accordingly.
- Determine the priority and impact, them map the alert to the kill chain or MITRE framework.
- Establish the Time to Detection (TTD) and Time to Response (TTR) for the alert to assess the SIEM's effectiveness and analysts' performance.
- Create a Standard Operating Procedure (SOP) for managing alerts.
- Outline the process for refining alerts based on SIEM monitoring.
- Develop an Incident Response Plan (IRP) to address true positive incidents.
- Set Service Level Agreements (SLAs) and Operational Level Agreements (OLAs) between teams for handling alerts and following the IRP.
- Implement and maintain an audit process for managing alerts and incident reporting by analysts.
- Create documentation to review the logging status of machines or systems, the bases for creating alerts and their triggering frequency.
- Establish a knowledge base document for essential information and updates to case management tools.

### Example 1 (Microsoft Build Engine Started By An Office Application)

![[us1.webp]]

Attackers [exploit MSBuild](https://blog.talosintelligence.com/building-bypass-with-msbuild/)'s ability to include malicious source code within its configuration or project file.
To address this risk, we create a detection use case in our SIEM solution that monitors instances of MSBuild initiated by Excel or Word, as this behavior could indicate a malicious script payload execution.
Next, let's define prioruty, impact and map the alert to the kill chain/MITRE framework.
This varies on the organization's specific context and landscape. For this one, it is in a high global risk category.
To define TTD and TTR, we need to focus on the rule's execution interval and the data ingestion pipeline. For this example, we set the rule to run every 5 minutes.
When creating an SOP or documenting alert handling, consider:
- process.name
- process.parent.name
- event.action
- machine where the alert was detected
- user associated with the machine
- user activity within +/- 2 days of the alert's generation
After this, engage with the user and examine the machine to analyze system logs, antivirus logs and proxy logs from the SIEM for full visibility.
For rule fine-tuning, while the Build Engine is common among Windows developers, its use by non-engineers isn't, so exclude legitimate parent process names from the rule.

### Example 2 (MSBuild Making Network Connections)

![[us2.png]]

This is focusing once again on the MsBuild.exe binary, but this time, a machine attempts outbound communication with a remote or potentially malicious IP address.
To address this risk, we need a monitoring solution capable of detecting instances where MSBuild is responsible for malicious outbound connections.
Unlike the previous example, this situation could occur whenever MsBuild.exe establishes an outbound connection. This could be a legitimate IP address, making it MEDIUM severity.
It also falls under the Execution (TA0002) tactic.
The IRP will differ when handling this specific type of alert. Defenders will focus on event.action, IP address ans reputation off the IP, among other factors.
