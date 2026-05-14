# .NET 10 Upgrade - Execution Tasks

**Status**: In Progress
**Branch**: upgrade-to-NET10
**Target Framework**: net10.0
**Strategy**: Bottom-Up (5 Phases)

---

## Progress Summary

- [ ] Phase 1: Foundation (1 task)
- [ ] Phase 2: Core Libraries (6 tasks)
- [ ] Phase 3: Secondary Layer (13 tasks)
- [ ] Phase 4: AspNetCore Layer (15 tasks)
- [ ] Phase 5: Test Suite Validation (3 tasks)

**Total**: 38 tasks

---

## TASK-001: Phase 1 - Verify Foundation (Analyzers)

**Phase**: 1 - Foundation
**Priority**: CRITICAL
**Effort**: Minimal (5 min)
**Status**: [ ] Not Started

### Description
Verify that `ModelContextProtocol.Analyzers` project is stable. This is netstandard2.0 and should not require changes for .NET 10.0 upgrade.

### Actions
1. [ ] Build ModelContextProtocol.Analyzers project
2. [ ] Verify no compilation errors
3. [ ] Verify no warnings

### Commands
```powershell
dotnet build src/ModelContextProtocol.Analyzers/ModelContextProtocol.Analyzers.csproj -c Release
```

### Success Criteria
- ✅ Build succeeds
- ✅ Zero errors
- ✅ Zero warnings

### Dependencies
None (foundation project)

---

## TASK-002: Phase 2 - Update ModelContextProtocol.Core Target Frameworks

**Phase**: 2 - Core Libraries
**Priority**: CRITICAL (Blocks 15+ projects)
**Effort**: Low (5 min)
**Status**: [ ] Not Started

### Description
Update `ModelContextProtocol.Core.csproj` to maintain multi-targeting with net10.0 already included. Verify current targets are: net10.0;net9.0;net8.0;netstandard2.0

### Actions
1. [ ] Open src/ModelContextProtocol.Core/ModelContextProtocol.Core.csproj
2. [ ] Verify TargetFrameworks includes net10.0
3. [ ] If missing, add net10.0 to TargetFrameworks

### File to Modify
`src/ModelContextProtocol.Core/ModelContextProtocol.Core.csproj`

### Expected Change
```xml
<TargetFrameworks>net10.0;net9.0;net8.0;netstandard2.0</TargetFrameworks>
```

### Success Criteria
- ✅ TargetFrameworks includes net10.0
- ✅ Project file syntax valid

### Dependencies
TASK-001 (Phase 1 complete)

---

## TASK-003: Phase 2 - Update NuGet Packages in ModelContextProtocol.Core

**Phase**: 2 - Core Libraries
**Priority**: CRITICAL
**Effort**: Moderate (10 min)
**Status**: [ ] Not Started

### Description
Update Microsoft.Extensions.* packages from 10.0.7 to 10.0.8 in ModelContextProtocol.Core.

### Actions
1. [ ] Update Microsoft.Extensions.Caching.Abstractions to 10.0.8
2. [ ] Update Microsoft.Extensions.Caching.Memory to 10.0.8
3. [ ] Update Microsoft.Extensions.DependencyInjection to 10.0.8
4. [ ] Update Microsoft.Extensions.Hosting.Abstractions to 10.0.8
5. [ ] Update Microsoft.Extensions.Logging to 10.0.8
6. [ ] Update Microsoft.Extensions.Logging.Abstractions to 10.0.8
7. [ ] Update Microsoft.Extensions.Options to 10.0.8
8. [ ] Run dotnet restore

### File to Modify
`src/ModelContextProtocol.Core/ModelContextProtocol.Core.csproj`

### Commands
```powershell
dotnet restore src/ModelContextProtocol.Core/ModelContextProtocol.Core.csproj
```

### Success Criteria
- ✅ All packages updated to 10.0.8
- ✅ dotnet restore succeeds
- ✅ No package conflicts (NU1605, NU1608)

### Dependencies
TASK-002 (Core framework updated)

---

## TASK-004: Phase 2 - Build ModelContextProtocol.Core (All Targets)

