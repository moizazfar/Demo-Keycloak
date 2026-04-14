# Keycloak + Supabase Integration


---

## Big Picture — What We Are Setting Up

```
YOUR AIOPS DATA IN SUPABASE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  system_metrics          → CPU, memory, network
  anomaly_detection       → ML anomaly results
  ml_predictions          → model outputs
  alerts_incidents        → alerts and incidents
  log_analysis            → log data
  infrastructure_monitoring → infra data

WHO GETS WHAT ACCESS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  aiops-admin    → full read + write + delete ALL tables
  ml-engineer    → read all + write predictions/anomalies
  aiops-analyst  → read only ALL tables
  external-api   → read only specific tables

HOW IT WORKS AFTER INTEGRATION:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  User/Service → Login Keycloak → Get Token
  Token → Sent to Supabase API
  Supabase → Checks role in token
  Role matches → Data returned
  Role wrong → 403 Forbidden
```

---

# PART 1 — Keycloak UI Setup

## Step 1: Login to Keycloak Admin Console

```
Open browser:
https://<your-keycloak-route>/admin

Username: admin
Password: Admin@12345
```

---

## Step 2: Switch to aiops-dev Realm

```
Top left corner → click dropdown
Currently shows "master"
Click "aiops-dev"

You are now inside aiops-dev realm
All next steps happen inside this realm
```

---

## Step 3: Update Existing Roles (Add Missing Ones)

We already have 3 roles. We need to add one more for external APIs.

```
Left sidebar → Realm roles

You should see:
  ✅ aiops-admin
  ✅ aiops-analyst
  ✅ ml-engineer

We need to ADD:
  ➕ external-api
```

Click **Create role**:
```
┌──────────────────────────────────────────────┐
│  Role name:   external-api                   │
│  Description: External apps and APIs         │
│               read-only access to AIOps data │
└──────────────────────────────────────────────┘
Click Save
```

---

## Step 4: Create Supabase Client

```
Left sidebar → Clients → Create client
```

**Page 1 — General Settings:**
```
┌──────────────────────────────────────────────┐
│  Client type:    OpenID Connect              │
│  Client ID:      supabase                    │
│  Name:           Supabase AIOps Platform     │
│  Description:    Keycloak auth for Supabase  │
│  Always display: OFF                         │
└──────────────────────────────────────────────┘
Click Next
```

**Page 2 — Capability Config:**
```
┌──────────────────────────────────────────────┐
│  Client authentication:    ON  ✅            │
│  Authorization:            OFF               │
│                                              │
│  Authentication flow:                        │
│  ✅ Standard flow                            │
│  ✅ Direct access grants                     │
│  ☐  Implicit flow          OFF               │
│  ☐  Service accounts roles OFF               │
└──────────────────────────────────────────────┘
Click Next
```

**Page 3 — Login Settings:**
```
┌──────────────────────────────────────────────┐
│  Root URL:                                   │
│  https://<your-supabase-studio-route>        │
│                                              │
│  Valid redirect URIs:                        │
│  https://<your-supabase-studio-route>/*      │
│  (click + to add another)                    │
│  http://localhost:3000/*                     │
│                                              │
│  Valid post logout redirect URIs:            │
│  https://<your-supabase-studio-route>/*      │
│                                              │
│  Web origins:                                │
│  https://<your-supabase-studio-route>        │
│  +                                           │
└──────────────────────────────────────────────┘
Click Save
```

---

## Step 5: Get Client Secret

```
Clients → supabase → Credentials tab

┌──────────────────────────────────────────────┐
│  Client Authenticator: Client Id and Secret  │
│                                              │
│  Client secret: xxxxxxxx-xxxx-xxxx-xxxx      │
│                 [Copy]  ← CLICK THIS         │
└──────────────────────────────────────────────┘

SAVE THIS SECRET SOMEWHERE
We need it in Supabase config later
```

---

## Step 6: Add Role Claim to Token (MOST IMPORTANT STEP)

This is what tells Supabase WHO the user is and WHAT they can access.

```
Clients → supabase → Client scopes tab

You will see:
  supabase-dedicated   ← Click this
  supabase-basic
```

Inside supabase-dedicated:
```
Click "Add mapper" button (top right)
Click "By configuration"
```

**Mapper 1 — Add Realm Roles to Token:**
```
Choose: User Realm Role

┌──────────────────────────────────────────────┐
│  Name:              realm-roles              │
│  Multivalued:       ON  ✅                   │
│  Token Claim Name:  roles                    │
│  Claim JSON Type:   String                   │
│                                              │
│  Add to ID token:        ON ✅               │
│  Add to access token:    ON ✅               │
│  Add to userinfo:        ON ✅               │
│  Add to token introspection: ON ✅           │
└──────────────────────────────────────────────┘
Click Save
```

