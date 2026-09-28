<div align="center">

# 💎 Diamond Model of Intrusion Analysis — Applied to IR-001

![Framework](https://img.shields.io/badge/framework-Diamond%20Model-2563EB?style=flat-square)
![Based%20On](https://img.shields.io/badge/based%20on-IR--001%20SSH%20Brute--Force-6B7280?style=flat-square)

</div>

---

## 📖 What This Is

The Diamond Model is a simple way analysts break down any single intrusion event into four connected parts. Instead of just writing "an attack happened," it forces you to separate out:

- **Who** did it (or what you can tell about them)
- **What** they used to do it
- **Where/how** they reached you from
- **Who/what** got targeted

```
          Adversary
         /          \
  Capability ------ Infrastructure
         \          /
          Victim
```

Below, this is applied to the existing [IR-001-ssh-brute-force.md](../incident-reports/IR-001-ssh-brute-force.md) incident from this homelab.

---

## 🧑‍💻 Adversary

*Who carried this out?*

From the log evidence alone (source IP, timing, technique), the only conclusion supportable is: a single actor using automated tooling, capable of downloading and running an offensive security tool (Hydra) and sourcing a common leaked-password wordlist. No further attribution (identity, organization, motive) is possible purely from network/host logs — this reflects a realistic constraint: in most real incidents, attribution requires additional intelligence sources beyond your own SIEM (threat intel feeds, law enforcement data, etc.), which is out of scope for a single-incident SIEM investigation.

**In this lab:** the "adversary" is the Kali attacker VM, standing in for an external attacker.

---

## 🛠️ Capability

*What tool/technique did they use?*

- **Tool:** Hydra — a free, widely-available password-guessing tool
- **Technique:** automated dictionary/brute-force attack (MITRE ATT&CK **T1110**)
- **Wordlist:** `rockyou.txt` — a well-known leaked-password list bundled with Kali by default

This is a low-sophistication, high-availability capability — no custom malware, no zero-day, just a commodity tool anyone can download. That's worth noting: **most real-world attacks use "off the shelf" capability, not custom tooling** — which is exactly why detecting commodity tool signatures (like Hydra's rapid, uniform login attempts) has high practical value.

---

## 🌐 Infrastructure

*Where did the attack come from?*

- **Source IP:** 192.168.1.177 (Kali attacker VM)

In a real (non-lab) investigation, this is where you'd add: ASN/hosting provider, geolocation, whether the IP appears on any threat-intel blocklists (e.g., AbuseIPDB), and whether it's a known VPN/proxy/Tor exit node. Since this is an internal lab network, that enrichment doesn't apply here — but documenting *where that enrichment would go* is itself worth showing, since it demonstrates awareness of the full real-world process even where the lab's scope stops short of it.

---

## 🎯 Victim

*What was targeted?*

- **System:** Ubuntu-Victim (192.168.x.x)
- **Service:** SSH (port 22)
- **Targeted account:** `socanalyst`

The victim here is a single, specific service/account — not a broad, opportunistic scan. This is consistent with a targeted brute-force attempt against a known account name, rather than mass scanning of an unknown network range.

---

## 🔗 The Edges (Relationships Between Vertices)

The Diamond Model's real value isn't just naming the four things — it's the relationships between them:

- **Adversary → Capability:** the adversary chose a simple, effective, freely-available tool rather than investing in custom malware — consistent with an opportunistic or low-resourced actor, not an advanced persistent threat
- **Capability → Infrastructure:** Hydra was run directly from the source host with no additional obfuscation (no proxy chain, no botnet) — meaning the true source IP was directly visible in the logs, unlike more sophisticated attacks that route through anonymizing infrastructure
- **Infrastructure → Victim:** the source host had direct network-layer access to the victim's SSH port — highlighting that network segmentation (restricting which hosts can even reach SSH) is a control that would have prevented this attack chain from ever starting, regardless of password strength

---

## 💡 Why This Matters

Filling out all four vertices — even when some fields are limited ("no further attribution possible") — is itself informative: it tells a reader (or a more senior analyst) exactly what is and isn't known, and where a real investigation would need external intelligence sources to go further. This is a more disciplined, complete way to document an incident than a narrative summary alone, and it's the same structure CTI (Cyber Threat Intelligence) analysts use to track and pivot between related intrusions over time — recognizing when two seemingly separate incidents actually share the same capability or infrastructure, for example.
