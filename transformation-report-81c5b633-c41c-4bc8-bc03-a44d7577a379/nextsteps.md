# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The transformation appears to have completed successfully. No build errors were detected across any of the projects in the solution:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

The following steps outline how to validate, test, and deploy the migrated solution.

---

## 1. Restore Dependencies

Run a full NuGet package restore to ensure all dependencies are resolved correctly:

```bash
dotnet restore
```

Review the output for any warnings related to package compatibility or version conflicts. Address any packages that may have been resolved to unexpected versions.

---

## 2. Build the Solution

Perform a full solution build to confirm there are no compilation issues:

```bash
dotnet build --configuration Release
```

Ensure the build completes with zero errors and review any warnings, particularly those related to nullable reference types, deprecated APIs, or platform compatibility.

---

## 3. Run Unit Tests

Execute the test project to verify that existing business logic behaves as expected after the migration:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test results carefully. Any failing tests should be investigated to determine whether they indicate a regression introduced during migration or a pre-existing issue.

---

## 4. Validate the Data Layer

Since `Bookstore.Data` handles data access, verify the following:

- **Database provider**: Confirm that the correct cross-platform compatible NuGet package is referenced (e.g., `Microsoft.EntityFrameworkCore.SqlServer`, `Npgsql.EntityFrameworkCore.PostgreSQL`, or equivalent).
- **Connection strings**: Ensure connection strings in configuration files (e.g., `appsettings.json`) are correct for the target environment.
- **Migrations**: If Entity Framework Core is in use, verify that existing migrations are intact and apply cleanly:

```bash
dotnet ef database update --project app/Bookstore.Data/Bookstore.Data.csproj --startup-project app/Bookstore.Web/Bookstore.Web.csproj
```

---

## 5. Run and Validate the Web Application

Start the web application locally and verify core functionality:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Manually test key application flows such as browsing, searching, and any data entry features. Pay particular attention to areas that interact with the data layer or any platform-specific APIs that may have been affected by the migration.

---

## 6. Review the CDK Project

The `Bookstore.Cdk` project likely defines infrastructure. Review it to ensure:

- Any environment-specific configuration values (e.g., region, account IDs, resource names) are correctly set for the target deployment environment.
- The CDK version referenced is compatible with the current AWS CDK CLI version installed on your machine.

Synthesize the CDK stack to confirm it produces valid output:

```bash
cd app/Bookstore.Cdk
cdk synth
```

Address any synthesis errors before proceeding to deployment.

---

## 7. Deploy the Infrastructure and Application

Once validation steps above are complete, deploy the CDK stack to provision or update infrastructure:

```bash
cdk deploy
```

After infrastructure is in place, publish and deploy the web application to the target environment:

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

Copy the contents of the `./publish` directory to the appropriate hosting environment (e.g., an EC2 instance, Elastic Beanstalk, or App Service).

---

## 8. Post-Deployment Verification

After deployment, perform the following checks:

- Confirm the application starts and responds to HTTP requests.
- Verify database connectivity from the deployed environment.
- Review application logs for any runtime errors that did not surface during local testing.