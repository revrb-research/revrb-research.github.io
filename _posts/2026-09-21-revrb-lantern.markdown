---
layout: post
title:  "REVRB-LANTERN: Security Research on Lantronix Autonomous Out-of-Band Devices"
date:   2026-09-21 08:28:19 -0500
tags: security lantronix cve ot
---

RE/VRb LLC conducts independent research to improve cyber infrastructure security. The following report details research conducted without contract or bounty, representing over four months of discovery and coordination by a single researcher. You can support our ongoing efforts [here](https://ko-fi.com/revrb).

**Table of Contents:**

* TOC
{:toc}

## Summary

RE/VRb discovered and coordinated fourteen vulnerabilities in Lantronix Autonomous Out-of-Band Devices, including chains that can lead to unauthenticated remote code execution. These devices appear to be prevalent in data centers, telecom networks, and in state and local services, among other markets. By design, these devices may introduce a secondary path into a user network for managing network or other serial-managed devices in case of a primary network failure. In typical deployments, these devices sit alongside management-layer infrastructure and may hold credentials and/or provide control over devices and physical systems that communicate over serial.

Per Lantronix, the reported CVEs are addressed in recent patches (v9.7.0.5 for the SLC8000 and v9.7.0.1 for the EMG series), but per our research into v9.7.0.5 for the SLC8000, REVRB-LANTERN-13 (CVE-2026-80154) remains unremediated.

The following devices were confirmed to be vulnerable to one or more of the findings detailed below:

- Lantronix SLC8000
- Lantronix EMG8500
- Lantronix EMG7500
- Lantronix SLB882
- Lantronix SLCx-03
- Lantronix SLCx-02

Additionally, we asked Lantronix about the applicability of our findings to the newly released SLC9000, which we assessed may share portions of the same codebase as the devices listed above. Despite multiple requests, Lantronix did not confirm or deny our findings against the SLC9000, and asked that we "refrain from mentioning it in [our] publication." Our findings concerning the SLC9000 can be found below. If you own an SLC9000 and would like to coordinate further research, please reach out to research@revrb.net.

We recommend SLC9000 owners review the 18 September 2026 firmware update, which Lantronix published citing security vulnerabilities, reach out to Lantronix support with any questions about said vulnerabilities, and apply where appropriate.

SLC8000 owners and owners of EMG-series devices are recommended to patch their devices as soon as possible to obtain available remediations and to adopt/continue defense-in-depth and strong network monitoring practices to respond to yet-unknown vulnerabilities.

Owners of SLB-series devices are recommended to disconnect or layer defenses over their SLBs and to reach out to Lantronix concerning patch availability; Lantronix's product discontinuation notice indicates warranty and software support through 31 December 2028[^slb-eol], but no patch appears to have been made available.

Owners of SLCx-02 and SLCx-03 devices are recommended to decommission these devices, as they are end-of-support and no patch is expected to be made available.

[^slb-eol]: Lantronix, [*Product Discontinuation Notice - December 18, 2023*](https://cdn.lantronix.com/wp-content/uploads/pdf/PCN-881-SLB-PRODUCT-FAMILY-DISCONTINUATION-NOTICE.pdf)


## Device Market

The following market and deployment information is aggregated from publicly available sources external to RE/VRb. RE/VRb makes no claim that any named or unnamed organization is currently a Lantronix customer, currently operates a product affected by the vulnerabilities described in this report, or is inherently or immediately vulnerable due to this disclosure. Where a source attributes a deployment to a named organization, the attribution is the source's; RE/VRb has not independently verified any deployment described here. This information is provided solely to demonstrate the scale and reach of the mentioned devices and the importance of this research.

**AI and High-Performance Computing:** SambaNova documentation explicitly identifies the SLC8000 as the serial console server in its SambaRack SN40L-16 infrastructure,[^sambanova-slc8000] while a recent Lantronix earnings call states that Lantronix won a SambaNova design for the newer SLC9000.[^sambanova-slc9000] SambaNova has separately documented DataScale, SambaNova Suite, or SN40L infrastructure deployments at multiple US Department of Energy National Laboratories,[^usdoe-argonne][^usdoe-oakridge][^usdoe-losalamos][^usdoe-lawrencelivermore] US and international high-performance computing (HPC) centers,[^hpc-tacc][^hpc-riken] and to power international sovereign AI providers.[^sambanova-sovai]

**Government and Public Safety Infrastructure:** The communications-equipment inventory published with the US Defense Information Systems Agency's (DISA) Capacity Services Communications III (CSC III) indefinite-delivery/indefinite-quantity solicitation lists multiple SLC8000 models alongside other enterprise equipment.[^disa-csciii] CSC III is intended to provide scalable communications infrastructure services throughout DISA and other approved locations across the DoD worldwide. During a 2019 earnings call, while discussing the SLC8000, Lantronix reported strong demand for its out-of-band management devices from customers including the Swedish Defence Administration.[^ltrx-2019-q2] The SLC8000 also appears as a component of Motorola Solutions ASTRO 25 public-safety radio infrastructure.[^moto-contract] We were able to identify multiple US state and city radio acquisition contracts that itemized the SLC8000 or its expansion modules.

**Telecommunications, Data Centers, and Enterprise IT:** A 2018 use-case describes an unnamed "telecom giant" deploying SLC8000s across geographically distributed data centers.[^telecom-giant] Similarly, a 2016 use-case describes an unnamed "leading nationwide provider of cable television" deploying SLC8000s across hundreds of data centers or points-of-presence.[^cable-giant] A 2016 Lantronix SLC8000 battle card states that the SLC8000 is "Used by Avaya, Cisco, Brocade, NetApp, T-Mobile and others in their development labs and data centers."[^battle-card] Similarly, on a slide discussing share gain for the SLC8000, a 2017 Lantronix investor presentation lists NetApp, F5, Hewlett Packard Enterprise, Avaya, Brocade, Samsung, T-Mobile, Symantec, Yahoo!, PwC, and Vodafone as "Select Customers"[^select-customers]

**EMG and SLB Deployments:** A 2023 Lantronix case study indicates that the EMG8500 is used by the University of Cambridge.[^u-cambridge] Lantronix earnings calls explicitly describe a large Verizon SLB rollout and continued purchases[^ltrx-2016-q1], and separately describe an SLB rollout through a solution provider that had won a contract with the Four Seasons Hotel Group.[^ltrx-2019-q2] Finally, a 2014 Lantronix publication states that an SLB was deployed at the Aloha Cabled Observatory, approximately three miles below the sea surface.[^aloha-observatory]

[^sambanova-slc8000]: SambaNova, [*Third-party Components*](https://docs.sambanova.ai/docs/en/sambastack/resources/thirdparty)
[^sambanova-slc9000]: Roic AI, [*Lantronix, Inc. (LTRX) Q4 FY2026 Earnings Call Transcript - August 26, 2026*](https://www.roic.ai/quote/LTRX/transcripts/2026-year/4-quarter)
[^usdoe-argonne]: SambaNova, [*Argonne National Laboratory Deploys SambaNova Suite to Advance AI Inference In Science Research*](https://sambanova.ai/blog/argonne-national-laboratory-enhances-ai-testbed-for-scientific-research)
[^usdoe-oakridge]: SambaNova, [*Oak Ridge National Laboratory Deploys SambaNova Suite, Enabling Energy-Efficient AI Inference for Science*](https://sambanova.ai/blog/oak-ridge-national-laboratory-deploy-sambanova-suite-for-ai-for-science)
[^usdoe-losalamos]: SambaNova, [*Los Alamos National Laboratory expands partnership with SambaNova*](https://sambanova.ai/blog/los-alamos-national-laboratory-expands-partnership-with-sambanova)
[^usdoe-lawrencelivermore]: SambaNova, [*SambaNova and Lawrence Livermore National Laboratory Scale Up Collaboration to Accelerate AI for Science*](https://sambanova.ai/press/sambanova-and-lawrence-livermore-national-laboratory-scale-up-collaboration-to-accelerate-ai-for-science)
[^hpc-tacc]: SambaNova, [*Texas Advanced Computing Center Deploys SambaNova Suite, Enabling AI Inference for Science*](https://sambanova.ai/blog/tacc-deploys-sambanova-suite-ai-inference-for-scientific-research)
[^hpc-riken]: SambaNova, [*SambaNova to provide SambaNova DataScale to RIKEN*](https://sambanova.ai/ja/press/sambanova-systems-to-deliver-sambanova-datascale-to-riken)
[^sambanova-sovai]: SambaNova, [*SambaNova Powers the AI Backbone for Three Sovereign AI Providers Across Australia, Europe and the UK*](https://sambanova.ai/press/sambanova-powers-the-ai-backbone-for-three-sovereign-ai-providers-across-australia-europe-and-the-u.k)

[^disa-csciii]: DISA CSC III, [*PWS Appendix 1 - Comm Equipment Inventory.xlsx* (GovTribe mirror)](https://govtribe.com/file/government-file/pws-appendix-1-comm-equipment-inventory-dot-xlsx)
[^moto-contract]: Commonwealth of Pennsylvania / Motorola Solutions, [ASTRO contract pricing and equipment descriptions](https://www.emarketplace.state.pa.us/FileDownload.aspx?file=4400027237%5CChangeNotice.pdf)

[^telecom-giant]: Lantronix, [*Enabling Robust Data Center Infrastructure Access & Management for Telecom Giant*](https://www.lantronix.com/resources/application-spotlights/enabling-robust-data-center-infrastructure-access-management-for-telecom-giant/)
[^cable-giant]: Lantronix, [*Ensuring the Highest Levels of Reliability and Service for Cable and Video Streaming*](https://www.lantronix.com/resources/application-spotlights/ensuring-highest-levels-reliability-service-cable-video-streaming-band-management/)
[^battle-card]: Lantronix, [*Why the SLC 8000 is the Only Advanced Modular Console Manager for Enterprise*](https://www.lantronix.com/wp-content/uploads/pdf/SLC8000_BattleCard_Final_021216-1.pdf)
[^select-customers]: Lantronix, [*Investor Presentation - 19th Annual Needham Growth Conference*](https://www.lantronix.com/wp-content/uploads/pdf/LTRX-Q2-Prelim-Needham-IR-Preso-Jan-2017.pdf)

[^u-cambridge]: Lantronix, [*Lantronix Out-of-Band Solutions Power University of Cambridge*](https://cdn.lantronix.com/wp-content/uploads/pdf/Univ.-of-Cambridge-CS_FNL_no-bleed.pdf)
[^aloha-observatory]: Lantronix, [*Lantronix Highlights Out of Band Management Solutions at Data Center World*](https://www.lantronix.com/newsroom/press-releases/lantronix-highlights-management-solutions-data-center-world/)

[^ltrx-2016-q1]: Roic AI, [*Lantronix, Inc. (LTRX) Q1 FY2016 Earnings Call Transcript*](https://www.roic.ai/quote/LTRX/transcripts/2016-year/1-quarter)
[^ltrx-2019-q2]: Roic AI, [*Lantronix, Inc. (LTRX) Q2 FY2019 Earnings Call Transcript*](https://www.roic.ai/quote/LTRX/transcripts/2019-year/2-quarter)


## Findings

**Descriptions:**

Internal Marking | Associated CVE | CVE Description | Recommended CVSS3.1 
-|-|-|-
REVRB-LANTERN-01 | CVE-2026-80143 | An attacker that can authenticate as any user to the terminal/CLI of Lantronix Autonomous Out-of-Band devices can execute shell commands as root. This can cause complete loss of confidentiality, integrity, and availability for the affected device with the potential to impact downstream serial-attached devices. | 9.9/Critical
REVRB-LANTERN-02 | CVE-2026-80144 | An attacker that can authenticate as any user to the terminal/CLI of Lantronix Autonomous Out-of-Band devices can execute shell commands as root. This can cause complete loss of confidentiality, integrity, and availability for the affected device with the potential to impact downstream serial-attached devices. | 9.9/Critical
REVRB-LANTERN-03 | CVE-2026-80145 | An attacker that can authenticate as any user with the ‘services’ permission to the terminal/CLI of Lantronix Autonomous Out-of-Band devices can execute shell commands as root. This can cause complete loss of confidentiality, integrity, and availability for the affected device with the potential to impact downstream serial-attached devices. | 9.1/Critical
REVRB-LANTERN-04 | CVE-2026-80146 | An attacker that can authenticate as any user to the terminal/CLI of Lantronix Autonomous Out-of-Band devices can cause a stack-based buffer overflow which may lead to code execution. This can cause a complete loss of confidentiality, integrity, and availability for the affected device with the potential to impact downstream serial-attached devices. | 9.9/Critical
REVRB-LANTERN-05 | CVE-2026-80147 | An attacker that can authenticate as any user to the terminal/CLI of Lantronix Autonomous Out-of-Band devices can cause a stack-based buffer overflow which may lead to code execution. This can cause a complete loss of confidentiality, integrity, and availability for the affected device with the potential to impact downstream serial-attached devices. | 9.9/Critical
REVRB-LANTERN-06 | CVE-2026-80148 | An unauthenticated attacker that can access the WebSSH/WebTelnet listener on Lantronix Autonomous Out-of-Band devices can force a server-side request forgery that causes the affected device to create SSH connections to attacker-defined endpoints. An attacker could use this capability to enumerate and/or communicate with endpoints they otherwise would not have access to. | 8.6/High
REVRB-LANTERN-07 | CVE-2026-80149 | An unauthenticated attacker that can access the WebSSH/WebTelnet listener on Lantronix Autonomous Out-of-Band devices can force a server-side request forgery that causes the affected device to create SSH connections to attacker-defined endpoints. An attacker could use this capability to enumerate and/or communicate with endpoints they otherwise would not have access to. | 8.6/High
REVRB-LANTERN-08 | CVE-2026-80150 | An unauthenticated attacker that can access the WebSSH/WebTelnet listener on Lantronix Autonomous Out-of-Band devices can force a server-side request forgery that causes the affected device to create telnet connections to attacker-defined endpoints. An attacker could use this capability to enumerate and/or communicate with endpoints they otherwise would not have access to. | 8.6/High
REVRB-LANTERN-09 | CVE-2018-16789 | An unauthenticated attacker that can access the WebSSH/WebTelnet listener on Lantronix Autonomous Out-of-Band devices can force the listener into an infinite loop using CVE-2018-16789, denying service to the endpoint. | 7.5/High
REVRB-LANTERN-10 | CVE-2026-80151 | An attacker that can authenticate as any user with the ‘services’ permission to the terminal/CLI of Lantronix Autonomous Out-of-Band devices can execute shell commands as root. This can cause complete loss of confidentiality, integrity, and availability for the affected device with the potential to impact downstream serial-attached devices. | 9.1/Critical
REVRB-LANTERN-11 | CVE-2026-80152 | An attacker that can authenticate as any user with the ‘services’ permission to the terminal/CLI of Lantronix Autonomous Out-of-Band devices can execute shell commands as root. This can cause complete loss of confidentiality, integrity, and availability for the affected device with the potential to impact downstream serial-attached devices. | 9.1/Critical
REVRB-LANTERN-12 | N/A | This represents an internal finding that was later discovered to be a false positive | N/A
REVRB-LANTERN-13 | CVE-2026-80154 | An unauthenticated attacker that can access the web management portal on Lantronix Autonomous Out-of-Band devices can derive session tokens of logged-in users and bypass validation of those tokens in order to elevate privileges. | 9.6/Critical
REVRB-LANTERN-14 | CVE-2026-80155 | An unauthenticated attacker that can access the web management portal on Lantronix Autonomous Out-of-Band devices can bypass authentication checks to pull key configuration files (such as usernames and hashed passwords) and upload files to key filesystem locations, leading to remote code execution and the ability to impact downstream serial-connected devices. | 10.0/Critical
REVRB-LANTERN-15 | CVE-2026-80156 | An attacker that can authenticate to the upload endpoint of the web management portal of Lantronix Autonomous Out-of-Band devices can write arbitrary data to any location on that device’s disk, leading to remote code execution and the ability to impact downstream serial-connected devices. | 9.1/Critical

**By Device/Firmware Applicability:**

Internal Marking | Associated CVE | SLC8000 | EMG8500/EMG7500 | SLB882 | SLCx-03 | SLCx-02
-|-|-|-|-|-|-|-
REVRB-LANTERN-01 | CVE-2026-80143 | <v9.7.0.2 | <v9.7.0.1 | All versions | All versions | All versions
REVRB-LANTERN-02 | CVE-2026-80144 | <v9.7.0.2 | <v9.7.0.1 | All versions | All versions | All versions
REVRB-LANTERN-03 | CVE-2026-80145 | <v9.7.0.2 | <v9.7.0.1 | All versions | All versions | All versions
REVRB-LANTERN-04 | CVE-2026-80146 | <v9.7.0.2 | <v9.7.0.1 | All versions | All versions | All versions
REVRB-LANTERN-05 | CVE-2026-80147 | <v9.7.0.2 | <v9.7.0.1 | All versions | All versions | All versions
REVRB-LANTERN-06 | CVE-2026-80148 | <v9.7.0.3 | <v9.7.0.1 | All versions | N/A | N/A
REVRB-LANTERN-07 | CVE-2026-80149 | <v9.7.0.3 | <v9.7.0.1 | All versions | N/A | N/A
REVRB-LANTERN-08 | CVE-2026-80150 | <v9.7.0.3 | <v9.7.0.1 | All versions | N/A | N/A
REVRB-LANTERN-09 | CVE-2018-16789 | <v9.7.0.3 | <v9.7.0.1 | All versions | N/A | N/A
REVRB-LANTERN-10 | CVE-2026-80151 | <v9.7.0.3 | <v9.7.0.1 | All versions | All versions | All versions
REVRB-LANTERN-11 | CVE-2026-80152 | <v9.7.0.3 | <v9.7.0.1 | All versions | All versions | All versions
REVRB-LANTERN-13 | CVE-2026-80154 | All versions | All versions | All versions | All versions | All versions
REVRB-LANTERN-14 | CVE-2026-80155 | <v9.7.0.5 | <v9.7.0.1 | All versions | All versions | All versions
REVRB-LANTERN-15 | CVE-2026-80156 | <v9.7.0.5 | <v9.7.0.1 | All versions | All versions | All versions

### Technical Details

#### Web Management Portal

**REVRB-LANTERN-13:** Lantronix Autonomous Out-of-Band devices create their session token from the device model and the current time at a resolution of one second and automatically expire them after 15 minutes. For the span of time a session token could possibly be valid, there are only 900 possible session tokens, with some minor variation based on how often the cookie expiration script runs. An attacker can retrieve a current unauthenticated session token from `login.htm`, deriving the device model and time to generate all 900 possible active tokens.

An attacker must then overcome the source IP and User-Agent validation stored in `/tmp/.save/cookies/sessions.txt`. In certain areas of the web server's path handling, file extension checks are performed that allow for bypassing this check. An attacker can create a crafted URI that abuses these checks to allow the use of a stolen session token.

**REVRB-LANTERN-14:** The web configuration server, when validating the session token for upload endpoint, performs the following steps:

* Splits the cookie by '=' and parses the time from the second half, only checking that it's newer than the boot time of the device.
* Checks that the cookie exists as a file in `/tmp/.save/cookies/`; i.e.: `/tmp/.save/cookies/session=S0F00280d48546-38845`
* Checks that the contents of that file don't begin with "COOKIE_USER"
* Loads the username and permissions from the cookie file

An attacker can bypass the first check simply by creating a cookie with the correct timestamp - as mentioned before, the cookies are derived from the device model and a timestamp.

For check number 2, an attacker can abuse a `snprintf` call used to build the filepath to check - `snprintf(filename, 129, "%s/%s", "/tmp/.save/cookies", cookie)`. By providing a cookie of a specific length, said attacker can cause the filepath to truncate on the required `=` and leverage path traversal to identify an arbitrary file on disk for checks 3 and 4.

An attacker can use these to point the server to `/etc/.lusers` - a file on-device that stores the local users and starts with `sysadmin` - in order to bypass checks 3 and 4.

The final cookie ends up looking something _like_ `Cookie: ../../..///////////////////////////////////////////////////////////////////////////////////////////etc/.lusers=S0F00280d48546-38845` - everything past the '=' passing cookie format and timestamp checks, and everything prior becoming truncated to `../../..///////////////////////////////////////////////////////////////////////////////////////////etc/.lusers` as the file to use to check the validity of the cookie.

This allows an attacker to upload to and/or pull from critical file locations, such as key stores.

**REVRB-LANTERN-15:** When validating the upload filename to avoid pathing characters, the web management portal first checks for `\` and strips them. If `\` is discovered, however, then checks for `/` never occur, allowing an attacker to upload a filename like `pre\../../../bin/busybox`, allowing arbitrary data to be uploaded anywhere on disk instead of the intended controlled locations.

#### WebShell

Lantronix Autonomous Out-of-Band devices use a custom `shellinaboxd` to provide terminal access across the web browser. SLCx-02 and SLCx-03 devices do not have this feature, and are therefore not vulnerable to the following.

**REVRB-LANTERN-09:** The custom `shellinaboxd` in-use is vulnerable to CVE-2018-16789, which allows an attacker to send a malformed multipart-form request that sends the service into an infinite loop.

**REVRB-LANTERN-06:** The custom shellinaboxd builds its connection target using user input for the username passed to a `snprintf` call: `snprintf((char *)&host,0x200,"%s@%s",input_buf,this_device_ip)`.  Because the `@<device_ip>` comes _after_ user input, and because user input is unbounded, an attacker can pass in an overly-long connection string and truncate the device IP entirely. By padding an IP with 0s, an attacker can point this connection string to any IP (though they will have to perform octal conversions first, since 0-prefixed numbers are interpreted as octal on the SLC and EMG devices.) i.e.: `root@000...00012.0.0.1` allows an attacker to point the resulting SSH connection to `root@10.0.0.1`.

**REVRB-LANTERN-07, 08:** When building the terminal connection for the user, the custom `shellinaboxd` uses the `rooturl` parameter provided by the web connection to determine its own IP address. An attacker can modify this parameter to cause the terminal connection to be made to an arbitrary host or IP.  This is reported as two separate findings, one for the SSH section of the codebase and one for the Telnet portion.

#### CLI

**REVRB-LANTERN-01, 02, 04, 05:** An undocumented set of commands exists in the `cli` management binary. Among them are `mfc eeprom read` and `mfc eeprom write`, which both pass unbounded/unsanitized user input into a bounded stack buffer, and then to a call to `system`.

**REVRB-LANTERN-03:** `set cifs password` passes unsanitized user input into a call to `system`.

**REVRB-LANTERN-10:** `set nfs download` passes unsanitized user input into a call to `system`.

**REVRB-LANTERN-11:** `set script schedule` passes unsanitized user input into a call to `system`.


## Footnote Concerning the SLC9000

It is our belief that the SLC9000 may be vulnerable to the web management portal vulnerabilities disclosed here, but we are resource-constrained from validating them. 

We first asked Lantronix about shared code concerning the SLC9000 in June. Our initial inquiry was based on the SLC9000 User Guide which demonstrated identical CLI commands to the SLC8000, and even retained the PDF meta title "SLC8000 Advanced Console Server User Guide".[^slc9k-userguide] Lantronix's response was to remove any mention of the SLC9000 from our coordination document, stating that the SLC9000 was "not yet in customer hands". 

In August, after they had published firmware version 9.7.0.1 for the SLC9000, we disclosed REVRB-LANTERN-13, 14, and 15 to them, and asked again how we should handle these findings in relation to the SLC9000. Instead of answering, Lantronix stated they had "serious concerns" about our assertions of shared code from June, prior to the device's release, calling it "a bit disturbing". After we assured them that we did not access anything non-public, and explained our reasoning for expecting the web server to be the same between them, Lantronix's response was, "Consistent user experience does not imply shared code base". 

It was also around this time that they responded to our notice of a firm disclosure date with "The disclosure shall be limited to SLC8000, EMG8500, and EMG7500". We followed up and asked whether they meant their own disclosure or ours, to which they responded, "We believe your publication should be limited to SLC8000, EMG8500, and EMG7500."

In early September, Shodan scanned an SLC9000, providing the HTML response of its login page. The following is a diff between the login pages of the SLC8000 and SLC9000.

```diff
--- a/slc8000.html
+++ b/slc9000.html
@@ -1,105 +1,105 @@
 <!DOCTYPE html PUBLIC "-//W3C//Dtd html 4.0 transitional//EN">
 <htmL>
 <head>
-<title>Lantronix SLC 8016</title>
+<title>Lantronix SLC9016</title>
 
 <meta http-equiv=Content-Type content="text/html; charset=iso-8859-1">
 
 <link href="images/style.css" type=text/css rel=stylesheet>
 
 <style>
 <!--
 html { visibility: hidden }
 -->
 </style>
 
 </head>
 
 <script>
 <!--
 
 function applyForm(id)
 {
   var login = document.getElementById('text_login');
   var pass = document.getElementById('text_pass');
   if (login.value == "")
   {
     alert("Login: required field.");
     login.focus();
     return false;
   }
   if (pass.value == "")
   {
     alert("Password: required field.");
     pass.focus();
     return false;
   }
   document.theform.submit();
   return true;
 } // applyForm
 
 window.onload = function() {
   var login = document.getElementById('text_login');
   if (self == top) {
     document.documentElement.style.visibility = 'visible';
   } else {
     top.location = self.location;
   }
   login.focus();
 }
 
 -->
 </script>
 
 <body class="auth" leftMargin=0 topMargin=0 marginwidth=0 marginheight=0>
 
 <div class=authTopbar>
   <div class=logo>
     <a href="http://www.lantronix.com"><img src="images/ltrx_logo_new.gif" border=0></a>
   </div>
-  <div class=product>SLC 8016</div>
+  <div class=product>SLC9016</div>
 </div>
 <div class=authLine></div>
 
 <form name=theform method=post autocomplete="off" onSubmit="return applyForm()">
 
 <div style="height: 350px" align=center>
   <div></div>
   <div class=loginBanner>
 Welcome to the SLC
       <br>&nbsp;
   </div>
-  <div class=loginTitle>Login to SLC 8016</div>
+  <div class=loginTitle>Login to SLC9016</div>
   <div class=authPortletGroup>
     <div class=portletContent style="background-color: #f2f2f2">
       <table width=370 border=0>
         <tr>
           <td class=authPadding align=center colspan=2>
           </td>
         </tr>
         <tr>
           <td class="fieldName authPadding">Login:</td>
-<td class="fieldValue authPadding"><input class="fieldValue" type=text name=slcloginS0E20260c28116-29547 id="text_login" size=32 maxlength=32 value=""></td>
+<td class="fieldValue authPadding"><input class="fieldValue" type=text name=slcloginS0S00261q59716-69109 id="text_login" size=32 maxlength=32 value=""></td>
         </tr>
         <tr>
           <td class="fieldName authPadding">Password:</td>
-<td class="fieldValue authPadding"><input class="fieldValue" type=password name=slcpasswordS0E20260c28116-29547 id="text_pass" size=32 maxlength=64 value=""></td>
+<td class="fieldValue authPadding"><input class="fieldValue" type=password name=slcpasswordS0S00261q59716-69109 id="text_pass" size=32 maxlength=64 value=""></td>
         </tr>
         <tr>
           <td class="authPadding" align=center colspan=2>
             <input class=pushButton type=submit value="Login" onClick="return applyForm()">
           </td>
         </tr>
       </table>
     </div>
   </div>
   <div>&nbsp;<br>&nbsp;</div>
 
   <div class="authLine authFooter">
-  &copy; 2003-2023 Lantronix, Inc.
+  &copy; 2003-2026 Lantronix, Inc.
   </div>
 </div>
 
 </form>
 </body>
 </html>
```

The `name` properties of the changed input tags, in our research, implicate portions of code responsible for REVRB-LANTERN-13 and 14. Using code from the SLC8000, we were able to reverse the transposition on the 'name' property of the SLC8000's homepage to "S??8016-092126022547" - this maps to \<model\>-\<timestamp:MMddyyhhmmss\> (though the second and third characters of the model get overwritten). Using the same code on the 'name' property for the SLC9000, we obtain "S??9016-090726165109", indicating that code responsible for this dynamic, authentication-tied server response is consistent with shared code between the SLC8000 and SLC9000.

On 18 September, Lantronix published firmware updates for the EMG7500, EMG8500, SLC8000, and SLC9000, each citing "security vulnerabilities" in their respective release notes. We do not claim to know what vulnerabilities were patched on the SLC9000.

If you own an SLC9000 and would like to coordinate further research, please reach out to research@revrb.net.

[^slc9k-userguide]: Lantronix, [*SLC 9000 Advanced Console Server User Guide*](https://cdn.lantronix.com/wp-content/uploads/pdf/PMD-00347A-SLC9K-UG-release.pdf)


## GNU General Public License (GPL) Code

We at RE/VRb believe that code obtained, modified, and/or monetized under GNU General Public Licensing makes the world go 'round.[^linux-share] As a matter of practice, we ask for GPL-covered code whenever we identify it in an investigation, and we aim to share it.

During this investigation, we asked Lantronix multiple times, as far back as April, for code covered by the GPLv2. Despite frequent communication on other topics, Lantronix did not answer for the location or availability of source code for GPL-covered binaries until late August, when we received this response: "Why is this needed?"

We at RE/VRb make no claims about Lantronix's legal compliance with any version of the GPL - only that we identified GPL-covered code on devices we obtained for this investigation, that we asked repeatedly for the corresponding source code or a path to obtain it, and that we have received none - and therefore have none to share.

[^linux-share]: w3techs.com, [*Usage Statistics and Market Share of Linux for Websites, September 2026*](https://w3techs.com/technologies/details/os-linux)


## Future Work

While RE/VRb stands by this report and the work that generated it, we don't believe this report to be a comprehensive audit of these devices. The code patterns that spawned these findings are still prevalent elsewhere - below is a table showing the number of potentially unsafe libc calls in the newest version of the SLC8000's web management binary, per Ghidra's automated analysis and based largely on GitHub's banned.h:

libc Call | Number of Cross-References
-|-
`strcpy` | 499
`strcat` | 372
`strncpy` | 37
`strncat` | 2
`strtok` | 120
`strtok_r` | 8
`sprintf` | 433
`system` | 146
`popen` | 91

These devices are still deeply interesting targets for security research.


## Corrections and Coordination
{:.no_toc}

We can be reached concerning corrections to this advisory at research@revrb.net.

## Citations
{:.no_toc}