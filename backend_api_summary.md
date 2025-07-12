## Resumo da API do Backend (FederacaoMad API)

A análise da documentação da API Swagger e do conteúdo do `pasted_content.txt` revela que o backend do seu aplicativo VoIP, denominado 'FederacaoMad API', é uma aplicação Node.js robusta e bem estruturada, com diversas funcionalidades para gerenciamento de usuários, clãs, federações, canais de comunicação (texto e voz), missões (QRR), convites, solicitações de entrada, notificações e uploads de arquivos.

**Principais Categorias de Rotas e Funcionalidades:**

1.  **Administração (`/api/admin`):**
    *   Gerenciamento de usuários: listar, alterar papel (role), resetar senha, suspender.

2.  **Autenticação (`/api/auth`):**
    *   Registro de novos usuários (`/register`).
    *   Login e obtenção de token de autenticação (`/login`).
    *   Obtenção do perfil do usuário autenticado (`/profile`).

3.  **Canais de Comunicação (`/api/channels`):**
    *   Criação, obtenção (por tipo e entidade, por ID, todos), atualização e exclusão de canais.
    *   Funcionalidades para entrar e sair de canais.
    *   Obtenção de mensagens de um canal.

4.  **Missões do Clã (`/api/clan-missions`):**
    *   Confirmação de presença em missões.
    *   Adição de estratégia (upload de imagem) para missões.
    *   Criação, listagem (por clã, por ID), cancelamento, confirmação de presença e adição de mídia de estratégia.

5.  **Clãs (`/api/clans`):**
    *   Obtenção (todos, por ID de federação, por ID específico), criação, atualização e exclusão de clãs.
    *   Gerenciamento de banners de clãs.
    *   Funcionalidades para entrar e sair de clãs.
    *   Gerenciamento de membros: promover a sub-líder, rebaixar a membro comum, transferir liderança, expulsar.
    *   Gerenciamento de alianças e inimizades com outros clãs.
    *   Gerenciamento de cargos personalizados (adicionar, atualizar, deletar, atribuir, remover).

6.  **Federações (`/api/federations`):**
    *   Obtenção (todas, por ID), criação, atualização e exclusão de federações.
    *   Transferência de liderança e atualização de banners de federações.
    *   Adição e remoção de clãs de federações.
    *   Gerenciamento de sub-líderes (promover, rebaixar).
    *   Gerenciamento de alianças e inimizades com outras federações.

7.  **Canais Globais (`/api/global-channels`):**
    *   Listagem de todos os canais globais (texto e voz).
    *   Criação de novos canais globais (apenas ADM).
    *   Listagem de canais de texto e voz globais separadamente.
    *   Funcionalidades para entrar e sair de canais de voz globais.
    *   Exclusão de canais globais (apenas ADM).

8.  **Chat Global (`/api/global-chat`):**
    *   Envio e obtenção de mensagens no chat global.

9.  **Convites (`/api/invites`):**
    *   Envio de convites para usuários.
    *   Listagem de convites (do usuário logado, específicos).
    *   Aceitar e rejeitar convites.

10. **Solicitações de Entrada (`/api/join-requests`):**
    *   Criação de solicitações para entrar em clãs ou federações.
    *   Listagem de solicitações (do usuário logado, pendentes).
    *   Aprovação, rejeição e cancelamento de solicitações.

11. **QRR (Quests, Raids e Recompensas) (`/api/qrrs`):**
    *   Criação, obtenção (por clã, disponíveis, por ID), atualização e exclusão de QRRs.
    *   Atualização de status de QRRs.
    *   Funcionalidades para entrar e sair de QRRs.
    *   Marcação de presença em QRRs.
    *   Atualização de desempenho de participantes.
    *   Conclusão e cancelamento de QRRs.
    *   Aceitar e rejeitar QRRs de federação para um clã.

12. **Notificações Segmentadas (`/api/notifications`):**
    *   Envio de notificações para todos os usuários (apenas ADMs), membros de clãs (líderes de clã) e membros de federações (líderes de federação).
    *   Pré-visualização de destinatários para notificações.

13. **Estatísticas (`/api/stats`):**
    *   Obtenção de estatísticas globais do sistema e estatísticas de usuários específicos.

14. **Uploads (`/api/upload`):**
    *   Upload de fotos de perfil, bandeiras de clãs e tags de federações.
    *   Obtenção de informações de identidade visual de usuários.
    *   Listagem de todas as bandeiras de clãs e tags de federações.

15. **Usuários (`/api/users`):**
    *   Upload de foto de perfil.
    *   Listagem de membros de um clã específico.
    *   Obtenção de informações de um usuário por ID.
    *   Atualização de informações de perfil de um usuário.
    *   Atualização de status online/offline de um usuário.

16. **Canais de Voz (`/api/voice-channels`):**
    *   Obtenção de todos os canais de voz (globais, de clã, de federação).
    *   Criação de novos canais de voz.
    *   Funcionalidades para entrar e sair de canais de voz.
    *   Exclusão de canais de voz.

17. **VoIP (`/api/voip`):**
    *   Iniciação, aceitação, rejeição e encerramento de chamadas VoIP (1x1).
    *   Obtenção do histórico de chamadas VoIP do usuário.
    *   As rotas de VoIP parecem estar integradas com Jitsi (mencionado `jitsiToken` e `roomName`).

**Observações Importantes do `pasted_content.txt`:**

O arquivo `pasted_content.txt` também continha erros de resolução de referência no Swagger para `ClanChatMessage` e `FederationChatMessage` nas rotas de chat de clã e federação. Isso sugere que os esquemas para esses objetos podem estar faltando ou incorretamente referenciados na definição da API, o que pode causar problemas na interpretação da API por ferramentas ou no desenvolvimento do frontend.

**Próximos Passos:**

Com base nesta análise, o backend parece ser abrangente para um aplicativo VoIP com funcionalidades sociais e de gerenciamento de comunidades (clãs/federações). O próximo passo é correlacionar essa compreensão com o código fonte do backend e, posteriormente, com o frontend, para identificar a causa raiz dos problemas que você está enfrentando. Estou pronto para receber os códigos do backend e frontend, juntamente com o logcat atualizado, para prosseguir com a análise.

