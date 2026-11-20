# Foliolight

A free, fully local, open-source viewer that turns machine-readable security packages into something a human can read, filter, and review.

Think of it as a "PDF reader for machine-readable authorization packages."

> **Status: planning.** There is no application code yet. The plan and specifications are in [docs/PLAN.md](docs/PLAN.md). Feedback is welcome.

## Why the name

**Foliolight** joins two words:

- **Folio**: a collection of pages or documents. An authorization package is a folio: plans, assessment results, remediation records, and attachments.
- **Light**: shining a light on the package so you can read what is inside. It also describes the tool: light, fast, and local.

## Why it exists

Machine-readable authorization packages (for example, OSCAL in JSON, XML, or YAML) are better for automation, but they are hard for people to read. Agency reviewers who manage packages from many cloud service providers need one consistent, vendor-neutral way to open any provider's package and review it.

Foliolight is a **stopgap and a demonstration**. It shows how much easier review becomes with modern packages, so agencies can ask more of the tools they buy. It is not a product, has no revenue plan, and is not meant to be maintained forever.

## What it will do

- Browse a folder of packages organized by provider and document type, without parsing anything until you open it.
- Show any JSON, XML, or YAML file as a readable outline, with OSCAL-aware views for common package documents.
- Display embedded images and open linked local files safely in your default application.
- Let you build your own views for new file types with a friendly schema builder, and load ready-made view packs from partners.
- Let reviewers keep local comments that survive when a provider sends a newer version.
- Provide a command-line companion for scripting.

## What it will never do

- Send your data anywhere. No telemetry, no analytics, no accounts, no cloud features.
- Make any network connection except an optional update check that you can turn off.
- Run in the background, install services, or start automatically.
- Modify, move, or delete the package files you review.
- Run code from packages or view packs.

## Disclaimer

Foliolight is provided "as is," without warranty of any kind. It is not an official government product, is not endorsed by any government agency or program, and is not a substitute for official review of any authorization package. See the [LICENSE](LICENSE) file for the full warranty disclaimer and limitation of liability.

## Documentation

| Document | Contents |
| --- | --- |
| [Project plan](docs/PLAN.md) | Vision, scope, architecture, technology choice, roadmap, risks, assumptions |
| [Specifications](docs/specs/) | Library convention, formats, view schemas, opening files and checks, export, schema packs, comments, configuration, CLI, UI |
| [Threat model](docs/security/threat-model.md) | Security analysis and controls |
| [Release plan](docs/release/release-and-distribution.md) | Signing, distribution, and updates |
| [Test strategy](docs/testing/test-strategy.md) | Synthetic packages, hostile inputs, performance, accessibility |
| [Open questions](docs/community/open-questions.md) | Decisions we would like the community to weigh in on |
| [Licensing and governance](docs/community/licensing-and-governance.md) | License choice, contribution terms, trademark, dependency policy |

## Contributing

Contributions are welcome from anyone acting in good faith. Please read [CONTRIBUTING.md](CONTRIBUTING.md) and the [Code of Conduct](CODE_OF_CONDUCT.md). Commits must be signed off under the [Developer Certificate of Origin](DCO) (`git commit -s`). No separate agreement is required.

**Never attach real package data to issues or pull requests.** Use the synthetic samples instead.

To report a security issue, see [SECURITY.md](SECURITY.md).

## License

Licensed under the [Apache License, Version 2.0](LICENSE). Copyright The Foliolight Contributors. The license does not grant rights to the Foliolight name or logo; see [TRADEMARKS.md](TRADEMARKS.md).