**Phase**: 2 - Core Libraries
**Priority**: CRITICAL
**Effort**: Moderate (10 min)
**Status**: [ ] Not Started

### Description
Build ModelContextProtocol.Core for all target frameworks and fix any compilation errors from source incompatibilities.

### Actions
1. [ ] Build for net10.0
2. [ ] Build for net9.0
3. [ ] Build for net8.0
4. [ ] Build for netstandard2.0
5. [ ] Fix any compilation errors (Api.0002)
6. [ ] Ensure zero warnings

### Commands
```powershell
dotnet build src/ModelContextProtocol.Core/ModelContextProtocol.Core.csproj -c Release
```

### Success Criteria
- ✅ Builds successfully for all targets
- ✅ Zero compilation errors
- ✅ Zero warnings

### Dependencies
TASK-003 (NuGet packages updated)

### Notes
Watch for:
- Obsolete API usage
- Signature changes in Microsoft.Extensions.* APIs
- Multi-target conditional compilation needs

---

## TASK-005: Phase 2 - Update ModelContextProtocol.TestOAuthServer

**Phase**: 2 - Core Libraries
**Priority**: LOW (Only used by AspNetCore.Tests)
**Effort**: Low (5 min)
**Status**: [ ] Not Started

### Description
Update TestOAuthServer framework targets and dependencies.

### Actions
1. [ ] Verify TargetFrameworks: net10.0;net9.0;net8.0
2. [ ] Update any Microsoft.Extensions.* packages to 10.0.8
3. [ ] Build project

### File to Modify
`tests/ModelContextProtocol.TestOAuthServer/ModelContextProtocol.TestOAuthServer.csproj`

### Commands
```powershell
dotnet build tests/ModelContextProtocol.TestOAuthServer/ModelContextProtocol.TestOAuthServer.csproj -c Release
```

### Success Criteria
- ✅ Build succeeds
- ✅ Zero warnings

### Dependencies
TASK-004 (Core builds successfully)

---

## TASK-006: Phase 2 - Run Core Tests

**Phase**: 2 - Core Libraries
**Priority**: CRITICAL (Validates Phase 2)
**Effort**: Moderate (15 min)
**Status**: [ ] Not Started

### Description
Run ModelContextProtocol.Tests to validate no regressions in Core functionality after .NET 10 upgrade.

### Actions
1. [ ] Build ModelContextProtocol.Tests
2. [ ] Run tests for net10.0 target
3. [ ] Verify all tests pass
4. [ ] Investigate any failures

### Commands
```powershell
dotnet build tests/ModelContextProtocol.Tests/ModelContextProtocol.Tests.csproj -c Release
dotnet test tests/ModelContextProtocol.Tests/ModelContextProtocol.Tests.csproj -c Release -f net10.0 --no-build --logger "console;verbosity=normal"
```

### Success Criteria
- ✅ All tests pass
- ✅ No test failures related to Core changes
- ✅ No behavioral regressions detected

### Dependencies
TASK-004, TASK-005

### Blocker
If tests fail, Phase 3 cannot start until issues resolved.

---

## TASK-007: Phase 2 - Commit Phase 2 Changes

**Phase**: 2 - Core Libraries
**Priority**: HIGH
**Effort**: Low (5 min)
**Status**: [ ] Not Started

### Description
Commit all Phase 2 changes with descriptive message.

### Actions
1. [ ] Stage all modified .csproj files
2. [ ] Commit with message

### Commands
```powershell
git add src/ModelContextProtocol.Core/
git add tests/ModelContextProtocol.TestOAuthServer/
git commit -m "Phase 2: Update Core libraries to .NET 10.0

- Updated ModelContextProtocol.Core to net10.0 (maintained multi-target)
- Updated Microsoft.Extensions.* packages to 10.0.8
- Updated ModelContextProtocol.TestOAuthServer
- All Core tests passing (net10.0)

Resolved:
- 235 issues in Core
- 45 issues in TestOAuthServer
- No source incompatibilities found
"
```

### Success Criteria
- ✅ Changes committed
- ✅ Commit message clear

