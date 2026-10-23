# Deploy the backend version

The browser never sees the Gemini API key. It calls a Supabase Edge Function, which holds the key and the prompts.

## 1. Get a free Gemini API key
1. Go to https://aistudio.google.com/app/apikey and sign in with a Google account.
2. Click **Create API key**, then pick a Google Cloud project or let AI Studio create one.
3. Copy the key and keep it private.
4. Restrict the key to the Gemini API only (AI Studio or Google Cloud Console), since Google has said unrestricted keys will be blocked.
5. Current details: https://ai.google.dev/gemini-api/docs/api-key

## 2. Create a Supabase project
Create a free project at https://supabase.com. From **Project Settings > API**, copy the **Project URL** and the **anon public** key.

## 3. Deploy the function
```bash
npm i -g supabase
supabase login
supabase link --project-ref YOUR-PROJECT-REF
supabase secrets set GEMINI_API_KEY=your-key
supabase functions deploy study-buddy
```

## 4. Point the app at it
In `index.html`, set `BACKEND_URL` (`https://YOUR-PROJECT-REF.supabase.co/functions/v1/study-buddy`) and `ANON_KEY`. The anon key is safe to ship in the page; the Gemini key never is.

## 5. Host the page
Upload `index.html` to GitHub Pages, Netlify or Cloudflare Pages.

## Before going public
- Set `Access-Control-Allow-Origin` in the function to your site's origin instead of `*`.
- Add rate limiting (for example, per-IP counts in a Supabase table) and keep an eye on your usage in AI Studio. The free tier has rate limits, so a bot could use up your daily quota.

## Troubleshooting
- **"The buddy is unavailable"**: open the Supabase dashboard, go to Edge Functions > study-buddy > Logs, and check the error.
- **500 "server not configured"**: the `GEMINI_API_KEY` secret is missing or misspelled.
- **502 "upstream 4xx"**: Gemini rejected the request. Check the key, its API restriction, or the model name (override with a `GEMINI_MODEL` secret).
- **401 from Supabase**: the `ANON_KEY` in `index.html` is wrong or from a different project.
- **429**: you hit the free-tier rate limit. Wait a minute and try again.
