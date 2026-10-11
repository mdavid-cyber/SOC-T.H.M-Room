SOC Simulator: Introduction to Phishing
Platform: TryHackMe SOC Simulator 
SIEM: Splunk 
Alert: #8814 
Outcome: True Positive, escalated
Overview
I worked this scenario as a Tier 1 SOC analyst. I picked up alert #8814, "Inbound Email Containing Suspicious External Link" (Medium severity, Phishing), investigated it in Splunk, classified it as a true positive, and submitted an incident report with escalation.
The Alert
 
 
Screenshot -Alert 8814




Field	Value
Alert ID	8814
Raised	4 Oct 2026, 18:08
Sender	onboarding@hrconnex.thm
Recipient	j.garcia@thetrydaily.thm
Subject	Action Required: Finalize Your Onboarding Profile
Attachment	None
Link	https://hrconnex.thm/onboarding/15400654060/j.garcia

The email tells the recipient to click a link to "finalise" her onboarding. The alert description told me to check firewall or proxy logs for any attempt to reach the URL.
Investigation
Search 1: the sender address
 Screenshot- Splunk search Email 
I searched Splunk for the sender address. It returned two events, both copies of the phishing email (datasource: email). No firewall or proxy events, because those logs record connections to a URL or domain, not email addresses.

Search 2: the domain
  Screenshot- Splunk search log/Domain   
I widened the search to the domain hrconnex.thm. It returned three events:



Time	Event
18:06:16	Phishing email delivered: onboarding@hrconnex.thm to j.garcia@thetrydaily.thm
18:07:49	Internal email: h.harris@thetrydaily.thm to j.carter@thetrydaily.thm, subject "Unable to Receive Onboarding Email from hrconnex.thm". It says Garcia has not received her onboarding email, calls hrconnex.thm "our new third-party HR partner" and says the address is correct.
18:12:15	The same phishing email delivered to j.garcia again
No proxy or firewall events for the domain appeared in the 6-hour window I searched, so these logs show no evidence that the link was clicked.
Findings
1.	Domain mismatch. The sender domain (hrconnex.thm) does not match the recipient's company domain (thetrydaily.thm). On its own this is not conclusive, since third-party HR vendors are common, but it is a flag.
2.	Timeline contradiction. The internal email at 18:07:49 says the onboarding email never arrived. It had been delivered at 18:06:16, 1 minute 33 seconds earlier.
3.	Illogical request. Garcia already has an active company mailbox, so an external "finalise your access" request does not fit an employee who is already onboarded.
4.	Domain vouching. The internal email endorses the suspicious domain just before the same phishing email is sent again. This could point to a spoofed or compromised internal account. I could not confirm that from the logs, so it needs investigating.
5.	Repeat delivery. The phishing email arrived twice within about six minutes, which suggests a sequence of messages rather than a single attempt.
The TryDetectThis URL check returned "clean". A newly registered phishing domain with no reports would look the same, so I based the classification on behavioural and contextual evidence rather than the reputation result.




Classification
  Screenshot- Conclusion Positive  
True Positive, based on findings 1 to 5 above.

Case Report
  Screenshot- Incident report 1,2,3    








Indicators of compromise
Type	Value
Sender domain	hrconnex.thm
Sender address	onboarding@hrconnex.thm
URL	https://hrconnex.thm/onboarding/15400654060/j.garcia
Behaviour	Internal "non-delivery" email sent after the email was already delivered


MITRE ATT&CK: T1566.002, Phishing: Spear phishing Link

Recommended actions
•	Block hrconnex.thm at the email gateway and firewall, and add the URL to the blocklist
•	Tell Ms. Garcia not to click the link, and check proxy/firewall logs for any access attempt
•	Investigate whether h.harris's account has been compromised
•	Warn other staff about the campaign
Escalation: Yes. This is a confirmed phishing attempt against an active employee, part of a multi-email sequence.

Key Takeaways:
Triage sets the order, not the verdict. Sorting alerts by severity tells me what to look at first, but this alert was only Medium and still turned out to be a real phishing attack, so lower-severity alerts still need a proper look.
Search broader when the first search comes up empty. Searching the sender address only returned copies of the email. Searching the domain pulled in the internal email that cracked the case.
Evidence beats a "clean" verdict.  The URL check came back clean, but the domain mismatch and the timestamp contradiction were enough to classify it as malicious. A tool saying "clean" only means nothing has been reported yet.

