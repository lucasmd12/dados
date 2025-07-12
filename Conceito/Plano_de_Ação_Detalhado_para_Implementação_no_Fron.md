# Plano de Ação Detalhado para Implementação no Frontend Flutter

Este plano é um guia prático e direto para você implementar as mudanças necessárias no seu frontend Flutter, aproveitando ao máximo o que já existe e focando na experiência imersiva de Federações (Países) e Clãs (Estados), além do controle de acesso baseado em função (RBAC).

## 1. Preparação e Correção da Base

Antes de qualquer nova funcionalidade, é crucial garantir que a base do seu aplicativo esteja sólida.

### 1.1. Correção do Erro de `Provider` em `main.dart`

**Problema:** O erro `Could not find the correct Provider<UserService>` indica que seus serviços não estão sendo injetados corretamente na árvore de widgets.

**Ação:**
1.  Abra `lib/main.dart`.
2.  Localize o `MultiProvider`.
3.  **Verifique e adicione/ajuste todos os `ChangeNotifierProvider` e `Provider.value` para *todos* os serviços que você utiliza no seu aplicativo.** O erro que você viu é para `UserService`, mas é provável que outros serviços também precisem ser declarados ali para serem acessíveis globalmente.

    ```dart
    // lib/main.dart
    import 'package:provider/provider.dart';
    import 'package:lucasbeatsfederacao/services/api_service.dart'; // Certifique-se de importar
    import 'package:lucasbeatsfederacao/services/user_service.dart';
    import 'package:lucasbeatsfederacao/services/auth_service.dart';
    import 'package:lucasbeatsfederacao/services/clan_service.dart';
    import 'package:lucasbeatsfederacao/services/federation_service.dart';
    import 'package:lucasbeatsfederacao/services/socket_service.dart';
    import 'package:lucasbeatsfederacao/services/signaling_service.dart';
    import 'package:lucasbeatsfederacao/services/mission_service.dart';
    import 'package:lucasbeatsfederacao/services/notification_service.dart';
    import 'package:lucasbeatsfederacao/services/voip_service.dart';
    import 'package:lucasbeatsfederacao/services/firebase_service.dart';
    import 'package:lucasbeatsfederacao/services/chat_service.dart';
    import 'package:lucasbeatsfederacao/providers/auth_provider.dart'; // Seu AuthProvider
    import 'package:lucasbeatsfederacao/providers/call_provider.dart'; // Seu CallProvider
    import 'package:lucasbeatsfederacao/providers/connectivity_provider.dart'; // Seu ConnectivityProvider
    import 'package:lucasbeatsfederacao/providers/mission_provider.dart'; // Seu MissionProvider
    import 'package:lucasbeatsfederacao/services/qrr_service.dart';

    void main() {
      runApp(
        MultiProvider(
          providers: [
            // Seus serviços principais
            Provider<ApiService>(create: (_) => ApiService()), // ApiService deve ser o primeiro se outros dependem dele
            ChangeNotifierProvider(create: (context) => AuthService(context.read<ApiService>())),
            ChangeNotifierProvider(create: (context) => UserService(context.read<ApiService>())),
            ChangeNotifierProvider(create: (context) => SocketService()),
            ChangeNotifierProvider(create: (context) => SignalingService()),
            ChangeNotifierProvider(create: (context) => FederationService(context.read<ApiService>())),
            ChangeNotifierProvider(create: (context) => ClanService(context.read<ApiService>(), context.read<AuthService>())),
            ChangeNotifierProvider(create: (context) => MissionService(context.read<ApiService>())),
            ChangeNotifierProvider(create: (context) => NotificationService()),
            ChangeNotifierProvider(create: (context) => VoIPService()),
            ChangeNotifierProvider(create: (context) => FirebaseService(context.read<ApiService>())),
            ChangeNotifierProvider(create: (context) => ChatService(
              firebaseService: context.read<FirebaseService>(),
              authService: context.read<AuthService>())),
            ChangeNotifierProvider(create: (context) => QRRService(context.read<ApiService>())),

            // Seus Providers
            ChangeNotifierProvider(create: (context) => AuthProvider(
              context.read<SocketService>(),
              context.read<AuthService>(),
            )),
            ChangeNotifierProvider(create: (context) => CallProvider(authService: context.read<AuthService>())),
            ChangeNotifierProvider(create: (context) => ConnectivityProvider()),
            ChangeNotifierProvider(create: (context) => MissionProvider(
              context.read<MissionService>(),
            )),
          ],
          child: MyApp(),
        ),
      );
    }
    ```

