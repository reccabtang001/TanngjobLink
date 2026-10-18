TANGJOBLINK MEMBER GROUP + ADMIN DASHBOARD — SETUP GUIDE

WHAT THIS PACKAGE INCLUDES
- Public preview of five learning groups and topic titles only.
- Member registration, sign-in, password reset and sign-out through Supabase Auth.
- Selected-group M-Pesa payment-reference submission for manual verification.
- Private member dashboard on this website after payment verification; no separate group invite is needed.
- Five selectable member groups with separate fees: Beginner KSh 300, Intermediate KSh 500, Advanced KSh 700, Professional KSh 1,000 and Elite KSh 1,300. Each group has a Join Group button.
- Member-only group discussion, protected meeting schedule and US college/FAFSA/scholarship/loan/tax/FEMA resources.
- Admin dashboard to review registrations, approve a member only after independently verifying payment, reject requests, revoke/restore membership access, and schedule/delete meetings.
- Supabase Row Level Security protects group messages, meetings and member lessons.

IMPORTANT
This is a setup package, not an already connected or deployed system. You must configure Supabase, run the SQL, add the public project URL/key to index.html and deploy the site. M-Pesa verification is manual; there is no automatic Daraja callback in this package. A member selects one group and sees its fee. The admin must independently verify the matching payment, then use Approve member to activate access. The member dashboard shows the selected group. Public visitors see category/topic titles only; detailed lessons, discussions and meetings require active membership.

STEP 1 — CREATE SUPABASE PROJECT
1. Visit https://supabase.com/ and create a separate project named TangjobLink.
2. Open SQL Editor > New query. Paste all of supabase_setup.sql and click Run.
3. Open Project Settings / API or Connect. Copy the Project URL and anon/public/publishable key. Do NOT use a service_role or secret key in browser code.

STEP 2 — CONNECT THE WEBSITE
1. Open index.html in Notepad or VS Code.
2. Near the bottom, replace PASTE_YOUR_SUPABASE_PROJECT_URL_HERE and PASTE_YOUR_SUPABASE_ANON_KEY_HERE with your Project URL and anon/publishable key. Keep quotation marks.
3. Save the file.
4. In Supabase Authentication > URL Configuration, add your final Netlify URL to Site URL and Redirect URLs.
5. Keep email confirmation enabled for public accounts where possible.

STEP 3 — MAKE YOURSELF THE ADMIN
1. Create your owner account through the website and confirm your email.
2. In Supabase Dashboard > Authentication > Users, copy your account's User UID.
3. In SQL Editor, run this command after replacing OWNER_AUTH_USER_UID with the copied UID:
   insert into public.member_roles(user_id, role) values ('OWNER_AUTH_USER_UID', 'admin') on conflict (user_id) do update set role='admin';
4. Sign out and sign back in. The Administrator control panel appears for an active admin account. If the owner account is pending, verify its payment and activate it using Step 4.
5. To activate the owner's dashboard without processing a learner payment, the owner may run this SQL for their own account after assigning the admin role: update public.member_profiles set membership_status='active' where user_id='OWNER_AUTH_USER_UID'; Replace the placeholder with the same verified owner UID.
6. Never hardcode administrator email/password in website files, share admin credentials, or allow users to insert themselves into member_roles. Create/secure the admin login through Supabase Auth and assign its UID the admin role privately in SQL.

STEP 4 — PAYMENT AND MEMBER ACCESS
1. Learner selects a group, registers, pays that group’s fee using Paybill 247247 and account 0112825493, then submits the M-Pesa transaction code.
2. Verify the transaction independently in official M-Pesa records and confirm the amount matches the selected group. A typed code alone is not proof of payment.
3. To inspect registrations in SQL Editor:
   select user_id, email, full_name, phone, selected_group, payment_reference, membership_status, created_at from public.member_profiles order by created_at desc;
4. Preferred: sign in as admin, locate the member, confirm their selected group, fee and transaction reference, then click Approve member. The SQL alternative for a payment already verified is:
   update public.member_profiles set membership_status='active' where email='member@example.com';
5. The member signs in and accesses the dashboard, group discussions, meetings and member-only resources directly on TangjobLink.
6. To remove group access:
   update public.member_profiles set membership_status='removed' where email='member@example.com';
   The admin panel also has Approve member / Reject request controls for pending registrations, plus Remove member / Restore access controls. Removal revokes group access but does not delete the Auth account.

STEP 5 — SCHEDULE MEETINGS
- Sign in with the configured admin account and open the Administrator control panel.
- Enter title, group, date/time, optional HTTPS meeting link and notes, then click Publish meeting.
- Active members see meetings in their dashboard. Admin can delete meetings.

STEP 6 — DEPLOY TO NETLIFY
1. Upload the configured TangjobLink folder to the production deploy area of your existing Netlify site, or deploy through your Git repository.
2. Add the final Netlify URL to Supabase Authentication > URL Configuration.
3. Test signup, email confirmation, login, password reset, pending access, active access, group messages, admin controls and meeting scheduling.

SECURITY NOTES
- Never put a Supabase service_role key, M-Pesa secret, password, M-Pesa PIN, SSN, bank credential or tax document in website code or group messages.
- The anon/publishable key is safe for browser use only with the supplied RLS rules enabled.
- Admin role assignment is done in SQL by the owner; users cannot assign themselves admin or change their own membership status. The admin dashboard uses a protected database function for member status changes.
- Proxy lessons should focus on lawful privacy, security and platform terms. Dark-web lessons are for lawful cybersecurity awareness and defense. No unauthorized access, fraud, identity deception or bypassing job-platform eligibility.
- US college, FAFSA, loan, tax and FEMA eligibility varies. Use official links and never promise aid, loans, refunds or FEMA grants.
- M-Pesa verification is manual; automatic STK Push/callback requires a separate secure server integration.

SUPPORT
TangjobLink: 0797858453 | WhatsApp: https://wa.me/254797858453 | rtltd.jobs@gmail.com


REGISTRATION FLOW UPDATE
- Every Join Group button now scrolls to the registration form and preselects that group, even before Supabase is configured.
- Registration asks for full name, email, ID number, username, and password. The Register button asks the user to enter the group fee and requires it to match the selected price.
- Supabase Auth sends the confirmation link only when email confirmation is enabled in Supabase Authentication settings. Configure the site's URL and redirect URLs there.
- The entered amount is a declaration, NOT proof of payment. The member must pay via Paybill 247247, account 0112825493, submit the M-Pesa transaction code, and wait for admin verification and approval.
- The updated SQL includes username, ID number and declared amount columns. If you already ran an older version of the SQL in Supabase, run these migration statements in SQL Editor before deploying:
  alter table public.member_profiles add column if not exists username text;
  alter table public.member_profiles add column if not exists id_number text;
  alter table public.member_profiles add column if not exists amount_entered integer;
  create unique index if not exists member_profiles_username_unique on public.member_profiles (lower(username)) where username is not null;
- ID numbers are sensitive personal information. Restrict database access, do not display ID numbers in public pages, and collect/store them only if you genuinely need them.
