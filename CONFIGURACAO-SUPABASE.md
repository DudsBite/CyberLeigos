# Configuração de contas reais — CyberLeigos

O site pode ser usado como visitante sem criar conta. O progresso de visitante fica somente na sessão atual do navegador (`sessionStorage`) e não é sincronizado nem guardado permanentemente. Para salvar XP entre visitas e dispositivos, configure o Supabase.

## 1. Criar o projeto
1. Acesse https://supabase.com/ e crie um projeto.
2. Em **Project Settings → API**, copie a **Project URL** e a chave **anon/public**.
3. Abra `assets/supabase-config.js` e preencha `url` e `anonKey`.
4. Nunca use a chave `service_role` no site.

## 2. Criar a tabela de progresso
No **SQL Editor** do Supabase, execute:

```sql
create table if not exists public.profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  xp integer not null default 0 check (xp >= 0),
  completed jsonb not null default '[]'::jsonb,
  daily jsonb not null default '{}'::jsonb,
  streak integer not null default 0,
  last_day date,
  updated_at timestamptz not null default now()
);

alter table public.profiles enable row level security;

drop policy if exists "Users read own profile" on public.profiles;
drop policy if exists "Users insert own profile" on public.profiles;
drop policy if exists "Users update own profile" on public.profiles;

create policy "Users read own profile"
on public.profiles for select using (auth.uid() = id);

create policy "Users insert own profile"
on public.profiles for insert with check (auth.uid() = id);

create policy "Users update own profile"
on public.profiles for update using (auth.uid() = id)
with check (auth.uid() = id);
```

## 3. Ativar confirmação de e-mail
1. No painel Supabase, abra **Authentication → Providers → Email**.
2. Ative **Confirm email**.
3. Em **Authentication → URL Configuration**, defina a **Site URL** como `https://dudsbite.github.io/CyberLeigos/`.
4. Adicione também `https://dudsbite.github.io/CyberLeigos/**` em **Redirect URLs**.
5. Se quiser e-mails personalizados, configure SMTP próprio em **Authentication → SMTP Settings**. Sem SMTP próprio, o serviço padrão pode ter limites de envio.

## 4. Testar
1. Publique as alterações no GitHub.
2. Abra `conta.html`, crie uma conta com um e-mail seu e confirme pelo link recebido.
3. Volte ao site e faça login.
4. Ganhe XP em uma atividade; depois abra o site em outro dispositivo e entre na mesma conta para verificar a sincronização.

## Como o progresso funciona
- **Visitante:** XP e atividades ficam em armazenamento temporário da sessão. Fechar a sessão encerra esse progresso.
- **Conta autenticada:** o progresso é carregado e salvo na tabela `profiles`, protegido por Row Level Security (RLS).
- O banco só aceita que cada usuário leia ou altere a própria linha.

**Segurança:** a chave `anon/public` pode estar no JavaScript do site, mas a proteção dos dados depende das políticas RLS acima. Nunca publique `service_role`, senhas ou tokens privados.
