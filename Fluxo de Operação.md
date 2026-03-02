# 🔄 Documentação Técnica: Fluxo de Operação

Este documento detalha a jornada física e digital do cliente no Toca Music Bar, mapeando a interação entre as três interfaces do sistema e as mudanças de estado no banco de dados.

## 📱 Visão Geral do Ecossistema

O sistema é composto por três interfaces distintas que conversam entre si em tempo real:

- **App Cliente (Web/Mobile):** Acessado via QR Code no celular do cliente. Focado em aquisição de dados e direcionamento para a avaliação.
- **Totem (Tablet):** Fixo no caixa. Serve como display de atração (screensaver) e como a máquina de giro da roleta.
- **Painel Admin:** Operado pela bartender/caixa em um computador ou tablet seguro para gerenciar liberações da fila e resgates de prêmios.

---

## 📍 Fase 1: Atração e Captação (Tablet + Celular)

- **Estado do Tablet:** O tablet no caixa está em modo _Screensaver_. A tela exibe uma animação chamativa da Roleta, um QR Code gigante e a mensagem: _"Escaneie para avaliar o Toca Music Bar e ganhe um giro na Roleta!"_. Há também um botão no rodapé: _"Já tenho passe liberado"_ (atalho para a Fase 4).
- **Ação do Cliente:** O cliente, da sua própria mesa ou na fila, aponta a câmera do celular para o QR Code do tablet (ou de displays impressos espalhados pelo bar).

## 📝 Fase 2: Cadastro e Avaliação (Celular do Cliente)

- **A Interface (Celular):** O QR Code abre uma página web leve (App Cliente).
- **O Cadastro:** O cliente preenche um formulário super rápido contendo: `Nome`, `WhatsApp` e `CPF`.
- **A Ação Principal:** Ao enviar o cadastro, a tela muda para um grande botão: **"Ir Avaliar"**.
- **O Redirecionamento:** O botão abre o link direto do Google Meu Negócio do bar.
- **O Retorno:** Após deixar as 5 estrelas, o cliente volta para a aba do sistema no celular. A tela exibe a mensagem de instrução: _"Tudo certo! Mostre para a Bartender que você avaliou para liberar o seu giro."_
- **Background (Sistema):** O banco de dados registra esse usuário e cria uma intenção de giro com o status `AWAITING_VALIDATION`.

## 🛡️ Fase 3: Validação Visual e Liberação (Admin / Bartender)

- **Ação Física:** O cliente vai até o caixa e mostra a tela do seu celular (com a avaliação do Google feita) para a bartender.
- **A Interface (Admin):** A bartender acessa o Painel Admin. Ela vê uma lista em tempo real dos CPFs/Nomes que estão com status `AWAITING_VALIDATION`.
- **A Liberação:** A bartender localiza o nome do cliente na fila e clica em **"Liberar Roleta"**.
- **Background (Sistema):** O status daquele CPF no banco de dados muda para `READY_TO_SPIN`. Neste exato milissegundo, o backend roda o algoritmo de pesos (_probabilidades_) e define qual prêmio este CPF vai ganhar, garantindo a segurança contra manipulações no frontend.

## 🎰 Fase 4: O Momento do Giro (Tablet no Caixa)

- **Ação do Cliente:** O cliente toca no botão **"Já tenho passe liberado"** no tablet do caixa.
- **A Identificação:** O tablet exibe um teclado numérico grande. O cliente digita o seu **CPF**.
- **A Validação:** O tablet consulta o banco. Como a bartender já liberou aquele CPF na Fase 3, a tela avança. _(Se não estivesse liberado, o sistema exibiria o aviso: "Peça para a bartender liberar seu passe primeiro!")._
- **O Giro:** A roleta gigante aparece na tela do tablet. O cliente aperta **"GIRAR"**. A animação acontece fluida e é forçada a parar no prêmio pré-determinado pelo servidor na etapa anterior.

## 🎉 Fase 5: Celebração e Carteira de Prêmios (Tablet + Admin)

- **A Tela do Tablet:** Confetes virtuais aparecem com a mensagem: _"Parabéns, [Nome]! Você ganhou [Prêmio]!"_.
- **A Decisão do Cliente:** O tablet apresenta duas opções de resgate:
  - **"Retirar Agora":** O Painel da Bartender emite um alerta sonoro/visual avisando que o cliente do CPF 'X' está retirando o prêmio 'Y' naquele momento. Ela entrega o produto e dá baixa no sistema (status muda para `CLAIMED`).
  - **"Guardar na Carteira":** O prêmio fica salvo no CPF do cliente com status `PENDING_RESCUE` para ele retirar mais tarde (na mesma noite ou em outro dia). O tablet volta imediatamente para o _Screensaver_ (Fase 1), liberando a fila.
