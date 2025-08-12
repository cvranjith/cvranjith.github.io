**TL;DR**:
`DBMS_CLOUD.PUT_OBJECT` can upload binaries, but it doesn’t let you attach **user-defined metadata**. To set metadata (the `opc-meta-*` headers), call the **Object Storage REST API** directly from PL/SQL using `DBMS_CLOUD.SEND_REQUEST` and authenticate with an **OCI API Signing Key** credential (not an Auth Token).

---

## Why this trick?
If you need to stamp files not only with business context—say customerId=C-1029 or docType=invoice—but also with technical metadata like content-type=application/pdf —you must send those as `opc-meta-*` HTTP headers on the **PUT** request. That’s not supported by `DBMS_CLOUD.PUT_OBJECT`, so we invoke the REST endpoint ourselves with `DBMS_CLOUD.SEND_REQUEST`.

> **Heads-up on credentials**: For `SEND_REQUEST`, use an **API key** (RSA key + fingerprint) credential. Auth Tokens are great for other `DBMS_CLOUD` operations, but they don’t work reliably for `SEND_REQUEST` calls to the Object Storage REST endpoints.

---

## Step 1 — Create an API Key credential in ADB
Create a credential backed by an **OCI API Signing Key** (user OCID, tenancy OCID, **PEM private key**, and **fingerprint**).

```plsql
-- API Key credential (DBMS_CLOUD.CREATE_CREDENTIAL)
BEGIN
  DBMS_CLOUD.CREATE_CREDENTIAL(
    credential_name => 'obj_store_cred',
    user_ocid       => 'ocid1.user.oc1..aaaaaaaaa...',
    tenancy_ocid    => 'ocid1.tenancy.oc1..aaaaaaaa...',
    private_key     => '-----BEGIN PRIVATE KEY-----
YOUR PEM=
-----END PRIVATE KEY-----
OCI_API_KEY',
    fingerprint     => 'yo:ur:fi:ng:er:pr:in:t...'
  );
END;
/
```

**Tip:** Make sure your IAM user/group has a policy that allows put/write on the target bucket/namespace/region.

---

## Step 2 — PUT the object with metadata via REST
Build the HTTP headers (including your `opc-meta-*` pairs) and invoke **PUT** to the regional Object Storage endpoint.

```plsql
DECLARE
  l_resp  DBMS_CLOUD_TYPES.RESP;
  l_hdrs  JSON_OBJECT_T := JSON_OBJECT_T();
  l_blob  BLOB;
BEGIN
  -- Get your file as BLOB (example: from a table)
  SELECT file_content INTO l_blob FROM your_table WHERE id = 1;

  -- Required content type + user-defined metadata
  l_hdrs.put('content-type', 'application/pdf');
  l_hdrs.put('opc-meta-customerId', 'C-1029');
  l_hdrs.put('opc-meta-docType',    'invoice');

  -- Upload with metadata
  l_resp := DBMS_CLOUD.SEND_REQUEST(
    credential_name => 'obj_store_cred',
    uri             => 'https://objectstorage.ap-seoul-1.oraclecloud.com/n/<your_namespace>/b/<your_bucket>/o/demo/test.pdf',
    method          => DBMS_CLOUD.METHOD_PUT,
    headers         => l_hdrs.to_clob,  -- pass JSON headers as CLOB
    body            => l_blob           -- your file BLOB
  );

  DBMS_OUTPUT.PUT_LINE('HTTP status: ' || l_resp.status_code);
END;
/
```

That’s it—your object is stored with metadata like `opc-meta-customerId=C-1029`.

---

## (Optional) Verify the metadata
You can issue a **HEAD** request and inspect the response headers.

```plsql
DECLARE
  l_resp  DBMS_CLOUD_TYPES.RESP;
BEGIN
  l_resp := DBMS_CLOUD.SEND_REQUEST(
    credential_name => 'obj_store_cred',
    uri             => 'https://objectstorage.ap-seoul-1.oraclecloud.com/n/<your_namespace>/b/<your_bucket>/o/demo/test.pdf',
    method          => DBMS_CLOUD.METHOD_HEAD
  );
  DBMS_OUTPUT.PUT_LINE(l_resp.headers); -- contains the opc-meta-* pairs
END;
/
```

---

## Troubleshooting
- **401/403**: Confirm IAM policy, correct **region** in the endpoint, and the **namespace** in the URI.  
- **Credential name mismatch**: Use the same `credential_name` you created.  
- **Auth Token vs API Key**: If you used an **Auth Token** credential and get auth errors with `SEND_REQUEST`, switch to an **API Signing Key** credential.  
- **Content-Type**: Set a proper `content-type` for the uploaded object so clients recognize it correctly.

---

## References
- Oracle docs — *User-defined metadata uses the `opc-meta-` prefix*  
  https://docs.public.content.oci.oraclecloud.com/en-us/iaas/compute-cloud-at-customer/topics/object/managing-storage-objects.htm  
- Oracle docs — *When to use Auth Tokens vs. API Signing Keys in DBMS_CLOUD*  
  https://docs.oracle.com/en-us/iaas/autonomous-database/doc/dbms_cloud-access-management.html  
- Oracle blog — *`DBMS_CLOUD.SEND_REQUEST` basics (headers/body as JSON, returns `DBMS_CLOUD_TYPES.RESP`)*  
  https://blogs.oracle.com/developers/post/reliably-requesting-rest-results-right-from-your-sql-scripts

---

*Last updated: 2025-08-12*
