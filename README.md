# Hozaho TV & Radio: website + backend

## What this is

- **Website** (`public/`): home page with video hero, bouncing logo, LIVE button, on-air card, news by category, videos, newsletter.
- **Backend** (`netlify/functions/api.mjs`): one file that runs the radio, news, newsletters and subscribers.
- **Admin page** (`/admin.html`): your team adds songs, schedules recorded news, writes stories and newsletters.

## How the radio works

- Songs in the **Playlist** play one after another all day, then start again.
- A **Recorded news** item plays at its time every day (Kigali time). The songs wait, then continue where they stopped.
- The website works out what is playing from the clock, so everyone hears the same thing at the same second.
- The admin page reads the **length** of each audio file by itself. You only choose the file.

## Step 1: Database on Aiven

1. In your Aiven MySQL service, open **Databases**, click **Create database** and name it `hozaho` (a separate database, so nothing mixes with Websound).
2. Copy the **Service URI** from the Overview page. Change `defaultdb` at the end to `hozaho`. Keep it private.

## Step 2: Audio storage on Backblaze B2

1. Create a free B2 account at backblaze.com.
2. **Create a bucket.** Name: `hozaho-audio` (or another name). Files: **Public**.
3. Open the bucket and note its **Endpoint**, for example `s3.us-west-004.backblazeb2.com`. The part after `s3.` and before `.backblazeb2.com` is your **region** (`us-west-004`).
4. **Application keys** > **Add a new application key**. Give it access to this bucket only. Copy the **keyID** and the **applicationKey** now (the key is shown only once).
5. **CORS (needed for uploads from the admin page).** Open the bucket > **Bucket Settings** > **CORS Rules**. Choose **Share everything in this bucket with every origin**. If it asks which API, choose **S3 Compatible** (or **Both**). Save.

If uploading from the admin page does not work, you can still paste a direct link to an audio file instead.

## Step 3: GitHub and Netlify

1. Create a new GitHub repository, for example `hozaho-radio`.
2. Upload the files. Keep the folders: `public/` and `netlify/functions/`, and `netlify.toml` and `package.json` at the top.
   Tip: GitHub's upload box refuses folders. Create `public/keep.txt` and `netlify/functions/keep.txt` first (Add file > Create new file), then upload the files into those folders.
3. On Netlify: **Add new project** > **Import from Git** > choose the repository. Leave the build settings as they are.

## Step 4: Environment variables (Netlify > Site configuration > Environment variables)

| Name | Value |
|---|---|
| `DATABASE_URL` | the Aiven URI with `/hozaho` at the end |
| `JWT_SECRET` | any long random text, 30+ characters |
| `ADMIN_PASSWORD_HASH` | a bcrypt hash of the admin password (see below) |
| `B2_KEY_ID` | Backblaze keyID |
| `B2_APP_KEY` | Backblaze applicationKey |
| `B2_BUCKET` | the bucket name |
| `B2_REGION` | for example `us-west-004` |

**Admin password hash:** use the hash from your Websound `admin-login.php` if you want the same password, or make a new one at any bcrypt generator (cost 10) and paste the result that starts with `$2a$` or `$2y$`.

After adding the variables, go to **Deploys** > **Trigger deploy**.

## Step 5: First use

1. Open `/admin-login.html` and log in.
2. Open `/setup.html` and press **Create the tables**.
3. Open `/admin.html`. Add a few songs, then schedule one recorded news item.
4. Open the home page and press **Listen live**.

## Things to know

- **Audio size:** up to 30 MB per file. Use mp3 at 64 to 96 kbps to save data. Speech sounds fine at that size.
- **Backblaze free limits:** 10 GB of storage is free, and free downloads are limited each month. If the radio grows, Backblaze charges a very small price per extra GB, or the audio can move to another storage later.
- The newsletter box saves emails. You see them in the admin page (Subscribers). Sending emails automatically is not built yet.
- One shared admin password for now. Separate logins for each team member can be added later.
- Change the placeholders: the WhatsApp link, your phone and email in the footer, and add your video as `public/assets/hero.mp4` (10 to 15 seconds, no sound, under 3 MB).
