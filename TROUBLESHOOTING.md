# Gmail OAuth 403 Access Denied - Troubleshooting Guide

## Error Details
You're getting a 403 Access Denied error when trying to authenticate with Google OAuth for Gmail.

## Root Causes

### 1. Restricted Scopes Requiring Verification ⚠️ (MOST LIKELY)

Your OAuth request includes these scopes:
- `https://mail.google.com/` - **Full Gmail access (RESTRICTED)**
- `https://www.googleapis.com/auth/gmail.modify` - Modify Gmail
- `https://www.googleapis.com/auth/gmail.compose` - Compose emails
- `https://www.googleapis.com/auth/gmail.labels` - Manage labels
- Gmail addons scopes

The `https://mail.google.com/` scope is a **restricted scope** that requires Google app verification for production use.

## Solutions

### Option 1: Use Testing Mode (QUICKEST FIX)

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Select your project
3. Navigate to: **APIs & Services > OAuth consent screen**
4. Set **Publishing status** to **"Testing"**
5. Add your Gmail account as a **Test User**:
   - Click "ADD USERS" under "Test users"
   - Enter your Gmail address
   - Save

**Important**: In testing mode:
- Only test users can authenticate
- No verification needed
- Works immediately
- Limited to 100 test users

### Option 2: Use Less Restrictive Scopes (RECOMMENDED)

Modify n8n's Gmail credential to use only these scopes:
- `https://www.googleapis.com/auth/gmail.modify` - Read, send, delete, modify emails
- `https://www.googleapis.com/auth/gmail.labels` - Manage labels

These scopes should be sufficient for deleting emails and don't require verification.

**How to change scopes in n8n:**
1. Delete existing Gmail credential
2. Create new credential
3. In Google Cloud Console, update the OAuth consent screen scopes
4. Remove `https://mail.google.com/` from enabled scopes
5. Keep only `gmail.modify` and `gmail.labels`

### Option 3: Submit for Google Verification (PRODUCTION)

If you need unrestricted access:

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Navigate to: **APIs & Services > OAuth consent screen**
3. Complete all required information:
   - App name, logo, privacy policy, terms of service
   - Authorized domains
   - Developer contact information
4. Click **"PUBLISH APP"**
5. Submit for verification

**Note**: Verification can take 4-6 weeks and requires:
- Privacy policy URL
- Terms of service URL
- YouTube video demo
- Detailed justification for restricted scopes

### Option 4: Check OAuth Consent Screen Configuration

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Navigate to: **APIs & Services > OAuth consent screen**
3. Verify:
   - ✅ User type is set (Internal or External)
   - ✅ App name is filled in
   - ✅ User support email is set
   - ✅ Developer contact email is set
   - ✅ Redirect URI matches: `https://n8n.int.bierstadt.xyz/rest/oauth2-credential/callback`

4. Navigate to: **APIs & Services > Credentials**
5. Click on your OAuth 2.0 Client ID
6. Under "Authorized redirect URIs", ensure you have:
   ```
   https://n8n.int.bierstadt.xyz/rest/oauth2-credential/callback
   ```

### Option 5: Check API Enablement

Ensure Gmail API is enabled:

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Navigate to: **APIs & Services > Library**
3. Search for "Gmail API"
4. Click on it and ensure it's **ENABLED**

## Quick Fix Steps (Recommended)

**If this is for personal/testing use:**

1. **Enable Testing Mode:**
   - Google Cloud Console > OAuth consent screen
   - Set to "Testing" mode
   - Add your email as test user

2. **Verify Redirect URI:**
   - Credentials > Your OAuth Client
   - Add: `https://n8n.int.bierstadt.xyz/rest/oauth2-credential/callback`

3. **Enable Gmail API:**
   - APIs & Services > Library
   - Search "Gmail API" > Enable

4. **Re-authenticate in n8n:**
   - Delete old credential
   - Create new Gmail OAuth2 credential
   - Complete OAuth flow

## Still Getting 403?

Check these additional issues:

### Domain Restrictions
- Your Google Workspace might restrict OAuth apps
- Contact your workspace admin

### Account Issues
- Try with a personal Gmail account (not workspace)
- Ensure account doesn't have 2FA issues

### Redirect URI Mismatch
- Ensure exact match (including https://)
- No trailing slashes
- Case-sensitive

### Clear Browser Cache
- Clear cookies and cache
- Try incognito mode
- Try different browser

## Testing the Fix

After making changes:
1. Wait 1-2 minutes for Google to propagate changes
2. Clear browser cookies/cache
3. Try OAuth flow again in n8n
4. Check error message for any new details

## Additional Resources

- [Google OAuth Verification](https://support.google.com/cloud/answer/9110914)
- [Gmail API Scopes](https://developers.google.com/gmail/api/auth/scopes)
- [OAuth 2.0 Guide](https://developers.google.com/identity/protocols/oauth2)

## Need More Help?

Provide these details:
1. Is this a personal Gmail or Google Workspace account?
2. OAuth consent screen status (Testing/In Production)?
3. Are you listed as a test user?
4. Screenshot of the full error message
5. OAuth consent screen configuration screenshot
