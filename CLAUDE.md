# CLAUDE.md - Da Vinci PAS Implementation Guide

## Project Overview

This repository contains the **Da Vinci Prior Authorization Support (PAS) FHIR Implementation Guide**. PAS defines a FHIR-based interface for submitting prior authorization requests from EHR systems, enabling direct integration rather than fax or payer-specific portals.

- **Package ID**: `hl7.fhir.us.davinci-pas`
- **Current Version**: 2.2.0-ballot (STU 2.2)
- **FHIR Version**: R4 (4.0.1)
- **Canonical URL**: http://hl7.org/fhir/us/davinci-pas
- **GitHub**: https://github.com/HL7/davinci-pas
- **Publisher**: HL7 International / Financial Management

## Technology Stack

- **FHIR Shorthand (FSH)**: Profile and resource definitions in `input/fsh/`
- **SUSHI**: FSH compiler that generates FHIR JSON from FSH files
- **HL7 FHIR IG Publisher**: Builds the complete implementation guide HTML
- **Jekyll**: Used for page generation (handled by IG Publisher)

## Directory Structure

```
davinci-pas/
├── input/
│   ├── fsh/                    # FHIR Shorthand source files (profiles, extensions, examples)
│   │   ├── Bundle.fsh          # Request/Response Bundle profiles
│   │   ├── Claim.fsh           # PAS Claim profiles (request, inquiry)
│   │   ├── ClaimResponse.fsh   # PAS ClaimResponse profiles
│   │   ├── Claim-update.fsh    # Claim update profiles
│   │   ├── ClaimOperation.fsh  # $submit and $inquire operations, CapabilityStatements
│   │   ├── Coverage.fsh        # Coverage profile
│   │   ├── Encounter.fsh       # Encounter profile
│   │   ├── Examples.fsh        # Example instances
│   │   ├── Metric.fsh          # Metrics profiles
│   │   ├── Organization.fsh    # Organization profiles (Requestor, Insurer)
│   │   ├── Patient.fsh         # Patient/Beneficiary profile
│   │   ├── Practitioner.fsh    # Practitioner profiles
│   │   ├── Request.fsh         # Request extensions
│   │   ├── Task.fsh            # Task profile for additional info requests
│   │   ├── Terminology.fsh     # CodeSystems and ValueSets
│   │   └── USCoreAliases.fsh   # US Core profile aliases
│   ├── pagecontent/            # Markdown documentation pages
│   │   ├── index.md            # Home page
│   │   ├── specification.md    # Technical specification
│   │   ├── usecases.md         # Use cases and overview
│   │   ├── background.md       # Technical background
│   │   ├── conformance.md      # Conformance expectations
│   │   ├── additionalinfo.md   # Request for additional info
│   │   ├── changelog.md        # Version history
│   │   └── ...
│   ├── images/                 # PNG/SVG images for documentation
│   ├── images-source/          # Source files for images (Visio, PowerPoint)
│   ├── resources/              # Additional JSON resources
│   └── ignoreWarnings.txt      # Suppressed validation warnings
├── sushi-config.yaml           # SUSHI/IG configuration
├── ig.ini                      # IG Publisher configuration
├── _genonce.sh                 # Build script (Linux/Mac)
├── _genonce.bat                # Build script (Windows)
├── _updatePublisher.sh         # Update IG Publisher script
├── publication-request.json    # HL7 publication metadata
├── FHIR-us-davinci-pas.xml     # IG artifact registry
└── oids.ini                    # OID mappings
```

## Key Concepts

### Prior Authorization Workflow

1. **Client (EHR)** constructs a PAS Request Bundle containing a Claim resource
2. Bundle is sent via `POST [base]/Claim/$submit` operation
3. **Intermediary/Payer** processes request (may convert to X12 278)
4. Response Bundle with ClaimResponse is returned
5. For pended requests, subscriptions notify of updates

### Core FHIR Profiles

| Profile | Description |
|---------|-------------|
| `PASRequestBundle` | Collection bundle for prior auth requests |
| `PASResponseBundle` | Collection bundle for prior auth responses |
| `PASClaim` | Prior authorization request |
| `PASClaimUpdate` | Updates to existing prior auth |
| `PASClaimInquiry` | Query for existing authorizations |
| `PASClaimResponse` | Prior authorization response |
| `PASBeneficiary` | Patient/member profile |
| `PASCoverage` | Insurance coverage profile |
| `PASRequestor` | Requesting organization |
| `PASInsurer` | Payer organization |