### 1.2. Atualização do `AuthService` e Modelo `User`

**Ação:** Garanta que o `AuthService` armazene e o modelo `User` contenha as informações de `role`, `clanId`, `federationId`, `clanRole`, `federationRole` após o login.

1.  Abra `lib/services/auth_service.dart`.
2.  No método de login (ou onde você busca os dados do usuário após a autenticação), certifique-se de que o objeto `User` retornado do backend seja completamente mapeado para o seu modelo `User` no Flutter.
3.  Abra `lib/models/user.dart` (ou similar).
4.  **Verifique se a classe `User` possui todos os campos relevantes do backend:** `id`, `username`, `avatar`, `bio`, `status`, `clan` (ID), `clanRole`, `federation` (ID), `federationRole`, `role`, `online`, `lastSeen`.
5.  Adicione métodos `get` convenientes no `AuthService` para verificar a `role` do usuário (ex: `isAdmin`, `isClanLeader`, `isFederationLeader`, `isClanMember`).

    ```dart
    // lib/services/auth_service.dart (exemplo)
    class AuthService extends ChangeNotifier {
      // ...
      User? _currentUser;
      User? get currentUser => _currentUser;

      bool get isAdmin => _currentUser?.role == 'ADM';
      bool get isClanLeader => _currentUser?.clanRole == 'Leader' || _currentUser?.clanRole == 'leader';
      bool get isSubLeader => _currentUser?.clanRole == 'SubLeader' || _currentUser?.clanRole == 'subleader';
      bool get isFederationLeader => _currentUser?.federationRole == 'leader';
      bool get isClanMember => _currentUser?.clan != null; // Ou uma verificação mais específica
      // ...
    }
    ```

## 2. Implementação do Controle de Acesso (RBAC) e Navegação Dinâmica

### 2.1. Rotas Protegidas e Redirecionamento

**Ação:** Para cada tela que requer uma `role` específica, adicione uma verificação no método `build` ou no `initState`.

1.  **Para Telas de ADM (ex: `AdminManageUsersScreen`):**
    *   Abra `lib/screens/admin_manage_users_screen.dart`.
    *   No `build` ou `initState`, adicione a lógica de verificação:

        ```dart
        // lib/screens/admin_manage_users_screen.dart
        import 'package:provider/provider.dart';
        import 'package:lucasbeatsfederacao/services/auth_service.dart';

        class AdminManageUsersScreen extends StatefulWidget {
          // ...
        }

        class _AdminManageUsersScreenState extends State<AdminManageUsersScreen> {
          @override
          void initState() {
            super.initState();
            WidgetsBinding.instance.addPostFrameCallback((_) {
              final authService = Provider.of<AuthService>(context, listen: false);
              if (!authService.isAdmin) {
                Navigator.of(context).pushReplacementNamed('/access-denied'); // Ou para a home
              }
            });
          }
          // ... build method
        }
        ```
2.  **Crie uma `AccessDeniedScreen`:** Uma tela simples informando que o usuário não tem permissão.

### 2.2. Navegação Principal Dinâmica (`BottomNavigationBar`)

**Ação:** Modifique o widget que contém sua `BottomNavigationBar` (provavelmente `lib/screens/home_screen.dart` ou `lib/screens/main_app_screen.dart`) para renderizar itens de navegação com base na `role` do usuário.