### Dependencies
TASK-006 (Tests pass)

---

## TASK-008: Phase 3 - Update ModelContextProtocol (Main Package)

**Phase**: 3 - Secondary Layer
**Priority**: CRITICAL (Unblocks AspNetCore)
**Effort**: Low (5 min)
**Status**: [ ] Not Started

### Description
Update main package framework targets and dependencies.

### Actions
1. [ ] Verify TargetFrameworks: net10.0;net9.0;net8.0;netstandard2.0
2. [ ] Update NuGet packages if needed
3. [ ] Build project

### File to Modify
`src/ModelContextProtocol/ModelContextProtocol.csproj`

### Commands
```powershell
dotnet build src/ModelContextProtocol/ModelContextProtocol.csproj -c Release
```

### Success Criteria
- ✅ Build succeeds (all targets)
- ✅ Zero warnings

### Dependencies
TASK-007 (Phase 2 complete)

---

## TASK-009: Phase 3 - Update Console Sample Apps to net10.0

**Phase**: 3 - Secondary Layer
**Priority**: MEDIUM
**Effort**: Moderate (20 min - can parallelize)
**Status**: [ ] Not Started

### Description
Update all 8 console sample applications from net8.0/net9.0 to net10.0.

### Projects to Update
1. [ ] samples/ChatWithTools/ChatWithTools.csproj (net8.0 → net10.0)
2. [ ] docs/concepts/elicitation/samples/client/ElicitationClient.csproj (net9.0 → net10.0)
3. [ ] docs/concepts/logging/samples/client/LoggingClient.csproj (net9.0 → net10.0)
4. [ ] docs/concepts/progress/samples/client/ProgressClient.csproj (net9.0 → net10.0)
5. [ ] samples/QuickstartClient/QuickstartClient.csproj (net8.0 → net10.0)
6. [ ] samples/ProtectedMcpClient/ProtectedMcpClient.csproj (net9.0 → net10.0)
7. [ ] tests/ModelContextProtocol.Analyzers.Tests/ModelContextProtocol.Analyzers.Tests.csproj (net9.0 → net10.0)
8. [ ] samples/InMemoryTransport/InMemoryTransport.csproj (net8.0 → net10.0)

### Actions per Project
1. [ ] Update TargetFramework to net10.0
2. [ ] Update Microsoft.Extensions.* packages to 10.0.8 (if present)
3. [ ] Build project
4. [ ] Verify no errors

### Commands (Example for one)
```powershell
# Repeat for each project
dotnet build samples/ChatWithTools/ChatWithTools.csproj -c Release
```

### Success Criteria
- ✅ All 8 projects build successfully
- ✅ Zero warnings across all projects

### Dependencies
TASK-008 (Main package ready)

---

## TASK-010: Phase 3 - Update Test Utilities

**Phase**: 3 - Secondary Layer
**Priority**: HIGH (Needed for Phase 5)
**Effort**: Moderate (15 min)
**Status**: [ ] Not Started

### Description
Update test utility projects to maintain multi-targeting with net10.0.

### Projects to Update
1. [ ] tests/ModelContextProtocol.TestServer/ModelContextProtocol.TestServer.csproj
   - Verify: net10.0;net9.0;net8.0;net472
2. [ ] tests/ModelContextProtocol.ConformanceClient/ModelContextProtocol.ConformanceClient.csproj
   - Verify: net10.0;net9.0;net8.0
3. [ ] tests/ModelContextProtocol.ExperimentalApiRegressionTest/ModelContextProtocol.ExperimentalApiRegressionTest.csproj
   - Verify: net10.0;net9.0;net8.0

### Actions
1. [ ] Verify each has net10.0 in TargetFrameworks
2. [ ] Update NuGet packages if needed
3. [ ] Build all three projects

### Commands
```powershell
dotnet build tests/ModelContextProtocol.TestServer/ -c Release
dotnet build tests/ModelContextProtocol.ConformanceClient/ -c Release
dotnet build tests/ModelContextProtocol.ExperimentalApiRegressionTest/ -c Release
```

### Success Criteria
- ✅ All projects build (all targets)
- ✅ Zero warnings

