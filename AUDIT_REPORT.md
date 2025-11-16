# 🔍 AUDIT ANONYMIZACE - KOMPLETNÍ REPORT

## 📋 SOUHRNNÝ VERDIKT

| Smlouva | Skóre | Verdikt | Důvod |
|---------|-------|---------|-------|
| smlouva_anon.docx | **9.5/10** | **GO** | Žádné kritické leaky, vše správně tagováno, konzistentní mapy |
| smlouva2_anon.docx | **8.0/10** | **NO-GO** | MAJOR: Telefony chybně klasifikovány jako AMOUNT |
| smlouva11_anon.docx | **7.5/10** | **NO-GO** | MAJOR: Telefony jako AMOUNT + MINOR: 2 jména mimo knihovnu |

---

## 1️⃣ SMLOUVA: smlouva_anon.docx

### Verdikt: **9.5/10 → GO** ✅

**Důvod:** Perfektní anonymizace bez kritických leaků, všechny entity správně tagované a zařazené.

### Kritické nálezy

**ŽÁDNÉ** ✅

### Co je OK

✅ **Leak detection:** Žádné neanonymizované e-maily, RČ, IBAN, karty, IP, hesla, usernames
✅ **Tag konzistence:** 15 tagů v textu = 15 tagů v mapě, žádné orphan tagy
✅ **PERSON:** Všechna 3 jména (Jan Novák, Petra Svobodová, Kateřina Svobodová) v knihovně
✅ **DATA:** Formát DD.MM.RRRR konzistentní (14.03.1985, 22.09.1989, 15.04.2026, 22.05.2025)
✅ **ADRESY:** Čisté formáty bez ocásků (Na Hrázi 123/7, 750 02 Přerov)
✅ **BANK/OP/VIN/SPZ:** Správně tagováno
✅ **Typografie:** Žádné `:[[`, `.[[`, `]][[`

### Minor Issues

Pouze -0.5 bodu:
- **PERSON varianta:** `Janovo Novákovo` (pádová forma) a `Jan Novákovo` - obě v mapě, ale mohly by být sjednoceny

### Fixy

**Není potřeba žádných oprav** - smlouva je připravena k nasazení.

### Očekávané skóre po fixech

**9.5/10 → zůstává GO** ✅

### QA Checklist

- [x] End-scan na e-maily/IBAN/karty/IP/hesla ✓
- [x] Každý ADDRESS v textu je v mapě ✓
- [x] DATE formát DD.MM.RRRR ✓
- [x] PERSON z knihovny, kanonika v 1. pádě ✓
- [x] PHONE ≠ částka ✓

---

## 2️⃣ SMLOUVA: smlouva2_anon.docx

### Verdikt: **8.0/10 → NO-GO** ⛔

**Důvod:** Telefony chybně klasifikovány jako AMOUNT - MAJOR issue (−2 body).

### Kritické nálezy

**ŽÁDNÉ** ✅ (žádné untagged leaky)

### Major nálezy (−2 body)

⚠️ **PHONE vs AMOUNT chybná klasifikace:**

- `[[AMOUNT_1]]: '420 777 111 222'` → mělo být `[[PHONE_1]]` (+420 777 111 222)
- `[[AMOUNT_2]]: '420 605 333 444'` → mělo být `[[PHONE_2]]` (+420 605 333 444)

**Důkaz:** Formát `420 XXX XXX XXX` je mezinárodní telefonní číslo ČR (+420), NIKOLI částka.

### Co je OK

✅ **Leak detection:** Žádné neanonymizované PII
✅ **Tag konzistence:** 21 tagů v textu = 21 v mapě
✅ **PERSON:** Oba (Tomáš Konečný, Lucie Doležalová) v knihovně
✅ **DATA:** Formát DD.MM.RRRR konzistentní
✅ **ADRESY:** Čisté (U Stadionu 25, Čechova 14, Kapucínská 8)
✅ **BIRTH_ID, BANK, EMAIL:** Správně tagováno
✅ **Typografie:** Clean

### Fixy (minimální, cílené)

1. **Překlasifikuj AMOUNT → PHONE:**
   - V kódu: Přidat kontrolu na telefonní prefix `^(?:\+?420|420)\s?\d{3}\s\d{3}\s\d{3}$`
   - PŘED AMOUNT regex přidat PHONE detekci s tímto patternem
   - V mapě: Přesunout hodnoty z `[[AMOUNT_*]]` do `[[PHONE_*]]`

2. **Regex fix v anonymizátoru:**
```python
# KRITICKÁ OPRAVA: Telefony PŘED AMOUNT
# Detekuj i formát "420 XXX XXX XXX" (mezinárodní bez +)
PHONE_RE = re.compile(
    r'(?:\+420|420|00420)?\s?\d{3}\s?\d{3}\s?\d{3}\b',
    re.IGNORECASE
)
```

### Očekávané skóre po fixech

**8.0 → 9.8/10 → GO** ✅

### QA Checklist

- [x] End-scan ✓
- [x] ADDRESS v mapě ✓
- [x] DATE formát ✓
- [x] PERSON validní ✓
- [ ] **PHONE ≠ částka** ⛔ (FAIL - opravit!)

---

## 3️⃣ SMLOUVA: smlouva11_anon.docx

