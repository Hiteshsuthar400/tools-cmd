# Supabase Cloud Sync Setup — OSINT Tree v7.0 Pro

This guide connects the live GitHub Pages deployment (`https://hiteshsuthar400.github.io/tools-cmd/`) to a private Supabase PostgreSQL backend with authenticated Row Level Security (RLS).

## 1. Create a Supabase Project
1. Sign in at [https://supabase.com/dashboard](https://supabase.com/dashboard) and click **New project**.
2. Choose an organization, set a **Project Name** (e.g., `tools-cmd-osint`), generate a strong **Database Password**, select your closest region, and click **Create new project**.

## 2. Create the Database Table & RLS Policy
1. In the left sidebar, open **SQL Editor** and click **New query**.
2. Paste and run the SQL script to create `public.engagements` and enable RLS:
   ```sql
   create table if not exists public.engagements (
     user_id uuid references auth.users(id) on delete cascade not null default auth.uid(),
     domain text not null,
     config jsonb not null default '{}'::jsonb,
     checklist_state jsonb not null default '{}'::jsonb,
     flagged_state jsonb not null default '{}'::jsonb,
     notes_state jsonb not null default '{}'::jsonb,
     scratchpad text not null default '',
     updated_at timestamp with time zone default timezone('utc'::text, now()) not null,
     primary key (user_id, domain)
   );

   alter table public.engagements enable row level security;

   revoke all on table public.engagements from anon;
   grant select, insert, update, delete on table public.engagements to authenticated;

   create policy "Authenticated users manage own engagements"
   on public.engagements
   for all
   to authenticated
   using ((select auth.uid()) = user_id)
   with check ((select auth.uid()) = user_id);
