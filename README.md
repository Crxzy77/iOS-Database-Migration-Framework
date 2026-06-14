[Releases](https://github.com/Crxzy77/iOS-Database-Migration-Framework/releases)

# ⚡ iOS Database Migration Framework for Core Data and SwiftData

[![Releases](https://img.shields.io/badge/releases-View%20releases-blue?style=for-the-badge)](https://github.com/Crxzy77/iOS-Database-Migration-Framework/releases)

Welcome to a comprehensive framework for advanced database migrations on Apple platforms. This project provides versioned migrations, rollback capabilities, and data integrity tooling that work with Core Data and SwiftData. It focuses on reliability, performance, and a clear migration lifecycle that keeps user data safe during model evolutions. It is designed for teams that require disciplined migration flows, audit trails, and robust failure handling in production apps.

If you are here for the latest artifacts and release notes, the Releases page is the best starting point. You can find installers or packaged artifacts tailored for macOS and iOS environments there. From the Releases page, download the installer package or release asset and run it to set up the framework in your workspace. For convenience, you can also browse the repository’s releases from the top-level link provided, which will take you to the same page.

Contents
- About the project
- Core ideas and design principles
- Features
- Architecture and components
- Getting started
- Installation
- Quickstart with Core Data
- Quickstart with SwiftData
- Migration lifecycle and strategies
- Version control, rollback, and auditing
- Data integrity and validation
- Advanced topics
- API reference
- Testing, CI, and quality
- Examples and patterns
- Contributing
- Licensing

About the project
This framework is built to help iOS developers evolve their data models without risking data loss or inconsistent states. It brings a version-controlled migration system to Core Data and SwiftData, with reliable rollback support, thorough validation, and extensible hooks. It is designed to be used in apps ranging from small consumer apps to large enterprise systems that depend on stable data evolution.

Core ideas
- Versioned migrations: Each migration has a version and a deterministic plan.
- Rollback: If a migration fails or a user needs to revert, you can roll back to a prior stable version.
- Data integrity first: Validation steps verify row counts, data types, and cross-table relationships.
- Multiple data stores: Works with Core Data stores and SwiftData containers, with adapters to bridge to the migration engine.
- Auditability: Every migration is recorded with context to support debugging and compliance needs.
- Non-disruptive operations: Migrations are designed for live apps, with dry-run capabilities and minimal downtime where possible.
- Extensibility: Add custom steps, checks, and adapters as your data model evolves.

Features
- Versioned migration plans with explicit from/to versions
- Declarative migration steps that transform data safely
- Rollback support to a previous, known-good version
- Data integrity checks before, during, and after migrations
- Core Data adapter for model evolution workflows
- SwiftData adapter for modern Swift data stores
- Dry-run mode to test migrations without applying changes
- Conflict resolution strategies for schema and data conflicts
- Change auditing and exportable migration logs
- Configurable validation rules and constraints
- Integration with Swift Package Manager (SPM) for easy adoption
- Testing helpers and fixtures to simulate migrations in CI
- Production-friendly error handling and fallback paths
- Lightweight, modular architecture that scales with project size

Architecture and components
- MigrationEngine: The central orchestrator that coordinates plans, adapters, and validators.
- MigrationPlan: A sequence of steps that evolves the data model from a source version to a target version.
- CoreDataAdapter: Bridges Core Data stores to the migration engine, handling model mappings and store access.
- SwiftDataAdapter: Bridges SwiftData containers with migration hooks and transformations.
- VersionStore: Persisted record of applied migrations, current version, and rollback points.
- DataIntegrityValidator: Runs checks to ensure data remains consistent across steps.
- MigrationContext: Carries metadata, helpers, and shared state for each migration step.
- ChangeLog and AuditLog: Tracks who, when, and why migrations occurred.
- Hooks and Extensions: Custom blocks that run before, during, or after migrations.

Domain models and data safety
- Migrations are defined as explicit steps with idempotent transformations when possible.
- Steps can be composed to form complex transformations or split into smaller, testable units.
- Validation runs both pre- and post-migration, including checksums, row counts, and relationship integrity.
- Rollbacks restore the exact prior state using the version history and a reversible set of steps whenever feasible.
- For Core Data, model versioning aligns with standard model versioning practices, and the engine provides a pathway to migrate underlying persistent stores predictably.
- For SwiftData, container migrations occur through a compatible adapter layer that preserves data and relationships.

Getting started
This guide helps you adapt the framework to your project. The steps cover project setup, integration with Swift Package Manager (SPM), and a minimal working example to confirm the migration pipeline is functional.

Prerequisites
- macOS with Xcode for building and testing
- Swift 5.5 or newer (compatibility is maintained with recent Xcode versions)
- Core Data and/or SwiftData familiarity
- Basic knowledge of migrations, schema changes, and data transformations
- An existing data model or schema to evolve

Installation
To add the framework to your project, use Swift Package Manager or your preferred dependency manager. The typical process with SPM is straightforward:

- In Xcode, open your project, go to File > Swift Packages > Add Package Dependency.
- Enter the repository URL: https://github.com/Crxzy77/iOS-Database-Migration-Framework.git
- Choose the latest major version compatible with your app and add the library target to your app.
- Import the framework in code:

import IOSDatabaseMigration

Note: The Releases page may host installer artifacts for manual setups or platform-specific binaries. From the Releases page, download the installer package or release asset and run it to install the framework into your environment. If you encounter issues during installation, consult the Releases section for known-good binaries and compatibility notes.

Quickstart with Core Data
The Core Data path demonstrates how to initialize the migration engine and apply a plan to a Core Data store. The example shows a minimal, but representative, migration flow.

Code example (Swift)
import CoreData
import IOSDatabaseMigration

// Create a migration engine
let engine = MigrationEngine()

// Define a source Core Data store
let modelName = "AppModel"
let storeURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!.appendingPathComponent("AppModel.sqlite")

let coreDataAdapter = CoreDataAdapter(modelName: modelName, storeURL: storeURL)
engine.register(adapter: coreDataAdapter)

// Define a migration plan
let plan = MigrationPlan(fromVersion: "1.0.0", toVersion: "1.1.0")
plan.addStep(MigrationStep(name: "AddEmailIndex") { context in
    // Example transformation: create an index or update a field
    context.executeSQL("CREATE INDEX IF NOT EXISTS idx_user_email ON User(email);")
})

// Run a dry-run first to validate
engine.performMigration(plan: plan, dryRun: true) { result in
    switch result {
    case .success:
        print("Dry-run ready. Applying migration...")
        // Apply for real
        engine.performMigration(plan: plan, dryRun: false) { runResult in
            switch runResult {
            case .success:
                print("Migration completed.")
            case .failure(let error):
                print("Migration failed: \(error.localizedDescription)")
            }
        }
    case .failure(let error):
        print("Dry-run failed: \(error.localizedDescription)")
    }
}

Notes
- The CoreDataAdapter handles model versioning, mapping between old and new stores, and ensures the underlying store is migrated without data loss.
- You can add pre- and post-migration hooks to run custom code, such as data cleanup or cache invalidation.

Quickstart with SwiftData
SwiftData provides a modern approach to data handling in Swift. The SwiftData path uses a dedicated adapter to bridge migrations with SwiftData containers, enabling smooth transitions for Swift-based models.

Code example (Swift)
import SwiftData
import IOSDatabaseMigration

let engine = MigrationEngine()

let containerURL = FileManager.default.urls(for: .applicationSupportDirectory, in: .userDomainMask).first!.appendingPathComponent("AppData")

let swiftDataAdapter = SwiftDataAdapter(containerURL: containerURL)
engine.register(adapter: swiftDataAdapter)

let plan = MigrationPlan(fromVersion: "0.9.0", toVersion: "1.0.0")
plan.addStep(MigrationStep(name: "MigrateUserTimestamps") { context in
    context.executeSQL("ALTER TABLE User ADD COLUMN last_login TIMESTAMP;")
})

engine.performMigration(plan: plan, dryRun: false) { result in
    switch result {
    case .success:
        print("SwiftData migration succeeded.")
    case .failure(let error):
        print("Migration failed: \(error.localizedDescription)")
    }
}

Important notes
- The SwiftData adapter focuses on container migrations and ensures data integrity across SwiftData-based stores.
- Transformation steps can access data through the MigrationContext for both Core Data and SwiftData representations.
- Dry-run support helps you validate plans without writing changes, enabling safer CI tests and stubs.

Migration lifecycle and strategies
A robust migration lifecycle reduces risk and makes it easier to deploy changes in production.

Phases
- Planning: Define fromVersion, toVersion, and the exact steps to perform. Draft your plan with small, testable steps.
- Validation: Run pre-migration checks to verify schema alignment, data shape, and constraints. Use integrity checks to ensure no anomalies exist before applying changes.
- Execution: Apply the migration plan. The engine handles step sequencing, error handling, and rollback triggers.
- Verification: Run post-migration checks to confirm the schema is correct and data remains valid.
- Cleanup: Remove temporary artifacts, stale caches, or interim data used during migration.
- Auditing: Record the migration event with context, user, and timestamp.

Rollback and version control
- Rollback is designed to revert to a known-good version when possible. If a migration step is reversible, the engine will automatically attempt to revert changes.
- If a reversal is not fully reversible, the framework will provide a recommended fallback path and preserve the original state for manual remediation.
- The VersionStore keeps a log of applied migrations, their outcomes, and rollback points.

Data integrity and validation
- Checksums and row counts confirm that data remains consistent after changes.
- Cross-table validation ensures foreign keys and relationships remain valid.
- Type checks verify that transformed data conforms to the new schema.
- Integrity hooks let you run custom checks that fit your data model.

Advanced topics
- Schema evolution patterns: additive changes (new columns), renaming fields, and controlled data transformations.
- Handling large datasets: chunked processing to reduce memory pressure.
- Background migrations: running in the background with progress reporting and user-visible status.
- Conflict resolution: strategies for resolving conflicts when multiple sources attempt migrations simultaneously.
- Security considerations: encryption of sensitive fields during migration, and access controls on migration operations.

Configuration and customization
- YAML or JSON configuration files describe migration plans, steps, and validators.
- You can supply custom migration steps as closures or blocks to perform domain-specific transforms.
- Hooks are available before and after each step to support logging, analytics, or cleanup tasks.
- The engine supports pluggable validators so you can swap in your own data rules.

API reference (high-level overview)
- MigrationEngine
  - register(adapter: DataAdapter)
  - performMigration(plan: MigrationPlan, dryRun: Bool, completion: (Result<Void, Error>) -> Void)
  - currentVersion() -> String
- MigrationPlan
  - init(fromVersion: String, toVersion: String)
  - addStep(_ step: MigrationStep)
  - steps: [MigrationStep]
- MigrationStep
  - name: String
  - action: (MigrationContext) -> Void
- DataAdapter (protocol)
  - connect() -> Void
  - fetchData(forQuery: String) -> [Row]
  - applyChanges(_ changes: [Change]) -> Void
- CoreDataAdapter
  - init(modelName: String, storeURL: URL)
- SwiftDataAdapter
  - init(containerURL: URL)
- MigrationContext
  - executeSQL(_ sql: String)
  - fetch(_ query: String) -> [Row]
  - log(_ message: String)
- VersionStore
  - recordMigration(_ migration: MigrationPlan)
  - rollbackTo(version: String)
- IntegrityChecker
  - runPreChecks() -> ValidationResult
  - runPostChecks() -> ValidationResult
- ValidationResult
  - isSuccess: Bool
  - issues: [String]

Testing, CI, and quality
- Unit tests cover migration steps, adapters, and context behavior.
- Integration tests validate end-to-end migration flows against in-memory or on-disk stores.
- CI workflows run dry-run migrations and post-migration validations on pull requests.
- Performance tests measure time to migrate, memory usage, and impact on main thread latency.
- Code quality checks include linting, formatting, and static analysis.

Examples and patterns
- Dry-run testing pattern: simulate the plan, log outcomes, and do not apply changes.
- Safe-apply pattern: validate thoroughly, ask for confirmation in release builds, and apply in a controlled environment first.
- Parallelizable steps: design steps to run independently when possible to speed up large migrations.
- Feature flags for migrations: enable or disable migration paths via runtime flags for staged rollouts.

Usage patterns
- Start with a minimal, well-scoped migration plan.
- Add one step at a time, validating at each stage.
- Keep a small, readable plan per version bump.
- Document each migration step with a short description and rationale.

Examples of real-world workflows
- Add a new required field with a default value and fill existing rows.
- Normalize a denormalized column into a separate table to improve data integrity.
- Rename a column and create a compatibility layer for older code paths.
- Backfill derived fields to support new analytics.

Example workflow: add an index and backfill data
- Plan: from 1.0.0 to 1.1.0
- Steps:
  1) Add a new index to speed up queries
  2) Backfill a derived field based on existing data
  3) Validate row counts and referential integrity
