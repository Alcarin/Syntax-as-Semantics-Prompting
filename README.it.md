# Syntax as Semantics Prompting (SaSP)

*I vincoli di un prompt, scritti nella sintassi che li esprime.*

[English](README.md) · [Italiano](README.it.md)

[![DOI](https://zenodo.org/badge/1382252747.svg)](https://doi.org/10.5281/zenodo.23286981)

SaSP è una notazione e un metodo per scrivere i vincoli di un prompt in modo esplicito: rende visibili priorità, ambiti, condizioni e formati di uscita che nella prosa restano impliciti. È pensato per chi mantiene nel tempo prompt complessi per LLM, con molte regole di peso diverso, output strutturati o flussi a più fasi.

SaSP adotta quattro notazioni derivate da linguaggi diffusi sul web. Un prompt SaSP non è codice e non deve essere valido. È presentato come proposta aperta e non come metodo dimostrato: i limiti sono nel capitolo 8 del documento "Descrizione del metodo".

| Se il vincolo riguarda... | Scrivilo in... |
| --- | --- |
| la **struttura** del prompt | **XML** |
| le **prescrizioni**: obblighi, divieti, priorità, ambito | **CSS** |
| il **formato dei dati strutturati** e dove va l'output | **TypeScript** |
| il **flusso**: fasi, condizioni, punti in cui attendere l'utente | **JavaScript** |

## Esempio

Un prompt SaSP completo che valuta il sentiment di un feedback cliente e restituisce un JSON:

```xml
<system_prompt>
  <identity>
    role: "analista di feedback clienti";
    specialization: "estrazione del sentiment e categorizzazione";
  </identity>
  <objective>
    goal: "valutare il sentiment di un feedback e segnalare i casi da gestire";
  </objective>
  <context>
    source: "feedback scritti dai clienti";
    audience: "team di customer care";
  </context>
  <contracts>
    type EgressChannel = "chat_stream" | "artifact_file" | "workspace_source";

    interface Deliverable {
      egress: EgressChannel;
    }

    interface FeedbackAnalysis extends Deliverable {
      egress: "chat_stream";
      sentiment: "positivo" | "neutro" | "negativo";
      score: number; // da 1 a 10
      keyTopics: string[];
      needsEscalation: boolean;
    }
  </contracts>
  <specifications>
    rule[priority="100"] {
      language: italian !important;
    }
    rule[priority="100"][scope="EMIT"] {
      format: json-only !important;
    }
  </specifications>
  <workflow>
    const workflow = {
      initialState: "ANALYZE",
      states: {
        ANALYZE: {
          action: "classify_sentiment_score_and_extract_topics",
          transition: (ctx) =>
            ctx.sentiment === "negativo" && ctx.reportsDamageOrUnansweredSupport
              ? "ESCALATE"
              : "EMIT"
        },
        ESCALATE: {
          action: "set_needs_escalation_true",
          transition: () => "EMIT"
        },
        EMIT: {
          action: "return_analysis",
          deliverable: "FeedbackAnalysis"
        }
      }
    };
  </workflow>
</system_prompt>
```

Ogni vincolo compare in un solo punto: ruolo, obiettivo e contesto negli attributi; lingua, formato e tono nelle specifiche; la forma dei dati e il canale di uscita nel contratto; la logica di escalation nel flusso. La fase `EMIT` consegna `FeedbackAnalysis`, e la regola sul formato vale solo in quella fase (`scope="EMIT"`). Il capitolo 5 del documento "Descrizione del metodo" spiega l'esempio più in dettaglio.

## Contenuti

- **Descrizione del metodo:** la notazione e il metodo per intero [Italiano](Syntax_as_Semantics_Prompting_-_SaSP.it.md) · [English](Syntax_as_Semantics_Prompting_-_SaSP.en.md)
- **Transpiler:** [`transpiler/sasp_transpiler.en.xml`](transpiler/sasp_transpiler.en.xml) · [`transpiler/sasp_transpiler.it.xml`](transpiler/sasp_transpiler.it.xml): un prompt SaSP che aiuta nella conversione di specifiche da linguaggio naturale in prompt SaSP. Rileggi sempre il risultato.
- **Transpiler Skill**: [`transpiler/skill/sasp-transpiler`](transpiler/skill/sasp-transpiler/SKILL.md): una skill scritta in SaSP che aiuta nella conversione di specifiche da linguaggio naturale in prompt SaSP. Rileggi sempre il risultato.

## Come citare

Se usi questo lavoro, citalo con il pulsante "Cite this repository" di GitHub (vedi `CITATION.cff`).

## Contribuire

I risultati della tua esperienza d'uso sono il contributo più prezioso: il capitolo 9 del documento "Descrizione del metodo" spiega come confrontare SaSP con la prosa nel tuo caso. Apri una pull request per proporre pattern sintattici o adattamenti che hanno funzionato per te, o una issue per discutere idee.

## Licenza

[CC BY 4.0](LICENSE)
