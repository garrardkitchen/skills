# Semantic Kernel and Azure AI Security

Use this file when the user asks about Semantic Kernel, Azure OpenAI, Azure AI Foundry, function/tool calling, structured outputs, model deployment, or AI service integration in `.NET`.

## Must implement

- Keep system/developer instructions separate from user and retrieved content.
- Use explicit plugin/function allowlists.
- Validate function arguments before execution.
- Authorize downstream actions independently of model output.
- Prefer structured output schemas for machine-consumed responses.
- Apply content safety, abuse monitoring, rate limits, and cost controls.
- Use managed identity where Azure AI services support it.

## Function invocation guard

```csharp
public sealed class FunctionCallPolicy
{
    private static readonly HashSet<string> AllowedFunctions = new(StringComparer.Ordinal)
    {
        "orders.lookup",
        "tickets.create-draft"
    };

    public static void EnsureAllowed(string functionName)
    {
        if (!AllowedFunctions.Contains(functionName))
            throw new InvalidOperationException("Function is not allowlisted.");
    }
}
```

## Structured output model

```csharp
public sealed record SupportRecommendation(
    string Summary,
    string[] EvidenceIds,
    bool RequiresHumanApproval);
```

Model output used by code should be parsed, validated, and rejected when required fields, evidence, or approval flags are missing.

## Verification

- unapproved plugin/function is rejected
- malformed function arguments are rejected
- destructive action requires human approval
- model output without evidence is not promoted
- rate/cost limits stop runaway loops
- telemetry records model, prompt version, tool call, and correlation ID without leaking secrets

## Avoid

- auto-invoking every public method as a tool
- using model output directly as SQL, shell commands, URLs, or authorization decisions
- storing prompts or outputs containing personal data without retention controls
- giving one broad credential to all plugins

## Sources

- Microsoft Semantic Kernel documentation
- Microsoft Azure OpenAI and Azure AI Foundry security guidance
- OWASP Top 10 for Agentic Applications
- Azure Agentic AI Security Baseline
