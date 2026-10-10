# Syntax as Semantics Prompting (SaSP)

*Prompt constraints, written in the syntax that expresses them.*

[English](README.md) · [Italiano](README.it.md)

<!-- Zenodo DOI badge: add after the first release -->

SaSP is a notation and methodology for writing prompt constraints explicitly: it makes visible the priorities, scopes, conditions, and output formats that stay implicit in prose. It is meant for people who maintain complex LLM prompts over time, with many rules of different weight, structured outputs, or multi-stage workflows.

SaSP adopts four notations from widely used web languages. A SaSP prompt is not code and does not need to be valid. It is presented as an open proposal, not as a proven method: see Chapter 8 of the "Method description" document for its limitations.

| If the constraint concerns... | Write it in... |
| --- | --- |
| the **structure** of the prompt | **XML** |
| **prescriptions**: mandatory, forbidden, priority, scope | **CSS** |
| the **format of structured data** and where the output goes | **TypeScript** |
| the **workflow**: stages, conditions, points to wait for the user | **JavaScript** |

## Example

A complete SaSP prompt that evaluates customer feedback sentiment and emits strict JSON:

```xml
<system_prompt>
  <identity>
    role: "customer feedback analyst";
    specialization: "sentiment extraction and categorization";
  </identity>
  <objective>
    goal: "evaluate customer feedback sentiment and flag cases requiring escalation";
  </objective>
  <context>
    source: "customer written feedback";
    audience: "customer care team";
  </context>
  <contracts>
    type EgressChannel = "chat_stream" | "artifact_file" | "workspace_source";

    interface Deliverable {
      egress: EgressChannel;
    }

    interface FeedbackAnalysis extends Deliverable {
      egress: "chat_stream";
      sentiment: "positive" | "neutral" | "negative";
      score: number; // 1 to 10
      keyTopics: string[];
      needsEscalation: boolean;
    }
  </contracts>
  <specifications>
    rule[priority="100"] {
      language: english !important;
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
            ctx.sentiment === "negative" && ctx.reportsDamageOrUnansweredSupport
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

Each constraint lives in one place: role, objective, and context as attributes; language, format, and tone in the specifications; the output schema and its channel in the contracts; the escalation logic in the workflow. The `EMIT` state produces `FeedbackAnalysis`, and the format rule applies only to that stage (`scope="EMIT"`). Chapter 5 of the document "Method description" explains the example in more detail.

## Contents

- **Method description:** the full notation and methodology [English](Syntax_as_Semantics_Prompting_-_SaSP_en.md) · [Italiano](Syntax_as_Semantics_Prompting_-_SaSP_it.md)
- **Transpiler:** [`transpiler/sasp_transpiler.en.xml`](transpiler/sasp_transpiler.en.xml) · [`transpiler/sasp_transpiler.it.xml`](transpiler/sasp_transpiler.it.xml): a SaSP prompt that helps converting natural-language specifications into SaSP prompts. Always review its output.
- **Transpiler Skill**: [`transpiler/skill/sasp-transpiler`](transpiler/skill/sasp-transpiler/SKILL.md): a SaSP skill that helps converting natural-language specifications into SaSP prompts. Always review its output.

## Citing

If you use this work, please cite it using the "Cite this repository" button on GitHub (see `CITATION.cff`).

## Contributing

Results from your own use are the most valuable contribution: Chapter 9 of the "Method description" document explains how to compare SaSP with prose in your own case. Open a pull request to share syntactic patterns or adaptations that worked for you, or an issue to discuss ideas.

## License

[CC BY 4.0](LICENSE)