1.  Abra o arquivo da sua tela principal que contém a `BottomNavigationBar`.
2.  Use `Consumer<AuthService>` para reagir às mudanças na `role` do usuário.
3.  Construa a lista de `BottomNavigationBarItem` e as telas (`PageView` ou `IndexedStack`) dinamicamente.

    ```dart
    // Exemplo em lib/screens/home_screen.dart
    class HomeScreen extends StatefulWidget {
      const HomeScreen({Key? key}) : super(key: key);

      @override
      State<HomeScreen> createState() => _HomeScreenState();
    }

    class _HomeScreenState extends State<HomeScreen> {
      int _selectedIndex = 0;
      late PageController _pageController;

      @override
      void initState() {
        super.initState();
        _pageController = PageController(initialPage: _selectedIndex);
      }

      @override
      void dispose() {
        _pageController.dispose();
        super.dispose();
      }

      void _onItemTapped(int index) {
        setState(() {
          _selectedIndex = index;
          _pageController.jumpToPage(index);
        });
      }

      @override
      Widget build(BuildContext context) {
        final authService = Provider.of<AuthService>(context);

        List<Widget> _screens = [];
        List<BottomNavigationBarItem> _navItems = [];

        // Itens padrão para todos
        _screens.add(DashboardScreen()); // Sua tela de início
        _navItems.add(BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Início'));

        // Itens para ADM
        if (authService.isAdmin) {
          _screens.add(AdminDashboardScreen()); // Seu painel de ADM
          _navItems.add(BottomNavigationBarItem(icon: Icon(Icons.admin_panel_settings), label: 'ADM'));
          _screens.add(AdminManageUsersScreen()); // Tela de Gerenciar Usuários
          _navItems.add(BottomNavigationBarItem(icon: Icon(Icons.people), label: 'Usuários'));
          // ... adicione outras telas de ADM aqui
        }

        // Itens para Membros de Clã
        if (authService.isClanMember) {
          _screens.add(ClanDashboardScreen(clanId: authService.currentUser!.clanId!)); // Tela do Meu Clã
          _navItems.add(BottomNavigationBarItem(icon: Icon(Icons.group), label: 'Meu Clã'));
        } else { // Se não for membro de clã, mostra Explorar
          _screens.add(ExploreClansFederationsScreen()); // Nova tela para explorar clãs/federações
          _navItems.add(BottomNavigationBarItem(icon: Icon(Icons.explore), label: 'Explorar'));
        }

        // Itens globais
        _screens.add(GlobalChatScreen());
        _navItems.add(BottomNavigationBarItem(icon: Icon(Icons.chat), label: 'Global'));
        _screens.add(SettingsScreen());
        _navItems.add(BottomNavigationBarItem(icon: Icon(Icons.settings), label: 'Config'));

        return Scaffold(
          body: PageView(
            controller: _pageController,
            onPageChanged: (index) {
              setState(() {
                _selectedIndex = index;
              });
            },
            children: _screens,
          ),
          bottomNavigationBar: BottomNavigationBar(
            items: _navItems,
            currentIndex: _selectedIndex,
            onTap: _onItemTapped,
            type: BottomNavigationBarType.fixed, // Para mais de 3 itens
            // ... outros estilos
          ),
        );
      }
    }
    ```

### 2.3. Renderização Condicional de Widgets

**Ação:** Em qualquer widget onde a visibilidade de um elemento depende da `role` do usuário, use `if` statements ou `Visibility`.

```dart
// Exemplo: Botão de criar federação no painel do ADM
import 'package:lucasbeatsfederacao/services/auth_service.dart';

// ... dentro do build method de um widget
final authService = Provider.of<AuthService>(context);

if (authService.isAdmin) {
  ElevatedButton(
    onPressed: () { /* Lógica para criar federação */ },
    child: Text('Criar Federação'),
  ),
} else {
  // Opcional: SizedBox.shrink() para não ocupar espaço
  // Ou um widget alternativo
}
```

## 3. Arquitetura de Layouts e Navegação Imersiva

Esta é a parte que trará a sensação de "Países e Estados" e resolverá a bagunça das telas.

### 3.1. `ExploreClansFederationsScreen` (Nova Tela)

**Ação:** Crie uma nova tela `lib/screens/explore_clans_federations_screen.dart`.

*   Esta tela será acessível a todos os usuários (não membros de clã/federação) através da `BottomNavigationBar`.
*   Ela deve listar todas as Federações disponíveis. Ao clicar em uma Federação, navegue para o `FederationDetailScreen`.
*   Pode também ter uma seção para listar Clãs sem Federação, ou um filtro.

### 3.2. `FederationDetailScreen` (Painel da Federação - O País)

