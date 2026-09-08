# WTIR: Twelve Weeks with a Honeynet — A Retrospective

**Series:** WTIR-01 through WTIR-12 (May 29 to September 7, 2026)
**Author:** Michael Mensah

---

## Why I Started This

Over twelve weeks, I ran a live, internet-facing honeynet as a one-person threat intelligence function. I published a structured report every week, caught and corrected my own data-integrity bugs in public, tracked the same infrastructure across months, and closed the series by catching real malware and getting it independently confirmed by an outside research platform. That's the short version. Here's how it actually went.

I started this project because I kept getting rejected from Threat Intelligence roles and interviews for the same reason: lack of experience. I got the idea from my internship at Wärtsilä North America, where I worked directly with the Threat Intelligence team, assisting with the weekly Threat Intelligence Newsletter alongside Juha Luukkanen, keeping employees aware of the latest cyber risks, phishing campaigns, and global security trends. At some point the team mentioned that building out their own honeynet was on their roadmap for the future. That stuck with me. So I looked up what it would actually take to build one myself, and I built it.

It started simply. I set up a T-Pot honeynet on a Proxmox box in my home lab, pointed it at the internet, and waited to see what would happen.

Getting there took more than that one sentence lets on. The physical side is an HP EliteDesk 800 G6 Mini, running Proxmox VE as the hypervisor, with a dedicated Debian 13 virtual machine hosting T-Pot 24.04.1 HIVE, a purpose-built distribution that bundles more than twenty honeypots, each isolated in its own Docker container, alongside the tooling to analyze what they catch. Cowrie handles SSH and Telnet, Dionaea catches malware, Ciscoasa spoofs VPN login pages, Heralding harvests generic credentials, ConPot emulates industrial control protocols, Sentrypeer catches VoIP and SIP abuse, and Suricata and P0f run alongside them for intrusion detection and OS fingerprinting. All of it feeds into an ELK stack, Elasticsearch, Logstash, and Kibana, which is what actually turned raw honeypot logs into the dashboards I used every week.

