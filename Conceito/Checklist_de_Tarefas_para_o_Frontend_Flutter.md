# Checklist de Tarefas para o Frontend Flutter

Este checklist resume as ações necessárias para a reestruturação do seu frontend, seguindo o plano detalhado:

## 1. Preparação e Correção da Base

- [ ] **Correção do Erro de `Provider` em `main.dart`:**
  - [ ] Abra `lib/main.dart`.
  - [ ] Localize o `MultiProvider`.
  - [ ] Verifique e adicione/ajuste todos os `ChangeNotifierProvider` e `Provider.value` para *todos* os serviços que você utiliza no seu aplicativo.

- [ ] **Atualização do `AuthService` e Modelo `User`:**
  - [ ] Abra `lib/services/auth_service.dart`.
  - [ ] No método de login (ou onde você busca os dados do usuário após a autenticação), certifique-se de que o objeto `User` retornado do backend seja completamente mapeado para o seu modelo `User` no Flutter.
  - [ ] Abra `lib/models/user.dart`.
  - [ ] Verifique se a classe `User` possui todos os campos relevantes do backend: `id`, `username`, `avatar`, `bio`, `status`, `clan` (ID), `clanRole`, `federation` (ID), `federationRole`, `role`, `online`, `lastSeen`.
  - [ ] Adicione métodos `get` convenientes no `AuthService` para verificar a `role` do usuário (ex: `isAdmin`, `isClanLeader`, `isFederationLeader`, `isClanMember`).

## 2. Implementação do Controle de Acesso (RBAC) e Navegação Dinâmica

- [ ] **Rotas Protegidas e Redirecionamento:**
  - [ ] Para cada tela que requer uma `role` específica (ex: `AdminManageUsersScreen`), adicione uma verificação no método `build` ou no `initState` para redirecionar usuários não autorizados.
  - [ ] Crie uma `AccessDeniedScreen`.

- [ ] **Navegação Principal Dinâmica (`BottomNavigationBar`):**
  - [ ] Modifique o widget que contém sua `BottomNavigationBar` (provavelmente `lib/screens/home_screen.dart` ou `lib/screens/main_app_screen.dart`).
  - [ ] Use `Consumer<AuthService>` para reagir às mudanças na `role` do usuário.
  - [ ] Construa a lista de `BottomNavigationBarItem` e as telas (`PageView` ou `IndexedStack`) dinamicamente com base na `role`.

- [ ] **Renderização Condicional de Widgets:**
  - [ ] Em qualquer widget onde a visibilidade de um elemento depende da `role` do usuário, use `if` statements ou `Visibility`.

## 3. Arquitetura de Layouts e Navegação Imersiva

- [ ] **`ExploreClansFederationsScreen` (Nova Tela):**
  - [ ] Crie `lib/screens/explore_clans_federations_screen.dart`.
  - [ ] Implemente a listagem de todas as Federações e a navegação para `FederationDetailScreen`.

- [ ] **`FederationDetailScreen` (Painel da Federação - O País):**
  - [ ] Crie `lib/screens/federation_detail_screen.dart`.
  - [ ] Implemente a estrutura com `DefaultTabController`, `TabBar` e `TabBarView` para as abas (Visão Geral, Clãs Membros, Gerenciamento Condicional).
  - [ ] Use `FederationService` para buscar dados.

- [ ] **`ClanDetailScreen` (Painel do Clã - O Estado):**
  - [ ] Crie `lib/screens/clan_detail_screen.dart` (ou refatore `ClanManagementScreen`).
  - [ ] Implemente a estrutura com `DefaultTabController`, `TabBar` e `TabBarView` para as abas (Mural/Home, Membros, Missões QRR, Insta Clã, Voz, Configurações).
  - [ ] **Mova o conteúdo da sua tela atual de Missões QRR para uma aba dentro de `ClanDetailScreen`.**
  - [ ] Ajuste a lógica da aba "Voz" para que o ADM Master possa acessar mesmo sem ser membro.
  - [ ] Use `ClanService` para buscar dados.

- [ ] **`AdminDashboardScreen` (Painel do ADM Master):**
  - [ ] Refatore `lib/widgets/admin_dashboard.dart` para ser a tela principal do ADM.
  - [ ] Adicione navegação para `AdminManageUsersScreen`, `AdminManageClansScreen` (nova), `AdminManageFederationsScreen` (nova).
  - [ ] Implemente a navegação do ADM para `ClanDetailScreen` e `FederationDetailScreen` em modo de gerenciamento (`isAdminViewing: true`).
  - [ ] Adicione botões para `Criar Federação` e `Criar Clã`.

## 4. Implementação do "Insta Clã"

- [ ] **Modelo de Dados para `InstaClanPost`:**
  - [ ] Crie `lib/models/instaclan_post.dart` com a estrutura definida.

- [ ] **Serviço para `InstaClã`:**
  - [ ] Adicione métodos ao seu `ClanService` (ou crie um novo `InstaClanService`) para `createInstaClanPost`, `getInstaClanPosts`, `deleteInstaClanPost`.

- [ ] **Tela `InstaClanScreen`:**
  - [ ] Crie `lib/screens/instaclan_screen.dart` (ou refatore `lib/widgets/instaclan_feed_widget.dart`).
  - [ ] Implemente a exibição de postagens, formulário de criação e lógica de exclusão condicional.
  - [ ] Integre-a como uma aba em `ClanDetailScreen`.

## 5. Visualização de Usuários com Tags/Bandeiras

- [ ] **Atualizar Modelo `User` (se necessário):**
  - [ ] Certifique-se de que seu modelo `lib/models/user.dart` possa receber objetos aninhados para `clan` e `federation` (ou apenas os campos `name` e `tag`).

- [ ] **Exibição na UI:**
  - [ ] Em qualquer lugar onde você lista usuários (ex: `AdminManageUsersScreen`, lista de membros do clã, chat global), use os campos `clanInfo` e `federationInfo` para exibir as tags e bandeiras.

## 6. Testes e Refinamentos

- [ ] Realize testes completos de todas as funcionalidades, especialmente o controle de acesso para cada `role`.
- [ ] Ajuste a UI/UX para garantir a imersão e a clareza visual, seguindo a analogia de "Países e Estados".
- [ ] Adicione indicadores de carregamento, mensagens de erro e feedback visual para o usuário.
- [ ] Teste cada funcionalidade iterativamente.

