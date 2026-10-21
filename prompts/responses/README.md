# GPT agento atsakymai

Čia saugomi atsakymai, gauti pateikus [`../test_prompt.md`](../test_prompt.md) GPT agentui (ChatGPT):

- [`chatgpt_example_01.json`](chatgpt_example_01.json) – ChatGPT grąžintas JSON planas;
- `chatgpt_example_01.png` – pokalbio ekrano nuotrauka.

## Įvertinimas

Atsakymas patikrintas tuo pačiu `ResponseValidator` ir tais pačiais priėmimo kriterijais kaip Gemini rezultatai Colab notebook'e ([`../../data/y/example_01.json`](../../data/y/example_01.json)):

| Patikra | Rezultatas |
|---|---|
| JSON atitinka schemą, visi pratimų ID leidžiami | ✅ validus |
| `sessions_per_week` = 3 | ✅ |
| `max_session_minutes` ≤ 30 (apskaičiuota ~19, ~21, ~21 min) | ✅ |
| `allowed_equipment` – tik pratimai be įrangos | ✅ |
| `max_difficulty` – tik BEGINNER | ✅ |
| `notes_present` | ✅ |

**Rezultatas: 5/5 kriterijų.** Palyginus su Gemini: abu modeliai laikėsi visų taisyklių; ChatGPT pasirinko trumpesnį poilsį (30 s) ir trumpesnes treniruotes (~20 min iš leidžiamų 30).
