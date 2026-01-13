# Sistema de Lojas - Supabase

Este projeto é uma **plataforma completa para lojas online**, permitindo:  
- Cadastro e login de usuários  
- Criação de lojas e produtos  
- Upload de imagens de produtos  
- Carrinho de compras e checkout  
- Visualização e gerenciamento de pedidos  

O backend é feito com **Supabase**, usando Auth, Storage e Postgres, e o frontend é em **HTML/JS puro**.

---

## 1️⃣ Pré-requisitos

- Conta no [Supabase](https://supabase.com/)  
- Projeto criado no Supabase  
- Node.js (opcional, se for hospedar localmente ou usar bundlers)  

---

## 2️⃣ Configuração do Banco de Dados

### 2.1 Criar tabelas

No **SQL Editor do Supabase**, rode o seguinte script:

```sql
-- Tabelas principais
create table stores (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users(id),
  nome_loja text not null,
  descricao text,
  created_at timestamp with time zone default now()
);

create table products (
  id uuid primary key default gen_random_uuid(),
  store_id uuid references stores(id) on delete cascade,
  nome text not null,
  descricao text,
  preco numeric not null,
  estoque integer default 0,
  imagem_url text,
  created_at timestamp with time zone default now()
);

create table orders (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users(id),
  store_id uuid references stores(id),
  status text default 'pendente',
  total numeric,
  created_at timestamp with time zone default now()
);

create table order_items (
  id uuid primary key default gen_random_uuid(),
  order_id uuid references orders(id) on delete cascade,
  product_id uuid references products(id),
  quantidade integer not null,
  preco_unitario numeric not null,
  subtotal numeric not null
);

-- Ativar Row-Level Security (RLS)
alter table stores enable row level security;
alter table products enable row level security;
alter table orders enable row level security;
alter table order_items enable row level security;

-- Policies básicas de segurança
create policy "Usuário só vê e gerencia suas lojas" on stores
for select, insert, update, delete using (auth.uid() = user_id);

create policy "Usuário só vê seus pedidos" on orders
for select using (auth.uid() = user_id);

create policy "Usuário só insere pedidos" on orders
for insert with check (auth.uid() = user_id);