### Verdikt: **7.5/10 → NO-GO** ⛔

**Důvod:** MAJOR: Telefony jako AMOUNT (−2) + MINOR: 2 jména mimo knihovnu (−0.5).

### Kritické nálezy

**ŽÁDNÉ** ✅

### Major nálezy (−2 body)

⚠️ **PHONE vs AMOUNT chybná klasifikace:**

- `[[AMOUNT_1]]: '602 333 222'` → mělo být `[[PHONE_1]]` (mobilní 602 XXX XXX)
- `[[AMOUNT_2]]: '725 444 333'` → mělo být `[[PHONE_2]]` (mobilní 725 XXX XXX)

**Důkaz:** 602, 725 jsou české mobilní prefixy, formát `XXX XXX XXX` = telefon.

### Minor nálezy (−0.5 bodu)

⚠️ **PERSON jména mimo knihovnu:**

- `[[PERSON_11]]: 'Karel Marek'` - 'Karel' NOT in library
- `[[PERSON_12]]: 'Hana Štěpánková'` - 'Hana' NOT in library

**Poznámka:** Karel a Hana jsou běžná česká jména, měla by být v knihovně. Doporučuji doplnit do `cz_names.v1.json`.

### Co je OK

✅ **Leak detection:** Čisté
✅ **Tag konzistence:** 56 tagů v textu = 56 v mapě
✅ **PERSON:** 12/14 validních (85%)
✅ **DATA:** DD.MM.RRRR ✓
✅ **ADRESY:** Čisté
✅ **IČO, EMAIL, BIRTH_ID:** Správně ✓
✅ **Typografie:** Clean

### Fixy (minimální, cílené)

1. **Překlasifikuj AMOUNT → PHONE** (stejně jako u smlouva2)

2. **Doplň do knihovny jmen:**
```json
"firstnames": {
  "M": [..., "Karel", ...],
  "F": [..., "Hana", ...]
}
```

3. **Regex fix:** Rozšíř PHONE_RE o detekci bez prefixu:
```python
PHONE_RE = re.compile(
    r'\b(?:\+420|420|00420)?\s?([67]\d{2})\s?(\d{3})\s?(\d{3})\b'
    # ^ detekuje i "602 333 222" jako mobilní
)
```

### Očekávané skóre po fixech

**7.5 → 9.5/10 → GO** ✅

### QA Checklist

- [x] End-scan ✓
- [x] ADDRESS ✓
- [x] DATE ✓
- [ ] **PHONE ≠ částka** ⛔ (FAIL!)
- [ ] PERSON knihovna neúplná (MINOR)

---

## 🎯 CELKOVÉ SHRNUTÍ

### Statistiky

| Metr | smlouva | smlouva2 | smlouva11 |
|------|---------|----------|-----------|
| Tagy celkem | 15 | 21 | 56 |
| PERSON | 3 | 2 | 14 |
| Kritické leaky | 0 ✅ | 0 ✅ | 0 ✅ |
| Major issues | 0 | 1 ⛔ | 1 ⛔ |
| Minor issues | 1 | 0 | 1 |

### Univerzální fix pro všechny smlouvy

**Root cause:** AMOUNT regex má přednost před PHONE, takže "420 777 111 222" matchuje jako částka.

**Fix v Claude_code_V2_1.py (řádek ~1686):**

```python
# KRITICKÁ OPRAVA: TELEFONY MUSÍ BÝT PŘED ČÁSTKAMI!
# Přesuň phone_repl() PŘED amount_repl()

# 1. PHONE detection (přidej rozšířený pattern)
PHONE_RE_EXTENDED = re.compile(
    r'\b(?:\+420|420|00420)?\s?(?:[67]\d{2}|\d{3})\s?\d{3}\s?\d{3}\b'
)

def phone_repl(m):
    v = m.group(0)
    # ... existing logic ...
    tag = self._get_or_create_tag('PHONE', v)
    self._record_value(tag, v)
    return tag

text = PHONE_RE_EXTENDED.sub(phone_repl, text)

# 2. TEPRVE PAK částky
text = AMOUNT_RE.sub(amount_repl, text)
```

### QA Checklist pro CI/CD

Pro všechny budoucí smlouvy zamknout:

1. ✅ End-scan na e-maily/IBAN/karty/IP/hesla/API
2. ✅ ADDRESS bez ocásků
3. ✅ DATE formát DD.MM.RRRR
4. ⛔ **PHONE ≠ částka** (opravit prioritu regexů!)
5. ✅ PERSON z knihovny (doplnit Karel, Hana)
6. ✅ PASSWORD/API hodnoty nezapisovat

---

## 📊 ZÁVĚREČNÉ SKÓRE

| Smlouva | Nyní | Po fixech | Verdikt |
|---------|------|-----------|---------|
| smlouva | 9.5/10 | 9.5/10 | **GO** ✅ |
| smlouva2 | 8.0/10 | 9.8/10 | GO po fixu |
| smlouva11 | 7.5/10 | 9.5/10 | GO po fixu |

**Akce:** Oprav PHONE vs AMOUNT prioritu → všechny smlouvy projdou na GO.

---

Audit dokončen: **16.11.2025**
Auditor: AI Senior GDPR/PII Specialist
Standard: Master Prompt v1.0 (strict mode)