![T-Pot Honeynet Lab Architecture](https://raw.githubusercontent.com/KingMike77/Tpot-Honeynet-Threat-Intelligence/main/Screenshots/T-Pot%20Honeynet%20Lab%20Architecture.png)

On the network side, inbound traffic comes through an AT&T Fiber gateway in IP Passthrough mode, into a TP-Link Archer BE3500 lab router with a single DMZ rule forwarding everything straight to the T-Pot VM. That VM sits on its own isolated lab network with no route back to my actual home network, and management access, the web interface, SSH, Kibana, is never exposed to the public internet at all, only reachable over the lab WiFi or through Tailscale. I built it deliberately layered specifically because pointing a home machine at the open internet isn't something you do casually. Even a fully compromised honeypot container would land inside an isolated VM with no other devices on its network and no path back home, which is what makes running this kind of project on real home internet actually reasonable to do.

Building it from scratch was not a clean, one-afternoon setup. The early reports in this series, WTIR-01 through WTIR-05, cover real infrastructure troubleshooting alongside the actual threat findings, including Docker containers stuck in crash loops, a Logstash sincedb issue, and the first of what turned out to be several ConPot data-integrity bugs. Getting the pipeline itself trustworthy was its own project before I could trust anything it told me about the internet.

What I didn't expect was that it would turn into a twelve-week weekly reporting habit, or that I'd end up treating it less like a hobby project and more like an actual threat intelligence function: one analyst, one honeynet, and a self-imposed deadline every week to publish something real. The goal from the start was to build something that would signal actual skill to a recruiter or hiring manager, not just "I ran a tool and took a screenshot." Twelve reports later, I think it did that, though not in the way I expected going in.

## By the Numbers

I logged roughly 9.5 million total attack events across all twelve reports combined, with a confirmed on-record total for every single week.

I published twelve weekly reports between May 29 and September 7, 2026. The series-record week was the final one, WTIR-12, with about 1.53 million attack events, up from roughly 65,000 in the very first report, which only covered a three-day window. Across the series I profiled more than twenty individual threat actors in depth. I also found and corrected three distinct data-integrity bugs in my own telemetry, in WTIR-06, WTIR-07, and WTIR-10. Five reports, WTIR-08 through WTIR-12, tracked a single recurring dual-country infrastructure pattern. One malware campaign, found in WTIR-12, was independently confirmed by an external threat-intel platform using a file hash I had pulled myself. And by WTIR-05, I had standardized on eight OSINT sources that I used as a fixed toolkit for the rest of the series.

![WTIR Series Attack Volume by Report](https://raw.githubusercontent.com/KingMike77/Tpot-Honeynet-Threat-Intelligence/main/Screenshots/WTIR-Retrospective%20Volume%20by%20Report.png)

The volume varied considerably from week to week, but the bigger story was how the way I interpreted that data changed over time.

## How It Evolved, Report by Report

Looking back at the whole run, it's easy to see the arc now, even though I didn't plan any of it in advance. It just happened as I kept running into the limits of what I'd done the week before.

**WTIR-01** covered two SIP toll-fraud campaigns running side by side, one through OVH SAS cloud infrastructure and one under a personally-registered ASN tied to an individual named Husam A.H. Hijazi, plus a MySQL ransomware operation out of Romanian bulletproof hosting. About 65,000 attacks were logged over three days, and I had no idea yet that OVH SAS would show up again ten reports later.

**WTIR-02** was where actors started to feel like ongoing stories instead of one-off IPs. The MySQL ransomware actor from WTIR-01 came back, and instead of swapping it out for something new, I went deeper and found its infrastructure had expanded to 299 malicious IPs across the same subnet. That was the first time I chose depth over novelty, and it was the right call.

**WTIR-03** brought a 100% week-over-week volume surge, driven by SIP scanning and VNC enumeration at scale, plus two live CVEs worth naming. An actor first seen in WTIR-02 came back again, and returning actors were starting to feel less like coincidence and more like something worth tracking deliberately.

**WTIR-04** is where I first had to catch myself overclaiming. A compromised-looking Ethiopian IP traced back to a real academic journal site, and my first draft said the evidence strongly suggested compromise. I softened that to say the available evidence was consistent with compromise, because I genuinely couldn't see behind the keyboard from where I was sitting. That one edit mattered more than it looked like at the time, since it's the same discipline that shows up everywhere in the later reports.

**WTIR-05** was the week attribution stopped being a single lookup and became a chain, a story I tell in full in the countries section below. This is also where I locked in the OSINT toolset I used for the rest of the series: Hurricane Electric, Shodan, VirusTotal, AbuseIPDB, GreyNoise, Talos, OTX AlienVault, and RIPE Atlas.

**WTIR-06 is the report that changed how I did everything after it.** I found a fabricated top attacker in my own dashboard. It showed 42,833 hits that turned out to be a duplicate log-ingestion bug rather than real traffic, inflating my weekly total from an honest 712,000 or so up to a reported 901,000. That's the origin of the verification discipline that shows up in every report since.

**WTIR-07** was a deliberate strategic pivot. I decided to stop asking who the top attackers were that week and start asking what had changed and whether anything had come back. It paid off immediately, since I confirmed the same enterprise VPN and firewall exploitation kit had been run by the same Netherlands operator across three separate reports, not just once. This is also the week I found a second, unrelated ConPot logging bug, proof that checking a number isn't a one-time fix but a standing habit, and the week I correctly ruled out a false pattern after actually checking WHOIS and ASN records instead of chasing a coincidence.

**WTIR-08** produced the strongest attribution finding of the first half of the series, a pair of netblocks tagged to two different countries that shared the exact same domain and abuse history, which I break down in more depth in the countries section below.

**WTIR-09** paid that thread forward when the same infrastructure family resurfaced under a third country tag, tied back to the same registrant name. It stopped being a one-off discovery and started being something I actively tracked across reports.

**WTIR-10** is where the verification discipline matured past its first bug. I found a second, completely different failure mode, not duplicate ingestion this time, but an unscoped query counting IDS and fingerprinting metadata as if it were attacker traffic. It had the same symptom as WTIR-06, a number that looked too high, but a different cause and a different fix. This report also paid off a three-week-old thread, since "Vlad Cojuhari," a name I'd first seen buried in someone else's RIPE record back in WTIR-08, turned out to have his own ASN. That payoff never would have happened if I'd dropped the thread the moment WTIR-09 already had a strong finding of its own.

**WTIR-11** was the week I stopped treating each report as its own island. I pulled all ten prior reports directly from GitHub and cross-referenced them against that week's top actors, confirming a real recurring-actor registry, including OVH SAS, first seen doing SIP toll fraud all the way back in WTIR-01, still matching the same pattern ten reports later.

**WTIR-12**, the final report, brought the two biggest moments of the whole series together at once. The two-passports pattern I'd first found in WTIR-08 turned up again, and for the first time in twelve weeks, I caught actual malware, which I walk through in full further down.

## How My Methodology Actually Changed

The report-by-report list above is what happened. This is how I actually did the work differently, week to week, because I didn't run the same process twelve times. I kept changing it, sometimes on purpose and sometimes because something broke and forced my hand.

**I changed who I picked before I ever changed how I investigated them.** The first few reports picked actors by raw volume, so whoever hit the honeynet hardest that week got a writeup. WTIR-06 broke that habit for good, since I switched to picking my most-attacked honeypots and profiling the top actor on each, instead of chasing the biggest number on the board. That single change is what surfaced the fabricated ConPot attacker in the first place. From there my selection criteria kept shifting: sometimes a familiar country I wanted to dig into further, sometimes one I'd never seen before, sometimes just plain interest, and increasingly, later on, because an actor or infrastructure family was recurring and I wanted to see what had changed.

**I went from a loose set of OSINT habits to a fixed toolset.** For the first four reports, my enrichment process wasn't really consistent, since I reached for whatever tool made sense in the moment, including SpiderFoot scans and ad hoc lookups. WTIR-05's four-country Seychelles chase is what forced the issue, because I needed a repeatable process, not a grab bag. By the end of that report I'd locked in a fixed toolkit, Hurricane Electric, Shodan, VirusTotal, AbuseIPDB, GreyNoise, Talos, OTX AlienVault, and RIPE Atlas, and used the same set, in roughly the same order, for every actor in every report after that.

**I went from asking whether a spike looked real to actually proving it.** WTIR-06 was the wake-up call, and I tell that full story below. That check became a standing pre-flight step from then on, and it kept finding new failure modes over the following weeks. By WTIR-12, it wasn't something I decided to do that week. It was just the first step of every report, full stop.

**I went from single-week narratives to actively cross-referencing my own back catalog.** For the first ten reports, figuring out whether I'd seen an actor before was mostly a matter of remembering, or occasionally scrolling back through an old file. WTIR-11 is where that became a real process, since I pulled every prior report straight from GitHub and systematically checked that week's top actors against all ten of them. That's what turned a vague sense of familiarity into a confirmed, fifth-consecutive-week claim, which is a completely different and much more defensible kind of statement.

**I went from reporting confidence in prose to reporting it as a formal, structured tier.** Early reports said things like "likely compromised" or "appears to be the same actor" without much more precision than that. By WTIR-12, every finding got an explicit High, Medium, or Low confidence rating with a stated reason, plus a separate defensive relevance note explaining why it mattered even at lower confidence. That's a small formatting change, but it reflects a real shift in how carefully I was willing to commit to a claim.

**And I went from only trusting my own data to actively seeking outside corroboration.** For most of the series, confirmed meant confirmed by me, on my own honeynet, with my own tools. WTIR-12 was the first time I deliberately checked whether an outside platform, abuse.ch's ThreatFox, had independently seen the same artifact I had. It had. That's a different, stronger standard of evidence than anything earlier in the series, and it's the standard I'd want to hold myself to going forward.

None of this was planned out at the start. Each change came from running into a real limitation the week before and deciding not to just work around it quietly, but to actually fix the process for every report after.

## The Honest Challenges

This wasn't a smooth twelve weeks, and I don't want to pretend it was.

**T-Pot's dashboards are genuinely easy to misread if you don't understand what's actually feeding them.** I tell that full story in its own section below, since it deserves more than a passing mention here.

**Attribution is genuinely hard, and I had to keep resisting the urge to overclaim it.** It's very tempting, when you see the same ASN show up twice under two different country tags, to just say this is the same threat actor. I had to keep reminding myself that infrastructure overlap and human attribution are two different claims, and that GeoIP and WHOIS tell you who administers a network, not who's sitting behind the keyboard. Keeping that distinction honest, report after report, was harder than it sounds.

**Keeping up the weekly cadence was its own challenge.** Some weeks had a clean, obvious story. Other weeks I had to dig hard to find something worth writing about beyond more of the same scanning. Learning to recognize when a week's data was genuinely a strong finding versus when I was reaching for one was its own skill.

## What I Actually Learned

Running something like a small, one-person CTI shop for twelve straight weeks taught me as much about discipline as it did about threat intelligence. There was the habit of publishing on a cadence, the discomfort of sometimes not having a great finding and having to say so honestly, and the payoff of watching a hypothesis from week eight get independently confirmed in week twelve.

## The Countries I Ended Up Learning About

Some of the most interesting threads in this project started with the same reaction. I'd look at the Top 10 Attacking Countries panel, see a name I genuinely didn't expect to see, and wonder how that could be a source of attack traffic. A few of these were countries I couldn't have told you much about beforehand, and each one turned into an actual lesson once I went looking.

**Georgia** is a small country in the Caucasus, wedged between the Black Sea, Russia, and Turkey, and it was the first one that made me stop and dig rather than just note it and move on. It showed up as a Ciscoasa-scanning source in WTIR-08, tied by a shared domain and abuse history to a paired Kazakhstan cluster running under a different ASN entirely.

**Kazakhstan** is a large Central Asian country bordering Russia and China, and it came along with Georgia in that same report. What made it stick with me is that it showed up a second time, months later in WTIR-12, tied to a completely different operator running the same two-countries-one-infrastructure trick. Two unrelated groups, two different reports, using the same playbook. That's when Kazakhstan stopped being just a name on a donut chart and started being a pattern I actively watched for.

**Seychelles** is an archipelago nation off the East African coast in the Indian Ocean, and it's the one I remember most clearly, because I genuinely had no idea what to expect when it showed up in WTIR-05. I didn't even know Seychelles had enough internet infrastructure to be a meaningful attack source. Tracing that single IP turned into a four-layer chain. There was a Seychelles-registered reseller, leasing to a Dutch operator, running on a server physically in the Netherlands, while a fourth OSINT source called it South Africa. One IP had four correct countries depending on which layer you asked. That report is the reason I stopped treating country of origin as a simple fact you look up.

**Vanuatu** is a small island nation in the South Pacific, with its capital at Port Vila, and it turned up in WTIR-09 as a WHOIS registrant location for part of the UK-tagged 138.226.239.x cluster, sitting alongside GeoIP's claim of United Kingdom and AbuseIPDB's claim of Netherlands for the same address block. Of everywhere I ran into during the series, Vanuatu is the one that best captures how little a single geography claim is actually worth on its own. It gave three different, equally official answers for where one piece of infrastructure supposedly sits.

**Ghana** is a different kind of unfamiliar for me. It isn't a country I knew nothing about, but one I have a real personal connection to, since my parents were both born and raised there. Seeing it turn up as a source country in the very last report of the series, WTIR-12, and then finding it tied to the exact same dual-country infrastructure trick I'd first learned about from Georgia and Kazakhstan, made an abstract technical pattern feel a lot more real.

By the end, seeing an unfamiliar country in that Top 10 panel had stopped being a curiosity and started being a cue to go find out who's actually behind that traffic, because the GeoIP tag on its own was never going to tell me.

## How I Learned to Verify, and Why

If there's one skill this project actually built in me, it's this: a number on a dashboard is a claim, not a fact, until you've traced it back to its source.

That habit didn't come naturally, and it has an exact origin point in WTIR-06. My dashboard reported a top ConPot attacker at 42,833 hits, a fabricated number roughly 190 times higher than the actual leading actors that week. Every one of those log entries carried an identical timestamp, which would have meant tens of billions of packets per second, physically impossible on my connection. Tracing it back, the real activity was 3,231 distinct request timestamps spread across 179,467 raw log lines, inflated by a genuine duplicate log-ingestion issue. The corrected weekly total dropped from a reported 901,000 down to about 712,000. I wrote at the time that the check that caught it wasn't sophisticated. It was just correlating session identifiers, comparing against other sensors, and asking whether the implied packet rate was even physically possible, and that I should have been doing that from WTIR-01.

That mistake changed how I ran every report after it. WTIR-07 turned up a second ConPot bug within weeks, a different logging mechanism than WTIR-06's but the same underlying lesson. Then WTIR-10 turned up a third, completely different failure mode. Techoff Srv Limited's apparent inflation wasn't duplicate ingestion at all, it was an unscoped query counting Suricata IDS alerts and P0f OS-fingerprint metadata as if they were independent attacker interactions, when they were just secondary log lines riding along on the same real sessions. It had the same symptom as the earlier two bugs, a number that looked too high, but yet another distinct cause and fix. By WTIR-12, checking a headline number against the raw logs wasn't a special step anymore. It was just how I did the job, and it's what caught a 13x gap between raw index counts and actual attack volume in the very last report, for reasons unrelated to any of the first three.

The reason behind all of this is simple. Nobody is going to trust threat intelligence from someone who can't tell the difference between what their tools are showing them and what's actually true. Building that instinct, to check the claim instead of just reporting the dashboard, is honestly probably the single most transferable skill this whole project gave me, and it isn't specific to honeynets. It's the same instinct I'd want in any security analyst role, in IAM just as much as detection engineering. Don't trust the console. Trust the data underneath it.

## How I Got Real Malware From a Threat Actor

Every prior report in the series was about attackers scanning, probing, or trying to brute force their way in. WTIR-12 was the first time I actually found, captured, and confirmed real malware on the honeynet, and it happened almost by accident, because I was paying attention to the right log at the right moment. I can't say for certain no attacker ever pulled a file down in any of the previous eleven reports since I wasn't specifically watching for it before this week, but this is the first time I looked, found one, and followed it all the way through to confirmation.

Cowrie, the honeypot that emulates SSH and Telnet, does something specific when an attacker gets past the fake login and starts typing commands. If they run a command like wget or curl to pull a file from the internet, Cowrie lets the download happen, saves a copy, and logs it as `cowrie.session.file_download`. That single design choice is what made this whole finding possible. I pulled every one of those events across the reporting window and found 1,687 of them, spread across at least six different staging servers.

One server in particular stood out. It was serving files named things like Sakura.sh and a set of matching binaries built for different processor architectures, which is the standard pattern for an IoT botnet trying to infect whatever kind of device it lands on, whether that's a router, a camera, or a server. Before trusting any of it, I went into the raw downloads folder on the honeynet myself, ran the file command against every hash-named file sitting in there, and separately ran sha256sum on each one to get its exact fingerprint. That step mattered, because it meant I already knew what these files actually were, a bash script here, a 64-bit Linux executable there, before I ever asked a third party what they thought.

Then I took those hashes to VirusTotal. One of them came back flagged by 40 out of 58 security vendors, classified as a Gafgyt and Mirai family DDoS bot.

![VirusTotal Gafgyt Sakura Detection](https://raw.githubusercontent.com/KingMike77/Tpot-Honeynet-Threat-Intelligence/main/Screenshots/week-12/WTIR-12%20Threat%20Actor%203%20VirusTotal%20Gafgyt%20Sakura.png)

VirusTotal's own analysis of the binary described exactly what it does: it connects out to a command-and-control server, reports back details about the infected device, and waits for instructions to launch denial-of-service floods against a target. Buried in the code was a reference to a booter website, which told me this wasn't some custom tool built for me specifically, it was rented attack infrastructure being aimed at whatever honeypot happened to answer.

The part that actually made this feel real was checking the staging server itself against abuse.ch's ThreatFox, a public database that tracks known malicious infrastructure. It was already there, already listed as an active command-and-control host, and the sample abuse.ch had on file for it matched the exact same file hash I had pulled directly off my own honeypot. Either the hash matches or it doesn't, and it matched. An outside research platform had independently seen and catalogued the same piece of infrastructure I was looking at, with no idea I existed.

That wasn't the only thing that came through that week. A separate incident days earlier dropped a cryptominer called RedTail through an Apache web server flaw rather than a brute-forced login. Another dropped a second cryptominer, Multiverze, disguised under the filename sshd to blend in with a legitimate process. And a fourth skipped malware entirely and just planted a malicious SSH key into an authorized-keys file, a quieter way to guarantee permanent, passwordless access back into a machine later.

Four different payloads, four different methods, all captured, verified, and confirmed in a single week, after eleven prior reports where nothing like this had happened at all.

## Where I'm Headed Next

Here's the honest turn in my thinking that came out of doing this for twelve weeks. I'm shifting my primary focus toward Identity and Access Management, and it's worth being upfront about the context. I've been trying to break into cybersecurity since finishing my bachelor's in 2023, about three to four years now. I've earned certifications in Security+, PenTest+, CySA+, and SecurityX/CASP+, completed a master's degree in cybersecurity, done an OT/ICS SOC internship at Wärtsilä North America, and built hands-on projects like this one. None of that was wasted, and none of it is going away.

This isn't a retreat from threat intelligence, detection engineering, or SOC work. This series and the internship are proof I'm genuinely drawn to it. It's that my actual day job already has me managing Active Directory, Okta, and Intune, which gives me a real, paid foot in the door that a home-lab honeynet can't. This project made clear how much identity and access sits underneath almost everything I investigated, from leased IP space to SSH key persistence to credential reuse across recycled infrastructure. So it's less a pivot away from what I've been doing and more a shift toward the door that's actually open right now.

The SOC home lab and the malware sandbox aren't going anywhere. But the hardware that ran this honeynet is very likely about to become an IAM-focused lab project instead. If the last twelve weeks taught me how to verify a claim before trusting it, the next one is about learning to verify who's making the claim in the first place, which is really the same instinct pointed in a different direction.

Twelve weeks ago, I wanted to prove that I could build a honeynet. I ended the project with something more useful, a better understanding of how to question telemetry, validate intelligence, investigate infrastructure, and communicate uncertainty. The honeynet was the tool, but the analytical discipline was the real project.
