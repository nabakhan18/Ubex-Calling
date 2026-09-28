# UBEX Calling Services — client handover

This package contains the complete website source in `source/` and the currently published website text in `data/ubex/site-content.json`. The code includes the animated public website, the admin dashboard, email and password login, and dashboard controls to change both login email and password. No old account credentials or Vercel tokens are included.

## Before starting

- The client needs their own GitHub account and Vercel account, and Node.js installed on the computer used for setup.
- This is a commercial client website. Vercel's Hobby plan is restricted to personal, noncommercial use; select an eligible Pro or Enterprise plan for production commercial use: https://vercel.com/docs/limits/fair-use-guidelines
- Choose a fresh project name such as `ubex-calling-services`. The automatically assigned `.vercel.app` URL will depend on availability. The old project's URL will not automatically move to the new project.
- Keep the old website online until the new public URL, admin login and editing have all been checked. The current owner can unpublish the old project afterward.

## 1. Import the source

1. Extract this ZIP. Create a **private GitHub repository owned by the client**. Upload the *contents* of `source/` to the repository root (`app/`, `lib/`, `public/`, `package.json`, `pnpm-lock.yaml`, etc.). Keep the directory structure. The `data/` folder and this guide are backup/setup materials, not website source.
2. In the client's Vercel account, select **Add New → Project → Import Git Repository**, select that repository and choose a fresh project name. Keep the detected **Next.js** framework and root directory `./`.
3. Deploy once. The website can initially show its built-in default content. Admin login needs the setup below.

## 2. Create and connect private Blob storage

1. In the new Vercel project open **Storage → Create Storage (or Create Database) → Blob**.
2. Create a **Private** store owned by the client. Connect it to this project for **Production**; connect Preview too if previews need admin editing.
3. In **Project → Settings → Environment Variables**, confirm the new connection supplied `BLOB_READ_WRITE_TOKEN` for Production. This source uses that variable for admin and content reads/writes. If it is absent, use the Blob store's project connection/setup flow to supply a read-write token for the client's new store. Do **not** reuse a token from the old owner's Vercel account. `BLOB_STORE_ID` and `BLOB_WEBHOOK_PUBLIC_KEY`, if present, are managed by Vercel and do not need to be copied from the old project.

## 3. Create the client's own admin login

On the client's computer, in the extracted `source/` folder, run:

```bash
node scripts/generate-admin-secrets.mjs
```

Enter a new password of at least 12 characters. The script prints two values, without storing the password in a file. In **Project → Settings → Environment Variables**, add these to **Production**:

| Name | Value |
| --- | --- |
| `ADMIN_EMAIL` | The client's chosen admin email address. This is a login identifier; it does not send emails. |
| `ADMIN_PASSWORD_HASH` | The entire `salt:hash` string printed by the script. Do not enter the password itself here. |
| `ADMIN_SESSION_SECRET` | The random string printed by the script. |
| `BLOB_READ_WRITE_TOKEN` | Automatically supplied by the newly connected private Blob store, or its **new** read-write token if manual setup is required. |

Never publish these values in GitHub, screenshots, chat groups, or the ZIP. The client keeps their plain password in their own password manager. After setting the variables, **redeploy Production** from Vercel → Deployments (the previous deployment will not receive newly configured values).

## 4. Restore the currently published content

The file `data/ubex/site-content.json` contains the saved published and draft website text. Upload it to the **client's new private Blob store** at the exact pathname **`ubex/site-content.json`**, without a random filename suffix. A reliable method using Vercel CLI:

```bash
npm install -g vercel
vercel login
cd source
vercel link
vercel blob put ../data/ubex/site-content.json --pathname ubex/site-content.json --access private --content-type application/json
vercel blob list --prefix ubex/
```

During `vercel link`, choose the **client's account and new project**, not the old owner's. If the CLI prompts for a store, select the newly created private Blob store. The final listing must contain `ubex/site-content.json`. The dashboard may also offer Upload in the `ubex` folder; check the exact resulting pathname if using it. See https://vercel.com/docs/cli/blob

## 5. Verify and hand over

1. Open the new public URL; check the hero, services, industry sections, animation and contact email.
2. Open `https://NEW-URL/admin/login` and sign in with `ADMIN_EMAIL` and the password chosen in step 3.
3. Open the dashboard **Security** section. The client can change their login email or password there using their current password; the changes are stored in the new private Blob store and end previous sessions.
4. Edit a harmless draft field, save it, reload the dashboard to verify persistence, then publish only if that edit should go live. Confirm the public website updates.
5. After these checks pass, the original owner can unpublish the old Vercel project. Unpublishing/deleting it first would cause downtime. If using a custom domain later, point that domain to the client's new Vercel project separately.

### Troubleshooting

- Login fails: check the exact `ADMIN_EMAIL`, full `ADMIN_PASSWORD_HASH`, and `ADMIN_SESSION_SECRET` values in the client's Production environment; redeploy after changes. Ensure private Blob is connected. The login form uses the plaintext password chosen at setup; `ADMIN_PASSWORD_HASH` is never typed into the login form.
- Dashboard saves fail: confirm the project's Production `BLOB_READ_WRITE_TOKEN` belongs to the **new** private store. Make sure there is only one store connected for this app.
- Old content does not appear: list the new store and confirm the exact path `ubex/site-content.json` and that it is Private. Built-in defaults may look similar, so verify an edit persists.
- Changing `ADMIN_EMAIL` or `ADMIN_PASSWORD_HASH` in Vercel later will **not** override a prior email/password change in the dashboard: `ubex/admin-auth.json` in the new private store takes precedence. Use dashboard Security for routine changes. Keep that private file with the client's store.

No old Vercel credentials or Blob token are needed for the fresh deployment. The client owns the new GitHub repo, Vercel project, private Blob store and admin login.