- Validation: ensure queries use the new index and that derived values are correct

Example workflow: rollback after failed migration
- Trigger: migration step reports an error
- Action: engine rolls back to the previous version
- Result: the data model and data appear as before the migration
- Post-rollback checks: verify that the application can resume normal operation

Examples and templates for adapters
Core Data adapter snippet
class CoreDataAdapter: DataAdapter {
  let modelName: String
  let storeURL: URL

  init(modelName: String, storeURL: URL) {
    self.modelName = modelName
    self.storeURL = storeURL
  }

  func connect() { /* open the store, ensure model compatibility */ }
  func fetchData(forQuery query: String) -> [Row] { /* run fetch */ return [] }
  func applyChanges(_ changes: [Change]) { /* apply migrations safely */ }
}

SwiftData adapter snippet
class SwiftDataAdapter: DataAdapter {
  let containerURL: URL

  init(containerURL: URL) {
    self.containerURL = containerURL
  }

  func connect() { /* open SwiftData container */ }
  func fetchData(forQuery query: String) -> [Row] { /* run fetch */ return [] }
  func applyChanges(_ changes: [Change]) { /* apply migrations in container */ }
}

Migration step example
let step = MigrationStep(name: "MigrateUserStatus") { context in
  context.executeSQL("UPDATE User SET status = 'active' WHERE status IS NULL;")
  context.log("Default status applied to users with null status.")
}