**Ação:** Crie `lib/screens/federation_detail_screen.dart`.

*   Esta tela receberá o `federationId` como parâmetro.
*   **Estrutura:** Use um `DefaultTabController` com `TabBar` e `TabBarView` para as abas:
    *   **Visão Geral:** Exibe `FederationInfoWidget` (nome, descrição, banner).
    *   **Clãs Membros:** Lista os clãs que pertencem a esta federação. Ao clicar em um clã, navegue para `ClanDetailScreen`.
    *   **Gerenciamento (Condicional):** Se o usuário for `isFederationLeader` ou `isAdmin`, mostre uma aba ou botões para:
        *   Adicionar/Remover Clãs (chamando as APIs do backend).
        *   Editar informações da Federação.
*   **Dados:** Use `FederationService` para buscar os detalhes da federação e seus clãs membros.

### 3.3. `ClanDetailScreen` (Painel do Clã - O Estado)

**Ação:** Crie `lib/screens/clan_detail_screen.dart` (ou refatore `ClanManagementScreen` para ser mais genérica).

*   Esta tela receberá o `clanId` como parâmetro.
*   **Estrutura:** Use um `DefaultTabController` com `TabBar` e `TabBarView` para as abas:
    *   **Mural/Home:** Exibe `ClanInfoWidget` (nome, tag, banner) e um feed de notícias/avisos.
    *   **Membros:** Lista os membros do clã. Se o usuário for `isClanLeader`, `isSubLeader` ou `isAdmin`, mostre opções de gerenciamento (promover, remover).
    *   **Missões QRR:** **Mova o conteúdo da sua tela atual de Missões QRR para cá.** Apenas visível para `isClanMember` ou `isAdmin`.
    *   **Insta Clã:** A nova tela `InstaClanScreen` será integrada aqui.
    *   **Voz:** O canal de voz do clã. **Ajuste a lógica para que `isAdmin` possa acessar mesmo sem ser membro.**
    *   **Configurações (Condicional):** Se o usuário for `isClanLeader`, `isSubLeader` ou `isAdmin`, mostre opções para editar informações do clã.
*   **Dados:** Use `ClanService` para buscar os detalhes do clã e seus membros.

### 3.4. `AdminDashboardScreen` (Painel do ADM Master)

**Ação:** Refatore `lib/widgets/admin_dashboard.dart` para ser a tela principal do ADM.

*   **Estrutura:** Pode ser uma grade de botões ou um menu lateral que leva a telas de gerenciamento específicas.
*   **Funcionalidades:**
    *   **Gerenciar Usuários:** Navegue para `AdminManageUsersScreen` (que já existe e será corrigida pelo Provider).
    *   **Gerenciar Clãs:** Navegue para uma nova tela `AdminManageClansScreen` que lista todos os clãs. Ao clicar em um clã, o ADM pode navegar para `ClanDetailScreen` com um parâmetro `isAdminViewing: true`.
    *   **Gerenciar Federações:** Navegue para uma nova tela `AdminManageFederationsScreen` que lista todas as federações. Ao clicar em uma federação, o ADM pode navegar para `FederationDetailScreen` com `isAdminViewing: true`.
    *   **Criação Direta:** Adicione botões para `Criar Federação` e `Criar Clã` que chamam os modais ou navegam para as telas de criação.

## 4. Implementação do "Insta Clã" (Urgente)

O backend já foi atualizado com as APIs para o Insta Clã. Agora, o foco é no frontend.

### 4.1. Modelo de Dados para `InstaClanPost`

**Ação:** Crie um novo arquivo `lib/models/instaclan_post.dart`.

