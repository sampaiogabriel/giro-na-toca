# 🗄️ Documentação Técnica: Modelagem de Dados e Entidades (ER)

Este documento detalha a estrutura de dados relacional que sustenta o sistema de Roleta Gamificada. O banco de dados foi projetado para operar no **Neon (PostgreSQL)**, utilizando o **Prisma ORM**.

A modelagem foca em alta performance para consultas via _polling_ e na integridade do estado do prêmio (desde a intenção de giro até o resgate físico).

## 1. Diagrama de Entidades Principal

O sistema gravita em torno de 4 entidades fundamentais:

1. **Customer (Cliente)**
2. **Campaign (Campanha)**
3. **Prize (Prêmio)**
4. **Spin (Giro/Transação)**

---

## 2. Dicionário de Dados

### 2.1. Tabela `Customer` (Cliente)

Armazena os dados de identificação do usuário capturados via celular. O CPF atua como chave primária de negócio (Unique) para centralizar a carteira de prêmios e evitar duplicidade estrutural.

| Campo      | Tipo   | Descrição                            | Regra               |
| :--------- | :----- | :----------------------------------- | :------------------ |
| `id`       | UUID   | Identificador único do registro.     | Primary Key         |
| `name`     | String | Nome completo ou apelido do cliente. | Obrigatório         |
| `cpf`      | String | CPF para validação no Totem.         | Unique, Obrigatório |
| `whatsapp` | String | Contato para CRM futuro do bar.      | Obrigatório         |

### 2.2. Tabela `Campaign` (Campanha)

Gerencia o contexto da roleta. Permite que o bar crie roletas temáticas com prazos de validade e regras distintas sem perder o histórico anterior.

| Campo       | Tipo     | Descrição                                        | Regra           |
| :---------- | :------- | :----------------------------------------------- | :-------------- |
| `id`        | UUID     | Identificador único da campanha.                 | Primary Key     |
| `name`      | String   | Nome interno (Ex: "St. Patrick's Day").          | Obrigatório     |
| `isActive`  | Boolean  | Define se esta é a campanha rodando no Totem.    | Default: `true` |
| `startDate` | DateTime | Início da vigência da campanha.                  | Obrigatório     |
| `endDate`   | DateTime | Fim da vigência (opcional para campanhas fixas). | Opcional        |

### 2.3. Tabela `Prize` (Prêmio)

Catálogo de itens vinculados a uma campanha. Contém as regras matemáticas para o sorteio e configurações visuais para o frontend.

| Campo        | Tipo   | Descrição                                        | Regra                      |
| :----------- | :----- | :----------------------------------------------- | :------------------------- |
| `id`         | UUID   | Identificador único do prêmio.                   | Primary Key                |
| `campaignId` | UUID   | Chave estrangeira ligando à Campanha.            | Foreign Key                |
| `name`       | String | Nome exibido (Ex: "Shot de Catuaba").            | Obrigatório                |
| `weight`     | Int    | Peso numérico para o algoritmo de probabilidade. | Obrigatório                |
| `color`      | String | Hexadecimal para a fatia na UI da roleta.        | Obrigatório                |
| `stock`      | Int    | Quantidade limite de prêmios por campanha.       | Opcional (Null = Infinito) |

### 2.4. Tabela `Spin` (Transação / Carteira)

É a tabela mais crítica (Máquina de Estados). Ela registra toda a jornada do cliente, desde o cadastro na mesa até o momento em que ele bebe o prêmio no balcão.

| Campo        | Tipo | Descrição                                    | Regra                          |
| :----------- | :--- | :------------------------------------------- | :----------------------------- |
| `id`         | UUID | Identificador da transação.                  | Primary Key                    |
| `customerId` | UUID | Cliente atrelado a este giro.                | Foreign Key                    |
| `campaignId` | UUID | Campanha na qual o giro ocorreu.             | Foreign Key                    |
| `prizeId`    | UUID | Prêmio sorteado (preenchido após liberação). | Foreign Key, Nullable          |
| `status`     | Enum | Estado atual do giro (ver seção 3).          | Default: `AWAITING_VALIDATION` |

## 2.5. Tabela `Admin` (Staff / Usuários do Sistema)

Gerencia a equipe do bar que tem acesso ao Painel de Controle. A autenticação real (login/senha) é delegada ao **Clerk**, e esta tabela atua como um espelho local para controle de autorização (RBAC - Role Based Access Control) e auditoria de ações.

| Campo       | Tipo     | Descrição                            | Regra               |
| :---------- | :------- | :----------------------------------- | :------------------ |
| `id`        | String   | O exato `user_id` gerado pelo Clerk. | Primary Key         |
| `name`      | String   | Nome do funcionário.                 | Obrigatório         |
| `email`     | String   | E-mail corporativo ou de acesso.     | Unique, Obrigatório |
| `role`      | Enum     | Nível de permissão (ver seção 3.2).  | Default: `ADMIN`    |
| `createdAt` | DateTime | Data de criação do usuário.          | Default: `now()`    |

---

## 3. Máquinas de Estado e Permissões (Enums)

### 3.1. `SpinStatus` (Estado do Giro)

_(Mantém-se o mesmo de antes: AWAITING_VALIDATION, READY_TO_SPIN, PENDING_RESCUE, CLAIMED)_

### 3.2. `AdminRole` (Nível de Acesso)

A coluna `role` da tabela `Admin` dita o que o usuário logado pode ver e fazer no painel:

1. **`SUPER_ADMIN`**: O dono do bar ou o gerente. Tem acesso total. Pode criar novas Campanhas, cadastrar Prêmios, alterar pesos, ver relatórios financeiros e gerenciar a equipe.
2. **`ADMIN`**: O operador de caixa ou bartender. Tem acesso restrito apenas à tela de Operação: ver a fila de `AWAITING_VALIDATION`, clicar em "Liberar Roleta" e dar baixa em prêmios (`CLAIMED`).

---

## 4. Atualização Crítica de Relacionamentos (Auditoria)

Com a introdução da tabela `Admin`, atualizamos a tabela `Spin` para registrar a **Auditoria das Ações**. Isso evita fraudes internas (ex: um bartender liberando giros para amigos sem eles terem avaliado no Google).

**Novos campos na tabela `Spin` (Transação):**

- `approvedById` (String, Nullable): FK apontando para `Admin.id`. Registra qual bartender clicou em "Liberar Roleta" (Fase 3).
- `claimedById` (String, Nullable): FK apontando para `Admin.id`. Registra qual bartender entregou o prêmio físico e deu baixa (Fase 5).

**Nova Cardinalidade:**

- **Admin (1) ↔ (N) Spin:** Um administrador pode aprovar ou dar baixa em milhares de giros ao longo de seu turno.
