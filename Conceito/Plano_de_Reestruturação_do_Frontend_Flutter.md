# Plano de Reestruturação do Frontend Flutter

Este documento detalha o plano para organizar e otimizar o frontend do seu aplicativo Flutter, garantindo uma experiência de usuário coesa e um controle de acesso robusto, refletindo a hierarquia de Federações (Países) e Clãs (Estados).

## 1. Correção da Base: `Provider` e Gerenciamento de Estado

O erro de `Provider` é crítico e precisa ser resolvido primeiro. Ele indica que os serviços (como `UserService`) não estão acessíveis onde são necessários. Além disso, o gerenciamento do estado do usuário (incluindo sua `role`) deve ser centralizado.

### 1.1. Injeção de Dependências com `MultiProvider`

**Problema:** `Could not find the correct Provider<UserService> above this AdminManageUsersScreen Widget`.

**Solução:** No seu arquivo `lib/main.dart`, envolva o `MaterialApp` com um `MultiProvider`. Isso garante que todos os serviços necessários estejam disponíveis para toda a árvore de widgets.

```dart
// lib/main.dart
import 'package:provider/provider.dart';
// Importe todos os seus serviços aqui
import 'package:lucasbeatsfederacao/services/user_service.dart';
import 'package:lucasbeatsfederacao/services/auth_service.dart';
import 'package:lucasbeatsfederacao/services/clan_service.dart';
import 'package:lucasbeatsfederacao/services/federation_service.dart';
// ... outros serviços

void main() {
  runApp(
    MultiProvider(
      providers: [
        // Certifique-se de que todos os seus serviços estejam listados aqui
        ChangeNotifierProvider(create: (_) => UserService()),
        ChangeNotifierProvider(create: (_) => AuthService()),
        ChangeNotifierProvider(create: (_) => ClanService(context.read<ApiService>(), context.read<AuthService>())),
        ChangeNotifierProvider(create: (_) => FederationService(context.read<ApiService>())),
        // ... adicione outros serviços que usam ChangeNotifierProvider
        // Para serviços que não precisam de notificação de mudança, use Provider.value
        // Provider<SocketService>.value(value: socketService),
      ],
      child: MyApp(), // Seu widget principal que contém o MaterialApp
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      // ... suas configurações de MaterialApp
      home: Consumer<AuthService>(
        builder: (context, authService, _) {
          // Lógica de redirecionamento baseada no status de autenticação
          if (authService.authStatus == AuthStatus.unknown) {
            return const SplashScreen();
          } else if (authService.authStatus == AuthStatus.authenticated) {
            return const HomeScreen(); // Ou a tela principal após login
          } else {
            return const LoginScreen();
          }
        },
      ),
    );
  }
}
```

### 1.2. Centralização do Estado do Usuário (`AuthService`)

**Ação:** Garanta que seu `AuthService` (ou `AuthProvider` como você já tem) armazene e forneça as informações cruciais do usuário após o login:

*   `userId`
*   `username`
*   `role` (ADM_MASTER, CLAN_LEADER, FEDERATION_LEADER, CLAN_MEMBER, USER)
*   `clanId` (se pertencer a um clã)
*   `federationId` (se pertencer a uma federação)

**Implementação:** Após a chamada de login bem-sucedida ao backend, o `AuthService` deve atualizar seu estado com esses dados. O backend já retorna a `role` e outras informações no payload do token ou em uma resposta de `/api/users/me`.

```dart
// Exemplo simplificado dentro de AuthService
class AuthService extends ChangeNotifier {
  User? _currentUser;
  AuthStatus _authStatus = AuthStatus.unknown;

  User? get currentUser => _currentUser;
  AuthStatus get authStatus => _authStatus;

  Future<void> login(String email, String password) async {
    // ... chamada à API de login
    // Supondo que a API retorna um objeto User com role, clanId, etc.
    final response = await apiService.post('/auth/login', {'email': email, 'password': password});
    _currentUser = User.fromJson(response.data); // Seu modelo de usuário
    _authStatus = AuthStatus.authenticated;
    notifyListeners();
  }

  bool get isAdmin => _currentUser?.role == 'ADM_MASTER';
  bool get isClanLeader => _currentUser?.role == 'CLAN_LEADER';
  // ... outros getters para roles
}
```