### Dependencies
TASK-008 (Main package ready)

---

## TASK-011: Phase 3 - Run Sample Applications

**Phase**: 3 - Secondary Layer
**Priority**: MEDIUM (Validation)
**Effort**: Low (10 min)
**Status**: [ ] Not Started

### Description
Run at least 2 console samples to ensure they execute without runtime errors.

### Actions
1. [ ] Run QuickstartClient
2. [ ] Run ChatWithTools (or another sample)
3. [ ] Verify no runtime errors

### Commands
```powershell
dotnet run --project samples/QuickstartClient/QuickstartClient.csproj
dotnet run --project samples/ChatWithTools/ChatWithTools.csproj
```

### Success Criteria
- ✅ Samples start without errors
- ✅ No unexpected exceptions

### Dependencies
TASK-009 (Samples built)

### Notes
May need sample-specific inputs; basic validation is sufficient.

---

## TASK-012: Phase 3 - Commit Phase 3 Changes

**Phase**: 3 - Secondary Layer
**Priority**: HIGH
**Effort**: Low (5 min)
**Status**: [ ] Not Started

### Description
Commit all Phase 3 changes.

### Actions
1. [ ] Stage all modified .csproj files
2. [ ] Commit with message

### Commands
```powershell
git add src/ModelContextProtocol/
git add samples/
git add docs/concepts/
git add tests/ModelContextProtocol.TestServer/
git add tests/ModelContextProtocol.ConformanceClient/
git add tests/ModelContextProtocol.ExperimentalApiRegressionTest/
git commit -m "Phase 3: Update secondary layer to .NET 10.0

- Updated main package (ModelContextProtocol)
- Updated 8 console sample apps to net10.0
- Updated 3 test utilities (maintained multi-target)
- All projects building successfully
- Sample applications run without errors

Resolved:
- 11 mandatory framework issues
- All samples validated
"
```

### Success Criteria
- ✅ Changes committed
- ✅ Clean git status

### Dependencies
TASK-011 (Validation complete)

---

## TASK-013: Phase 4 - Update ModelContextProtocol.AspNetCore

**Phase**: 4 - AspNetCore Layer
**Priority**: CRITICAL (Blocks 13 projects)
**Effort**: Moderate (20 min)
**Status**: [ ] Not Started

### Description
Update AspNetCore library to .NET 10.0, fixing any ASP.NET Core API breaking changes.

### Actions
1. [ ] Verify TargetFrameworks: net10.0;net9.0;net8.0
2. [ ] Update Microsoft.AspNetCore.* packages to 10.0.8
3. [ ] Update Microsoft.Extensions.* packages to 10.0.8
4. [ ] Build project (all targets)
5. [ ] Fix any compilation errors related to ASP.NET Core API changes
6. [ ] Verify McpEndpointRouteBuilderExtensions.cs compiles

### File to Modify
`src/ModelContextProtocol.AspNetCore/ModelContextProtocol.AspNetCore.csproj`

### Key File to Watch
`src/ModelContextProtocol.AspNetCore/McpEndpointRouteBuilderExtensions.cs`
- Uses MapGroup, MapPost, MapGet, MapDelete
- Verify these APIs still work in .NET 10

### Commands
```powershell
dotnet build src/ModelContextProtocol.AspNetCore/ModelContextProtocol.AspNetCore.csproj -c Release
```

### Success Criteria
- ✅ Builds successfully (all targets)
- ✅ Zero compilation errors
- ✅ Zero warnings
- ✅ Endpoint mapping APIs still work

### Dependencies
TASK-012 (Phase 3 complete)

### Notes
CRITICAL BLOCKER: If this fails, all 13 AspNetCore-dependent projects blocked.

---

## TASK-014: Phase 4 - Update AspNetCore Sample Apps

**Phase**: 4 - AspNetCore Layer
**Priority**: MEDIUM
**Effort**: Moderate (20 min - can parallelize)
**Status**: [ ] Not Started

### Description
Update all AspNetCore sample applications to net10.0.

