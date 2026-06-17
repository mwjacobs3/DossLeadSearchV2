# Salesforce Dedup Rules

The whole point of this pipeline is to surface companies **not already in Salesforce**. Salesforce
data is inconsistent (industry/revenue fields are messy), so **website domain is the primary
matching key**, with company name as a fuzzy secondary.

## Domain normalization
Normalize both the candidate domain and any Salesforce `Website` before comparing:
1. Lowercase.
2. Strip scheme (`http://`, `https://`) and any path/query (`/...`, `?...`).
3. Strip leading `www.`.
4. Keep the registrable root domain (e.g. `shop.acme.com` → `acme.com`; `acme.co.uk` stays).
5. Compare roots for equality.

Examples: `https://www.AcmeBrand.com/products` → `acmebrand.com`.

## Matching procedure (per candidate)
Run these checks; a hit on **any** = the company is already KNOWN → exclude.

**1. Account by domain** (primary):
```sql
SELECT Id, Name, Website, Account_Status__c, Type, OwnerId
FROM Account
WHERE Website LIKE '%<root_domain>%'
```
(`<root_domain>` without the `www.`, e.g. `%acmebrand.com%`.) Confirm the normalized roots
actually match — `LIKE` can over-match, so verify in code/judgment.

**2. Account by name** (secondary, catches accounts with blank/wrong Website):
```sql
SELECT Id, Name, Website, Account_Status__c
FROM Account
WHERE Name LIKE '%<core name>%'
```
Use the distinctive core of the name (drop `Inc`, `LLC`, `Co.`). Or SOSL for fuzzy matching:
```sql
FIND {"<company name>"} IN NAME FIELDS RETURNING Account(Id, Name, Website, Account_Status__c)
```

**3. ZoomInfo id** (after enrichment, belt-and-suspenders):
```sql
SELECT Id, Name FROM Account
WHERE ZoomInfo_Company_ID__c = '<zi_id>' OR DOZISF__ZoomInfo_Id__c = '<zi_id>'
```

**4. Leads** (if `settings.salesforce.check_leads`):
```sql
SELECT Id, Name, Company, Website, Status FROM Lead
WHERE Website LIKE '%<root_domain>%' OR Company LIKE '%<core name>%'
```

## Treat as KNOWN regardless of status
Exclude matches in **any** status — `Customer`, `Churned`, `Closed Lost`, `Disqualified`,
prospect, etc. We don't want to re-surface accounts that are already owned or already judged.
Record the matched `Id`, `Name`, and `Account_Status__c` in the run log for the audit trail.

## Edge cases
- **Parent/child brands:** a candidate may be a brand owned by a parent already in SFDC. If the
  parent is a customer, note it but you may still surface the distinct brand — flag the relationship.
- **Multiple domains:** some brands use a `.co` / `.shop` / regional TLD. Check the primary domain
  ZoomInfo returns too, not just the one from the publication.
- **Over-broad `LIKE`:** always verify normalized-root equality; don't trust the SQL `LIKE` alone.
