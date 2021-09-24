# Unified_Org_Migration_Pipeline_Hackathon
Unified_Org_Migration_Pipeline_Hackathon

**Why? **

currently, Org migration is getting challenging with many pre-checks & Magnitude of services getting onboarding (CDI & CDI-E & Dq ) & manual intervention for every step by Operations Team. From starting a migration to debug migration errors. The unified p2pms pipeline will bring everything on the same page and will address this issue. This pipeline will make feel operations team, Platform QA & External QA seamlessly run the migration of the org from one pod to another. Every migration we have a downtime setup and once we see failures we need to ask GCS to talk to the customer and get the window extended. But with a unified pipeline and process in place and we can estimate and reduce the downtime as the pipeline helps narrow down issues with a proper error message. Adding to it, currently, the Operations team uses an ops-CLI tool which is an outdated way to migrate an org. This unified pipeline will leverage seamless integration with current ops operation cycles. 

**Issues during Migration**: Accommodating org on Destination pod issues Service onboarded and enabled differ from pods to pods POD sync Issues (Missing connectors and packages issues on destination pod compared to source pod) 

**How Does a unified pipeline address the above issues?** Reduces confusion for the Operation Team, Platform QA and External QA Perform Org Eligibility criteria to accommodate Org on destination pod By Comparing the registered services on both pods before any migration initiation Harvest Migration status on the same unified pipeline Instigate troubleshoot steps in the pipeline for failing migrations - with error details ( To route to respective service owners) 

**Time saved**: More than 6 hours to Operations teams for a single org migration No Need to debug Platform QA & External QA for migration failures, As a unified pipeline takes care of it. 

**Direct Benefiter**: Operations Team Platform QA External QA 

**Indirect Benefiter**: R&D Team ( Can skip joining calls until a real exception returned from this unified pipeline) Customer downtime can be reduced with this unified pipeline

**Tech used**: node js Newman Axios js fs js Jenkins Groovy scripting