## 2. Controle de Acesso Baseado em Função (RBAC) e Navegação Dinâmica

O controle de acesso será implementado em três níveis: rotas, navegação principal e renderização de widgets.

### 2.1. Rotas Protegidas (Guards de Navegação)

**Conceito:** Impedir que usuários não autorizados acessem certas telas. Em Flutter, isso é feito verificando a `role` do usuário antes de construir a tela ou redirecionando-o.

**Implementação:** Use o `Consumer` ou `Provider.of` no widget raiz de cada rota protegida para verificar a `role` do usuário. Se não autorizado, redirecione para uma tela de acesso negado ou para a tela de login.

```dart
// Exemplo para uma rota de administração
class AdminDashboardScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final authService = Provider.of<AuthService>(context);

    if (!authService.isAdmin) {
      // Redireciona para uma tela de acesso negado ou home
      return AccessDeniedScreen(); 
    }
    // ... constrói o dashboard do admin
    return Scaffold(
      appBar: AppBar(title: Text('Painel do ADM Master')),
      body: // ... conteúdo do painel
    );
  }
}
```

### 2.2. Navegação Principal Dinâmica (`BottomNavigationBar`)

**Problema:** A aba "Missões QRR" está visível para todos.

**Solução:** A `BottomNavigationBar` deve ser construída dinamicamente com base na `role` do usuário logado.

```dart
// Exemplo dentro da sua HomeScreen ou widget que contém a BottomNavigationBar
class MainAppScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final authService = Provider.of<AuthService>(context);

    List<BottomNavigationBarItem> adminItems = [
      BottomNavigationBarItem(icon: Icon(Icons.dashboard), label: 'ADM'),
      // ... outros itens específicos do ADM
    ];

    List<BottomNavigationBarItem> clanMemberItems = [
      BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Início'),
      BottomNavigationBarItem(icon: Icon(Icons.group), label: 'Meu Clã'), // Nova aba
      BottomNavigationBarItem(icon: Icon(Icons.chat), label: 'Global'),
      BottomNavigationBarItem(icon: Icon(Icons.settings), label: 'Config'),
    ];

    List<BottomNavigationBarItem> defaultUserItems = [
      BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Início'),
      BottomNavigationBarItem(icon: Icon(Icons.explore), label: 'Explorar'), // Lista pública de clãs/federações
      BottomNavigationBarItem(icon: Icon(Icons.chat), label: 'Global'),
      BottomNavigationBarItem(icon: Icon(Icons.settings), label: 'Config'),
    ];

    List<BottomNavigationBarItem> currentItems;
    if (authService.isAdmin) {
      currentItems = adminItems;
    } else if (authService.currentUser?.clanId != null) {
      currentItems = clanMemberItems;
    } else {
      currentItems = defaultUserItems;
    }

    return Scaffold(
      body: // ... seu PageView ou IndexedStack para as telas
      bottomNavigationBar: BottomNavigationBar(
        items: currentItems,
        // ...
      ),
    );
  }
}
```

### 2.3. Renderização Condicional de Widgets

**Conceito:** Mostrar ou esconder elementos da UI (botões, campos, seções) com base na `role` do usuário.

**Implementação:** Use `if` statements ou `Visibility` widgets diretamente no código da UI.

```dart
// Exemplo em uma tela de gerenciamento de clã
class ClanManagementScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final authService = Provider.of<AuthService>(context);

    return Column(
      children: [
        Text('Detalhes do Clã'),
        if (authService.isClanLeader || authService.isAdmin) 
          ElevatedButton(
            onPressed: () { /* Editar clã */ },
            child: Text('Editar Clã'),
          ),
        if (authService.isAdmin) 
          ElevatedButton(
            onPressed: () { /* Excluir clã */ },
            child: Text('Excluir Clã'),
          ),
        // ... outros widgets
      ],
    );
  }
}
```

## 3. Arquitetura de Layouts e Navegação Imersiva (Países e Estados)

Esta é a chave para a experiência imersiva e organizada. Criaremos layouts dedicados para cada nível hierárquico.

### 3.1. `MainLayout` (Base do Aplicativo)

*   **Função:** Contém a lógica de autenticação e a `BottomNavigationBar` dinâmica. É o ponto de entrada principal após o splash screen.
*   **Conteúdo:** Um `PageView` ou `IndexedStack` que alterna entre as telas principais (Início, Explorar/Meu Clã, Global, Configurações).

