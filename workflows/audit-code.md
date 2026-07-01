# Code Review Workflow

Workflow ini hanya mengarahkan review. Sumber aturan inti ada di `references/Reviewer.md`, dengan detail keamanan di `references/Security.md`, performa di `references/Performance.md`, dan networking di `references/Networking.md`.

---

## Step 1: Project Scan

- Map struktur proyek dan scope perubahan.
- Identifikasi file yang terdampak dan dependensinya.
- Catat apakah review ini menyentuh server, client, shared, atau data layer.

## Step 2: Attach References

- `references/Reviewer.md`
- `references/Security.md`
- `references/Performance.md`
- `references/Networking.md`
- `references/Luau.md`
- `references/topics/TestingQA.md`

## Step 3: Review Order

1. Critical risk dan data-loss risk.
2. Runtime failure dan error handling.
3. Architecture, dependency direction, dan module boundaries.
4. Security quick-check.
5. Performance quick-check.
6. Readability, maintainability, dan test coverage.
7. Production readiness dan refactor safety.

## Step 4: Report

- Tulis findings dulu, severity paling tinggi dulu.
- Sertakan file dan line kalau tersedia.
- Bedakan bug nyata, risk, dan improvement.
- Akhiri dengan readiness status dan top fix list.
