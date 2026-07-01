# Security Audit Workflow

Workflow ini hanya mengarahkan audit. Sumber aturan inti ada di `references/Security.md`, dengan detail remote dan payload di `references/Networking.md` serta pattern implementasi di `references/bridges/ImplementationPatterns.md`.

---

## Step 1: Inventory

- Daftarkan semua remote surface.
- Catat arah data, owner, consumer, dan frekuensi.
- Cari client-trusted logic, data exposure, dan permission boundary.

## Step 2: Attach References

- `references/Security.md`
- `references/Networking.md`
- `references/bridges/ImplementationPatterns.md`
- `references/topics/TestingQA.md`

## Step 3: Audit Order

1. Remote surface dan trust boundary.
2. Validation, authorization, dan ownership check.
3. Rate limit, cooldown, dan payload size.
4. Data exposure dan replicated secrets.
5. Receipt safety, transaction safety, dan anti-duplication.
6. Admin / permission / audit logging.

## Step 4: Report

- Tulis temuan dari critical ke low.
- Sertakan attack path, server-owned state, dan fix minimal.
- Prioritaskan exploit yang bisa langsung dimanfaatkan pemain.