### 3.2. `AdminLayout` (Painel do ADM Master)

*   **Função:** Interface centralizada para o `ADM_MASTER` gerenciar todo o sistema.
*   **Estrutura:** Pode ser um `Scaffold` com um `Drawer` (menu lateral) para navegação entre as diferentes seções de gerenciamento (Usuários, Clãs, Federações, Missões QRR, etc.).
*   **Acesso:** Apenas para `ADM_MASTER`.
*   **Correção do Erro de Usuários:** A tela `AdminManageUsersScreen` será uma das rotas dentro deste layout, e o `UserService` já estará disponível via `MultiProvider`.

### 3.3. `FederationLayout` (Painel da Federação - O País)

*   **Função:** Representa o "País". Exibe detalhes da federação e lista seus clãs membros.
*   **Estrutura:** Um `Scaffold` com `AppBar` mostrando o nome da federação, e um `TabBar` ou `ListView` para navegar entre:
    *   **Visão Geral:** Informações da federação, mural de notícias.
    *   **Clãs Membros:** Lista de clãs que pertencem a esta federação. Ao clicar em um clã, o usuário "entra" no `ClanLayout`.
    *   **Gerenciamento (para Líder de Federação/ADM):** Seções para adicionar/remover clãs, gerenciar aliados/inimigos, editar informações da federação.
*   **Acesso:** Membros da federação, Líder de Federação, ADM Master. Usuários comuns podem ver uma versão pública.

### 3.4. `ClanLayout` (Painel do Clã - O Estado)

*   **Função:** Representa o "Estado". O "espaço único" do clã, com funcionalidades exclusivas para seus membros.
*   **Estrutura:** Um `Scaffold` com `AppBar` mostrando o nome do clã, e um `TabBar` para navegar entre:
    *   **Mural/Home:** Notícias, avisos do líder.
    *   **Membros:** Lista de membros do clã.
    *   **Missões QRR:** **Esta é a nova localização da tela de missões.** Apenas membros do clã (e ADM) podem ver e interagir com as missões aqui.
    *   **Insta Clã:** A nova funcionalidade de feed de atividades/galeria do clã.
    *   **Voz:** O canal de voz do clã. A lógica aqui deve permitir que o ADM Master acesse mesmo sem ser membro do clã.
    *   **Configurações (para Líder/Sublíder/ADM):** Edição de informações do clã, gerenciamento de membros.
*   **Acesso:** Membros do clã, Líder de Clã, Sublíder de Clã, ADM Master.

## 4. Implementação do "Insta Clã" (Urgente)

### 4.1. Backend (Verificar/Implementar APIs)

*   **`POST /api/clans/:clanId/instaclan/posts`:** Para criar uma postagem (texto, imagem, etc.).
*   **`GET /api/clans/:clanId/instaclan/posts`:** Para listar as postagens do clã.
*   **`DELETE /api/clans/:clanId/instaclan/posts/:postId`:** Para excluir uma postagem.

### 4.2. Frontend (Flutter)

1.  **Crie o Modelo de Dados:** Defina uma classe `InstaClanPost` em `lib/models/` para mapear as respostas da API.
2.  **Crie o Serviço:** Adicione métodos ao seu `ClanService` ou crie um `InstaClanService` para interagir com as novas APIs.
3.  **Crie a Tela/Widget:** Desenvolva `lib/screens/instaclan_screen.dart` (ou `lib/widgets/instaclan_feed_widget.dart` como já existe, mas precisa ser integrado).
    *   Use um `ListView.builder` para exibir as postagens.
    *   Implemente um formulário para criar novas postagens.
    *   Adicione botões de exclusão condicionalmente visíveis (autor da postagem, Líder/Sublíder de Clã, ADM Master).
4.  **Integre no `ClanLayout`:** Adicione `InstaClanScreen` como uma nova aba no `TabBar` do `ClanLayout`.

## 5. Correções Específicas para o ADM Master

### 5.1. Acesso Irrestrito a Conteúdo de Clã/Federação

**Problema:** ADM Master sendo impedido de ver conteúdo de clã/federação.

**Solução:** Em todas as verificações de permissão para visualização de conteúdo (ex: na aba "Voz", na tela de Missões QRR), adicione uma condição `|| authService.isAdmin`.

