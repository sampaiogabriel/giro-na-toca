# 🎡 Toca Music Bar - Sistema de Roleta Gamificada

Um sistema completo de gamificação para o ambiente físico do **Toca Music Bar**, focado em conversão de avaliações no Google Meu Negócio e retenção de clientes através de prêmios sorteados via Roleta da Fortuna.

Desenvolvido por [codesampa.io](https://codesampa.io).

## 📌 Visão Geral do Produto

O sistema resolve um problema clássico de operações em bares: como incentivar o engajamento do cliente (avaliações) sem gerar filas ou atrito no caixa.

Para isso, a arquitetura foi dividida em um ecossistema de 3 pontas, garantindo que o cliente faça o trabalho demorado (cadastro e avaliação) no próprio celular, utilizando o Totem do bar apenas para a experiência rápida e visual do sorteio.

### O Ecossistema

1. **App Cliente (Mobile Web):** Acessado via QR Code nas mesas. Onde o cliente se cadastra (CPF) e é redirecionado para o Google.
2. **Painel Admin (Caixa/Bartender):** Interface protegida onde o _staff_ do bar gerencia as campanhas, aprova os giros após conferir a avaliação física e dá baixa nos prêmios.
3. **Totem (Tablet no Caixa):** Tela interativa onde o cliente digita o CPF, gira a roleta e descobre o prêmio.

## ⚙️ Arquitetura e Fluxo de Operação

O fluxo foi desenhado com foco em **Segurança** (o frontend não sabe o resultado do sorteio) e **Escalabilidade** (prêmios ficam salvos em uma "carteira" atrelada ao CPF do cliente).

1. **Captação:** Cliente escaneia o QR Code, insere Nome, WhatsApp e CPF no próprio celular, e é redirecionado para avaliar o bar.
2. **Validação:** Cliente mostra a tela de avaliação concluída para a Bartender.
3. **Aprovação e Sorteio:** A Bartender aprova a liberação no Painel Admin. Neste exato milissegundo, o **Backend** roda o algoritmo de pesos (probabilidades), define o prêmio vencedor e salva no banco de dados.
4. **O Giro:** Cliente vai ao Totem, digita o CPF. A roleta é liberada, gira e é forçada matematicamente a parar no prêmio pré-definido pelo servidor.
5. **Resgate ou Carteira:** O cliente pode retirar o prêmio na hora (Painel avisa a Bartender) ou guardar em sua "Carteira" digital para retirar mais tarde.

## 🛠️ Stack Tecnológica

O projeto utiliza uma stack moderna, focada em performance na Edge, tempo real simulado de baixo custo e alta produtividade:

- **Framework:** [Next.js 15](https://nextjs.org/) (App Router + Server Actions)
- **Linguagem:** TypeScript
- **Banco de Dados:** [Neon](https://neon.tech/) (Serverless PostgreSQL)
- **ORM:** [Prisma](https://www.prisma.io/) (Tipagem e modelagem de dados relacional)
- **Autenticação (Admin):** [Clerk](https://clerk.com/) (Proteção das rotas do Backoffice)
- **Gerenciamento de Estado/Polling:** [React Query / TanStack Query](https://tanstack.com/query/latest) (Para o Totem escutar a liberação do caixa)
- **UI e Animação da Roleta:** `react-custom-roulette` e `shadcn/ui`
- **Estilização:** Tailwind CSS
- **Formulário** React Hook Form
- **Validação** Zod

## 🗄️ Estrutura do Banco de Dados

O sistema é baseado em 4 entidades principais (veja a documentação completa do modelo de dados para mais detalhes):

- `Customer` (Cliente único via CPF)
- `Campaign` (Campanhas sazonais ativas/inativas)
- `Prize` (Itens sorteáveis com pesos de probabilidade e controle de estoque)
- `Spin` (A tabela pivô de histórico, status de liberação e resgate)

---

_Documentação gerada para alinhamento de arquitetura e regras de negócio antes do início do desenvolvimento._
