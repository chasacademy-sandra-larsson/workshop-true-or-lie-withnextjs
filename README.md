# Workshop: Fakta eller påhitt? 🤯

Bygg en AI-driven webbapp där spelaren ska avgöra om ett påstående är sant eller påhittat. AI:n (Google Gemini) genererar påståendena!

## Kom igång

### 1. Skaffa API-nyckel

Gå till [Google AI Studio](https://aistudio.google.com/app/apikey), logga in med ett Google-konto och klicka "Create API Key".

### 2. Skapa ett nytt Next.js-projekt

```bash
npx create-next-app@latest fakta-eller-pahitt
```

Välj rekommonderade inställningar och App-router. 


### 3. Installera Gemini-paketet

```bash
npm install @google/genai
```

### 4. Skapa env-fil

Skapa filen `.env.local` i projektets rot och lägg till:

```
GEMINI_API_KEY=din-api-nyckel
```

### 5. Starta utvecklingsservern

```bash
npm run dev
```

Öppna [http://localhost:3000](http://localhost:3000) i webbläsaren.

---

## API-route — serverkomponent (färdig kod)

Skapa filen `src/app/api/generate/route.ts` och klistra in denna kod:

```typescript
import { GoogleGenAI } from "@google/genai";
import { NextRequest, NextResponse } from "next/server";

const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY! });

export async function POST(request: NextRequest) {
  const { topic } = await request.json();

  const shouldBeTrue = Math.random() > 0.5;

  const prompt = `Ge mig ett ${shouldBeTrue ? "SANT" : "FALSKT och påhittat"} påstående om ${topic}. Svara BARA med JSON i exakt detta format, inget annat: { "statement": "...", "isTrue": ${shouldBeTrue} }`;

  try {
    const response = await ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: prompt,
    });

    const text = response.text?.trim() || "";

    // Ta bort eventuella markdown-backticks
    const jsonString = text
      .replace(/```json\n?/g, "")
      .replace(/```\n?/g, "")
      .trim();
    const data = JSON.parse(jsonString);

    return NextResponse.json({
      statement: data.statement,
      isTrue: data.isTrue,
    });
  } catch (error) {
    console.error("Gemini API error:", error);
    return NextResponse.json(
      { error: "Kunde inte generera påstående" },
      { status: 500 }
    );
  }
}
```

API:et tar emot en POST-request med `{ topic: "animals" }` och returnerar `{ statement: "...", isTrue: true/false }`.

---

## Uppgifter

### 1. Bygg en startsida — klientkomponent (`src/app/page.tsx`)

1. Visa en titel och en kort beskrivning av spelet
2. Skapa knappar eller kort där spelaren kan välja ämne (t.ex. djur, rymden, historia, mat)
3. När ett ämne väljs — navigera till `/game/[topic]` (t.ex. `/game/animals`)

**Tips:** Använd `useRouter` från `next/navigation` för att navigera programmatiskt:

```typescript
import { useRouter } from "next/navigation";

const router = useRouter();
router.push("/game/animals");
```

Kom ihåg att komponenter med interaktivitet (onClick, useState) behöver `"use client"` högst upp i filen.

---

### 2. Bygg spelsidan — klientkomponent (`src/app/game/[topic]/page.tsx`)

1. Hämta ett påstående från `/api/generate` när sidan laddas (använd `fetch` med POST)
2. Visa påståendet
3. Skapa två knappar: "Fakta" och "Påhitt" — jämför spelarens svar med `isTrue`
4. Visa feedback (rätt/fel) efter varje svar
5. Lägg till en "Nästa fråga"-knapp som hämtar ett nytt påstående


**Filer att skapa:** `src/app/game/[topic]/page.tsx`

**Tips:** Använd `useState` för att hålla koll på state (påstående, feedback). Använd `useEffect` för att hämta första frågan. Så här anropar du API:et:

```typescript
const res = await fetch("/api/generate", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ topic: "animals" }),
});
const data = await res.json();
// data.statement = "Sandra Bullock är 50 år"
// data.isTrue = false
```

---

## Diskutera när ni är klara

1. Vad är skillnaden mellan klient- och serverkomponenter i Next.js? Vilka av era filer är vilka?
2. Varför ligger API-anropet till Gemini i en API-route istället för direkt i klientkomponenten?
3. Hur skulle ni kunna använda server-komponenter för att förbättra prestandan?
