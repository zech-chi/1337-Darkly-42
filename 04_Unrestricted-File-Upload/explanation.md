#Chapter 4 — Unrestricted File Upload
#unrestricted-upload #file-upload 

The settings page lets me change my avatar. It should only accept images, but the server checks nothing — it accepts ANY file type. I logged in as jdoe and uploaded an SVG file (not a normal image) to `POST /upload/avatar`. The server accepted it and redirected me to the settings page, where the flag appeared: `location: /profile/me/settings?upload_flag=FLAG{...}` 
`🚩🚩🚩 FLAG{unr3str1ct3d_upl0ad_g0_brrr}`. 

I used an SVG on purpose: it can hold a `<script>` inside it. I even added a smiley drawing so it looks like a normal avatar, but the script is still there. This is the bridge to the next attack — stored XSS. 

### Impact

Because the server accepts any file, an attacker can upload dangerous files (like an SVG with a script). The file is served with its real type (`image/svg+xml`), so when it is opened as a page, the script runs in the victim's browser. This leads to stored XSS and can steal sessions.

### Prevention
- Check the real file type on the server (magic bytes), not the extension or the browser's word.
- Only allow safe image types (png, jpg, gif) and block everything else.
- Store uploads outside the web root, or serve them as downloads (`Content-Disposition: attachment`) so they can't run.
- Give uploaded files random names.