**Mapper 2 — Add Email to Token:**
```
Go back → Add mapper → By configuration
Choose: User Property

┌──────────────────────────────────────────────┐
│  Name:              email                    │
│  Property:          email                    │
│  Token Claim Name:  email                    │
│  Claim JSON Type:   String                   │
│                                              │
│  Add to ID token:        ON ✅               │
│  Add to access token:    ON ✅               │
│  Add to userinfo:        ON ✅               │
└──────────────────────────────────────────────┘
Click Save
```

**Mapper 3 — Add Username to Token:**
```
Go back → Add mapper → By configuration
Choose: User Property

┌──────────────────────────────────────────────┐
│  Name:              username                 │
│  Property:          username                 │
│  Token Claim Name:  preferred_username       │
│  Claim JSON Type:   String                   │
│                                              │
│  Add to access token:    ON ✅               │
│  Add to userinfo:        ON ✅               │
└──────────────────────────────────────────────┘
Click Save
```

---

## Step 7: Create Service Account Users

These are non-human users for your AIOps ML services:

```
Left sidebar → Users → Create new user
```

**ML Service User:**
```
┌──────────────────────────────────────────────┐
│  Username:     ml-service-account            │
│  Email:        ml-service@aiops.internal     │
│  First name:   ML                            │
│  Last name:    Service                       │
│  Enabled:      ON ✅                         │
└──────────────────────────────────────────────┘
Click Create

→ Credentials tab
  Password: MLService@2026
  Temporary: OFF
  Click Save password

→ Role mapping tab
  Click Assign role
  Select: ml-engineer
  Click Assign
```

**External API User:**
```
Create new user:
┌──────────────────────────────────────────────┐
│  Username:     external-api-user             │
│  Email:        api@aiops.internal            │
│  First name:   External                      │
│  Last name:    API                           │
│  Enabled:      ON ✅                         │
└──────────────────────────────────────────────┘
Click Create

→ Credentials tab
  Password: ExtApi@2026
  Temporary: OFF
  Click Save password

→ Role mapping tab
  Click Assign role
  Select: external-api
  Click Assign
```

---

## Step 8: Configure Token Settings

```
Left sidebar → Realm settings → Tokens tab

┌──────────────────────────────────────────────┐
│  Default Signature Algorithm: RS256  ✅      │
│  (Supabase needs RS256 — NOT HS256)          │
│                                              │
│  Access Token Lifespan:   5 minutes          │
│  (short = more secure)                       │
│                                              │
│  SSO Session Idle:        30 minutes         │
│  SSO Session Max:         8 hours            │
└──────────────────────────────────────────────┘
Click Save
```

---

## Step 9: Get Keycloak Public Key

Supabase needs this to verify tokens are genuine.

```
Left sidebar → Realm settings → Keys tab

You will see a list of keys.
Find the row where:
  Algorithm = RS256
  Use = SIG

Click "Public key" button on that row

┌──────────────────────────────────────────────┐
│  MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKC   │
│  AQEAxxx...xxxx==                            │
│                      [Copy] ← CLICK          │
└──────────────────────────────────────────────┘

SAVE THIS KEY
We need it in Supabase config
```

Also note your Realm URLs:
```
Left sidebar → Realm settings → General tab

OpenID Endpoint Configuration → Click this link
It opens a JSON page. Note these values:

issuer:  https://<keycloak-route>/realms/aiops-dev
jwks_uri: https://<keycloak-route>/realms/aiops-dev/protocol/openid-connect/certs
```

---

# PART 2 — Supabase UI Setup

## Step 10: Login to Supabase Studio

```
Open browser:
https://<your-supabase-studio-route>

Login with your Supabase admin credentials
```

---

## Step 11: Configure JWT Settings in Supabase

```
Left sidebar → Project Settings (gear icon)
→ API section

You will see:
┌──────────────────────────────────────────────┐
│  JWT Settings                                │
│                                              │
│  JWT Secret: [current value]                 │
│                                              │
│  We need to configure this to work          │
│  with Keycloak RS256 tokens                 │
└──────────────────────────────────────────────┘
```

---

## Step 12: Add Keycloak as Auth Provider

```
Left sidebar → Authentication
→ Providers tab (or Sign In / Up tab)
→ Scroll down to find "Custom OIDC" or "OpenID Connect"
```

