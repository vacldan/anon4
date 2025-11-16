Anonymizátor (offline) – README

Cíl: Tento nástroj automaticky anonymizuje osobní údaje dle GDPR v textových dokumentech (CZ/EN, případně vícejazyčně) a vytvoří anonymizovanou verzi dokumentu + mapu náhrad (JSON i TXT). Navržen pro zcela offline provoz (např. Electron + Python backend).

Klíčové vlastnosti
🔒 Offline: žádná data neopouští zařízení.

🧭 Detekce PII: jména, adresy, e-maily, tel. čísla, bankovní účty/IBAN, rodná čísla, IČ/DIČ, SPZ/poznávací značky, čísla dokladů (OP, pas), názvy firem (volitelné), uživatelská jména, a další.

🏷️ Jednotné štítky: každá entita nahrazena stabilním štítkem typu [[UŽIVATEL_1]], [[ADRESA_3]], [[ÚČET_2]] atd.

🗺️ Mapa náhrad: strojově čitelný map.json + lidsky čitelný map.txt.

📄 Bezztrátová struktura: zachování odstavců, prázdných řádků a většiny interpunkce.

⚙️ Konfigurovatelné: zapínání/vypínání kategorií, vlastní slovníčky/whitelisty/blacklisty, přemapování názvů štítků.

🧪 Testovatelné: sada unit/integračních testů + demo vstupy.

Co přesně anonymizujeme (GDPR PII)
Níže jsou defaultní kategorie. Lze je měnit v konfiguraci.

Jména fyzických osob (české i cizí; včetně pádů a titulů). Příklady: „Jan Novák“, „Ing. Petra Černá, Ph.D.“

Adresy (ulice, č.p./č.o., PSČ, město, stát). Příklady: „Křenová 14, 602 00 Brno, CZ“

Kontakty: e‑maily, telefonní čísla (CZ/EU formáty), uživatelská jména.

Bankovní identifikátory: IBAN, čísla účtů (CZ formáty: 123456789/0100, CZ65 0800 …).

Identifikátory státu: rodné číslo, číslo OP/pasu/řidičáku.

Registrace vozidel: SPZ/RZ.

Daňové/firmní: IČ, DIČ, názvy firem (volitelné, typicky se anonymizují jen pokud identifikují fyzickou osobu).

Jiné unikátory: čísla smluv, zákaznická ID…, pokud mohou identifikovat FO (volitelné podle nastavení).

Pozn.: „Osobní údaj“ = jakákoli informace, která vede (samostatně či v kombinaci) k identifikaci živé fyzické osoby.

Zásady a pravidla anonymizace
3.1 Principy

✂️ Minimalizace: nahrazujeme pouze to, co je nutné pro de‑identifikaci.

🔁 Stabilita náhrad: stejný originál → vždy stejný štítek v rámci jednoho běhu.

🔍 Detekce více metodami: regexy + jazykové heuristiky + (volitelně) slovníky.

🧩 Morfologie (čeština): skloňování jmen pokryto pravidly (např. „Nováka“, „Novákovi“, „s Novákem“ → [[UŽIVATEL_1]]).

🧰 Konfigurovatelné: granularita štítků, whitelisty (co nezakrývat, např. veřejné subjekty), blacklisty (co vždy zakrýt).

3.2 Formát štítků

[[UŽIVATEL_{n}]], [[ADRESA_{n}]], [[EMAIL_{n}]], [[TELEFON_{n}]], [[ÚČET_{n}]], [[IBAN_{n}]], [[RČ_{n}]], [[OP_{n}]], [[PAS_{n}]], [[SPZ_{n}]], [[IČ_{n}]], [[DIČ_{n}]], [[FIRMA_{n}]], [[ID_{n}]]

n je pořadové číslo v dané kategorii, od 1.

Štítky jsou uzavřené v [[...]] kvůli snadnému hledání.

3.3 Strategie nahrazování

Vždy nahrazuj nejdelší shodu (Longest‑Match‑Wins), aby se předešlo částečným náhradám uvnitř delších entit.

U entit s vnitřní strukturou (např. IBAN) nahrazuj celek, ne po částech.

Pro víceslovné názvy (např. „Jan Karel Novák“) použij jeden štítek.

Pokud si detektor není jistý (< práh jistoty), ponech původní text a přidej varování do logu/reportu.

3.4 Citlivé kontexty

Pokud je jméno součástí citace nebo právního označení (např. „J. N.“, iniciály), anonymizuj konzistentně: „J. N.“ → [[UŽIVATEL_1]] (lze volitelně zachovat iniciály dle konfigurace).

Výstupy
Po zpracování získáte tři soubory ve složce output/:

dokument_anonymizovany. – text s nahrazenými PII štítky.

map.json – strojově čitelná mapa náhrad.

map.txt – čitelný přehled pro člověka.

4.1 map.json – specifikace

{ "version": "1.0", "generated_at": "2025-11-03T12:34:56Z", "source_file": "vstup.txt", "entities": [ {"type": "UŽIVATEL", "label": "[[UŽIVATEL_1]]", "original": "Jan Novák", "occurrences": 5}, {"type": "ADRESA", "label": "[[ADRESA_1]]", "original": "Křenová 14, 602 00 Brno", "occurrences": 2}, {"type": "ÚČET", "label": "[[ÚČET_1]]", "original": "123456789/0100", "occurrences": 1} ], "notes": ["Morfologické varianty jmen jsou sloučeny pod jeden label."] }

4.2 map.txt – specifikace

UŽIVATEL → [[UŽIVATEL_1]] : Jan Novák (výskyty: 5) ADRESA → [[ADRESA_1]] : Křenová 14, 602 00 Brno (výskyty: 2) ÚČET → [[ÚČET_1]] : 123456789/0100 (výskyty: 1) ... 5) Vstup a formáty

Textové soubory: .txt, .md, .rtf (po konverzi), .docx/.pdf (přes interní převodník – doporučeno předem převést do TXT).

Kódování: UTF‑8 doporučeno.

Jazyk: CZ/EN (ostatní jazyky fungují omezeně dle pravidel a slovníků).

VTUP příklad: Dne 12. 5. 2024 uzavřel Jan Novák, nar. 1. 1. 1988, bytem Křenová 14, 602 00 Brno, smlouvu s Papin s.r.o. Číslo účtu: 123456789/0100. Kontakt: jan.novak@example.com, +420 777 123 456. SPZ vozidla ABC1234.

VÝSTUP zkráceně: Dne 12. 5. 2024 uzavřel [[UŽIVATEL_1]], nar. [[ID_1]], bytem [[ADRESA_1]], smlouvu s Papin s.r.o. Číslo účtu: [[ÚČET_1]]. Kontakt: [[EMAIL_1]], [[TELEFON_1]]. SPZ vozidla [[SPZ_1]].

MAPA příklad: UŽIVATEL → [[UŽIVATEL_1]] : Jan Novák (výskyty: 1) ID → [[ID_1]] : 1. 1. 1988 (výskyty: 1) ADRESA → [[ADRESA_1]] : Křenová 14, 602 00 Brno (výskyty: 1) ÚČET → [[ÚČET_1]] : 123456789/0100 (výskyty: 1) EMAIL → [[EMAIL_1]] : jan.novak@example.com (výskyty: 1) TELEFON → [[TELEFON_1]] : +420 777 123 456 (výskyty: 1) SPZ → [[SPZ_1]] : ABC1234 (výskyty: 1)
