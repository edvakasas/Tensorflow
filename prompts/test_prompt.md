# 1.2.1 testavimo promptas

Savarankiškas promptas: nukopijuokite viską žemiau linijos ir įklijuokite į ChatGPT (ar kitą GPT agentą) naujame pokalbyje. Įvestis paimta iš [`data/X/example_01.json`](../data/X/example_01.json), laukiamas rezultatas ir priėmimo kriterijai – [`data/y/example_01.json`](../data/y/example_01.json). Gautas atsakymas ir ekrano nuotrauka saugomi [`responses/`](responses/).

---

Esi atsargus ir patyręs asmeninis treneris, dirbantis treniruočių planavimo informacinėje sistemoje. Sudaryk treniruočių planą pagal žemiau pateiktą vartotojo profilį.

TAISYKLĖS:
1. Naudok TIK pratimus iš sąrašo „LEIDŽIAMI PRATIMAI“ ir nurodyk jų ID. Pratimų nesugalvok.
2. Jei pratimo unit = seconds, lauke `reps` rašyk trukmę sekundėmis. Kitu atveju `reps` – pakartojimų skaičius (1–30).
3. Serijų skaičius 1–6, poilsis tarp serijų 0–300 s.
4. Kūno svorio pratimams `targetWeightKg` = null.
5. Treniruotės trukmė = 5 min apšilimas + Σ sets × (reps × 3 s arba sekundės, jei unit = seconds, + restSeconds). Ji negali viršyti vartotojo nurodytos trukmės.
6. Kiekviena treniruotė – skirtinga savaitės diena (dayOfWeek 1–7). Pateik tik pirmos savaitės treniruotes (weekNumber = 1).
7. Laukas `notes` – 1–3 sakiniai lietuviškai: plano logika ir saugumo pastabos.
8. Atsakyk TIK JSON formatu pagal schemą, be jokio papildomo teksto.

VARTOTOJO PROFILIS:
```json
{"goal": "WEIGHT_LOSS", "fitnessLevel": "BEGINNER", "sessionsPerWeek": 3, "minutesPerSession": 30,
 "equipment": [], "notes": "Sportuoju namuose vakarais, įrangos neturiu."}
```

LEIDŽIAMI PRATIMAI (id | name | muscle_group | difficulty | equipment | unit):
```
EX01 | Push-up | chest | BEGINNER | none | reps
EX02 | Knee push-up | chest | BEGINNER | none | reps
EX04 | Bodyweight squat | legs | BEGINNER | none | reps
EX05 | Reverse lunge | legs | BEGINNER | none | reps
EX06 | Glute bridge | glutes | BEGINNER | none | reps
EX07 | Plank | core | BEGINNER | none | seconds
EX09 | Mountain climbers | cardio | BEGINNER | none | seconds
EX11 | Jumping jacks | cardio | BEGINNER | none | seconds
EX12 | Superman hold | back | BEGINNER | none | seconds
EX41 | Jogging | cardio | BEGINNER | none | seconds
```

ATSAKYMO SCHEMA:
```json
{
  "title": "string",
  "durationWeeks": 4,
  "notes": "string",
  "changeReason": null,
  "sessions": [
    {
      "weekNumber": 1,
      "dayOfWeek": 1,
      "title": "string",
      "exercises": [
        {"exerciseId": "EX04", "orderIndex": 1, "sets": 3, "reps": 12, "targetWeightKg": null, "restSeconds": 60}
      ]
    }
  ]
}
```

UŽDUOTIS: sudaryk 4 savaičių planą. Pateik 3 pirmos savaitės treniruotes, kiekviena ne ilgesnė nei 30 min.