Click **Add new provider** or find the OIDC section:
```
┌──────────────────────────────────────────────┐
│  Provider:        OpenID Connect (OIDC)      │
│  Enabled:         ON ✅                      │
│                                              │
│  Name:            keycloak                   │
│                                              │
│  Client ID:       supabase                   │
│  (same as what we created in Keycloak)       │
│                                              │
│  Client Secret:   paste-secret-from-step-5  │
│                                              │
│  Issuer URL:                                 │
│  https://<keycloak-route>/realms/aiops-dev   │
│                                              │
│  (Supabase auto-discovers all endpoints      │
│   from this URL)                             │
│                                              │
│  Skip nonce check:     ON ✅                 │
└──────────────────────────────────────────────┘
Click Save
```

---

## Step 13: Enable Row Level Security on AIOps Tables

```
Left sidebar → Table Editor
```

Do this for EACH of your AIOps tables:

### For system_metrics table:
```
Click on "system_metrics" table
Top right → click "RLS disabled" toggle
Click "Enable RLS"

┌──────────────────────────────────────────────┐
│  Row Level Security                          │
│  Status: ENABLED ✅                          │
│                                              │
│  Without policies = NO ONE can read data     │
│  We add policies next                        │
└──────────────────────────────────────────────┘
```

Repeat for ALL tables:
```
✅ system_metrics          → Enable RLS
✅ anomaly_detection       → Enable RLS
✅ ml_predictions          → Enable RLS
✅ alerts_incidents        → Enable RLS
✅ log_analysis            → Enable RLS
✅ infrastructure_monitoring → Enable RLS
```

---

## Step 14: Create RLS Policies via SQL Editor

```
Left sidebar → SQL Editor
Click "New query"
```

**Paste and run this SQL — creates all policies for all tables:**

```sql
-- ============================================================
-- KEYCLOAK + SUPABASE RLS POLICIES
-- AIOps Platform — All Tables
-- ============================================================

-- ─── SYSTEM METRICS ───────────────────────────────────────

-- Admins: full access
CREATE POLICY "admins full access on system_metrics"
ON public.system_metrics
FOR ALL
USING (
  (auth.jwt() ->> 'roles') LIKE '%aiops-admin%'
);

-- Analysts + Authenticated: read only
CREATE POLICY "analysts read system_metrics"
ON public.system_metrics
FOR SELECT
USING (
  auth.role() = 'authenticated'
);

-- ─── ANOMALY DETECTION ────────────────────────────────────

-- Admins: full access
CREATE POLICY "admins full access on anomaly_detection"
ON public.anomaly_detection
FOR ALL
USING (
  (auth.jwt() ->> 'roles') LIKE '%aiops-admin%'
);

-- ML Engineers: read + write
CREATE POLICY "ml engineers write anomaly_detection"
ON public.anomaly_detection
FOR INSERT
WITH CHECK (
  (auth.jwt() ->> 'roles') LIKE '%ml-engineer%'
);

CREATE POLICY "ml engineers update anomaly_detection"
ON public.anomaly_detection
FOR UPDATE
USING (
  (auth.jwt() ->> 'roles') LIKE '%ml-engineer%'
);

-- All authenticated: read only
CREATE POLICY "authenticated read anomaly_detection"
ON public.anomaly_detection
FOR SELECT
USING (auth.role() = 'authenticated');

-- ─── ML PREDICTIONS ───────────────────────────────────────

-- Admins: full access
CREATE POLICY "admins full access on ml_predictions"
ON public.ml_predictions
FOR ALL
USING (
  (auth.jwt() ->> 'roles') LIKE '%aiops-admin%'
);

-- ML Engineers: read + write predictions
CREATE POLICY "ml engineers write ml_predictions"
ON public.ml_predictions
FOR INSERT
WITH CHECK (
  (auth.jwt() ->> 'roles') LIKE '%ml-engineer%'
);

CREATE POLICY "ml engineers update ml_predictions"
ON public.ml_predictions
FOR UPDATE
USING (
  (auth.jwt() ->> 'roles') LIKE '%ml-engineer%'
);

-- All authenticated: read
CREATE POLICY "authenticated read ml_predictions"
ON public.ml_predictions
FOR SELECT
USING (auth.role() = 'authenticated');

-- ─── ALERTS AND INCIDENTS ─────────────────────────────────

-- Admins: full access
CREATE POLICY "admins full access on alerts_incidents"
ON public.alerts_incidents
FOR ALL
USING (
  (auth.jwt() ->> 'roles') LIKE '%aiops-admin%'
);

-- ML Engineers: can create alerts
CREATE POLICY "ml engineers insert alerts"
ON public.alerts_incidents
FOR INSERT
WITH CHECK (
  (auth.jwt() ->> 'roles') LIKE '%ml-engineer%'
);

-- All authenticated: read
CREATE POLICY "authenticated read alerts"
ON public.alerts_incidents
FOR SELECT
USING (auth.role() = 'authenticated');

-- External API: read only
CREATE POLICY "external api read alerts"
ON public.alerts_incidents
FOR SELECT
USING (
  (auth.jwt() ->> 'roles') LIKE '%external-api%'
);

-- ─── LOG ANALYSIS ─────────────────────────────────────────

-- Admins: full access
CREATE POLICY "admins full access on log_analysis"
ON public.log_analysis
FOR ALL
USING (
  (auth.jwt() ->> 'roles') LIKE '%aiops-admin%'
);

-- All authenticated: read
CREATE POLICY "authenticated read logs"
ON public.log_analysis
FOR SELECT
USING (auth.role() = 'authenticated');

-- ─── INFRASTRUCTURE MONITORING ────────────────────────────

-- Admins: full access
CREATE POLICY "admins full access on infrastructure_monitoring"
ON public.infrastructure_monitoring
FOR ALL
USING (
  (auth.jwt() ->> 'roles') LIKE '%aiops-admin%'
);

-- All authenticated: read
CREATE POLICY "authenticated read infrastructure"
ON public.infrastructure_monitoring
FOR SELECT
USING (auth.role() = 'authenticated');

-- External API: read only infrastructure
CREATE POLICY "external api read infrastructure"
ON public.infrastructure_monitoring
FOR SELECT
USING (
  (auth.jwt() ->> 'roles') LIKE '%external-api%'
);
```

