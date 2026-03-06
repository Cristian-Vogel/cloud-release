# Why the GitHub Action and the CLI Are Separate

While they might look like they do the same thing on the surface, they each have a distinct purpose and work together to give us a smooth CI/CD experience.

## 1️⃣ The GitHub Action Is a Wrapper

Think of the `cloud-release` GitHub Action as a convenience layer around the `cn` CLI. It handles all the “plumbing” that would otherwise require a bunch of manual steps in the workflow file:

- **Environment detection** – automatically picks the right binary for Linux (x64/ARM), macOS, or Windows.
- **Dependency management** – downloads the correct version of the `cn` CLI for the detected OS.
- **Permissions** – makes sure the binary is executable (`chmod +x`).
- **Configuration** – maps your GitHub secrets (e.g., `api-key`) straight to the CLI’s expected inputs.

Without this action we’d end up writing 10‑15 lines of `curl` and conditional logic in every repo.

## 2️⃣ Interoperability (Why We Need Output Variables)

The `cn release draft` command prints a JSON object to stdout, but GitHub Actions can’t read that object directly. In the GitHub Actions world:

- **Stdout** is just human‑readable text in the logs.
- **Outputs** (e.g., `${{ steps.draft.outputs.release-id }}`) are special variables that later steps can consume.

The action captures the CLI’s text output, extracts the hidden JSON, and explicitly tells GitHub *“this piece of text is the Release ID – store it for the next step.”* That bridge lets us use the data downstream without writing custom parsers.

## 3️⃣ Separation of Concerns

- **The CLI (`cn`)** – the engine. It contains the core logic for talking to CrabNebula Cloud’s API and can be used directly in a terminal or script.
- **The Action (`cloud-release`)** – the UI/integration. It adapts the engine to the GitHub environment, exposing its functionality as native workflow variables.

### TL;DR

The CLI does the heavy lifting. The Action makes that work discoverable and usable as variables within a GitHub workflow, sparing us from writing custom parsing scripts in every project.