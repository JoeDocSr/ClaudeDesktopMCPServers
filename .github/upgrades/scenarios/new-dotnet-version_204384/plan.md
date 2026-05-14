# .NET 10 Upgrade Plan

**Repository**: csharp-sdk (Model Context Protocol C# SDK)
**Scenario**: Upgrade from mixed framework versions (.NET 8/9) to .NET 10.0
**Strategy**: Bottom-Up (Dependency-First)
**Status**: Planning Phase

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Migration Strategy](#migration-strategy)
3. [Detailed Dependency Analysis](#detailed-dependency-analysis)
4. [Project-by-Project Plans](#project-by-project-plans)
5. [Package Update Reference](#package-update-reference)
6. [Expected Breaking Changes](#expected-breaking-changes)
7. [Risk Management](#risk-management)
8. [Testing & Validation Strategy](#testing--validation-strategy)
9. [Complexity & Effort Assessment](#complexity--effort-assessment)
10. [Source Control Strategy](#source-control-strategy)
11. [Success Criteria](#success-criteria)

---

## Executive Summary

### Project Overview
- **Total Projects**: 32 (4 core libraries, 8 samples, 10 documentation samples, 10 test projects)
- **Current State**: Mixed frameworks - .NET 8.0, 9.0, with some multi-targeting (net10.0;net9.0;net8.0)
- **Target State**: .NET 10.0 (LTS) as primary target
- **Scope**: 1,116 total issues (39 mandatory, 1,077 potential)

### Classification: **MEDIUM-COMPLEX**
- **Decision Factors**:
  - 32 projects with clear dependency hierarchy (5 levels)
  - 39 mandatory issues (framework updates + binary incompatibilities)
  - 858 behavioral changes requiring code review
  - 198 source incompatibilities requiring fixes
  - 11 NuGet packages recommended for upgrade
  - Clear dependency ordering possible

### Chosen Strategy: **BOTTOM-UP (DEPENDENCY-FIRST)**
This strategy respects the codebase's natural dependency hierarchy:

**Why Bottom-Up?**
1. **Foundation-First**: Core libraries (Analyzers, Core) have 0-1 mandatory issues each
2. **Clear Ordering**: 5-tier dependency graph enables phased approach
3. **Stability**: Each tier builds on stable foundation before moving up
4. **Risk Mitigation**: High-complexity test projects (AspNetCore.Tests) upgraded last when lower dependencies are proven stable
5. **Incremental Validation**: Test each tier before proceeding

### Iteration Strategy
**4 Phases** based on dependency tiers:
- **Phase 1**: Foundation (0 dependencies) - 1 project
- **Phase 2**: Core Libraries (depends on Foundation) - 2 projects  
- **Phase 3**: Secondary Libraries & Simple Apps (depends on Phases 1-2) - 11 projects
- **Phase 4**: Complex Apps & Test Suites (depends on all phases) - 18 projects

### Critical Issues
- **Binary Incompatibilities (Api.0001)**: 9 occurrences - must be addressed in Phase 1-2
- **Source Incompatibilities (Api.0002)**: 198 occurrences - requires code changes
- **Behavioral Changes (Api.0003)**: 858 occurrences - requires review and testing
- **NuGet Updates**: 11 packages need version bumps (Microsoft.Extensions.* family)

### Expected Effort Distribution
- **Phase 1**: Minimal - 1 foundation project, no issues
- **Phase 2**: Moderate - 2 core libraries with mandatory fixes
- **Phase 3**: Moderate - 11 projects, mix of simple samples and mid-tier libraries
- **Phase 4**: High - 18 projects including comprehensive test suite with 514 issues

### Success Criteria
✅ All 32 projects target net10.0 (with multi-target support where needed)
✅ All builds succeed without errors or warnings
✅ All tests pass (critical for Phase 4)
✅ All 11 NuGet packages upgraded
✅ No security vulnerabilities remain

---

## Migration Strategy

### Approach: BOTTOM-UP PHASED MIGRATION

**Why Bottom-Up?**
- Respects natural dependency ordering without multi-targeting complexity
- Each tier is upgraded after its dependencies are proven stable
- Reduces risk of cascading failures
- Enables incremental testing and validation
- Aligns with MCP SDK's layered architecture

### Execution Flow

#### PHASE 1: Foundation & Analyzers (Tier 1)
**Duration**: Minimal (1 project, 0 issues)
**Scope**: `ModelContextProtocol.Analyzers`

**Steps**:
1. Verify Analyzers project (netstandard2.0 → no change needed)
2. Confirm no build issues
3. Mark complete

**Blockers**: None
**Success**: Clean build

---

#### PHASE 2: Core Libraries (Tiers 1-2)
**Duration**: Moderate (2 projects, 235+ issues)
**Scope**: `ModelContextProtocol.Core`, `ModelContextProtocol.TestOAuthServer`

**Execution Order**:
1. **ModelContextProtocol.Core** (PRIMARY)
   - 235 issues (1 mandatory framework update)
   - Update multi-target: `net10.0;net9.0;net8.0;netstandard2.0`
   - Fix source incompatibilities (Api.0002)
   - Review behavioral changes (Api.0003)
   - Run unit tests for Core

2. **ModelContextProtocol.TestOAuthServer** (DEPENDENT)
   - 45 issues (1 mandatory)
   - Inherits Core fixes
   - Update multi-target if needed
   - Validate builds

**Blockers**: Core must be complete before TestOAuthServer
**Success**: Both projects build, Core tests pass

---

#### PHASE 3: Secondary Libraries & Apps (Tiers 1-3)
**Duration**: High (11 projects, ~300 issues)
**Scope**: Main package, console samples, test utilities

**Execution Order (Parallelizable within tier)**:

**Group 3A**: Main Package (CRITICAL)
1. `ModelContextProtocol` (3 issues, 1 mandatory)
   - Update multi-target: `net10.0;net9.0;net8.0;netstandard2.0`
   - Depends on: Core (✅ done)
   - Unblocks all AspNetCore work

**Group 3B**: Simple Console Apps (Can start after Phase 2)
1. `ChatWithTools` (net8.0 → net10.0)
2. `ElicitationClient` (net9.0 → net10.0)
3. `LoggingClient` (net9.0 → net10.0)
4. `ProgressClient` (net9.0 → net10.0)
5. `QuickstartClient` (net8.0 → net10.0)
6. `ProtectedMcpClient` (net9.0 → net10.0)
7. `ModelContextProtocol.Analyzers.Tests` (net9.0 → net10.0)

Migration for each: Framework update + NuGet updates + review behavioral changes

**Group 3C**: Test Utilities (After Phase 2 Core)
1. `ModelContextProtocol.TestServer` (update multi-target)
2. `ModelContextProtocol.ExperimentalApiRegressionTest` (update multi-target)
3. `ModelContextProtocol.ConformanceClient` (update multi-target)

**Blockers**: Phase 2 must complete
**Success**: All projects build, samples run, test utilities available for Phase 4

---

#### PHASE 4: AspNetCore Layer & Applications (Tiers 1-4)
**Duration**: High (13 projects, ~400+ issues)
**Scope**: AspNetCore library, AspNetCore samples, complex applications

**Execution Order**:

**Step 1**: `ModelContextProtocol.AspNetCore` (CRITICAL HUB)
- 11 issues (1 mandatory + API changes)
- Update multi-target: `net10.0;net9.0;net8.0`
- Fix source incompatibilities (ASP.NET Core APIs may have breaking changes)
- Unblocks 13+ dependent projects

**Step 2**: AspNetCore Sample Apps (Can parallelize)
- `AspNetCoreMcpPerSessionTools`, `AspNetCoreMcpServer`, `Elicitation`, `EverythingServer`, `HttpContext`, `Logging`, `Progress`
- Each: net9.0 → net10.0
- Inherit AspNetCore fixes
- Review sample-specific behavioral changes

**Step 3**: Advanced/Hosting Apps (After AspNetCore ready)
- `InMemoryTransport` (net8.0 → net10.0)
- `LongRunningTasks` (net9.0 → net10.0, has behavioral changes)
- `ProtectedMcpServer` (net9.0 → net10.0, depends on AspNetCore)
- `QuickstartWeatherServer` (net8.0 → net10.0, depends on main package)
- `TestServerWithHosting` (multi-target: add net10.0)

**Step 4**: Special Projects
- `ModelContextProtocol.AotCompatibility.TestApp` (already net10.0)
  - Verify no regressions after all dependencies upgraded
- `ModelContextProtocol.ConformanceServer` (net10.0;net9.0;net8.0)
  - Update multi-target, runs conformance tests

**Blockers**: Phase 3 must complete, AspNetCore must be ready
**Success**: All projects build, sample applications run

---

#### PHASE 5: Comprehensive Test Suite (Tiers 1-5)
**Duration**: Very High (2 projects, 700+ issues combined)
**Scope**: Full test coverage for entire SDK

**Execution Order**:

**Primary**: `ModelContextProtocol.Tests` (191 issues, 5 mandatory)
- Multi-target: net10.0;net9.0;net8.0;net472 (keep net472 for .NET Framework compatibility)
- Comprehensive unit tests for Core, main package
- Depends on: TestServer, TestOAuthServer (✅ Phases 2-3)
- Run full test suite

**Secondary**: `ModelContextProtocol.AspNetCore.Tests` (514 issues, 4 mandatory - LARGEST)
- Multi-target: net10.0;net9.0;net8.0
- Integration tests across entire HTTP transport layer
- Depends on: 7 projects (all previous tiers)
- Runs conformance client + server interactions
- Run full test suite - most comprehensive validation

**Success Criteria**:
- ✅ All tests pass for net10.0 target
- ✅ All tests pass for multi-targets (if testing those)
- ✅ No regressions in behavioral changes
- ✅ All 1,116 issues resolved

---

### NuGet Package Update Strategy

**Timing**: Apply to each project as it's upgraded

**Packages to Update** (11 total, mostly Microsoft.Extensions.*):
```
Microsoft.AspNetCore.Authentication.JwtBearer:    10.0.7 → 10.0.8
Microsoft.Extensions.Caching.Abstractions:        10.0.7 → 10.0.8
Microsoft.Extensions.Caching.Memory:              10.0.7 → 10.0.8
Microsoft.Extensions.DependencyInjection:         10.0.7 → 10.0.8
Microsoft.Extensions.Hosting:                     10.0.7 → 10.0.8
Microsoft.Extensions.Hosting.Abstractions:        10.0.7 → 10.0.8
Microsoft.Extensions.Logging:                     10.0.7 → 10.0.8
Microsoft.Extensions.Logging.Abstractions:        10.0.7 → 10.0.8
Microsoft.Extensions.Logging.Console:             10.0.7 → 10.0.8
Microsoft.Extensions.Options:                     10.0.7 → 10.0.8
```

**Application**:
- Phase 2: Update in Core projects
- Phase 3: Update in packages/samples that reference
- Phase 4: Update in AspNetCore-dependent projects
- Phase 5: Ensure test projects have latest

---

### Parallel Execution Within Tiers

**Can Parallelize**:
- All 8 console apps in Phase 3B (no interdependencies)
- All 7 AspNetCore samples in Phase 4 Step 2 (depend only on AspNetCore)
- All advanced apps in Phase 4 Step 3 (once AspNetCore ready)

**Cannot Parallelize Across Phases**:
- Phase 2 → Phase 3 (dependency chain)
- Phase 3 → Phase 4 (AspNetCore depends on main package)
- Phase 4 → Phase 5 (tests depend on everything)

---

### Risk Management During Migration

**Biggest Risks**:
1. **AspNetCore breaking changes** (Phase 4) - High impact on 13+ dependent projects
2. **Behavioral changes in Core** (Phase 2) - Affects entire SDK
3. **Multi-target complexity** - Some projects maintain 3-4 targets

**Mitigation**:
- Test each tier before proceeding
- Focus on Phase 2 Core for blocking issues
- Review all Api.0002 (source incompatibilities) before compiling
- Keep detailed logs of breaking changes found
- Run test suites immediately after each phase

---

### Expected Timeline (Relative Complexity)
- Phase 1: ⏱️ Minimal (5 min)
- Phase 2: ⏱️ Moderate (30 min)
- Phase 3: ⏱️ High (60 min)
- Phase 4: ⏱️ Very High (90 min)
- Phase 5: ⏱️ Critical (120 min) - comprehensive testing

**Total**: ~300 min of focused work, with significant testing at end

---

## Detailed Dependency Analysis

### Dependency Graph by Migration Tier

```
TIER 1 (Foundation - 0 dependencies):
└── ModelContextProtocol.Analyzers
    • Current: netstandard2.0 → Target: netstandard2.0 (unchanged)
    • Issues: 0
    • Role: Roslyn analyzers, used by Core
    • Migration: NO CHANGES NEEDED

TIER 2 (Core Libraries - depends only on Tier 1):
├── ModelContextProtocol.Core
│   • Current: net10.0;net9.0;net8.0;netstandard2.0 → Target: net10.0;net9.0;net8.0;netstandard2.0
│   • Issues: 235 (1 mandatory - framework update)
│   • Role: Core protocol implementation, fundamental library
│   • Dependents: 15+ projects
│
└── ModelContextProtocol.TestOAuthServer
    • Current: net10.0;net9.0;net8.0 → Target: net10.0;net9.0;net8.0 (no change)
    • Issues: 45 (1 mandatory)
    • Role: Test utility server
    • Dependents: AspNetCore.Tests only

TIER 3 (Secondary Libraries & Apps - depends on Tiers 1-2):
├── ModelContextProtocol (main package)
│   • Current: net10.0;net9.0;net8.0;netstandard2.0 → Target: same
│   • Issues: 3 (1 mandatory - framework update)
│   • Depends on: ModelContextProtocol.Core
│   • Dependents: AspNetCore, 5 simple samples
│
├── Simple Console/CLI Apps (8 projects)
│   • ChatWithTools, ElicitationClient, LoggingClient, ProgressClient, QuickstartClient,
│     ProtectedMcpClient, ModelContextProtocol.Analyzers.Tests
│   • Current: net8.0-net9.0 → Target: net10.0
│   • Depends on: ModelContextProtocol.Core (and Analyzers for one)
│   • Role: Simple one-file samples
│   • Migration: Straightforward framework update + NuGet updates
│
└── Test Utilities (3 projects)
    • ModelContextProtocol.TestServer, ModelContextProtocol.ExperimentalApiRegressionTest,
      ModelContextProtocol.ConformanceClient
    • Depends on: ModelContextProtocol.Core
    • Migration: Simple framework updates

TIER 4 (AspNetCore Libraries & Mid-tier Apps - depends on Tiers 1-3):
├── ModelContextProtocol.AspNetCore (CRITICAL)
│   • Current: net10.0;net9.0;net8.0 → Target: net10.0;net9.0;net8.0
│   • Issues: 11 (1 mandatory + API changes)
│   • Depends on: ModelContextProtocol
│   • Dependents: 13+ projects
│   • Role: HTTP transport layer - HIGH IMPACT
│   • Migration: Medium complexity - ASP.NET Core APIs may have breaking changes
│
├── AspNetCore Sample Apps (6 projects)
│   • AspNetCoreMcpPerSessionTools, AspNetCoreMcpServer, Elicitation, EverythingServer,
│     HttpContext, Logging, Progress
│   • Current: net9.0 → Target: net10.0
│   • Depends on: ModelContextProtocol.AspNetCore
│   • Role: Samples demonstrating server patterns
│   • Migration: Inherit AspNetCore fixes + sample-specific updates
│
├── Advanced Samples (4 projects)
│   • InMemoryTransport, LongRunningTasks, ProtectedMcpServer, QuickstartWeatherServer
│   • Depends on: ModelContextProtocol (and AspNetCore for ProtectedMcpServer)
│   • Migration: Medium complexity - some have behavioral changes
│
├── Special Cases (2 projects)
│   • TestServerWithHosting: Multi-target (net10.0;net9.0;net8.0;net472)
│   • ModelContextProtocol.AotCompatibility.TestApp: Already targets net10.0
│
└── Multi-Tier Utility (1 project)
    • ModelContextProtocol.ConformanceServer: Depends on AspNetCore

TIER 5 (Comprehensive Test Suite - depends on all previous tiers):
└── ModelContextProtocol.AspNetCore.Tests
    • Current: net10.0;net9.0;net8.0 → Target: net10.0;net9.0;net8.0
    • Issues: 514 (4 mandatory - largest test suite)
    • Depends on: 7 projects (entire test infrastructure)
    • Role: Integration & conformance testing - CRITICAL VALIDATION POINT
    • Migration: HIGH COMPLEXITY - test for all lower tiers, behavioral changes review needed

Additional: ModelContextProtocol.Tests
    • Current: net10.0;net9.0;net8.0;net472 → Target: net10.0;net9.0;net8.0;net472
    • Issues: 191 (5 mandatory)
    • Role: Core functionality tests
    • Tier: Mixed - depends on Tier 3 projects (TestServer) and Tier 4 projects
```

### Dependency-Based Ordering Rationale

**Why this order ensures success:**

1. **Tier 1 First** (Foundation): Analyzer is static, no runtime changes needed
   - ✅ Safe to start here - 0 issues
   - ✅ Unblocks Tier 2

2. **Tier 2 Second** (Core): Core.csproj is the most-used library
   - ✅ 15+ projects depend on it
   - ✅ Small scope (235 issues) focused on one project
   - ✅ Once Core works, all dependents can consume upgrades

3. **Tier 3 Third** (Secondary): Simple apps and test utilities
   - ✅ All depend only on Tier 1-2 (already upgraded)
   - ✅ Mix of simple (8 console apps) and mid-complexity
   - ✅ Can be parallelized within tier
   - ⚠️ Some have behavioral changes to review

4. **Tier 4 Fourth** (AspNetCore Layer): HTTP/web focus
   - ✅ AspNetCore.csproj is the HTTP layer
   - ✅ 13+ dependents waiting for it
   - ✅ Upgrade AspNetCore first, then samples inherit fixes
   - ⚠️ May have ASP.NET Core framework-specific breaking changes

5. **Tier 5 Last** (Test Suite): Comprehensive validation
   - ✅ Tests everything below it
   - ✅ Largest scope (514 issues) but most valuable - validates all changes
   - ✅ Only runs after all libraries proven stable
   - ✅ Best indicator of "ready for production"

### No Circular Dependencies Detected
✅ Verified: Dependency graph is a strict DAG (Directed Acyclic Graph)
✅ No blocking circular references

### Multi-Target Consideration
Several projects already multi-target:
- `ModelContextProtocol.Core`: net10.0;net9.0;net8.0;netstandard2.0
- `ModelContextProtocol.AspNetCore`: net10.0;net9.0;net8.0
- `ModelContextProtocol.Tests`: net10.0;net9.0;net8.0;net472
- `TestServerWithHosting`: net10.0;net9.0;net8.0;net472

**Strategy**: 
- Maintain multi-targeting where it exists (these are libraries/test utilities)
- Add net10.0 to existing multi-targets if not present
- Single-target projects (samples, simple apps) → upgrade to net10.0 only

---

## Project-by-Project Plans

[To be filled]

---

## Package Update Reference

### NuGet Package Upgrade Summary

**Total Packages to Update**: 11 (all Microsoft.Extensions.* family for .NET 10.0 compatibility)

### Packages Requiring Updates

| Package | Current | Target | Phase | Projects Affected | Notes |
|---------|---------|--------|-------|------------------|-------|
| Microsoft.AspNetCore.Authentication.JwtBeaker | 10.0.7 or 9.0.11 | 10.0.8 | 4 | ProtectedMcpServer, AspNetCore samples | Auth package upgrade |
| Microsoft.Extensions.Caching.Abstractions | 10.0.7 | 10.0.8 | 2+ | Core, Main package, AspNetCore | Caching abstraction |
| Microsoft.Extensions.Caching.Memory | 10.0.7 | 10.0.8 | 2+ | Core, Main package | Memory caching |
| Microsoft.Extensions.DependencyInjection | 10.0.7 | 10.0.8 | 2+ | Core, Main package, AspNetCore | DI container |
| Microsoft.Extensions.Hosting | 10.0.7 | 10.0.8 | 3+ | Samples, test apps | Hosting |
| Microsoft.Extensions.Hosting.Abstractions | 10.0.7 | 10.0.8 | 2+ | Core, AspNetCore | Hosting abstractions |
| Microsoft.Extensions.Logging | 10.0.7 | 10.0.8 | 2+ | Core, Main package | Logging |
| Microsoft.Extensions.Logging.Abstractions | 10.0.7 | 10.0.8 | 2+ | Core, Main package | Logging abstractions |
| Microsoft.Extensions.Logging.Console | 10.0.7 | 10.0.8 | 3+ | Samples, test apps | Console logging |
| Microsoft.Extensions.Options | 10.0.7 | 10.0.8 | 2+ | Core, AspNetCore | Options/configuration |

### Compatible Packages (No Update Needed)

The following packages are compatible with .NET 10.0 and don't require upgrades:

- Anthropic (12.17.0)
- coverlet.collector (10.0.0)
- JsonSchema.Net (9.2.0)
- Microsoft.CodeAnalysis.Analyzers (3.3.4)
- Microsoft.CodeAnalysis.CSharp (4.8.0)
- Microsoft.Extensions.AI (10.5.2)
- Microsoft.Extensions.AI.Abstractions (10.5.2)
- Microsoft.Extensions.AI.OpenAI (10.5.2)
- Microsoft.Extensions.TimeProvider.Testing (10.1.0)
- Microsoft.NET.Test.Sdk (18.5.1)
- Microsoft.SourceLink.GitHub (8.0.0)
- Moq (4.20.72)
- NETStandard.Library (2.0.3)
- OpenTelemetry (1.15.3)
- OpenTelemetry.* (various - all compatible)
- Serilog (4.3.1)
- Serilog.* (various - all compatible)
- xunit.* (various - all compatible)

### Application Strategy

#### Phase 2: Core Libraries
Update in `ModelContextProtocol.Core`:
- Microsoft.Extensions.Caching.Abstractions (10.0.7 → 10.0.8)
- Microsoft.Extensions.Caching.Memory (10.0.7 → 10.0.8)
- Microsoft.Extensions.DependencyInjection (10.0.7 → 10.0.8)
- Microsoft.Extensions.Hosting.Abstractions (10.0.7 → 10.0.8)
- Microsoft.Extensions.Logging (10.0.7 → 10.0.8)
- Microsoft.Extensions.Logging.Abstractions (10.0.7 → 10.0.8)
- Microsoft.Extensions.Options (10.0.7 → 10.0.8)

Update in `ModelContextProtocol` (main package):
- Microsoft.Extensions.Logging (if not inherited from Core)

#### Phase 3: Secondary Packages & Apps
Update in `ModelContextProtocol.Analyzers.Tests` and samples that reference:
- Microsoft.Extensions.Hosting (10.0.7 → 10.0.8)
- Microsoft.Extensions.Logging.Console (10.0.7 → 10.0.8)

#### Phase 4: AspNetCore Layer
Update in `ModelContextProtocol.AspNetCore`:
- Microsoft.AspNetCore.Authentication.JwtBearer (10.0.7 → 10.0.8)
- Microsoft.Extensions.DependencyInjection (if not inherited)
- Other Extensions packages

Update in samples and applications:
- Apply package updates based on what's referenced

#### Phase 5: Test Projects
Verify all test projects have latest versions:
- `ModelContextProtocol.Tests`
- `ModelContextProtocol.AspNetCore.Tests`

### Update Method

For each project:

1. **Edit .csproj directly** (preferred):
```xml
<PackageReference Include="Microsoft.Extensions.Logging" Version="10.0.8" />
```

2. **Or use NuGet restore**:
```powershell
dotnet add package Microsoft.Extensions.Logging --version 10.0.8
```

3. **Or update via Solution Package Manager** (if using Visual Studio)

### Validation After Updates

After updating each set of packages:
- ✅ Run `dotnet restore` (update lock files)
- ✅ Run `dotnet build` (ensure packages are compatible)
- ✅ Run `dotnet test` (validate no behavioral changes from package versions)
- ✅ Check for package conflicts (`NU1605` or `NU1608` warnings)

---

## Expected Breaking Changes

### Overview
The assessment identified 1,077 potential issues, with the main breaking change categories being:

1. **Binary Incompatibilities (Api.0001)**: 9 occurrences - Will require recompilation
2. **Source Incompatibilities (Api.0002)**: 198 occurrences - Will require code changes
3. **Behavioral Changes (Api.0003)**: 858 occurrences - Code may behave differently at runtime

### Known Breaking Change Categories

#### 1. ASP.NET Core API Changes (Phase 4 Impact)

**Potential areas**:
- Middleware registration APIs
- Configuration changes
- Authentication/authorization patterns
- Minimal hosting APIs
- Endpoint mapping patterns

**Current file likely affected**:
- `McpEndpointRouteBuilderExtensions.cs` (already shown - uses MapGroup, MapPost, MapGet, MapDelete)
- These APIs are largely stable in .NET 10, but review needed

**Mitigation**: 
- Test AspNetCore samples immediately after update
- Review ASP.NET Core release notes for breaking changes
- Check if any MapGroup/MapPost/MapGet method signatures changed

#### 2. System.Text.Json Changes

**Potential areas** (if SDK uses JSON serialization):
- Default serialization options
- Enum serialization behavior
- Null handling changes

**Mitigation**:
- Verify JSON test cases pass
- Check if any custom converters need updates
- Ensure backward compatibility in JSON-RPC messages

#### 3. Threading & Async Changes

**Potential areas**:
- Thread pool behavior
- Task scheduling changes
- Timeout handling

**Mitigation**:
- Run full test suite (most async issues caught by tests)
- Look for timing-sensitive tests that might flake

#### 4. Collection & LINQ Changes

**Potential areas**:
- Collection initialization behavior
- LINQ query performance/semantics
- Dictionary/Set behavior

**Mitigation**:
- Run comprehensive tests
- Benchmark if performance is critical

#### 5. Reflection & Type Loading

**Potential areas** (if code uses reflection):
- Type discovery behavior
- Assembly loading paths
- Generic type constraints

**Mitigation**:
- Test analyzer and reflection-based tool discovery
- Verify `ModelContextProtocol.Analyzers` still works correctly

#### 6. Dependency Injection Changes

**Potential areas**:
- Service registration ordering
- Service resolution semantics
- Factory pattern changes

**Mitigation**:
- Core DI tests must pass
- AspNetCore DI samples must work
- Hosted services initialization tested

### By Project: Likely Affected Areas

#### Phase 2: ModelContextProtocol.Core
- **Source Incompatibilities (Api.0002)**: Expected ~50-100
- **Likely**: API signature changes, possibly obsolete APIs
- **Action**: Compile and fix errors as they appear
- **Review**: Watch for serialization, threading APIs

#### Phase 4: ModelContextProtocol.AspNetCore
- **Source Incompatibilities (Api.0002)**: Expected ~5-10
- **Likely**: ASP.NET Core endpoint mapping, middleware changes
- **Action**: Compile and fix errors
- **Review**: Check MapGroup, MapPost, MapGet, MapDelete are still valid
- **File Example**: `McpEndpointRouteBuilderExtensions.cs` probably fine, but verify

#### Phase 5: Test Suites
- **Behavioral Changes (Api.0003)**: 1000+ to review
- **Likely**: Timing changes, assertion library updates, test infrastructure
- **Action**: Run tests, fix failures one at a time
- **Review**: Some may be false failures due to test timing

### Behavioral Changes Review Process

For the 858 behavioral changes (Api.0003):

**Strategy**: These typically don't cause compilation errors but may cause runtime differences.

1. **Phase 2-4**: Run tests after each phase to catch behavioral issues early
2. **Phase 5**: Focus on AspNetCore.Tests which validates all behavioral changes
3. **If tests fail**:
   - Check if test assertion is too strict
   - Check if actual behavior changed (read .NET 10 release notes)
   - Adapt test if behavior change is intentional and correct
   - Fix code if behavior is undesired

### .NET 10 Specific Considerations

#### Multi-Target Maintenance
Several projects multi-target (net10.0;net9.0;net8.0):
- **Issue**: Some breaking changes only affect net10.0
- **Mitigation**: Use conditional compilation if needed
- **Example**: `#if NET10_0_OR_GREATER` guards if needed

#### Native AOT Support
`ModelContextProtocol.AotCompatibility.TestApp` already targets net10.0:
- **Ensure**: JSON source generators still work
- **Check**: No reflection-only code is introduced
- **Validate**: This app still builds and runs

#### Platform-Specific Changes
If net472 target is maintained (in test projects):
- **Note**: net472 changes are minimal (it doesn't change)
- **Just ensure**: net10.0 specific issues don't affect net472 behavior

### Testing Strategy for Breaking Changes

#### Phase 2-4 Testing
- After each phase, compile and verify no errors
- Run unit tests for that phase's projects
- Quick smoke tests for dependent projects

#### Phase 5 Full Testing
- Run entire test suite for net10.0 target
- Review all test failures systematically
- Don't skip or suppress failures without understanding

#### Behavioral Change Validation
- For each failing test:
  1. Read error message carefully
  2. Check if it's a real regression or test artifact
  3. If regression: check .NET 10 breaking change docs
  4. Fix either code or test accordingly

### Reference: Common .NET 10 Breaking Changes

(These are examples - actual breaking changes should be verified from Microsoft docs)

Potential areas based on typical .NET releases:
- Reflection APIs: Changed return types, null handling
- Collections: Behavior with null, empty collections
- Async APIs: CancellationToken handling, task scheduling
- JSON: Default serialization options for new scenarios
- HTTP: Client behavior, timeout handling
- ASP.NET Core: Endpoint mapping (covered above), middleware order

**Always check**: [.NET 10 Breaking Changes Documentation](https://learn.microsoft.com/en-us/dotnet/core/compatibility/10.0)

### Issues Encountered Log

During execution, maintain a log of breaking changes found:

**Template**:
```
## Breaking Changes Found

### [Category]: [Issue Title]
- **Project**: [Project.csproj]
- **Error/Issue**: [Description]
- **Root Cause**: [What changed in .NET 10]
- **Resolution**: [How it was fixed]
- **Files Modified**: [List of files]
```

This log becomes valuable documentation for code review and future upgrades.

---

## Risk Management

### High-Risk Items

| Risk | Severity | Impact | Probability | Mitigation |
|------|----------|--------|-------------|-----------|
| AspNetCore breaking changes hidden | 🔴 Critical | 13+ projects blocked | Medium | Test AspNetCore samples immediately, review release notes |
| Core.csproj behavioral changes | 🔴 Critical | Cascades to all 15+ dependents | Medium | Run Core tests first, validate before Phase 3 |
| Large test suite failures (514 issues) | 🔴 Critical | Unclear what's actually broken | High | Systematic review, one failure at a time |
| Multi-target package conflicts | 🟠 High | Some targets won't build | Low | Test each target (net10.0, net9.0, net8.0, netstandard2.0) |
| Binary incompatibilities (9 issues) | 🟠 High | Unexpected runtime failures | Low | Force recompilation after package updates |
| Source incompatibilities (198 issues) | 🟠 High | Compilation errors | High | Will be caught at compile time, fix as they appear |
| Behavioral changes in tests | ⚠️ Medium | False test failures | High | Review each failure, don't suppress without understanding |

### Per-Phase Risk Assessment

#### Phase 1: Foundation
- **Risk Level**: ✅ MINIMAL
- **What Can Go Wrong**: Nothing (no changes needed)
- **Mitigation**: Just verify project

#### Phase 2: Core Libraries
- **Risk Level**: ⚠️ MEDIUM-HIGH
- **What Can Go Wrong**:
  - Core APIs have hidden behavioral changes
  - Test suite reveals unforeseen issues
  - Multi-target combinations break
- **Mitigation**:
  - Run Core tests immediately after Phase 2
  - Verify all framework targets build
  - If Core tests fail, fix before proceeding to Phase 3
  - Consider rolling back and investigating if critical

#### Phase 3: Secondary Layer
- **Risk Level**: ⚠️ MEDIUM
- **What Can Go Wrong**:
  - Cascade failures from Core issues not caught by tests
  - Behavioral changes in samples cause runtime errors
  - Test utilities don't work with updated Core
- **Mitigation**:
  - All Core tests must pass before starting Phase 3
  - Compile all Phase 3 projects before running
  - Run samples to validate they work
  - If failures: check if Core changes are compatible

#### Phase 4: AspNetCore Layer
- **Risk Level**: 🔴 HIGH
- **What Can Go Wrong**:
  - ASP.NET Core has breaking changes not in Microsoft docs
  - Endpoint mapping changed (our code heavy with MapGroup, etc)
  - Middleware order/registration has subtle issues
  - 13+ projects cascade failures
- **Mitigation**:
  - Review ASP.NET Core 10.0 release notes carefully
  - Test AspNetCore samples immediately after AspNetCore library update
  - Run samples and verify HTTP endpoints work
  - If AspNetCore breaks: all dependent projects blocked
  - Consider maintaining AspNetCore on net9.0 temporarily if net10.0 breaks (not ideal, but option)

#### Phase 5: Test Suite
- **Risk Level**: 🔴 CRITICAL
- **What Can Go Wrong**:
  - 514+ issues in AspNetCore.Tests = massive review effort
  - Many failures are behavioral not actual breaks
  - Difficulty determining which are real issues vs test artifacts
  - Takes longest to resolve
  - Risk of declaring "done" while actual issues remain
- **Mitigation**:
  - Only run after all other phases verified
  - Systematic failure review (one at a time, don't batch fix)
  - Keep detailed log of each resolution
  - Consider running with reduced scope first (subset of tests)
  - May need to run with different logging levels to diagnose

### Rollback Strategy

**If Phase N fails critically**:

1. **Identify**: Is this Phase N issue or inherited from Phase N-1?
   - Build a simple test (if possible)
   - Check if issue is in this phase's code or dependencies

2. **Scope**: Can it be fixed in Phase N, or does it require Phase N-1 fix?
   - If this phase only: fix and continue
   - If previous phase: may need to rollback that phase

3. **Rollback Steps**:
   ```bash
   # Reset to last known good
   git checkout upgrade-to-NET10  # Back to last commit
   # OR start fresh from main
   git reset --hard origin/main
   git checkout -b upgrade-to-NET10
   ```

4. **Investigate**: Root cause analysis
   - Was breaking change not expected?
   - Is dependency conflict?
   - Package compatibility issue?

5. **Decide**: 
   - Can fix and retry Phase N
   - Need to adjust strategy for Phase N+1
   - Need to escalate (e.g., wait for package fix)

### Communication & Decision Points

#### Decision: Proceed to Next Phase
**Gate**: Before moving to next phase, confirm:
- ✅ All projects in current phase compile
- ✅ All projects have no warnings
- ✅ All tests pass (if project has tests)
- ✅ No cascading failures in dependent projects

#### Decision: Phase Modification
**If issues found in current phase**:
1. Document what changed
2. Fix code to match new .NET 10 behavior
3. Update this plan if expectations were wrong
4. Continue with same phase

#### Decision: Strategy Change
**If entire strategy needs adjustment**:
1. Document why current approach won't work
2. Pause phase execution
3. Revise plan
4. Continue with revised approach

Example: If AspNetCore.csproj fundamentally incompatible with .NET 10, may need to skip it temporarily and continue with other branches.

### Contingency Plans

#### Contingency 1: Core.csproj Has Major Issues
**Symptom**: Core won't compile in net10.0, or tests fail extensively
**Action**:
1. Stop Phase 2
2. Investigate root cause (likely .NET 10 breaking change)
3. Options:
   a. Fix in Core (most likely)
   b. Add conditional compilation for net10.0 if possible
   c. Skip net10.0 support temporarily, stick with net9.0 (not preferred)

#### Contingency 2: AspNetCore Breaking Changes
**Symptom**: AspNetCore.csproj won't compile or samples fail
**Action**:
1. Stop Phase 4
2. Review ASP.NET Core 10.0 breaking changes documentation
3. Check if issue is temporary (e.g., package not updated yet)
4. Options:
   a. Fix code to work with new APIs
   b. Wait for package updates
   c. Temporarily keep AspNetCore on net9.0 multi-target

#### Contingency 3: Test Suite Too Large to Review
**Symptom**: 514 failures in AspNetCore.Tests, can't manage
**Action**:
1. Reduce scope - test only critical tests first
2. Filter by category (unit vs integration)
3. Run tests in isolation to understand failures
4. May extend timeline

#### Contingency 4: Package Conflicts
**Symptom**: Package versions don't align, NU1605/NU1608 warnings
**Action**:
1. Review package resolution
2. May need to wait for packages to update
3. Can temporarily use floating versions (`10.0.*`) to get compatible latest
4. Update plan once packages stabilize

#### Contingency 5: Performance Regression
**Symptom**: Tests pass but performance is significantly slower
**Action**:
1. Log performance regression
2. Investigate if it's a .NET 10 issue or our code
3. May not be blocking for upgrade (could be follow-up task)
4. Document for post-upgrade optimization

### Success Criteria with Risk Management

#### Phase 2 Complete = Core is "Safe"
- [x] All Core tests pass
- [x] No warnings in Core build
- [x] Dependent projects build successfully
- [x] Decision: Proceed to Phase 3? YES/NO
  - If NO: debug Core before Phase 3

#### Phase 3 Complete = Secondary Layer Stable
- [x] All secondary projects build
- [x] Samples run without errors
- [x] No cascading failures from Core
- [x] Decision: Proceed to Phase 4? YES/NO
  - If NO: fix Phase 3 issues before Phase 4

#### Phase 4 Complete = AspNetCore Ready
- [x] AspNetCore builds (all targets)
- [x] AspNetCore samples run
- [x] No new failures in dependent projects
- [x] Decision: Proceed to Phase 5? YES/NO
  - If NO: fix AspNetCore before Phase 5

#### Phase 5 Complete = PRODUCTION READY
- [x] All tests pass in net10.0 target
- [x] All tests pass in multi-targets (if testing those)
- [x] Zero warnings in builds
- [x] No performance regressions identified
- [x] APPROVED FOR PRODUCTION: YES/NO
  - If NO: iterate until approved

---

## Testing & Validation Strategy

### Multi-Level Testing Approach

Testing occurs at 3 levels to ensure comprehensive validation:

1. **Compilation Level**: Does code compile?
2. **Unit/Sample Level**: Do individual projects work?
3. **Integration Level**: Do all projects work together?

---

### Phase-by-Phase Testing

#### Phase 1: Foundation Verification
**Steps**:
1. Verify `ModelContextProtocol.Analyzers` project hasn't changed
2. Attempt to build
3. Confirm build succeeds

**Validation Checklist**:
- ✅ Build succeeds
- ✅ No warnings

**Effort**: < 5 min

---

#### Phase 2: Core Libraries Testing

**Step 1**: Update Framework & Packages
```powershell
# ModelContextProtocol.Core
dotnet build src/ModelContextProtocol.Core/ModelContextProtocol.Core.csproj
```

**Test Targets**:
- net10.0 (primary)
- net9.0 (backward compat)
- net8.0 (backward compat)
- netstandard2.0 (library compat)

**Step 2**: Run Core Tests
```powershell
dotnet test tests/ModelContextProtocol.Tests/ -c Release --no-build
```

**Validation Checklist**:
- ✅ Core builds without errors (all targets)
- ✅ Core builds without warnings
- ✅ All Core tests pass
- ✅ TestOAuthServer builds
- ⚠️ IF FAILS: Do not proceed to Phase 3 until fixed

**Effort**: 30 min

**Key Issues to Watch**:
- Serialization API changes (if SDK uses JSON APIs)
- DependencyInjection changes
- Threading/async changes
- Reflection-based tool discovery in Analyzers

---

#### Phase 3: Secondary Layer Testing

**Step 1**: Compile All Projects
```powershell
# Main package
dotnet build src/ModelContextProtocol/ModelContextProtocol.csproj

# Console apps (can parallelize)
dotnet build samples/ChatWithTools/ChatWithTools.csproj
dotnet build samples/QuickstartClient/QuickstartClient.csproj
# ... etc for all 8 console apps

# Test utilities
dotnet build tests/ModelContextProtocol.TestServer/
dotnet build tests/ModelContextProtocol.ConformanceClient/
```

**Step 2**: Run Sample Applications
```powershell
# Run a few samples to validate they work
dotnet run --project samples/QuickstartClient/QuickstartClient.csproj
```

**Step 3**: Verify No Cascading Failures
```powershell
# Rebuild everything Phase 3 depends on (phases 1-2)
dotnet build src/ModelContextProtocol.Core/
dotnet build tests/ModelContextProtocol.Tests/ -c Debug  # Quick build to check dependency
```

**Validation Checklist**:
- ✅ All Phase 3 projects build without errors
- ✅ All Phase 3 projects build without warnings
- ✅ At least 2 console samples run successfully
- ✅ Test utilities available for Phase 4
- ✅ No new failures in Phase 2 builds
- ⚠️ IF FAILS: Identify if Phase 2 or Phase 3 issue

**Effort**: 45 min

**Key Issues to Watch**:
- Behavioral changes in samples
- Runtime errors in console apps
- Package conflicts

---

#### Phase 4: AspNetCore Layer Testing

**Step 1**: Build AspNetCore
```powershell
dotnet build src/ModelContextProtocol.AspNetCore/ModelContextProtocol.AspNetCore.csproj
```

**Test Targets**:
- net10.0 (primary)
- net9.0 (backward compat)
- net8.0 (backward compat)

**Step 2**: Build AspNetCore Samples
```powershell
# All AspNetCore samples
dotnet build docs/concepts/elicitation/samples/server/Elicitation.csproj
dotnet build docs/concepts/logging/samples/server/Logging.csproj
dotnet build samples/AspNetCoreMcpServer/AspNetCoreMcpServer.csproj
# ... etc
```

**Step 3**: Run Sample Servers (Smoke Test)
```powershell
# Start a sample server and verify it initializes without error
# (May need custom test script to validate endpoints)
dotnet run --project samples/AspNetCoreMcpServer/ &
# Check if server starts and serves requests
# Kill process after validation
```

**Step 4**: Verify Endpoint Mapping Works
- **Focus**: Files like `McpEndpointRouteBuilderExtensions.cs` that use MapGroup, MapPost, MapGet, MapDelete
- **Validation**: Samples that use these endpoints start successfully

**Step 5**: Verify HTTP Stack
- Check if SSE (Server-Sent Events) endpoints work
- Check if streamable HTTP endpoints work
- No breaking changes in ASP.NET Core middleware

**Validation Checklist**:
- ✅ AspNetCore builds without errors (all targets)
- ✅ AspNetCore builds without warnings
- ✅ All AspNetCore samples build
- ✅ At least 2 AspNetCore samples start without errors
- ✅ HTTP/SSE endpoints initialize correctly
- ⚠️ IF FAILS: Do not proceed to Phase 5 until fixed

**Effort**: 60 min

**Key Issues to Watch**:
- Endpoint mapping API changes (MapGroup signatures)
- Middleware registration changes
- Authentication/authorization API changes
- HTTP response type changes

---

#### Phase 5: Comprehensive Test Suite

**Step 1**: Run Core Test Suite
```powershell
dotnet test tests/ModelContextProtocol.Tests/ -c Release --no-build --logger "console;verbosity=normal"
```

**Validation**:
- ✅ All tests pass for net10.0 target
- ⚠️ Note any failures for investigation

**Step 2**: Run AspNetCore Integration Tests (Largest)
```powershell
dotnet test tests/ModelContextProtocol.AspNetCore.Tests/ -c Release --no-build --logger "console;verbosity=normal"
```

**Validation**:
- ✅ All tests pass for net10.0 target
- ⚠️ Note failures, may take multiple iterations

**Step 3**: Investigate Test Failures Systematically
For each failing test:
1. Read error message carefully
2. Determine if it's a real code issue or test artifact
3. If real issue:
   - Identify which project needs fixing
   - Check .NET 10 breaking changes docs
   - Fix code
4. If test artifact (e.g., timing issue):
   - May need to adjust test timing
   - Document decision

**Step 4**: Run Tests Again
After fixing, re-run tests:
```powershell
dotnet test tests/ModelContextProtocol.AspNetCore.Tests/ --filter "Category=Fixed" -c Release
```

**Step 5**: Multi-Target Verification (Optional but Recommended)
Test against multiple targets to ensure backward compatibility:
```powershell
# If project supports multiple targets, test each
dotnet test tests/ModelContextProtocol.Tests/ -c Release -f net9.0
dotnet test tests/ModelContextProtocol.Tests/ -c Release -f net8.0
```

**Validation Checklist** (Phase 5 Complete):
- ✅ ModelContextProtocol.Tests: ALL tests pass (net10.0)
- ✅ ModelContextProtocol.AspNetCore.Tests: ALL tests pass (net10.0)
- ✅ Multi-target tests pass (net9.0, net8.0 if applicable)
- ✅ Zero warnings in any test build
- ✅ All test failures investigated and documented
- ✅ Behavioral changes understood and validated
- ✅ No regressions in existing functionality

**Effort**: 120 min (could be more depending on test failures)

**Key Issues to Watch**:
- Timing-sensitive tests (may need adjustment)
- JSON serialization test assertions
- Async/Task timing changes
- Logging level changes affecting test output

---

### Test Execution Commands Reference

#### Build All Projects
```powershell
dotnet build ModelContextProtocol.slnx -c Release
```

#### Run All Tests
```powershell
dotnet test ModelContextProtocol.slnx -c Release --no-build
```

#### Run Specific Test Project
```powershell
dotnet test tests/ModelContextProtocol.AspNetCore.Tests/ModelContextProtocol.AspNetCore.Tests.csproj -c Release --no-build -v normal
```

#### Run Specific Test Category
```powershell
dotnet test tests/ModelContextProtocol.Tests/ --filter "Category=Unit" -c Release
```

#### Run Tests for Specific Target Framework
```powershell
dotnet test tests/ModelContextProtocol.Tests/ -c Release -f net10.0
```

#### See Detailed Test Output
```powershell
dotnet test tests/ModelContextProtocol.Tests/ -c Release --logger "console;verbosity=detailed"
```

---

### Validation Checkpoints

| Checkpoint | Phase | Gate | Status |
|-----------|-------|------|--------|
| Analyzers verified | 1 | Proceed to Phase 2? | ⏳ Pending |
| Core tests pass | 2 | Proceed to Phase 3? | ⏳ Pending |
| Phase 3 builds + samples run | 3 | Proceed to Phase 4? | ⏳ Pending |
| AspNetCore + samples work | 4 | Proceed to Phase 5? | ⏳ Pending |
| All tests pass (net10.0) | 5 | **UPGRADE COMPLETE** | ⏳ Pending |
| Multi-targets tested (optional) | 5 | Production ready? | ⏳ Pending |

---

### Performance Validation (Optional)

After all tests pass, consider:

```powershell
# Measure startup time
Measure-Command { dotnet run --project samples/QuickstartWeatherServer/ }

# Measure test execution time
Measure-Command { dotnet test tests/ModelContextProtocol.Tests/ -c Release --no-build }
```

**Document if**: .NET 10 has significantly different startup/runtime performance

---

### Coverage Validation (Optional)

If coverage tools are available:

```powershell
# Run with coverage (if coverlet is configured)
dotnet test tests/ModelContextProtocol.Tests/ /p:CollectCoverage=true /p:CoverletOutput=coverage.json
```

**Ensure**: No significant coverage regressions from code changes

---

## Complexity & Effort Assessment

### Per-Phase Complexity Matrix

| Phase | Projects | Issues | Mandatory | Complexity | Risk Level | Effort | Parallel |
|-------|----------|--------|-----------|------------|-----------|--------|----------|
| **Phase 1** | 1 | 0 | 0 | 🟢 Minimal | ✅ Low | 5 min | N/A |
| **Phase 2** | 2 | 280 | 2 | 🟡 Moderate | ⚠️ Medium | 30 min | Sequential |
| **Phase 3** | 11 | 300 | 11 | 🟡 Moderate | ⚠️ Medium | 60 min | Yes* |
| **Phase 4** | 13 | 400+ | 5 | 🟠 High | 🔴 High | 90 min | Partial* |
| **Phase 5** | 2 | 700+ | 9 | 🔴 Very High | 🔴 Very High | 120 min | Sequential |

**Parallel Opportunities**:
- Phase 3: All 8 console apps can build simultaneously
- Phase 4: 7 AspNetCore samples can build simultaneously (after AspNetCore library ready)

### Project-Level Risk Assessment

#### TIER 1: Foundation
```
ModelContextProtocol.Analyzers
  • Complexity: 🟢 MINIMAL (0 issues, no changes)
  • Risk: ✅ NONE
  • Effort: Trivial (verify only)
  • Critical: Yes (used by Core)
```

#### TIER 2: Core Libraries (BLOCKING FOR ENTIRE SDK)
```
ModelContextProtocol.Core
  • Complexity: 🟡 MODERATE (235 issues, mixed severity)
  • Risk: ⚠️ MEDIUM
    - 1 mandatory framework update (blocking compilation)
    - 198+ source incompatibilities requiring code fixes
    - 858+ behavioral changes requiring review
    - Used by 15+ projects
  • Effort: 30-45 min
  • Critical: YES - everything depends on this
  • Mitigation: 
    - Fix all source incompatibilities before running tests
    - Review behavioral changes systematically
    - Comprehensive testing before Phase 3

TestOAuthServer
  • Complexity: 🟢 MINIMAL (45 issues, low impact)
  • Risk: ✅ LOW
  • Effort: 10 min
  • Critical: No (only used by AspNetCore.Tests)
```

#### TIER 3: Secondary Libraries & Apps
```
Group 3A: ModelContextProtocol (Main Package)
  • Complexity: 🟢 MINIMAL (3 issues, straightforward)
  • Risk: ✅ LOW
  • Effort: 5 min
  • Critical: YES - unblocks AspNetCore layer
  • Dependencies: Core (done)

Group 3B: Console Apps (8 apps)
  • Complexity: 🟢 MINIMAL (1 issue each - framework update)
  • Risk: ✅ LOW
  • Effort: 5 min each (can parallelize)
  • Critical: No (samples/documentation)
  • Dependencies: Core (done)

Group 3C: Test Utilities (3 projects)
  • Complexity: 🟡 MODERATE (7-15 issues per project)
  • Risk: ⚠️ MEDIUM
  • Effort: 15 min each
  • Critical: YES - needed by AspNetCore.Tests
  • Dependencies: Core (done)
```

#### TIER 4: AspNetCore Layer (SECOND BIGGEST BLOCKER)
```
ModelContextProtocol.AspNetCore
  • Complexity: 🟡 MODERATE (11 issues)
  • Risk: ⚠️ MEDIUM-HIGH
    - ASP.NET Core framework changes may introduce breaking changes
    - 13+ projects depend on it
    - HTTP transport is critical path
  • Effort: 20-30 min
  • Critical: YES - largest blocking dependency after Core
  • Mitigation:
    - Review all Api.0002 source incompatibilities
    - Test immediately after update
    - Samples in Phase 4 validate AspNetCore fixes

AspNetCore Samples (7 projects)
  • Complexity: 🟡 MODERATE (1-2 issues per project)
  • Risk: ⚠️ MEDIUM
  • Effort: 5 min each (parallelize)
  • Critical: No (samples, but good for validation)
  • Dependencies: AspNetCore (must wait)

Advanced Apps (4 projects)
  • Complexity: 🟡 MODERATE (4-22 issues per project)
  • Risk: ⚠️ MEDIUM
  • Effort: 10-15 min each
  • Critical: Some (LongRunningTasks has behavioral changes to review)
  • Dependencies: Mixed (some Core only, some AspNetCore)

Special Projects (3 projects)
  • Complexity: 🟢 MINIMAL-🟡 MODERATE
  • Risk: ✅ LOW
  • Effort: 5-15 min
  • Critical: AotCompatibility (already net10.0 - validation), ConformanceServer (conformance)
```

#### TIER 5: Test Suite (FINAL VALIDATION - HIGHEST COMPLEXITY)
```
ModelContextProtocol.Tests
  • Complexity: 🟠 HIGH (191 issues, 5 mandatory)
  • Risk: 🔴 HIGH
    - 5 mandatory issues (framework + APIs)
    - Comprehensive unit tests across Core + Main package
    - Tests may reveal issues in Phases 2-3 code changes
  • Effort: 45-60 min
  • Critical: YES - must pass before production
  • Mitigation:
    - All lower phases must be stable first
    - Review all test failures carefully (may indicate hidden issues)
    - Fix issues iteratively, don't skip tests

ModelContextProtocol.AspNetCore.Tests
  • Complexity: 🔴 VERY HIGH (514 issues, 4 mandatory)
  • Risk: 🔴 CRITICAL
    - Largest test suite (514 issues!)
    - 4 mandatory issues + extensive behavioral changes
    - Depends on 7 different projects
    - Integration tests + conformance testing
    - If this fails, entire HTTP layer needs rework
  • Effort: 60-90 min
  • Critical: YES - most comprehensive validation
  • Mitigation:
    - Run after ALL other phases complete
    - Every failed test = potential issue in lower tiers
    - Systematic review of test failures
    - This is the "ultimate gate" - must pass before declaring done
```

---

### Issue Severity Breakdown

#### Mandatory Issues (39 total - BLOCKING)
- **Project.0002**: 30 occurrences - Framework target updates
- **Api.0001**: 9 occurrences - Binary incompatibilities (must recompile)

**Action**: MUST be resolved before any project is considered "complete"

#### Potential Issues (1,077 total - IMPORTANT)
- **Api.0003**: 858 occurrences - Behavioral changes (code may behave differently)
- **Api.0002**: 198 occurrences - Source incompatibilities (code won't compile)
- **NuGet.0002**: 21 occurrences - Package upgrades recommended

**Action**: Should be resolved to ensure correctness and security

---

### Risk Mitigation Strategies

| Risk | Severity | Mitigation | When |
|------|----------|-----------|------|
| Core library breaking changes | 🔴 Critical | Test Core immediately after Phase 2 | After Phase 2 completes |
| AspNetCore breaking changes | 🔴 Critical | Test AspNetCore samples immediately | After AspNetCore updated |
| Multi-target complexity | 🟠 High | Verify each target builds | After each phase |
| Large test suite failures | 🔴 Critical | Review systematically, don't skip | Phase 5 - iterate until passing |
| Behavioral changes hidden bugs | 🟠 High | Run full test suite with coverage | Phase 5 end |
| Package version conflicts | ⚠️ Medium | Update packages per phase | As each project updated |

---

### Success Criteria Per Phase

#### Phase 1: ✅ Foundation Ready
- [x] Analyzers project verified (no changes needed)
- [x] Clean build

#### Phase 2: ✅ Core Stable
- [x] Core.csproj compiles without errors
- [x] Core.csproj compiles without warnings
- [x] All framework targets build (net10.0, net9.0, net8.0, netstandard2.0)
- [x] All mandatory issues resolved
- [x] TestOAuthServer builds successfully
- [x] Core tests pass (validate no regressions)

#### Phase 3: ✅ Secondary Layer Complete
- [x] Main package builds
- [x] All 8 console apps build
- [x] All 3 test utilities build
- [x] No cascading failures from Phase 2
- [x] All NuGet updates applied where used

#### Phase 4: ✅ AspNetCore Ready
- [x] AspNetCore.csproj compiles (all targets)
- [x] All AspNetCore samples build
- [x] All advanced apps build
- [x] Special projects updated
- [x] Sample applications run without errors
- [x] No new issues introduced

#### Phase 5: ✅ READY FOR PRODUCTION
- [x] ModelContextProtocol.Tests: ALL TESTS PASS
- [x] ModelContextProtocol.AspNetCore.Tests: ALL TESTS PASS
- [x] Zero compilation warnings across all projects
- [x] All 11 NuGet packages updated
- [x] All 39 mandatory issues resolved
- [x] No regressions in existing functionality
- [x] Conformance tests pass (if applicable)

---

## Source Control Strategy

### Branch Management

**Upgrade Branch**: `upgrade-to-NET10` (created and checked out)
**Source Branch**: `main`

**Workflow**:
1. All changes committed to `upgrade-to-NET10`
2. After completion, submit PR from `upgrade-to-NET10` → `main`
3. Code review and CI/CD validation before merge
4. After merge, delete upgrade branch

### Commit Strategy

**Commits by Phase** (one major commit per phase for clarity):

#### Phase 1 Commit
```
commit: "Phase 1: Verify foundation (Analyzers)"
- No changes, verification only
- Creates baseline commit
```

#### Phase 2 Commits
```
commit: "Phase 2a: Update ModelContextProtocol.Core to net10.0"
- Update project file
- Update NuGet packages
- Fix source incompatibilities
- Fix compilation errors

commit: "Phase 2b: Update ModelContextProtocol.TestOAuthServer"
- Update project file
- Inherit Phase 2a fixes
```

#### Phase 3 Commits
```
commit: "Phase 3a: Update main package and console apps to net10.0"
- ModelContextProtocol.csproj
- All 8 console app samples

commit: "Phase 3b: Update test utilities"
- TestServer, ConformanceClient, ExperimentalApiRegressionTest
```

#### Phase 4 Commits
```
commit: "Phase 4a: Update ModelContextProtocol.AspNetCore to net10.0"
- Update AspNetCore library (critical)
- Fix HTTP transport issues

commit: "Phase 4b: Update AspNetCore samples"
- All 7 AspNetCore samples

commit: "Phase 4c: Update advanced apps and special projects"
- InMemoryTransport, LongRunningTasks, ProtectedMcpServer, etc.
```

#### Phase 5 Commits
```
commit: "Phase 5: Update and validate test suites for net10.0"
- ModelContextProtocol.Tests
- ModelContextProtocol.AspNetCore.Tests
- All tests passing, ready for production
```

### Commit Message Format

Each commit includes:
- **What changed**: Files/projects updated
- **Why**: Phase purpose and breaking changes found
- **Validation**: Build status, test results

Example:
```
Phase 2a: Update ModelContextProtocol.Core to net10.0

Updated framework targets:
- ModelContextProtocol.Core: net10.0;net9.0;net8.0;netstandard2.0

Fixed issues:
- 198 source incompatibilities (Api.0002)
- 235 behavioral changes reviewed (Api.0003)
- 11 NuGet packages updated to 10.0.8

Validation:
- ✅ Builds without errors (all targets)
- ✅ Builds without warnings
- ✅ 45/45 Core tests passing
- ✅ No regressions detected

Related: Upgrade #204384 (NET10)
```

### Pull Request Strategy

**When PR is ready** (after Phase 5):

**PR Description**:
```markdown
## .NET 10 Upgrade

Upgrades MCP C# SDK from .NET 8/9 to .NET 10.0 LTS.

### Changes
- 32 projects updated to target net10.0 (with multi-target maintained)
- 11 NuGet packages updated
- 1,116 issues resolved (39 mandatory, 1,077 potential)
- 698 breaking changes addressed

### By Phase
- Phase 1: Foundation verified (Analyzers)
- Phase 2: Core libraries updated (Core, TestOAuthServer)
- Phase 3: Secondary layer complete (main package, 11 secondary projects)
- Phase 4: AspNetCore ready (13 projects)
- Phase 5: Test suites pass (comprehensive validation)

### Validation
- ✅ All projects build without errors
- ✅ All projects build without warnings
- ✅ All tests pass (net10.0)
- ✅ Multi-targets tested (net9.0, net8.0)
- ✅ Sample applications run successfully
- ✅ No regressions detected

### Breaking Changes Log
[See attached breaking-changes.md]

### Issues Addressed
- Closes #204384 (if this is linked to an issue)

### Risk Assessment
- **Risk Level**: Medium (large scope, but well-structured phases)
- **Rollback Risk**: Low (can revert to main anytime)
- **Post-Release Issues**: Unlikely (comprehensive testing)
```

**PR Checklist**:
- ✅ All commits present for all 5 phases
- ✅ All tests passing
- ✅ CI/CD pipeline green
- ✅ Code review checklist completed
- ✅ No merge conflicts with main
- ✅ Breaking changes documented

### Version Bumping

After successful upgrade:
1. Determine SemVer impact (Major/Minor/Patch)
   - This is likely **MINOR** (framework upgrade, no API removal)
2. Update `src/Directory.Build.props` version
3. Generate release notes
4. Create release tag

---

## Success Criteria

### Technical Criteria (MANDATORY)
- ✅ **All 32 projects target net10.0** (or maintain multi-target with net10.0 added)
- ✅ **All 11 NuGet packages upgraded** to recommended versions
- ✅ **All 39 mandatory issues resolved** (Project.0002, Api.0001 completely gone)
- ✅ **All projects build without errors** (dotnet build succeeds)
- ✅ **All projects build without warnings** (TreatWarningsAsErrors=true passes)
- ✅ **All tests pass** (ModelContextProtocol.Tests and AspNetCore.Tests at 100% pass rate)
- ✅ **No package dependency conflicts** (no NU1605 or NU1608 warnings)
- ✅ **Multi-target support verified** (projects targeting net9.0/net8.0 still build)
- ✅ **AOT compatibility maintained** (ModelContextProtocol.AotCompatibility.TestApp runs)

### Quality Criteria (IMPORTANT)
- ✅ **Code quality maintained** (no new warnings introduced)
- ✅ **Test coverage maintained** (coverage % not decreased)
- ✅ **Documentation updated** (release notes, breaking changes logged)
- ✅ **Performance acceptable** (no unexplained regressions)
- ✅ **Backward compatibility** (multi-targets still function)

### Process Criteria (REQUIRED)
- ✅ **Bottom-Up strategy followed** (phases executed in order)
- ✅ **Each phase validated** (Phase N complete before Phase N+1)
- ✅ **Breaking changes documented** (log created)
- ✅ **Commits organized** (one per phase, clear messages)
- ✅ **PR checklist complete** (ready for code review)
- ✅ **CI/CD pipeline passing** (all checks green)

### Sign-Off Criteria (FINAL)
- ✅ **Technical Lead**: Approves all phases
- ✅ **Automated Tests**: All passing
- ✅ **Code Review**: At least 2 approvals
- ✅ **Documentation**: Release notes complete
- ✅ **Go/No-Go Decision**: Ready for merge to main

---

## Next Steps

### Immediate (Before Starting Execution)
1. Review this plan.md
2. Confirm 5-phase approach is acceptable
3. Verify branch `upgrade-to-NET10` is correct
4. Identify code reviewers for PR

### During Execution (Per Phase)
1. Execute phase according to plan
2. Document actual findings vs plan predictions
3. Run validation steps
4. Commit changes
5. Move to next phase only after validation

### Post-Execution (After Phase 5)
1. Create PR with all commits
2. Submit for code review
3. Address review comments
4. Merge to main
5. Tag release
6. Update documentation
7. Close issue #204384

---

**Plan Version**: 1.0
**Created**: 2024
**Target Framework**: .NET 10.0 LTS
**Branches**: `main` → `upgrade-to-NET10` (PR back to main)
**Strategy**: Bottom-Up (Dependency-First, 5-Phase Approach)