### Projects to Update
1. [ ] samples/AspNetCoreMcpPerSessionTools/AspNetCoreMcpPerSessionTools.csproj (net9.0 → net10.0)
2. [ ] samples/AspNetCoreMcpServer/AspNetCoreMcpServer.csproj (net9.0 → net10.0)
3. [ ] docs/concepts/elicitation/samples/server/Elicitation.csproj (net9.0 → net10.0)
4. [ ] samples/EverythingServer/EverythingServer.csproj (net9.0 → net10.0)
5. [ ] docs/concepts/httpcontext/samples/HttpContext.csproj (net9.0 → net10.0)
6. [ ] docs/concepts/logging/samples/server/Logging.csproj (net9.0 → net10.0)
7. [ ] docs/concepts/progress/samples/server/Progress.csproj (net9.0 → net10.0)

### Actions per Project
1. [ ] Update TargetFramework to net10.0
2. [ ] Update NuGet packages if needed
3. [ ] Build project

### Commands (Example)
```powershell
dotnet build samples/AspNetCoreMcpServer/ -c Release
```

### Success Criteria
- ✅ All 7 projects build successfully
- ✅ Zero warnings

### Dependencies
TASK-013 (AspNetCore library ready)

---

## TASK-015: Phase 4 - Update Advanced Applications

**Phase**: 4 - AspNetCore Layer
**Priority**: MEDIUM
**Effort**: Moderate (15 min)
**Status**: [ ] Not Started

### Description
Update advanced sample and application projects.

### Projects to Update
1. [ ] samples/LongRunningTasks/LongRunningTasks.csproj (net9.0 → net10.0)
   - Has behavioral changes to review
2. [ ] samples/ProtectedMcpServer/ProtectedMcpServer.csproj (net9.0 → net10.0)
   - Depends on AspNetCore
3. [ ] samples/QuickstartWeatherServer/QuickstartWeatherServer.csproj (net8.0 → net10.0)
4. [ ] tests/ModelContextProtocol.ConformanceServer/ModelContextProtocol.ConformanceServer.csproj
   - Verify: net10.0;net9.0;net8.0

### Actions
1. [ ] Update each project's framework
2. [ ] Update NuGet packages
3. [ ] Build all projects

### Commands
```powershell
dotnet build samples/LongRunningTasks/ -c Release
dotnet build samples/ProtectedMcpServer/ -c Release
dotnet build samples/QuickstartWeatherServer/ -c Release
dotnet build tests/ModelContextProtocol.ConformanceServer/ -c Release
```

### Success Criteria
- ✅ All projects build successfully
- ✅ Zero warnings

### Dependencies
TASK-013 (AspNetCore ready)

---

## TASK-016: Phase 4 - Update Special Projects

**Phase**: 4 - AspNetCore Layer
**Priority**: LOW
**Effort**: Low (10 min)
**Status**: [ ] Not Started

### Description
Update special-purpose projects.

### Projects to Update
1. [ ] samples/TestServerWithHosting/TestServerWithHosting.csproj
   - Verify: net10.0;net9.0;net8.0;net472 (should already have net10.0)
2. [ ] tests/ModelContextProtocol.TestSseServer/ModelContextProtocol.TestSseServer.csproj
   - Verify: net10.0;net9.0;net8.0
3. [ ] tests/ModelContextProtocol.AotCompatibility.TestApp/ModelContextProtocol.AotCompatibility.TestApp.csproj
   - Already net10.0 - just verify no regressions

### Actions
1. [ ] Verify each has net10.0
2. [ ] Build all projects
3. [ ] Run AotCompatibility.TestApp to ensure AOT still works

### Commands
```powershell
dotnet build samples/TestServerWithHosting/ -c Release
dotnet build tests/ModelContextProtocol.TestSseServer/ -c Release
dotnet build tests/ModelContextProtocol.AotCompatibility.TestApp/ -c Release
```

### Success Criteria
- ✅ All projects build
- ✅ AOT compatibility maintained

### Dependencies
TASK-013 (AspNetCore ready)

---

## TASK-017: Phase 4 - Run AspNetCore Sample Servers

