# Manual MSSQL Certificate Restoration Guide

---

## 🔐 Prerequisites

1.  **Access:** SQL Server Management Studio (SSMS) or `sqlcmd` connected to the target instance.
2.  **Files:** Ensure `SQL.CER` and `SQL.pvk` are located at `C:\cert\` on the **SQL Server machine**.
3.  **Passwords:** Retrieve the following passwords from **AWS Systems Manager (SSM) Parameter Store**:
    *   **Master Key Password:** `test-environment-mssql-masterkey`
    *   **Private Key Password:** `test-environment-mssql-certificate-password`

---

## 🛠 Step-by-Step Restoration

### Step 1: Create the Master Key
The Master Key must exist in the `master` database to protect the certificate you are about to import.

```sql
USE master;
GO

-- Check if it already exists to avoid errors
IF NOT EXISTS (SELECT * FROM sys.symmetric_keys WHERE name = '##MS_DatabaseMasterKey##')
BEGIN
    -- Replace 'xxxxxxxx' with the value from SSM: test-environment-mssql-masterkey
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'xxxxxxxx';
END
GO


### Step 2: Create the Certificate from Files
This step imports the certificate and the private key so the server can decrypt databases encrypted with this specific cert.

```
USE master;
GO

-- Check if the certificate already exists
IF NOT EXISTS (SELECT * FROM sys.certificates WHERE name = 'TDE_Cert_STG')
BEGIN
    CREATE CERTIFICATE TDE_Cert_STG 
    FROM FILE = 'C:\cert\SQL.CER' 
    WITH PRIVATE KEY (
        FILE = 'C:\cert\SQL.pvk', 
        -- Replace 'xxxxxxxx' with the value from SSM: test-environment-mssql-certificate-password
        DECRYPTION BY PASSWORD = 'xxxxxxxx' 
    );
END
GO
```

### Step 3: Verification
Run these diagnostic queries to ensure the configuration was successful.

Check the Certificate
========================

```
SELECT name, subject, expiry_date 
FROM sys.certificates 
WHERE name = 'TDE_Cert_STG';
```

Check the Master Key
========================

```
SELECT is_master_key_encrypted_by_server 
FROM sys.databases 
WHERE name = 'master';
```




