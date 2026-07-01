# Debug Workflow

Workflow ini hanya mengarahkan proses debug. Sumber aturan inti ada di `references/Luau.md`, `references/topics/TestingQA.md`, `references/Security.md`, dan `references/Performance.md`.

---

## Step 1: Gather

- Ambil error output atau stack trace.
- Catat script, line, origin, dan frekuensi error.

## Step 2: Attach References

- `references/Luau.md`
- `references/topics/TestingQA.md`
- `references/Security.md`
- `references/Performance.md`

## Step 3: Diagnose

1. Klasifikasikan error: syntax, runtime, logic, security, atau performance.
2. Cari source code di sekitar stack trace.
3. Cocokkan dengan pola debug, validation, atau performance yang relevan.

## Step 4: Fix and Verify

- Ajukan fix terkecil yang aman.
- Uji ulang sampai error hilang atau perlu eskalasi.
- Jika bug berulang, gunakan regression note dan checklist testing.
