# DNSSEC And You: Let's Do A Learn

---

## What is DNSSEC?

A system to let your machine know that DNS results come from the right place, and haven't been tampered with.

It provides resilience against domain takeover, failing to resolve if shenanigans are detected.

```mermaid
---
config:
  layout: elk
---
flowchart LR
    U(["User asks for a website"]) --> R@{ label: "Resolver asks the domain's DNS" }
    R --> A(["DNS server replies"]) & T(["Parent zone confirms key"])
    A --> S(["Answer + digital seal"]) & K(["Public key"])
    S --> V{"Seal valid?"}
    K --> V
    T --> V
    V -- Yes --> OK(["Show the website address"])
    V -- No --> FAIL(["Tampering detected! Fail to resolve, show an error"])

    R@{ shape: stadium}
     T:::Pine
     K:::Pine
     OK:::Pine
     FAIL:::Rose
    classDef Pine stroke-width:1px, stroke-dasharray:none, stroke:#254336, fill:#27654A, color:#FFFFFF
    classDef Rose stroke-width:1px, stroke-dasharray:none, stroke:#FF5978, fill:#FFDFE5, color:#8E2236
```

---

## Where does the key come from?

We need to generate a key with a lifetime, which we will use to cryptographically sign DNS results.

That lifetime is usually a few months; 1-3 months is common.

---

## How does DNSSEC fail?

Since two bodies have to know the key, it can get out of sync.

```mermaid
---
config:
  layout: elk
---
flowchart LR
    U(["User asks for a website"]) --> R@{ label: "Resolver asks the domain's DNS" }
    R -- "domain.tld" --> A(["DNS server replies"])
    R -- ".tld" --> T(["Parent zone<br>confirms old key"])
    A --> S(["Answer + digital seal"]) & K(["New public key"])
    S --> V{"Seal cannot be valid!"}
    K --> V
    T --> V
    V -- <br> --> FAIL(["Show an error"])

    R@{ shape: stadium}
     T:::Peach
     K:::Pine

     FAIL:::Rose
    classDef Pine stroke-width:1px, stroke-dasharray:none, stroke:#254336, fill:#27654A, color:#FFFFFF
    classDef Peach stroke-width:1px, stroke-dasharray:none, stroke:#FBB35A, fill:#FFEFDB, color:#8F632D
    classDef Rose stroke-width:1px, stroke-dasharray:none, stroke:#FF5978, fill:#FFDFE5, color:#8E2236
```

---

## The two scenarios

We now need to consider:

1. DNS host is the same as domain registrar
2. DNS host is a different company than domain registrar

---

### Scenario 1: Registrar and DNS host are the same

When your domain registrar is also your DNS host, they can make sure the keys are kept in sync.

```mermaid
---
config:
  layout: elk
---
flowchart LR
    KE(["Key expires"]) --> RG(["DNS host generates new key"])
    RG -- ".tld" --> RZ(["DNS host sends new key to zone"])
    RG -- "domain.tld" --> DH(["DNS host updates DNS with new key"])
    RZ & DH--> YAY(["Everything works properly!"])

     RG:::Pine
     RZ:::Pine
     DH:::Pine
    classDef Pine stroke-width:1px, stroke-dasharray:none, stroke:#254336, fill:#27654A, color:#FFFFFF
```

---

### Scenario 2: Registrar and DNS host are the different

When your domain registrar and DNS host are separate, each one can't coordinate with the other.

```mermaid
---
config:
  layout: elk
---
flowchart LR
    KE(["Key expires"]) --> RG(["DNS host generates new key"])
    RG -- ".tld" --> RZ@{ label: "DNS host can't update zone" }
    RZ --> EX(["Zone is left with expired key"])
    RG -- "domain.tld" --> DH(["DNS host updates DNS with new key"])
    EX --> YAY(["Key mismatch! DNS stops resolving."])
    DH --> YAY

    RZ@{ shape: stadium}
     RG:::Pine
     RZ:::Peach
     EX:::Rose
     DH:::Pine
    classDef Pine stroke-width:1px, stroke-dasharray:none, stroke:#254336, fill:#27654A, color:#FFFFFF
    classDef Rose stroke-width:1px, stroke-dasharray:none, stroke:#FF5978, fill:#FFDFE5, color:#8E2236
    classDef Peach stroke-width:1px, stroke-dasharray:none, stroke:#FBB35A, fill:#FFEFDB, color:#8F632D
```

---

## So how does it ever work for different companies?

Simple. **You are responsible for manually updating the key**. You have to do this depending on the lifetime of the key.

---

## What if I forget to update the key?

If you forget to update the key, or there's some sort of organisational clusterfuck, **your DNS will stop resolving** until it's fixed.

This means your website will be unreachable, your email to that domain will stop working, and any other services relying on your domain name will be affected.

**That's bad**.

---

## Recommendation

Use the same company for domain registration and DNS.

Since we're using Cloudflare for Workers too, having that tight integration means everything can be managed in one place.

Consider that **you rarely log in to your registrar** if it's a different company.

---

## Caveats

Cloudflare has one restriction: **you can't point your domain's nameservers to anyone but Cloudflare.**

---

## Does that impact us?

In a word, no.

In two words, still no.

While the DNS has to be on Cloudflare, you can still point records anywhere you like.

---

## Keeping an eye on it

Regardless of our choice, we should implement monitoring for DNSSEC validity. I'm working on that; I'm trying to find a cheap or even free solution.

---

## Questions?

I hear people add this slide to their presentations a lot.

---

## License

[MIT License](LICENSE)
