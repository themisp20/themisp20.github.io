---
title: "CWEE Review: Caught in the Web"
date: 2024-11-01T19:51:12Z
draft: false
# Originally published on Medium: https://medium.com/@theminiop/cwee-review-caught-in-the-web-3f5f4b07aa18
---

Hello and Welcome!

If you have landed here then you are probably interested in the CWEE certification, and I heartily salute that interest. It was this very interest which prompted me to complete this certification myself, and ultimately culminated to this very article.

The [Certified Web Exploitation Expert](https://academy.hackthebox.com/preview/certifications/htb-certified-web-exploitation-expert) certification is a new Advanced Web Certification by Hack The Box which greatly improves and demonstrates a holders competency in Web Exploitation. This review will mostly focus on my experience and impressions in achieving this certification. All specific certification details regarding the path modules/path cost/exam duration, etc. can be found through the official Hack the Box website:

Thanks for reading!

## My Background

At the time of writing I am an undergraduate student at the University of Edinburgh, with high hopes of pursuing a long and challenging career in offensive security. At the time of preparing and taking the exam, I had no professional experience in any technical cybersecurity role of any capacity. Rather, all my skills and proficiencies were built through CTFs, labs, certifications and personal projects. You can find out more regarding my specific background on my [LinkedIn](https://www.linkedin.com/in/themisp) profile (always welcome a connection request).

Moreover, before commencing on the CWEE path I had completed the CBBH and CPTS paths, as well as the CPTS certification exam. I believe these activities to be the most relevant to my background (within the context of CWEE) as they pertain to integral elements of the CWEE. The CBBH builds the technical and knowledge foundations for web hacking, which serve as prerequisites for the CWEE, while the CPTS exam familiarizes successful candidates with the longer marathon-like nature of Hack the Box certification exams, as well as enforcing strict reporting requirements.

Of course the aforementioned are not rigid requirements, but I do believe they armed me with the confidence necessary for tackling the CWEE certification.

## Preparing for the Exam

Hack the Box claims that the skills and knowledge amassed from the path alone is sufficient preparation for the exam. I can gladly confirm this statement while also providing two important conditions:

1\. **Work through the modules with the primary intention of comprehension, not completion**: Module practical exercises often don’t cover all the material described in the module. As such fragments may slip by and get forgotten. These very fragments may popup on the exam and cause disarray. Also, ensure you understand not only how to exploit something, but why is it exploitable in the first place.

2\. **Treat Skill Assessments as Mini-Exams**: The exam will present permutations and combinations of vulnerabilities discussed in the path. As such, most vulnerabilities will not be immediately obvious or exploitable. Skill assessments are perfect preparation for getting used to such scenarios. Therefore, if you get stuck, maintain composure and evaluate the material and your approach. Don’t be to quick to look and ask for hints through Discord, as getting unstuck on your own builds resilience and tenacity. If you have absolutely no ideas or path forward, then it is recommended to read through the module again.

Should you follow the above principles and successfully complete the path, then you are in great condition to attack the exam. I started the exam 3 or 4 days after completing the path.

## Exam Experience

The purpose of the exam is not to be as challenging as possible, but to serve as a realistic simulation of real world engagements which entail advanced web techniques. This idea is also reinforced by the duration of the exam (10 days) and the significant and strict reporting aspect. After all, the ultimate goal is to produce a document which delivers quality to the client.

10 days is a considerable amount of time and as such the exam should be treated as a marathon, not a sprint. A total of 6 flags are available. Similar to my CPTS exam, the first day came and went and bore no fruit. However, the next four days yielded steady progress with 1 flag per day. A minor hiccup occurred on day 6, but no worries, because day 7 delivered both of the remaining flags. I submitted the report sometime on the 8th day. Overall, the exam was hugely enjoyable and very rewarding.

I highly recommend that any exam goer completes the report as they progress through the environment. Resisting the urge to continue pwning after exploiting a vulnerability is tough, however having to retroactively collect screenshots and evidence will be a huge nuisance and likely lead to an inferior report. I would hack freely until I exploited something of note, upon which I would stop to document all steps up until that exploitation. Then I would proceed with a new cycle of enumeration.

## CWEE Value

This certification is quite costly, and as such, it is very important to consider why you want to pursue this cert.

My personal reasons for achieving this certification were mostly based on a desire to upskill and learn new advanced web hacking techniques. Naturally, there was an element of wanting to improve my resume, however, if appeasing the view of recruiters is your sole focus, the CWEE is not the best option yet.

However, the CWEE and CPTS were huge reasons why I felt so confident in technical interviews I had for junior penetration testing positions. Additionally, while the CWEE may evade the eye of an initial HR screening, an experienced security professional will likely pick it up, and it may prove to be a great talking point.

## Answering Common Questions

— — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — —

### **Q: How long to receive feedback?**

I received my feedback a short **2 days** after submission. I suspect this response time may not be so prompt in the future, as the certification gains traction and exam attempts/submissions rise. I believe Hack the Box officially states **1–20 days** as the official response time window.

— — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — —

### Q: How did you prepare for code review?

*TL;DR CWEE Path supplemented with HTB Web CTFs*

Approximately **50%** of the path and exam is White-Box. Therefore it is imperative to be comfortable reading and understanding reasonably sized codebases. The path does a great job in developing this skill, as numerous of the later modules feature larger sized code bases. The mindset I had when going through these modules was to try understand the code base as a whole, rather than look for specific vulnerabilities. Shallow scans may land you a SQL injection, but a race condition or logic error only reveals itself through slow methodical analysis.

To further alleviate the intimidation of delving into larger codebases, I did a few medium/hard web CTF challenges on the HTB platform, where I once again really focused on the source as a whole.

— — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — —

### Q: Where can I find HTB template for the report?

PDF: <https://docs.sysreptor.com/assets/reports/HTB-CWEE-Report.pdf>

- — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — —

### Q: Did you use skeleton scripts for your exploits?

The path introduces you to numerous custom exploit scripts. These scripts are designed to be easily adaptable and can be swiftly amended to attack other endpoints. Therefore, you can easily save and use such scripts as skeleton scripts.

Going even further, I wrote my own version for most custom scripts I encountered in the path. This improved my understanding of the vulnerability in question, allowed me to customize the scripts to my liking, and massively improved my comfort in writing custom scripts.

If you follow a similar strategy, by the end of the path you will be well equipped with an arsenal of adaptable scripts, ready to exploit any vulnerable endpoints you discover in the exam. You would also be very capable of modifying existing scripts to match your findings.

Please don’t stress too much about having a skeleton script ready for every type of vulnerability as you have plenty of time in your exam.

- — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — —

### Q: For the vulnerability fix in the report, how do they expect you to correct the vulnerability?

*TL;DR Black-Box -\> Suggestions \| White-Box -\> Concrete Patches*

You are expected to provide remediation steps for all vulnerabilities found and described in your report. However, the remediation profile for vulnerabilities differs depending on the style of the engagement.

In a black-box engagement, you need to outline the vulnerability and how you found it, explain the likely cause, and describe potential fixes. You can theorize regarding the backend code, and provide concrete suggestions, however you’re not expected to write patches and/or fix code.

For vulnerabilities found in a white-box engagement, you must identify the vulnerable code snippet, explain why its vulnerable, provide a concrete and safe patch for the identified code snippet, and explain why the patch fixes the vulnerability.

- — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — —

### Q: What programming languages do you recommend to be proficient at for the exam?

*TL;DR Be comfortable writing python exploit scripts + be comfortable reading all languages in the path*

Python is my go-to language for writing custom exploit scripts. I highly recommend being familiar with this language, especially the **requests/threads** library packages. Of course some client side exploits will need to be written in **JavaScript**, however these are often simpler exploits, and can be copied and adjusted from the path material, or snipped together using ChatGPT.

From a code review standpoint, ensure that you are familiar with all the languages in CWEE path, as well as any frameworks. You don’t need to master these languages of course, but you should be comfortable reading them. A key strategy I implemented during the completion of the path was to keep track of language unique vulnerabilities. By doing so I developed methodologies for identifying them. For example: type juggling in PHP -\> searching for \`!=\` or \`==\` in source.

- — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — —

### Q: Do they want you to write exploit scripts for ALL of the exploits or just some?

*TL;DR Only ones that need custom scripts*

All scripts for vulnerabilities which warrant a custom script for successful exploitation must be included in your report. Such scripts should be functional and will be tested.

The CWEE path is a good guide for assessing if a given vulnerability requires a custom exploit script. If a custom script was used to exploit a given vulnerability in the path, then you should write an exploit script for that same/similar vulnerability in the exam.\
If its simpler, then just providing the payload in your report will be sufficient.

- — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — —

### Q: How long did it take you to go through the path?

It took around **30–40 days** to get through all the path modules which is slightly above the estimation made by Hack the Box **(27 days)**.

- — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — —

### Q: How many hours per day where you actively working on your CWEE exam?

Around **8–10** hours. I submitted the report on the 8th day.

- — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — —

### Q: Did you have any prior programming experience?

Yes, a lot of my coursework in university is quite programming heavy. I had ample exposure to Java and Python and a good grasp of Object Oriented Programming before starting the path.

- — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — -
