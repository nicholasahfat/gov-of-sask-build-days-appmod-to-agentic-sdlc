---
applyTo: "capstone-src/api-dotnet/tests/**/*.cs"
---

# .NET capstone test guidance

- Follow the existing xUnit convention: `[Fact]` tests with descriptive method names and `Assert.*`.
- For repository clock behavior, use the injectable `TimeProvider` seam in `src/WorkItems.Api/InMemoryWorkItemRepository.cs`.
- From the repository root, run `dotnet test capstone-src/api-dotnet/tests/WorkItems.Api.Tests/WorkItems.Api.Tests.csproj`.
- Keep tests isolated from live cloud services, network access, production data, and elapsed wall-clock timing.
