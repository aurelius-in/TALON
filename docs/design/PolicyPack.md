# Policy Pack (Thresholds & OPA)

**Customer/Vendor:** Blue Eagle Robotics (BER) / Reliable AI Network (RAIN)  
**Product:** TALON — Tactical Agentic Layer for Orchestrated eNvironments  
**Version/Date:** v0.1 • 2025‑09‑26 

---

## Purpose

This document defines threshold criteria for TALON and demonstrates how to implement them using
Open Policy Agent (OPA). Thresholds provide objective limits for key metrics (e.g., latency,
confidence) that drive decision‑making. OPA enables enforcement of these limits as code, ensuring
that infrastructure and perception jobs adhere to agreed standards.

## Threshold Definitions

Thresholds should reflect business requirements, system capabilities, and user expectations.
Values below or above the threshold trigger different outcomes (“pass,” “review,” or “fail”).

### Perception Flow

The perception pipeline returns a measured value, confidence score and latency. Thresholds
govern when results are accepted automatically or flagged for review.

| Threshold                   | Description                                               | Example value |
|----------------------------|-----------------------------------------------------------|--------------|
| **min_confidence**         | Minimum acceptable confidence score (0–1).                | 0.92         |
| **max_latency_ms**         | Maximum acceptable latency in milliseconds.               | 2,000        |
| **min_value** / **max_value** | Optional bounds for the measured value (e.g., PSI range). | 50 / 90      |

When a perception result has confidence below `min_confidence` or latency above `max_latency_ms`,
the job is not automatically considered successful. Instead it is marked for human review.

### Infrastructure Flow

Infrastructure changes are evaluated before execution. Thresholds may include:

| Threshold               | Description                                                   | Example value |
|------------------------|---------------------------------------------------------------|--------------|
| **max_plan_size**      | Maximum number of resources that a plan may add or modify.     | 50           |
| **allowed_prefixes**   | List of allowed resource name prefixes. Names outside this list cause a violation.| ["ber", "talon"] |
| **required_tags**      | Tags that must be present on all resources.                   | ["owner", "environment"] |
| **cost_diff_max**      | Maximum permitted increase in monthly cost (see example below). | \$5,000       |

These thresholds ensure that proposed changes are scoped appropriately and include essential
metadata (e.g., tags) for governance and cost reporting.

## Policy Structure

Policies are written in [Rego](https://www.openpolicyagent.org/docs/latest/policy-language/),
OPA’s declarative language. Each policy file typically defines one or more rules. In TALON’s
policy pack, rules should:

* **Check inputs** – examine fields such as `confidence`, `latency_ms`, `plan.summary`, and
  `tags`.
* **Compare against thresholds** – evaluate whether inputs meet or violate threshold values.
* **Return structured output** – produce an object (e.g., `out`) containing a message and a
  boolean indicating whether the policy failed. This pattern is commonly used to allow CI/CD
  pipelines to act on policy results.

Below are examples illustrating this pattern.

### Example: Perception Threshold Policy

This Rego policy checks if a perception result meets confidence and latency thresholds. If
either threshold is violated, the policy returns a `failed` flag and a descriptive message. If
both thresholds are satisfied, the policy allows the result to proceed.

```rego
package talon.perception

# Input expected: {
#   "confidence": number,
#   "latency_ms": number,
#   "thresholds": {
#     "min_confidence": number,
#     "max_latency_ms": number
#   }
# }

default allow := false

# deny[out] produces a failure message when thresholds are not met.
deny[out] {
  input.confidence < input.thresholds.min_confidence
  msg := sprintf(
    "Confidence %.3f is below minimum %.3f", [input.confidence, input.thresholds.min_confidence],
  )
  out := {"msg": msg, "failed": true}
}

deny[out] {
  input.latency_ms > input.thresholds.max_latency_ms
  msg := sprintf(
    "Latency %d ms exceeds maximum %d ms", [input.latency_ms, input.thresholds.max_latency_ms],
  )
  out := {"msg": msg, "failed": true}
}

# allow is true only if no deny rule generates output.
allow {
  not deny[_]
}
```

**Usage:** In your application, build an input document containing the relevant fields. After
OPA evaluation, act on `allow` and `deny` results: if `deny` is non‑empty, mark the job as
"review" or "fail"; otherwise, accept it.

### Example: Infrastructure Cost Threshold Policy

Inspired by an example from Infracost’s integration with OPA, this policy enforces a cost
diff threshold. It fails if the proposed change increases monthly cost beyond an allowed
limit. It demonstrates how to return a message and a fail flag for integration with CI/CD
systems【839862797257852†L160-L194】.

```rego
package talon.infrastructure

deny[out] {
  # Maximum permitted cost increase
  maxDiff = 5000.0

  # Extract the total monthly cost difference from input
  diff := to_number(input.diffTotalMonthlyCost)

  diff >= maxDiff

  msg := sprintf(
    "Total monthly cost diff must be less than $%.2f (actual diff is $%.2f)",
    [maxDiff, diff],
  )

  out := {"msg": msg, "failed": true}
}
```

In this example, `input.diffTotalMonthlyCost` would be provided by a cost estimation tool
(e.g., Infracost) during the plan stage. The policy checks if the diff is greater than
`maxDiff`. If so, it returns a message and sets `failed` to `true`.

### Example: Naming and Tagging Policy

This policy illustrates how to ensure resource names start with allowed prefixes and contain
required tags. It evaluates each resource in the input and accumulates violations.

```rego
package talon.infrastructure

default allow := false

# Define allowed prefixes and required tags as data or input
allowed_prefixes := {"ber", "talon"}
required_tags := ["owner", "environment"]

deny[out] {
  some resource
  resource := input.resources[_]

  # Check prefix
  not startswith(resource.name, allowed_prefixes[_])
  msg := sprintf(
    "Resource name %s must start with one of %v", [resource.name, allowed_prefixes],
  )
  out := {"msg": msg, "failed": true}
}

deny[out] {
  some resource
  resource := input.resources[_]

  # Check required tags
  missing := [t | t := required_tags[_]; not resource.tags[t]]
  count(missing) > 0
  msg := sprintf(
    "Resource %s is missing required tags %v", [resource.name, missing],
  )
  out := {"msg": msg, "failed": true}
}

allow {
  not deny[_]
}
```

This policy uses simple helper functions (`startswith`, `count`) to evaluate naming and tagging
rules. Violations generate descriptive messages, making it easy for engineers to correct issues
before applying infrastructure changes.

## Organizing the Policy Pack

Structure policies by domain (e.g., `perception`, `infrastructure`, `cost`), and place them
under a common repository directory, such as `policy/`. Each file should declare its package
name (e.g., `package talon.perception`). Keep thresholds and shared constants in separate
`data` files or embed them as inputs. Version policies using source control tags so teams
can track changes over time.

## Testing and CI/CD Integration

1. **Local testing:** Use the `opa eval` or `opa test` command to evaluate policies against
   sample inputs. Provide representative inputs and verify that policies behave as expected.
2. **CI/CD checks:** Integrate OPA into your pipeline (e.g., using [Conftest](https://www.conftest.dev/)
   or the `opa` CLI). Fail the build if any policy rules return a `failed` result.
3. **Threshold management:** Store threshold values in versioned configuration (e.g., YAML or
   JSON) and pass them as input to OPA. Avoid hard‑coding thresholds in Rego where possible to
   allow easy updates without redeploying code.

## Conclusion

Defining thresholds and codifying them with OPA policies allows TALON to enforce
consistency and guard against undesirable changes or results. The examples above illustrate
how to translate business requirements into Rego rules that produce actionable feedback for
developers and operators.
