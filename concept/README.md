<p align="center">CAE Concept</p>

---

<br>

### Clean Use Cases

An application capability brings several operations together to achieve an outcome. Creating an account, for example, may require checking whether the account already exists, applying creation rules, and saving the new account.

A **use case** expresses that capability as a workflow with an explicit contract, an implementation, and declared dependencies. The use-case implementation decides which operations are needed, when to perform them, and what their results mean for the requested outcome.

The use case's contract remains applicable whether the caller is an HTTP endpoint, command, or scheduled task.

<br>

#### Direct Use

Application code can invoke an assembled use case directly through its execution API, supplying business input when required and an [execution context](https://github.com/clean-arch-enablers-project/clean-execution-context/blob/feature/hello-world/concept/README.md). A web framework or dependency injection container can host the use case, but the capability's contract does not depend on either.

The caller invokes the use-case execution API rather than bypassing it to call the business implementation directly. That execution boundary coordinates the workflow lifecycle, recorded execution information, and configured autofeatures around the business work.

<br>

#### Contract and Implementation

The use-case contract defines the business input accepted and output produced.

The use-case implementation defines how the workflow achieves the application outcome: applying rules, coordinating dependencies, and interpreting their results.

Four contract shapes cover the presence or absence of business input and output:

- **Function**: input and output
- **Consumer**: input only
- **Supplier**: output only
- **Runnable**: neither input nor output

[Execution context](https://github.com/clean-arch-enablers-project/clean-execution-context/blob/feature/hello-world/concept/README.md) accompanies the invocation separately from business input. It carries correlation, actor information, and execution observations.

The absence of business input or output does not remove the execution context. Supplier and runnable use cases still participate in execution tracking and configured lifecycle behavior. A consumer or runnable use case communicates successful completion or failure even though its contract returns no business result.

<br>

#### Input Requirements

A use-case input model describes the data accepted by the capability. The input model can also declare requirements such as required values, nonempty collections, nonblank text, and validation of nested input models. Application-defined validation expresses additional input rules.

For example, ***Create Account*** may require a supplied account name containing meaningful text. Those input requirements are checked before the protected business workflow proceeds. Rules requiring dependency results, such as whether an account already exists, remain part of the use-case workflow.

The **autoverify** component supplied by the separate [Clean Autofeatures](https://github.com/clean-arch-enablers-project/clean-autofeatures/blob/feature/hello-world/concept/README.md) concept interprets the input-validation declarations. The use-case input model declares its requirements; the composed validation component performs the checks.

<br>

#### Capability Meaning and Boundaries

The separate [Clean Ontology](https://github.com/clean-arch-enablers-project/clean-ontology/blob/feature/hello-world/concept/README.md) concept supplies declarations describing a use case's purpose and participation:

- **Edge use case:** a capability intended for end users or other external actors.
- **Internal use case:** a capability implementing system mechanics, including event-driven synchronization between subdomains.

For example, the edge ***Create Account*** capability may publish an event that leads another application to execute the internal ***Synchronize Membership*** capability. Each local workflow is an **inner flow** within its application. The broader process connecting those workflows is an **outer flow**, as described by [Clean Execution Context](https://github.com/clean-arch-enablers-project/clean-execution-context/blob/feature/hello-world/concept/README.md#execution-identity-versus-flow-identity).

Use-case **metadata**, the information describing the capability, can also declare affected resource-owner types, read-only or manipulation operations, and authorization requirements. An affected owner type identifies whose resources the capability concerns; operation type distinguishes reading information from creating, changing, or deleting it. The separate [Clean Ontology](https://github.com/clean-arch-enablers-project/clean-ontology/blob/feature/hello-world/concept/README.md) concept defines these declarations.

Together, these declarations describe what the capability means and where it participates. Edge or internal describes the capability's purpose; function, consumer, supplier, or runnable describes its business input/output shape.

<br>

#### Ports

A port declares an operation the workflow requires from a dependency.

For example, retrieving an account by identifier is a dependency contract. The use case decides when to request it and what its outcome means for the workflow.

Dependency ports can use the same function, consumer, supplier, and runnable input/output shapes as use cases. A use case expresses the overall capability; a dependency port expresses an operation needed to fulfill it.

The use-case implementation depends on the port contract, while an adapter supplies the dependency behavior. This allows the same workflow to run with a database adapter, a remote-service adapter, or a test implementation of the required operation.

Calling a port's execution API also creates a tracked step in the current inner flow. The port execution records the operation's name, timing, outcome, and applicable input/output information, and provides a boundary for handling dependency failures. The adapter implements the dependency operation inside that boundary.

<br>

#### Adapters and Assembly

An adapter implements a port using a particular technology or external system.

The port defines the required operation. The adapter translates between that contract and the concrete dependency, such as a database or remote API.

An assembler constructs the use case with its selected implementations and configuration. This keeps dependency selection explicit and allows different adapters to fulfill the same contract.

An assembler can expose a default assembled use case through [lazy initialization](https://github.com/clean-arch-enablers-project/clean-initializers/blob/feature/hello-world/concept/README.md), constructing the use case when first accessed and retaining it for reuse. Reusing an assembled use-case instance does not mean reusing one mutable execution record across separate invocations.

Application composition belongs to the separate [Clean Setup](https://github.com/clean-arch-enablers-project/clean-setup/blob/feature/hello-world/concept/README.md) concept. Clean Use Cases depends on [Clean Setup](https://github.com/clean-arch-enablers-project/clean-setup/blob/feature/hello-world/concept/README.md) for the **autofeatures registry**: the selected and configured autofeature components that participate in workflow execution.

<br>

#### Use-Case Registration

A **use-case registry** makes assembled use-case instances available for discovery by application components and tooling. Registration associates the available capabilities with their use-case metadata, and interested components can be notified when a use case is registered.

The use-case registry identifies available capabilities. The autofeatures registry composed through [Clean Setup](https://github.com/clean-arch-enablers-project/clean-setup/blob/feature/hello-world/concept/README.md) identifies operational components used during execution. These registries serve different purposes; registering a use case does not by itself publish an endpoint or grant permission to invoke it.

<br>

#### Execution Lifecycle

A use-case invocation represents one inner-flow execution. Its [execution context](https://github.com/clean-arch-enablers-project/clean-execution-context/blob/feature/hello-world/concept/README.md) carries the correlation identifier, actor information, outer-flow identity, and metadata for that inner flow.

The use-case execution boundary coordinates the following work:

1. Start the inner-flow execution record and make applicable input information available.
2. Invoke configured pre-run autofeatures, including input verification, authorization, and eligible cache lookup. A failed precondition prevents the protected business operation from running; a cache hit can supply the result without repeating that operation.
3. Execute the business workflow when needed, invoking ports and recording participating operations. Configured during-run retry behavior applies at its declared boundary.
4. Record the inner flow's success or failure, end time, applicable output, and exception when present.
5. Coordinate flow-resource completion hooks and configured post-run autofeatures using the recorded outcome.

Port invocations contribute step unit metadata to the inner flow. Internal operations and ordinary method calls can also be recorded as steps without becoming dependency ports. A recovered step failure can coexist with a successful overall workflow outcome.

Operations within the use case can access resources shared within the inner flow through the [execution context](https://github.com/clean-arch-enablers-project/clean-execution-context/blob/feature/hello-world/concept/README.md). For example, several database operations may share one transaction whose completion actions commit or roll back according to the workflow's outcome. Resource ownership and completion-hook behavior remain explicit, including who releases the resource.

<br>

#### Autofeature Participation

[Clean Autofeatures](https://github.com/clean-arch-enablers-project/clean-autofeatures/blob/feature/hello-world/concept/README.md) provides reusable operational components. [Clean Setup](https://github.com/clean-arch-enablers-project/clean-setup/blob/feature/hello-world/concept/README.md) selects and configures those components; the use-case execution lifecycle invokes the registered components at their participation phases.

- **Pre-run:** autoverify checks input declarations, autoauth evaluates access requirements, and autocache looks for an eligible reusable result.
- **During-run:** autoretry applies declared retry policies to use cases or dependency ports, using [Clean Trier](https://github.com/clean-arch-enablers-project/clean-trier/blob/feature/hello-world/concept/README.md) as the underlying retry mechanism.
- **Post-run:** autolog, autometrics, and autonotify consume completed workflow and step information to produce logs, measurements, and operational notifications.
- **During-delivery:** autodoc inspects use-case contracts, declarations, and implementations to generate documentation as part of software delivery, separately from individual workflow invocations.

Caching can apply to a result-producing use case or an individual result-producing port. The cache boundary determines which work is skipped when a cached result is reused. Retry declarations likewise distinguish repeating one dependency operation from repeating the broader workflow, including earlier side effects.

The use-case implementation coordinates business decisions while the configured autofeature components provide the selected operational behavior. The separate [Clean Autofeatures](https://github.com/clean-arch-enablers-project/clean-autofeatures/blob/feature/hello-world/concept/README.md) concept defines each autofeature's behavior in detail.

<br>

#### Failure Handling

[Mapped exceptions](https://github.com/clean-arch-enablers-project/clean-mapped-exceptions/blob/feature/hello-world/concept/README.md) communicate workflow failures and retain their meaning as they propagate.

Unexpected exceptions are translated at the relevant execution boundary, preserving their original causes. Translation gives the failure meaning; retry or fallback policies decide whether recovery is possible.

A port boundary identifies the dependency operation that failed. A use-case boundary identifies the overall capability that failed. An already mapped failure propagates with its existing meaning; an unmapped exception receives an appropriate execution failure representation containing the original cause and the relevant operation's identity.

Use-case execution tracking records both successful and failed workflow outcomes. Completion actions should have explicit failure behavior so their own errors do not accidentally obscure the workflow's original failure.

<br>

#### An Explicit Workflow and Its Surroundings

Together, the use-case contract and implementation express the application capability, dependency ports declare the operations it needs, and adapters supply those operations. Assembly selects the concrete dependencies, while the execution lifecycle connects the workflow to recorded metadata, shared resources, and configured autofeatures.

These responsibilities keep the business workflow understandable as an application outcome while making its dependencies and operational behavior explicit.

<br>
<br>

---

<p align="center">clean software, made easy.</p>