**Phase**: 4 - AspNetCore Layer
**Priority**: HIGH (Validation)
**Effort**: Moderate (15 min)
**Status**: [ ] Not Started

### Description
Start at least 2 AspNetCore sample servers and verify they initialize without errors.

### Actions
1. [ ] Start AspNetCoreMcpServer
2. [ ] Verify server starts and logs no errors
3. [ ] Stop server
4. [ ] Start one other AspNetCore sample
5. [ ] Verify endpoints are mapped correctly

### Commands
```powershell
# Start sample (blocks terminal)
dotnet run --project samples/AspNetCoreMcpServer/

# In another terminal, test basic health (optional)
# curl http://localhost:5000/health

# Stop with Ctrl+C
```

### Success Criteria
- ✅ Servers start without errors
- ✅ Endpoints initialize correctly
- ✅ No HTTP/SSE transport errors

### Dependencies
TASK-014 (AspNetCore samples built)

### Notes
This validates McpEndpointRouteBuilderExtensions.cs and HTTP transport work.

---

## TASK-018: Phase 4 - Commit Phase 4 Changes

**Phase**: 4 - AspNetCore Layer
**Priority**: HIGH
**Effort**: Low (5 min)
**Status**: [ ] Not Started

### Description
Commit all Phase 4 changes.

### Actions
1. [ ] Stage all modified .csproj files
2. [ ] Commit with message

### Commands
```powershell
git add src/ModelContextProtocol.AspNetCore/
git add samples/
git add docs/concepts/
git add tests/ModelContextProtocol.ConformanceServer/
git add tests/ModelContextProtocol.TestSseServer/
git add tests/ModelContextProtocol.AotCompatibility.TestApp/
git commit -m "Phase 4: Update AspNetCore layer to .NET 10.0

- Updated ModelContextProtocol.AspNetCore (maintained multi-target)
- Updated 7 AspNetCore sample apps to net10.0
- Updated 4 advanced apps and special projects
- All AspNetCore samples start successfully
- HTTP/SSE endpoints validated

Resolved:
- 13+ AspNetCore-dependent projects
- Endpoint mapping APIs verified (MapGroup, MapPost, MapGet, MapDelete)
- No breaking changes in ASP.NET Core 10.0 found
"
```

### Success Criteria
- ✅ Changes committed
- ✅ Clean git status

### Dependencies
TASK-017 (Validation complete)

---

## TASK-019: Phase 5 - Build Test Projects

**Phase**: 5 - Test Suite Validation
**Priority**: CRITICAL
**Effort**: Moderate (10 min)
**Status**: [ ] Not Started

### Description
Build both comprehensive test suites to prepare for test execution.

### Projects to Build
1. [ ] tests/ModelContextProtocol.Tests/ModelContextProtocol.Tests.csproj
   - Verify: net10.0;net9.0;net8.0;net472
2. [ ] tests/ModelContextProtocol.AspNetCore.Tests/ModelContextProtocol.AspNetCore.Tests.csproj
   - Verify: net10.0;net9.0;net8.0

### Actions
1. [ ] Build ModelContextProtocol.Tests (all targets)
2. [ ] Build ModelContextProtocol.AspNetCore.Tests (all targets)
3. [ ] Verify no compilation errors

### Commands
```powershell
dotnet build tests/ModelContextProtocol.Tests/ -c Release
dotnet build tests/ModelContextProtocol.AspNetCore.Tests/ -c Release
```

### Success Criteria
- ✅ Both projects build successfully (all targets)
- ✅ Zero compilation errors
- ✅ Zero warnings

### Dependencies
TASK-018 (Phase 4 complete)

---

## TASK-020: Phase 5 - Run ModelContextProtocol.Tests

**Phase**: 5 - Test Suite Validation
**Priority**: CRITICAL
**Effort**: High (30 min)
**Status**: [ ] Not Started

### Description
Run full ModelContextProtocol.Tests suite for net10.0 and investigate any failures.

### Actions
1. [ ] Run tests for net10.0
2. [ ] Record test results
3. [ ] Investigate any failures
4. [ ] Fix issues or update tests
5. [ ] Re-run until all pass

### Commands
```powershell
dotnet test tests/ModelContextProtocol.Tests/ -c Release -f net10.0 --no-build --logger "console;verbosity=normal"
```

### Success Criteria
- ✅ All tests pass for net10.0
- ✅ Zero test failures
- ✅ All behavioral changes validated

### Dependencies
TASK-019 (Tests built)

### Notes
191 issues expected, 5 mandatory. Test failures indicate:
- Real code issues (fix code)
- Behavioral changes (validate expected, update test if correct)
- Test artifacts (update test)

Systematic review required.

---

## TASK-021: Phase 5 - Run ModelContextProtocol.AspNetCore.Tests

**Phase**: 5 - Test Suite Validation
**Priority**: CRITICAL (Final Gate)
**Effort**: Very High (60+ min)
**Status**: [ ] Not Started

### Description
Run full AspNetCore.Tests suite for net10.0 - the most comprehensive validation with 514 issues.

### Actions
1. [ ] Run tests for net10.0
2. [ ] Record test results
3. [ ] Investigate failures systematically (one at a time)
4. [ ] Fix issues or update tests
5. [ ] Re-run until all pass
6. [ ] Document all resolutions

### Commands
```powershell
dotnet test tests/ModelContextProtocol.AspNetCore.Tests/ -c Release -f net10.0 --no-build --logger "console;verbosity=detailed"
```

### Success Criteria
- ✅ ALL tests pass for net10.0
- ✅ Zero test failures
- ✅ All 514 issues resolved or validated
- ✅ No regressions in HTTP/SSE transport
- ✅ Conformance tests pass

### Dependencies
TASK-020 (Core tests pass)

### Notes
**LARGEST TASK**: 514 issues, 4 mandatory.
**CRITICAL GATE**: Do not declare upgrade complete until this passes.

May require multiple iterations:
1. Fix batch of issues
2. Rebuild
3. Re-test
4. Repeat

Keep detailed log of each failure and resolution.

---

## TASK-022: Phase 5 - Multi-Target Validation (Optional)

**Phase**: 5 - Test Suite Validation
**Priority**: MEDIUM
**Effort**: Moderate (20 min)
**Status**: [ ] Not Started

### Description
Run tests for net9.0 and net8.0 targets to ensure backward compatibility maintained.

### Actions
1. [ ] Run ModelContextProtocol.Tests for net9.0
2. [ ] Run ModelContextProtocol.Tests for net8.0
3. [ ] Run AspNetCore.Tests for net9.0
4. [ ] Run AspNetCore.Tests for net8.0
5. [ ] Verify all pass

### Commands
```powershell
dotnet test tests/ModelContextProtocol.Tests/ -c Release -f net9.0 --no-build
dotnet test tests/ModelContextProtocol.Tests/ -c Release -f net8.0 --no-build
dotnet test tests/ModelContextProtocol.AspNetCore.Tests/ -c Release -f net9.0 --no-build
dotnet test tests/ModelContextProtocol.AspNetCore.Tests/ -c Release -f net8.0 --no-build
```

### Success Criteria
- ✅ All tests pass for net9.0
- ✅ All tests pass for net8.0
- ✅ No regressions in older targets

### Dependencies
TASK-021 (net10.0 tests pass)

---

## TASK-023: Phase 5 - Commit Phase 5 Changes

**Phase**: 5 - Test Suite Validation
**Priority**: HIGH
**Effort**: Low (5 min)
**Status**: [ ] Not Started

### Description
Commit all Phase 5 test fixes and validation results.

### Actions
1. [ ] Stage all modified test files
2. [ ] Commit with message

### Commands
```powershell
git add tests/
git commit -m "Phase 5: Validate and fix test suites for .NET 10.0

- All ModelContextProtocol.Tests passing (net10.0)
- All ModelContextProtocol.AspNetCore.Tests passing (net10.0)
- 705 total issues resolved across test suites
- Multi-target validation complete (net9.0, net8.0)

Test Results:
- Core Tests: 100% pass rate
- AspNetCore Tests: 100% pass rate
- No regressions detected
- All behavioral changes validated

UPGRADE COMPLETE - Ready for production
"
```

### Success Criteria
- ✅ Changes committed
- ✅ All tests documented as passing

### Dependencies
TASK-021 or TASK-022

---

## TASK-024: Final - Build Entire Solution

**Phase**: Final Validation
**Priority**: CRITICAL
**Effort**: Moderate (10 min)
**Status**: [ ] Not Started

### Description
Build entire solution to verify all 32 projects compile successfully.

### Actions
1. [ ] Clean solution
2. [ ] Build entire solution (all configurations)
3. [ ] Verify zero errors
4. [ ] Verify zero warnings

### Commands
```powershell
dotnet clean ModelContextProtocol.slnx
dotnet build ModelContextProtocol.slnx -c Release
```

### Success Criteria
- ✅ Entire solution builds
- ✅ Zero compilation errors
- ✅ Zero warnings
- ✅ All 32 projects successful

### Dependencies
TASK-023 (Phase 5 complete)

---

## TASK-025: Final - Run All Tests

**Phase**: Final Validation
**Priority**: CRITICAL
**Effort**: High (30 min)
**Status**: [ ] Not Started

### Description
Run entire test suite across all projects to ensure end-to-end validation.

### Actions
1. [ ] Run all tests in solution
2. [ ] Verify all pass
3. [ ] Document results

### Commands
```powershell
dotnet test ModelContextProtocol.slnx -c Release --no-build --logger "console;verbosity=normal"
```

### Success Criteria
- ✅ All tests pass
- ✅ 100% pass rate
- ✅ No test failures

### Dependencies
TASK-024 (Solution builds)

---

## TASK-026: Final - Create Pull Request

**Phase**: Final Validation
**Priority**: HIGH
**Effort**: Moderate (20 min)
**Status**: [ ] Not Started

### Description
Create PR from upgrade-to-NET10 → main with comprehensive description.

### Actions
1. [ ] Push branch to remote
2. [ ] Create PR on GitHub
3. [ ] Fill out PR description (see plan.md)
4. [ ] Add reviewers
5. [ ] Wait for CI/CD validation

### Commands
```powershell
git push origin upgrade-to-NET10
# Then create PR via GitHub UI
```

### PR Template
```markdown
## .NET 10 Upgrade

Upgrades MCP C# SDK from .NET 8/9 to .NET 10.0 LTS.

### Summary
- 32 projects updated to net10.0
- 11 NuGet packages upgraded
- 1,116 issues resolved
- All tests passing (100%)

### Phases Completed
- ✅ Phase 1: Foundation verified
- ✅ Phase 2: Core libraries updated
- ✅ Phase 3: Secondary layer complete
- ✅ Phase 4: AspNetCore ready
- ✅ Phase 5: Test suites validated

### Validation
- ✅ Entire solution builds without errors or warnings
- ✅ All tests pass (net10.0, net9.0, net8.0)
- ✅ Sample applications run successfully
- ✅ Multi-target compatibility maintained
- ✅ AOT compatibility verified
- ✅ No regressions detected

### Breaking Changes
[See breaking-changes.md if any were found]

### Related
- Assessment: .github/upgrades/scenarios/new-dotnet-version_204384/assessment.md
- Plan: .github/upgrades/scenarios/new-dotnet-version_204384/plan.md
- Tasks: .github/upgrades/scenarios/new-dotnet-version_204384/tasks.md
```

### Success Criteria
- ✅ PR created
- ✅ Description complete
- ✅ Reviewers assigned

### Dependencies
TASK-025 (All tests pass)

---

## Execution Log

### Phase 1: Foundation
*To be filled during execution*

### Phase 2: Core Libraries
*To be filled during execution*

### Phase 3: Secondary Layer
*To be filled during execution*

### Phase 4: AspNetCore Layer
*To be filled during execution*

### Phase 5: Test Suite Validation
*To be filled during execution*

### Final Validation
*To be filled during execution*

---

**Last Updated**: [Timestamp will be updated during execution]
**Current Status**: Ready to Execute
**Next Task**: TASK-001 (Phase 1 - Verify Foundation)
