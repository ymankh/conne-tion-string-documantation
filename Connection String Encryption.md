# Setup Process for Providing an Encrypted Connection String for a Published API on IIS

This document provides step-by-step instructions to set up an encrypted connection string for a published API hosted on IIS. The setup requires PowerShell 7 and the .NET Hosting Bundle.

## Prerequisites

1. **Install IIS**: Ensure that IIS is installed and configured on your server.
2. **PowerShell 7**: Download and install PowerShell 7 from the [official website](https://github.com/PowerShell/PowerShell/releases/tag/v7.4.6).
3. **.NET Hosting Bundle**: Install the appropriate version of the .NET Hosting Bundle for your application. You can download it from the [.NET website](https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/runtime-aspnetcore-8.0.12-windows-hosting-bundle-installer).

---

## Step 1: Encrypt the Connection String

1. **Open PowerShell 7**:

   - Launch PowerShell 7 with administrative privileges.

2. **Run the Encryption Script**:
   - Copy and paste the following script into PowerShell:

```powershell
# Prompt the user for the connection string
$PlainText = Read-Host -Prompt "Enter the connection string to encrypt"

# Convert the plain text to bytes
$PlainTextBytes = [System.Text.Encoding]::UTF8.GetBytes($PlainText)

# Encrypt using DPAPI, tied to the machine
$EncryptedBytes = [System.Security.Cryptography.ProtectedData]::Protect(
    $PlainTextBytes,
    $null,
    [System.Security.Cryptography.DataProtectionScope]::LocalMachine
)

# Convert the encrypted bytes to Base64 for easy transfer or storage
$EncryptedBase64 = [Convert]::ToBase64String($EncryptedBytes)

# Output the encrypted connection string
Write-Output "Encrypted Connection String: $EncryptedBase64"
```

3. **Enter the Connection String**:

   - When prompted, enter the plaintext connection string.

4. **Save the Output**:
   - Copy the generated **Encrypted Connection String**. This will be used in the next step.

---

## Step 2: Store the Encrypted Connection String in IIS ASP.NET Core Environment Variables

I'll create a Markdown file with steps to add environment variables in IIS using ASP.NET Core configurations, including placeholders for images.

# Configuring Environment Variables for Connection Strings in ASP.NET Core with IIS

Create or update your `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "PostgresConnection": "Host=localhost;Port=5432;Database=tgis_config;Username=postgres;Password=postgres"
  }
}
```

## Step 2: Create Environment-Specific Configuration

In `Program.cs` or `Startup.cs`, ensure environment variable configuration:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseConfiguration(new ConfigurationBuilder()
                    .SetBasePath(Directory.GetCurrentDirectory())
                    .AddJsonFile("appsettings.json")
                    .AddEnvironmentVariables()
                    .Build());
                webBuilder.UseStartup<Startup>();
            });
}
```

## Step 3: Configure IIS Environment Variables


1. Open IIS Manager
2. Select your website/application
3. Open "Configuration Editor"
![IIS Environment Variables](./images/1.png)
4. Navigate to `system.webServer/aspNetCore/environmentVariables`
![IIS Environment Variables](./images/2.png)
5. Add new environment variables for the connection string with name ConnectionStrings__PostgresConnection:
![IIS Environment Variables](./images/3.png)
![IIS Environment Variables](./images/4.png)
6. Save the changes and restart IIS.

### You will see the connection string stored in the web.config file in the following form:


```xml
<environmentVariables>
  <environmentVariable name="ConnectionStrings__DefaultConnection" value="your-actual-connection-string" />
</environmentVariables>
```

## Troubleshooting Tips

- Double-check naming conventions
- Ensure IIS application pool has necessary permissions
- Restart IIS after configuration changes