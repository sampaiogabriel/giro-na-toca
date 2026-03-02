# 🏢 Documentação Técnica: Visão Geral da Arquitetura de Negócios

Este documento estabelece as fundações lógicas do sistema. Para garantir que a plataforma do Toca Music Bar seja escalável e adaptável a diferentes estratégias de marketing, a arquitetura de negócios não trata a roleta como uma funcionalidade estática, mas sim como um ecossistema baseado em **Campanhas** e **Ativos (Assets)**.

## 1. Entidade "Campanha"

A roleta **não é global**. Ela é uma ferramenta modular que sempre pertence a uma **Campanha ativa**. Isso permite que a administração do bar crie eventos temáticos simultâneos ou sazonais sem a necessidade de alterar o código-fonte.

Uma campanha define estritamente:

- **Regras de Participação:** O gatilho que autoriza o cliente a girar a roleta (Ex: "Avaliação no Google", "Comprar Combo de Gin").
- **Vigência:** O período de funcionamento (Data de início e Data de fim).
- **Inventário:** Os itens atrelados (Prêmios) àquela campanha e os seus respectivos **pesos/probabilidades matemáticos** de sorteio, além do controle de estoque.

---

## 2. Entidade "Carteira" (Wallet)

O prêmio sorteado não é tratado apenas como uma mensagem comemorativa temporária na tela do Totem. Ele se torna um **Asset (Ativo Digital)**.

- **O Conceito:** Assim que o sorteio é processado pelo backend, o prêmio é imediatamente vinculado ao identificador único do usuário (CPF / WhatsApp).
- **Ciclo de Vida e Retenção:** Esse ativo recebe o status de `PENDING_RESCUE`.
- **Impacto Operacional:** Isso cria uma "Carteira" para o cliente. Ele não é obrigado a consumir o prêmio no mesmo instante em que gira a roleta, o que evita gargalos físicos no caixa do bar e incentiva o retorno do cliente em ocasiões futuras para resgatar o que está guardado.
