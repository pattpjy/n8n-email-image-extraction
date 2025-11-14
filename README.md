# Gmail Delete by Keyword Workflow

This n8n workflow allows you to delete Gmail emails based on a keyword search.

## Workflow Overview

The workflow consists of 9 nodes and **automatically loops** until all matching emails are deleted:

1. **Manual Trigger** - Starts the workflow when you click "Test workflow"
2. **Set Keyword** - Defines the keyword to search for (default: "spam")
3. **Gmail - Search Emails** - Searches Gmail for emails matching the keyword (max 50 per iteration)
4. **Check if Results Found** - Validates that search returned results before proceeding
5. **Gmail - Delete Email** - Deletes all emails found in the search (only runs if results exist)
6. **Aggregate Deleted Emails** - Combines all deleted email results into one item
7. **Wait 30 Seconds** - Pauses execution to avoid Gmail API rate limits
8. **Preserve Keyword for Loop** - Maintains the search keyword and loops back to step 3
9. **No Emails Found** - End node when no matching emails are found (stops the loop)

## Setup Instructions

### 1. Import the Workflow

1. Open your n8n instance
2. Click on "Workflows" in the left sidebar
3. Click "Import from File"
4. Select the `gmail-delete-by-keyword-workflow.json` file

### 2. Configure Gmail Credentials

1. Click on the "Gmail - Search Emails" node
2. Click on "Create New Credential" for Gmail OAuth2
3. Follow the OAuth2 setup process:
   - Go to [Google Cloud Console](https://console.cloud.google.com/)
   - Create a new project or select an existing one
   - Enable the Gmail API
   - Create OAuth2 credentials (OAuth client ID)
   - Add authorized redirect URI: `https://your-n8n-instance/rest/oauth2-credential/callback`
   - Copy the Client ID and Client Secret to n8n
   - Complete the OAuth2 authentication flow

4. The same credentials will be automatically used for the "Gmail - Delete Email" node

### 3. Customize the Search Keyword

In the "Set Keyword" node, you can modify the keyword value to search for specific terms:

- **Current default**: `spam`
- **Examples**:
  - `newsletter` - to delete newsletters
  - `unsubscribe` - to delete emails with unsubscribe links
  - `promotion` - to delete promotional emails

### 4. Advanced Search Options

The workflow uses Gmail's search query syntax. You can modify the query in the "Gmail - Search Emails" node:

**Current query**: `subject:{{$json.keyword}}`

**Other query examples**:
- `from:sender@example.com` - Delete emails from a specific sender
- `subject:{{$json.keyword}} is:unread` - Only delete unread emails
- `{{$json.keyword}} older_than:30d` - Delete emails older than 30 days
- `in:spam {{$json.keyword}}` - Search only in spam folder
- `label:promotions` - Delete all promotional emails

### 5. Adjust Search Limit

By default, the workflow processes up to 50 emails at a time. You can:
- Increase the `limit` value in "Gmail - Search Emails" node
- Set `returnAll: true` to process all matching emails (use with caution!)

## Usage

1. Open the workflow in n8n
2. Modify the keyword in the "Set Keyword" node if needed
3. Click "Test workflow" to run
4. The workflow will **automatically loop** until all matching emails are deleted:
   - **Iteration 1**: Search for up to 50 emails → Delete → Wait 30s
   - **Iteration 2**: Search for up to 50 emails → Delete → Wait 30s
   - **Iteration 3**: Search for up to 50 emails → Delete → Wait 30s
   - **...continues until no more emails match...**
   - **Final Iteration**: Search returns 0 results → Stops

## How the Automatic Looping Works

### The Loop Mechanism

1. **Search**: Finds up to 50 emails matching your keyword
2. **Check**: If results found → proceed, if not → stop
3. **Delete**: Removes all found emails (processes each one)
4. **Aggregate**: Combines deletion results into one item
5. **Wait**: Pauses for 30 seconds (prevents API rate limits)
6. **Loop Back**: Returns to step 1 and searches again

### Why the 30-Second Delay?

- **Prevents Gmail API rate limiting** (250 quota units per user per second)
- **Allows Gmail to process deletions** before searching again
- **Safe and respectful** to Google's servers

### When Does It Stop?

The loop stops automatically when the Gmail search returns **zero results**, meaning all matching emails have been deleted.

## Example Execution

If you have 237 emails matching "spam":
- **Loop 1**: Delete 50 emails (187 remaining) → Wait 30s
- **Loop 2**: Delete 50 emails (137 remaining) → Wait 30s
- **Loop 3**: Delete 50 emails (87 remaining) → Wait 30s
- **Loop 4**: Delete 50 emails (37 remaining) → Wait 30s
- **Loop 5**: Delete 37 emails (0 remaining) → Wait 30s
- **Loop 6**: Find 0 emails → **Stop**

**Total time**: ~3 minutes (5 loops × 30s + processing time)

## ⚠️ Important Warnings

- **Automatic Looping**: The workflow will continue running until ALL matching emails are deleted
- **Long Execution Time**: If you have thousands of emails, the workflow may run for 30+ minutes
- **Permanent Deletion**: Deleted emails go to Trash and are permanently deleted after 30 days
- **Test First**: Start with a small limit and specific keyword to test
- **Backup**: Consider backing up important emails before running
- **Cannot Stop Mid-Execution**: Once started, the loop continues until no results are found
- **Review Results**: Check the execution log to see which emails were deleted in each iteration

## Scheduling the Workflow

To run this workflow automatically:

1. Replace the "Manual Trigger" node with a "Schedule Trigger" or "Cron" node
2. Configure the schedule (e.g., daily, weekly)
3. Activate the workflow

## Troubleshooting

- **Authentication Error**: Reauthorize your Gmail OAuth2 credentials
- **No Emails Found**: Check your search query syntax
- **Rate Limiting**: Gmail API has rate limits; add delays if processing many emails
- **Permission Error**: Ensure your OAuth2 app has Gmail delete permissions

## Security Notes

- Never share your OAuth2 credentials
- Review the permissions granted to the OAuth2 app
- Consider using a dedicated Gmail account for testing
- Regularly audit your Google account's third-party access

## License

This workflow is provided as-is for use with n8n.