```dart
// Exemplo na lógica da aba 


Voz ou Missões QRR)
```dart
// Exemplo: dentro do widget que exibe o conteúdo do clã
final authService = Provider.of<AuthService>(context);
final bool isMemberOfThisClan = authService.currentUser?.clanId == currentClan.id;

if (isMemberOfThisClan || authService.isAdmin) {
  // Mostra o conteúdo do canal de voz ou as missões QRR
  return ClanVoiceContent();
} else {
  return Text("Você não tem permissão para ver este conteúdo.");
}
```

### 5.2. Ações de Gerenciamento do ADM Master

**Ação:** No painel do ADM Master, ao listar clãs e federações, adicione botões ou opções para "Gerenciar" ou "Visualizar como ADM". Ao clicar, navegue para o `ClanLayout` ou `FederationLayout` correspondente, mas com um parâmetro indicando que é o ADM Master visualizando.

```dart
// Exemplo de navegação do ADM para um clã específico
Navigator.push(context, MaterialPageRoute(builder: (context) => ClanManagementScreen(
  clanId: selectedClan.id,
  isAdminViewing: true, // Parâmetro para indicar que é o ADM
)));
```

No `ClanLayout` e `FederationLayout`, a presença de `isAdminViewing: true` anularia as verificações de `role` para exibir todas as opções de gerenciamento.

## 6. Cronograma Sugerido (Etapas)

Este cronograma é uma sugestão e pode ser ajustado conforme a disponibilidade e complexidade de cada etapa.

1.  **Semana 1: Base e Autenticação**
    *   Corrigir o `MultiProvider` em `main.dart` para todos os serviços.
    *   Garantir que o `AuthService` armazene corretamente a `role` e IDs de clã/federação do usuário.
    *   Implementar a lógica de redirecionamento inicial (`SplashScreen`, `LoginScreen`, `HomeScreen`).

2.  **Semana 2: RBAC e Navegação Principal**
    *   Implementar a `BottomNavigationBar` dinâmica baseada na `role` do usuário.
    *   Criar os 


guarda-rotas para as telas de ADM e de Clã/Federação.
    *   Começar a refatorar a `HomeScreen` para ser mais genérica e redirecionar para os layouts específicos.

3.  **Semana 3-4: Layouts e Experiência Imersiva**
    *   Desenvolver o `AdminLayout` com suas sub-telas de gerenciamento (Usuários, Clãs, Federações).
    *   Desenvolver o `FederationLayout` com a lista de clãs membros e as funcionalidades de gerenciamento para Líder de Federação/ADM.
    *   Desenvolver o `ClanLayout` com suas abas internas (Mural, Membros, Missões QRR, Insta Clã, Voz, Configurações).
    *   **Mover a tela de Missões QRR para dentro do `ClanLayout` e remover da navegação principal.**

4.  **Semana 5: Implementação do "Insta Clã" e Refinamentos**
    *   Implementar as APIs do backend para o "Insta Clã" (se ainda não existirem).
    *   Desenvolver a tela `InstaClanScreen` e integrá-la como uma aba no `ClanLayout`.
    *   Refinar a lógica de exibição condicional de widgets em todas as telas.
    *   Testar exaustivamente as permissões e a navegação para todas as `roles`.

## 7. Próximos Passos e Considerações Finais

*   **Comunicação com o Backend:** Certifique-se de que todas as chamadas de API do frontend estejam usando o `ApiService` e que ele esteja configurado para lidar com autenticação (enviando o token JWT).
*   **Modelos de Dados:** Mantenha seus modelos de dados (classes Dart para `User`, `Clan`, `Federation`, `QRR`, etc.) atualizados e alinhados com os schemas do backend.
*   **Testes:** Realize testes rigorosos para cada `role` de usuário, garantindo que as permissões e a navegação funcionem conforme o esperado.
*   **UI/UX:** Ao construir as telas, sempre tenha em mente a analogia de "Países e Estados" para criar uma experiência visual e interativa que reforce a hierarquia e a imersão.

Este plano fornece um roteiro claro para a reestruturação do seu frontend. Comece pela correção da base (Providers e `AuthService`) e avance passo a passo. A organização e a clareza na arquitetura trarão grandes benefícios a longo prazo para o seu projeto.

