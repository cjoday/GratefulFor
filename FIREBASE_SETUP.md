# Firebase Setup Guide for User Stories Feature

This guide will help you set up Firebase Authentication and Firestore to enable the user stories feature on your Grateful Dead website.

## Prerequisites

- A Google account
- Basic familiarity with Firebase Console

## Step 1: Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click "Add project" or "Create a project"
3. Enter a project name (e.g., "grateful-for-dead")
4. Accept the terms and click "Continue"
5. Disable Google Analytics (optional) or configure it
6. Click "Create project"
7. Wait for the project to be created, then click "Continue"

## Step 2: Register Your Web App

1. In the Firebase Console, click the web icon (`</>`) to add a web app
2. Enter an app nickname (e.g., "Grateful Dead Web")
3. Check "Also set up Firebase Hosting" if you want to use Firebase Hosting (optional)
4. Click "Register app"
5. Copy the Firebase configuration object - you'll need this in Step 5

The configuration will look like this:

```javascript
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef"
};
```

## Step 3: Enable Google Authentication

1. In Firebase Console, go to "Build" → "Authentication"
2. Click "Get started" (if this is your first time)
3. Go to the "Sign-in method" tab
4. Click on "Google" in the providers list
5. Toggle "Enable"
6. Select a support email address from the dropdown
7. Click "Save"

## Step 4: Set Up Firestore Database

1. In Firebase Console, go to "Build" → "Firestore Database"
2. Click "Create database"
3. Choose "Start in **production mode**" (we'll add custom rules next)
4. Select a Cloud Firestore location (choose one close to your users)
5. Click "Enable"

## Step 5: Configure Firestore Security Rules

1. In Firestore Database, go to the "Rules" tab
2. Replace the default rules with the content from `firestore.rules` file in this repository
3. Or manually paste this:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /stories/{storyId} {
      allow read: if true;
      allow create: if request.auth != null
                    && request.resource.data.userId == request.auth.uid
                    && request.resource.data.userName is string
                    && request.resource.data.title is string
                    && request.resource.data.title.size() <= 100
                    && request.resource.data.content is string
                    && request.resource.data.content.size() <= 1000
                    && request.resource.data.dateKey is string
                    && request.resource.data.dateKey.matches('^[0-9]{2}-[0-9]{2}$');
      allow update, delete: if request.auth != null
                            && resource.data.userId == request.auth.uid;
    }
  }
}
```

4. Click "Publish"

## Step 6: Create Firestore Indexes

The user stories feature queries Firestore with a compound query. You need to create an index:

1. In Firestore Database, go to the "Indexes" tab
2. Click "Add index"
3. Configure the index:
   - **Collection ID**: `stories`
   - **Fields**:
     - Field: `dateKey`, Order: `Ascending`
     - Field: `createdAt`, Order: `Descending`
4. Click "Create"
5. Wait for the index to build (can take a few minutes)

**Alternative**: Wait for the first query error, and Firebase will provide a link to auto-create the index.

## Step 7: Update Your Website Configuration

1. Open `index.html` in your code editor
2. Find the Firebase configuration section (around line 697-705):

```javascript
const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT_ID.appspot.com",
    messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
    appId: "YOUR_APP_ID"
};
```

3. Replace the placeholder values with your actual Firebase configuration from Step 2
4. Save the file

## Step 8: Configure Authorized Domains (for Production)

If you're deploying to a custom domain:

1. In Firebase Console, go to "Authentication" → "Settings" → "Authorized domains"
2. Click "Add domain"
3. Enter your domain (e.g., `gratefulfordead.com`)
4. Click "Add"

Note: `localhost` is already authorized by default for development.

## Step 9: Deploy Your Website

You can deploy using:

### Option A: Firebase Hosting (Recommended)

```bash
# Install Firebase CLI
npm install -g firebase-tools

# Login to Firebase
firebase login

# Initialize Firebase in your project
firebase init

# Select:
# - Hosting
# - Use existing project (select your project)
# - Public directory: . (current directory)
# - Single-page app: No
# - Overwrite index.html: No

# Deploy
firebase deploy
```

### Option B: Any Static Hosting

Simply upload all files (`index.html`, `dead-history.json`) to your hosting provider:
- GitHub Pages
- Netlify
- Vercel
- AWS S3
- Any web server

## Testing Locally

1. Open `index.html` in a web browser
2. You should see the "Share Your Stories" section
3. Click "Sign In to Share"
4. Sign in with Google
5. Click "Add Your Story"
6. Fill out the form and submit
7. Your story should appear in the stories section

## Troubleshooting

### "Firebase not configured" Error

- Make sure you replaced the placeholder Firebase configuration with your actual config
- Check the browser console for detailed error messages

### Authentication Not Working

- Verify Google sign-in is enabled in Firebase Console
- Check that your domain is in the authorized domains list
- Make sure you're using `https://` (not `http://`) in production

### Stories Not Loading

- Check that the Firestore index is created and fully built
- Verify the security rules are published correctly
- Look at the browser console for error messages

### Permission Denied Errors

- Ensure Firestore security rules are correctly configured
- Verify the user is authenticated before submitting stories
- Check that the story data matches the validation rules

## Data Structure

Stories are stored in Firestore with this structure:

```javascript
{
  dateKey: "10-29",           // MM-DD format matching dead-history.json
  title: "Story title",       // Max 100 characters
  content: "Story content",   // Max 1000 characters
  userId: "user-uid",         // Firebase Auth user ID
  userName: "User Name",      // Display name from Google account
  userPhoto: "photo-url",     // Profile photo URL from Google
  createdAt: Timestamp        // Server timestamp
}
```

## Security Features

- Only authenticated users can create stories
- Users can only edit/delete their own stories
- Story titles are limited to 100 characters
- Story content is limited to 1000 characters
- All users can read stories (public)
- XSS protection through HTML escaping

## Cost Considerations

Firebase free tier (Spark plan) includes:

- **Authentication**: 50,000 monthly active users
- **Firestore**:
  - 1 GB storage
  - 50K reads/day
  - 20K writes/day
  - 20K deletes/day

This is more than sufficient for a community website. Monitor usage in Firebase Console.

## Support

For Firebase-specific issues:
- [Firebase Documentation](https://firebase.google.com/docs)
- [Firebase Support](https://firebase.google.com/support)

For feature-specific issues:
- Check the browser console for errors
- Review this setup guide
- Check Firestore rules and indexes