Click **Run** button (or Ctrl+Enter)

---

## Step 15: Verify Policies Created

```
SQL Editor → New query → paste and run:

SELECT
  tablename,
  policyname,
  cmd,
  qual
FROM pg_policies
WHERE schemaname = 'public'
ORDER BY tablename, policyname;

You should see all policies listed for all 6 tables
```

---

# PART 3 — Test the Integration

## Step 16: Test Login Flow in Browser

```
Open NEW browser tab (incognito)

Go to: https://<supabase-studio-route>

You should see:
┌──────────────────────────────────────────────┐
│  Sign in to Supabase Studio                  │
│                                              │
│  [Continue with Keycloak]  ← button appears  │
│                                              │
│  OR normal email login                       │
└──────────────────────────────────────────────┘

Click "Continue with Keycloak"

→ Redirects to Keycloak login page
→ Login with: testuser / Test@12345
→ Redirects back to Supabase Studio
→ You are logged in!
```

---

## Step 17: Verify Token in Supabase

```
Supabase Studio → Authentication → Users tab

You should see testuser listed here
This confirms Keycloak token was accepted
```

---

# Complete Access Matrix

```
WHO CAN DO WHAT ON EACH TABLE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TABLE                    │ aiops-admin │ ml-engineer │ aiops-analyst │ external-api
─────────────────────────┼─────────────┼─────────────┼───────────────┼─────────────
system_metrics           │ ALL         │ READ        │ READ          │ READ
anomaly_detection        │ ALL         │ READ+WRITE  │ READ          │ READ
ml_predictions           │ ALL         │ READ+WRITE  │ READ          │ READ
alerts_incidents         │ ALL         │ READ+INSERT │ READ          │ READ
log_analysis             │ ALL         │ READ        │ READ          │ READ
infrastructure_monitoring│ ALL         │ READ        │ READ          │ READ

ALL = SELECT + INSERT + UPDATE + DELETE
READ = SELECT only
READ+WRITE = SELECT + INSERT + UPDATE
READ+INSERT = SELECT + INSERT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

# If Something Does Not Work

```
PROBLEM 1: "Continue with Keycloak" button not showing
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
→ Go back to Supabase Auth → Providers
→ Make sure Keycloak OIDC provider is Enabled ON
→ Check Issuer URL is exactly correct

PROBLEM 2: Redirect fails after Keycloak login
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
→ Go to Keycloak → Clients → supabase → Settings
→ Check Valid redirect URIs has correct Supabase URL
→ Make sure URL ends with /*

PROBLEM 3: Data not showing even after login
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
→ Supabase SQL Editor → run this to check:
  SELECT auth.jwt() ->> 'roles';
→ Should return your role name
→ If NULL → mapper not configured (redo Step 6)

PROBLEM 4: 403 Forbidden on all tables
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
→ RLS is ON but policies not created yet
→ Go to SQL Editor and run the SQL from Step 14 again
```