```dart
// lib/models/instaclan_post.dart
import 'package:lucasbeatsfederacao/models/user.dart'; // Assumindo que você tem um modelo User

class InstaClanPost {
  final String id;
  final String clanId;
  final User author;
  final String content;
  final String? imageUrl;
  final List<String> likes; // Lista de IDs de usuários
  final List<Comment> comments;
  final DateTime createdAt;

  InstaClanPost({
    required this.id,
    required this.clanId,
    required this.author,
    required this.content,
    this.imageUrl,
    required this.likes,
    required this.comments,
    required this.createdAt,
  });

  factory InstaClanPost.fromJson(Map<String, dynamic> json) {
    return InstaClanPost(
      id: json['_id'],
      clanId: json['clan'],
      author: User.fromJson(json['author']), // Certifique-se que User.fromJson funciona com o objeto populado
      content: json['content'],
      imageUrl: json['imageUrl'],
      likes: List<String>.from(json['likes']),
      comments: (json['comments'] as List)
          .map((c) => Comment.fromJson(c))
          .toList(),
      createdAt: DateTime.parse(json['createdAt']),
    );
  }
}

class Comment {
  final String authorId;
  final String content;
  final DateTime createdAt;

  Comment({
    required this.authorId,
    required this.content,
    required this.createdAt,
  });

  factory Comment.fromJson(Map<String, dynamic> json) {
    return Comment(
      authorId: json['author'],
      content: json['content'],
      createdAt: DateTime.parse(json['createdAt']),
    );
  }
}
```

### 4.2. Serviço para `InstaClã`

**Ação:** Adicione métodos ao seu `ClanService` ou crie um novo `InstaClanService`.

```dart
// lib/services/clan_service.dart (ou novo instaclan_service.dart)
import 'package:lucasbeatsfederacao/models/instaclan_post.dart';

class ClanService extends ChangeNotifier {
  final ApiService _apiService;
  // ...

  Future<InstaClanPost> createInstaClanPost(String clanId, String content, {String? imageUrl}) async {
    final response = await _apiService.post('/clans/$clanId/instaclan/posts', {
      'content': content,
      'imageUrl': imageUrl,
    });
    return InstaClanPost.fromJson(response.data);
  }

  Future<List<InstaClanPost>> getInstaClanPosts(String clanId) async {
    final response = await _apiService.get('/clans/$clanId/instaclan/posts');
    return (response.data as List).map((json) => InstaClanPost.fromJson(json)).toList();
  }

  Future<void> deleteInstaClanPost(String clanId, String postId) async {
    await _apiService.delete('/clans/$clanId/instaclan/posts/$postId');
  }
  // ... métodos para likes/comments se desejar
}
```

### 4.3. Tela `InstaClanScreen`

**Ação:** Crie `lib/screens/instaclan_screen.dart` (ou refatore `lib/widgets/instaclan_feed_widget.dart` para ser uma tela completa).

*   Esta tela receberá o `clanId` como parâmetro.
*   Use `FutureBuilder` ou `ChangeNotifierProvider` para carregar as postagens do `ClanService`.
*   Exiba as postagens em um `ListView.builder`.
*   Adicione um formulário para criar novas postagens.
*   Implemente a lógica para mostrar o botão de exclusão apenas para o autor da postagem, Líder/Sublíder do clã ou ADM Master.

## 5. Visualização de Usuários com Tags/Bandeiras

O backend já foi ajustado para popular os campos `clan` e `federation` com `name` e `tag` nas APIs de usuário. Agora, o frontend precisa consumir e exibir isso.

### 5.1. Atualizar Modelo `User` (se necessário)

**Ação:** Certifique-se de que seu modelo `lib/models/user.dart` possa receber objetos aninhados para `clan` e `federation` (ou apenas os campos `name` e `tag`).

```dart
// lib/models/user.dart (exemplo de como lidar com objetos populados)
class User {
  // ... outros campos
  final ClanInfo? clanInfo; // Novo campo
  final FederationInfo? federationInfo; // Novo campo

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      // ... mapeamento de outros campos
      clanInfo: json['clan'] != null ? ClanInfo.fromJson(json['clan']) : null,
      federationInfo: json['federation'] != null ? FederationInfo.fromJson(json['federation']) : null,
    );
  }
}

class ClanInfo {
  final String id;
  final String name;
  final String tag;
  ClanInfo({required this.id, required this.name, required this.tag});
  factory ClanInfo.fromJson(Map<String, dynamic> json) {
    return ClanInfo(id: json['_id'], name: json['name'], tag: json['tag']);
  }
}

class FederationInfo {
  final String id;
  final String name;
  // Adicione a tag se a federação tiver uma
  FederationInfo({required this.id, required this.name});
  factory FederationInfo.fromJson(Map<String, dynamic> json) {
    return FederationInfo(id: json['_id'], name: json['name']);
  }
}
```

