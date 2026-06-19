Had done this sql into supabase: 

-- =========================================
-- EXTENSIONS
-- =========================================

create extension if not exists "pgcrypto";

-- =========================================
-- ENUMS
-- =========================================

create type campaign_status as enum (
    'draft',
    'active',
    'paused',
    'completed',
    'archived'
);

create type import_status as enum (
    'pending',
    'processing',
    'completed',
    'failed'
);

create type event_type as enum (
    'reply_received',
    'email_bounced',
    'unsubscribed',
    'campaign_completed'
);

-- =========================================
-- USERS
-- =========================================

create table users (
    id uuid primary key default gen_random_uuid(),

    full_name text not null,
    email text not null unique,

    role text default 'consultant',

    is_active boolean default true,

    created_at timestamptz default now(),
    updated_at timestamptz default now()
);

create index idx_users_email
on users(email);

-- =========================================
-- CAMPAIGNS
-- =========================================

create table campaigns (
    id uuid primary key default gen_random_uuid(),

    name text not null,

    user_id uuid not null
        references users(id)
        on delete cascade,

    smartlead_campaign_id bigint,

    status campaign_status
        default 'draft',

    created_at timestamptz default now(),
    updated_at timestamptz default now()
);

create index idx_campaigns_user
on campaigns(user_id);

create index idx_campaigns_status
on campaigns(status);

create index idx_campaigns_smartlead
on campaigns(smartlead_campaign_id);

-- =========================================
-- CAMPAIGN SEQUENCES
-- =========================================

create table campaign_sequences (
    id uuid primary key default gen_random_uuid(),

    campaign_id uuid not null
        references campaigns(id)
        on delete cascade,

    sequence_no integer not null,

    subject text not null,

    body text not null,

    delay_days integer default 0,

    created_at timestamptz default now(),

    unique(campaign_id, sequence_no)
);

create index idx_campaign_sequences_campaign
on campaign_sequences(campaign_id);

-- =========================================
-- CAMPAIGN LEADS
-- =========================================

create table campaign_leads (
    id uuid primary key default gen_random_uuid(),

    campaign_id uuid not null
        references campaigns(id)
        on delete cascade,

    email text not null,

    first_name text,

    company text,

    status text,

    created_at timestamptz default now()
);

create index idx_campaign_leads_campaign
on campaign_leads(campaign_id);

create index idx_campaign_leads_email
on campaign_leads(email);

create index idx_campaign_leads_status
on campaign_leads(status);

-- =========================================
-- MAILBOXES
-- =========================================

create table mailboxes (
    id uuid primary key default gen_random_uuid(),

    email text not null unique,

    display_name text,

    smartlead_account_id bigint,

    daily_limit integer default 50,

    is_active boolean default true,

    created_at timestamptz default now()
);

create index idx_mailboxes_email
on mailboxes(email);

create index idx_mailboxes_smartlead
on mailboxes(smartlead_account_id);

-- =========================================
-- CAMPAIGN MAILBOXES
-- =========================================

create table campaign_mailboxes (
    id uuid primary key default gen_random_uuid(),

    campaign_id uuid not null
        references campaigns(id)
        on delete cascade,

    mailbox_id uuid not null
        references mailboxes(id)
        on delete cascade,

    created_at timestamptz default now(),

    unique(campaign_id, mailbox_id)
);

create index idx_campaign_mailboxes_campaign
on campaign_mailboxes(campaign_id);

create index idx_campaign_mailboxes_mailbox
on campaign_mailboxes(mailbox_id);

-- =========================================
-- TEMPLATES
-- =========================================

create table templates (
    id uuid primary key default gen_random_uuid(),

    name text not null,

    industry text,

    subject text not null,

    body text not null,

    created_by uuid
        references users(id),

    created_at timestamptz default now()
);

create index idx_templates_industry
on templates(industry);

-- =========================================
-- REPLY LOGS
-- =========================================

create table reply_logs (
    id uuid primary key default gen_random_uuid(),

    campaign_id uuid not null
        references campaigns(id)
        on delete cascade,

    lead_email text not null,

    received_at timestamptz not null,

    forwarded_at timestamptz,

    created_at timestamptz default now()
);

create index idx_reply_logs_campaign
on reply_logs(campaign_id);

create index idx_reply_logs_email
on reply_logs(lead_email);

-- =========================================
-- CAMPAIGN STATS
-- =========================================

create table campaign_stats (
    campaign_id uuid primary key
        references campaigns(id)
        on delete cascade,

    sent_count integer default 0,

    reply_count integer default 0,

    bounce_count integer default 0,

    open_count integer default 0,

    click_count integer default 0,

    updated_at timestamptz default now()
);

-- =========================================
-- WEBHOOK EVENTS
-- =========================================

create table webhook_events (
    id uuid primary key default gen_random_uuid(),

    campaign_id uuid
        references campaigns(id)
        on delete set null,

    lead_email text,

    event_type event_type not null,

    payload jsonb,

    created_at timestamptz default now()
);

create index idx_webhook_events_campaign
on webhook_events(campaign_id);

create index idx_webhook_events_type
on webhook_events(event_type);

create index idx_webhook_events_created
on webhook_events(created_at desc);

-- =========================================
-- LEAD IMPORTS
-- =========================================

create table lead_imports (
    id uuid primary key default gen_random_uuid(),

    campaign_id uuid not null
        references campaigns(id)
        on delete cascade,

    file_name text,

    total_rows integer default 0,

    success_rows integer default 0,

    failed_rows integer default 0,

    status import_status
        default 'pending',

    created_at timestamptz default now()
);

create index idx_lead_imports_campaign
on lead_imports(campaign_id);

create index idx_lead_imports_status
on lead_imports(status);


=======================================

Had done this also:

alter table campaign_leads
add column phone text;

alter table campaign_leads
add column linkedin_url text;

alter table campaign_leads
add column company_url text;

alter table campaign_leads
add column custom_fields jsonb;

alter table campaigns
add column total_leads integer default 0;

alter table campaigns
add column launched_at timestamptz;

alter table campaigns
add column timezone text default 'Australia/Sydney';