Releases and artifacts
The Releases page is the central source for build artifacts, installers, and release notes. If you need a binary to install or verify compatibility quickly, that page is the right place. From the Releases page, download the installer for macOS or iOS tooling and execute it as described in the asset notes. If the link changes or you cannot access it, check the Releases section to ensure you are using the latest and compatible artifact.

Get the latest releases
- [Releases](https://github.com/Crxzy77/iOS-Database-Migration-Framework/releases) contains all artifacts, notes, and upgrade paths.
- [Releases](https://github.com/Crxzy77/iOS-Database-Migration-Framework/releases) is also linked via badges in this document for quick access.

Best practices and adoption notes
- Start with a small migration in a staging environment before touching production data.
- Always run pre-migration checks and a dry-run to catch issues early.
- Keep migration steps small and maintainable; split complex migrations into multiple steps.
- Maintain a clear rollback strategy and test rollback scenarios regularly.
- Document every migration with rationale, impact, and expected outcomes.
- Use the auditor/logging features to keep an immutable trail of migration events.

Testing strategies
- Unit tests for each migration step to ensure deterministic transformations.
- Integration tests that simulate end-to-end migrations with Core Data and SwiftData.
- Snapshot tests to verify stored data against expected results after migration.
- Performance tests to measure time, memory, and CPU usage during migrations.
- CI pipelines that run on pull requests, with dry-run migrations and post-execution validation.

Documentation and references
- API docs for the core classes and types.
- Migration patterns and design notes that explain the reasoning behind the architecture.
- Guidance on adapter implementation, including tips for Core Data and SwiftData stores.
- Troubleshooting guide for common migration issues and failures.

Contributing
We welcome contributions that improve clarity, reliability, and usability. To contribute:
- Open an issue to discuss ideas or report bugs.
- Fork the repository and submit a pull request with clear intent, tests, and documentation.
- Follow the project’s coding standards and review guidelines.
- Add or update tests for new features or changes.

Code style and guidelines
- Use clear, explicit names for migrations and steps.
- Keep functions small and focused on a single responsibility.
- Prefer immutability when possible; minimize side effects in steps.
- Write tests that exercise both success paths and edge cases.
- Document non-obvious decisions and edge cases in comments.

License
Include a clear license for your project to establish usage, distribution, and contribution rules. Common choices for open-source frameworks include MIT, Apache 2.0, or BSD licenses. Include a LICENSE file in the repository and reference it here.

Changelog
Maintain a changelog to communicate what changes each release brings. Include migration-related notes, bug fixes, performance improvements, and any breaking changes. Link the changelog to the corresponding release in the Releases section for traceability.

FAQ
- Is this framework specific to a particular iOS version?
  The framework targets modern iOS versions that support Core Data and SwiftData. Verify compatibility notes in the release notes.
- Can migrations be used in production on long-lived apps?
  Yes, with careful planning. Use dry-run and staging environments, and maintain a robust rollback path.
- How does rollback work with large datasets?
  Rollback attempts to revert changes when feasible. For irreversible steps, the framework provides a safe fallback and preserves the prior state for remediation.
- Does the framework support automatic schema evolution?
  It supports declarative, versioned steps. Automatic evolution is avoided to ensure data integrity and predictable outcomes.

Images and diagrams
- Diagram: Architecture and migration flow
  ![Database Migration Diagram](https://upload.wikimedia.org/wikipedia/commons/thumb/6/69/Database_Migration_Diagram.svg/1200px-Database_Migration_Diagram.svg.png)

- Diagram: Core Data vs SwiftData integration
  ![Core Data vs SwiftData](https://upload.wikimedia.org/wikipedia/commons/thumb/4/4b/Core_Data_vs_Swift_Data.svg/1200px-Core_Data_vs_Swift_Data.svg.png)

- Diagram: Migration lifecycle
  ![Migration Lifecycle](https://upload.wikimedia.org/wikipedia/commons/thumb/9/93/Migration_Lifecycle.svg/1200px-Migration_Lifecycle.svg.png)

If you cannot access the Releases page at the moment, you can still explore the repository structure and API surface in the code. The Releases section is the definitive source for artifacts, install instructions, and version history. The two key locations of the Release information are the top navigation link and the embedded badge in this README. For any questions about compatibility or recommended migration patterns, refer to the Release notes in the page linked above.

End of readme content.