### FHIR Operations

- `$submit` - Submit a prior authorization request
- `$inquire` - Query for prior authorization status

## Build Commands

### Prerequisites

1. Install Java 11+
2. Install Node.js 18+ and npm
3. Install SUSHI: `npm install -g fsh-sushi`

### Building the IG

```bash
# Update/download IG Publisher (first time or to update)
./_updatePublisher.sh

# Build the IG
./_genonce.sh
```

The build:
1. Runs SUSHI to compile FSH to FHIR JSON (`fsh-generated/`)
2. Runs IG Publisher to generate the complete IG (`output/`)

### Output

- Generated FHIR resources: `fsh-generated/resources/`
- Final IG HTML: `output/`
- Temporary files: `temp/`

## Dependencies

| Dependency | Version | Purpose |
|------------|---------|---------|
| US Core 3.1.1 | 3.1.1 | USCDI v1 support |
| US Core 6.1.0 | 6.1.0 | USCDI v3 support |
| US Core 7.0.0 | 7.0.0 | USCDI v4 support |
| Da Vinci HRex | 1.1.0 | Common Da Vinci resources |
| Da Vinci CRD | 2.1.0 | Coverage Requirements Discovery |
| Subscriptions Backport | 1.1.0 | R5 Subscriptions for R4 |
| SDC | 3.0.0 | Structured Data Capture |

## Code Conventions

### FSH (FHIR Shorthand)

- **Profiles**: Use `Profile:` keyword, inherit from FHIR base or US Core
- **Extensions**: Use `Extension:` keyword
- **ValueSets**: Use `ValueSet:` keyword
- **CodeSystems**: Use `CodeSystem:` keyword
- **Examples**: Use `Instance:` keyword with `InstanceOf:`
- **Invariants**: Use `Invariant:` keyword for constraints
- **RuleSets**: Use `RuleSet:` for reusable constraint sets

### Naming Conventions

- Profile IDs: `profile-<resource-type>` (e.g., `profile-claim`)
- Extension IDs: `extension-<name>` (e.g., `extension-authorizationNumber`)
- ValueSet IDs: Descriptive names (e.g., `X12278DiagnosisCodes`)
- Example IDs: `<Name>Example` (e.g., `ReferralAuthorizationExample`)

### Documentation

- Page content uses GitHub-flavored Markdown
- Jekyll Liquid templates for dynamic content
- Custom note blocks: `{: .stu-note}`, `{: .modified-content}`, `{: .note-to-balloters}`

## X12 Integration

This IG maps to X12 278 (Prior Authorization Request) and 275 (Additional Documentation) transactions:
- X12 code systems referenced but not publicly available
- Validation warnings for X12 ValueSets are expected (see `ignoreWarnings.txt`)
- Mapping details published separately by ASC X12N

## Common Tasks

### Adding a New Profile

1. Create/edit FSH file in `input/fsh/`
2. Define profile with constraints
3. Add examples using `Instance:` keyword
4. Build to validate: `./_genonce.sh`

### Adding Documentation

1. Add/edit markdown in `input/pagecontent/`
2. Register page in `sushi-config.yaml` under `pages:`
3. Add to menu in `sushi-config.yaml` under `menu:`

### Adding Terminology

1. Edit `input/fsh/Terminology.fsh`
2. Define CodeSystem or ValueSet
3. Reference in profiles using `from <ValueSet> (binding-strength)`

## Validation Notes

- Some X12 code systems/value sets cannot be validated (proprietary)
- Suppressed warnings are documented in `input/ignoreWarnings.txt`
- Build output shows validation results with errors/warnings

## Related Implementation Guides

- [Coverage Requirements Discovery (CRD)](http://hl7.org/fhir/us/davinci-crd)
- [Documentation Templates and Rules (DTR)](http://hl7.org/fhir/us/davinci-dtr)
- [Clinical Data Exchange (CDex)](http://hl7.org/fhir/us/davinci-cdex)
- [Da Vinci HRex](http://hl7.org/fhir/us/davinci-hrex)

## Support Resources

- **Discussion Forum**: https://chat.fhir.org/#narrow/stream/208874-Da-Vinci-PAS
- **Project Page**: https://confluence.hl7.org/pages/viewpage.action?pageId=42993876
- **Implementer Support**: https://confluence.hl7.org/display/DVP/PAS+Implementer+Support
- **JIRA Dashboard**: https://jira.hl7.org/secure/Dashboard.jspa?selectPageId=11813