### 5.2. Exibição na UI

**Ação:** Em qualquer lugar onde você lista usuários (ex: `AdminManageUsersScreen`, lista de membros do clã, chat global), use os campos `clanInfo` e `federationInfo` para exibir as tags e bandeiras.

```dart
// Exemplo em um ListTile para um usuário
ListTile(
  leading: CircleAvatar(backgroundImage: NetworkImage(user.avatar ?? 'default_avatar.png')),
  title: Text(user.username),
  subtitle: Row(
    children: [
      if (user.clanInfo != null) Text('[${user.clanInfo!.tag}] ${user.clanInfo!.name}'),
      if (user.federationInfo != null) Text(' (${user.federationInfo!.name})'),
      // Adicione aqui widgets para bandeiras (imagens) se tiver URLs no backend
    ],
  ),
  trailing: Text(user.role),
);
```

## 6. Cronograma de Implementação (Foco no Frontend)

Este cronograma é otimizado para a implementação prática e iterativa.

### Semana 1: Base Sólida e Navegação

*   **Dia 1-2:** Corrigir `main.dart` (`MultiProvider`) e garantir que todos os serviços estejam injetados corretamente. Testar `AdminManageUsersScreen` para confirmar que o erro do `Provider` foi resolvido.
*   **Dia 3-4:** Atualizar `AuthService` e o modelo `User` para armazenar e expor todas as informações de `role`, `clanId`, `federationId`, etc. Criar getters para `isAdmin`, `isClanLeader`, etc.
*   **Dia 5-7:** Implementar a `BottomNavigationBar` dinâmica em `HomeScreen` (ou `MainAppScreen`). Criar a `ExploreClansFederationsScreen` (tela pública de federações/clãs).

### Semana 2: Imersão em Federações e Clãs

*   **Dia 8-10:** Desenvolver `FederationDetailScreen` (Painel da Federação). Implementar a listagem de clãs membros e a navegação para `ClanDetailScreen`.
*   **Dia 11-14:** Desenvolver `ClanDetailScreen` (Painel do Clã). Mover o conteúdo da tela de Missões QRR para uma aba dentro de `ClanDetailScreen`. Ajustar a lógica da aba "Voz" para o ADM Master.

### Semana 3: Gerenciamento e Insta Clã

*   **Dia 15-17:** Refatorar `AdminDashboardScreen` para ser o hub central do ADM. Criar `AdminManageClansScreen` e `AdminManageFederationsScreen`. Implementar a navegação do ADM para `ClanDetailScreen` e `FederationDetailScreen` em modo de gerenciamento (`isAdminViewing: true`).
*   **Dia 18-21:** Implementar o modelo `InstaClanPost` e os métodos no `ClanService` (ou `InstaClanService`). Desenvolver a `InstaClanScreen` e integrá-la como uma aba em `ClanDetailScreen`.

### Semana 4: Refinamentos e Testes

*   **Dia 22-24:** Implementar a exibição de tags/bandeiras de clã/federação nas listas de usuários (ex: `AdminManageUsersScreen`, lista de membros do clã, chat global).
*   **Dia 25-28:** Testes completos de todas as funcionalidades, especialmente o controle de acesso para cada `role`. Ajustes de UI/UX para garantir a imersão e a clareza visual.

## 7. Considerações Finais

*   **Comunicação com Backend:** Certifique-se de que todas as chamadas de API estejam corretas e que os dados sejam mapeados para os modelos Flutter adequadamente.
*   **Reutilização de Componentes:** Aproveite ao máximo os widgets e componentes que você já possui (`clan_info_widget.dart`, `federations_panel_widget.dart`, etc.), adaptando-os para os novos layouts.
*   **Feedback Visual:** Adicione indicadores de carregamento, mensagens de erro e feedback visual para o usuário durante as operações de rede.
*   **Testes Iterativos:** Teste cada funcionalidade assim que a implementar, para identificar e corrigir problemas rapidamente.

Este plano é um roteiro detalhado. Siga-o passo a passo, e você transformará seu aplicativo em uma experiência organizada e imersiva. Boa sorte!

