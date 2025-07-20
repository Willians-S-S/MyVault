Excelente pergunta! Essa é uma das preocupações de segurança mais importantes em qualquer aplicação multiusuário: a **autorização**, ou seja, garantir que um usuário autenticado só possa realizar as ações que lhe são permitidas, como modificar apenas os seus próprios dados.

O código fornecido, como está, é vulnerável a essa falha. Se você criasse um endpoint para atualização de usuário (ex: `PUT /users/{id}`), um usuário mal-intencionado poderia descobrir o ID de outro usuário e enviar uma requisição para modificar os dados dele.

Existem duas abordagens principais e complementares no Spring Boot para resolver isso:

1. **Verificação Programática na Camada de Serviço (Service Layer)**
    
2. **Segurança Declarativa com Anotações (`@PreAuthorize`)**
    

Vamos detalhar cada uma.

---

### Abordagem 1: Verificação Programática na Camada de Serviço

Esta é a forma mais direta e explícita. A lógica de verificação de permissão é adicionada diretamente no método do serviço que realiza a operação.

**Princípio:** Antes de executar a atualização, você deve verificar se o nome de usuário (ou ID) do usuário atualmente logado é o mesmo do usuário que está prestes a ser modificado.

**Como Implementar:**

Vamos imaginar que você tenha um `UserService` com um método `updateUser`.

1. **Obter o Usuário Autenticado:** Você pode obter os detalhes do usuário logado a partir do `SecurityContextHolder`.
    
2. **Carregar o Usuário a ser Modificado:** Busque o usuário do banco de dados pelo ID fornecido na requisição.
    
3. **Comparar e Decidir:** Compare o identificador do usuário logado com o identificador do usuário a ser modificado. Se forem diferentes, lance uma exceção de acesso negado.
    

**Exemplo de Código (`UserService.java`):**

Supondo que você crie um método `updateUser` no seu `UserService`:

Java

```
// Dentro da classe UserService

import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.Authentication;
import org.springframework.security.access.AccessDeniedException; // Importante

// ... outras importações

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    // ... outros métodos como addUser ...

    public UserResponseDto updateUser(Long userIdToUpdate, UserUpdateRequestDto updateData) {
        // 1. Obter o nome do usuário atualmente autenticado
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        String currentPrincipalName = authentication.getName();

        // 2. Buscar o usuário que se deseja atualizar no banco
        User userToUpdate = userRepository.findById(userIdToUpdate)
                .orElseThrow(() -> new NotFoundException("Usuário com ID " + userIdToUpdate + " não encontrado."));

        // 3. A VERIFICAÇÃO CRÍTICA DE PROPRIEDADE
        // Compara o email (username) do usuário logado com o email do usuário a ser modificado.
        if (!userToUpdate.getEmail().equals(currentPrincipalName)) {
            // Se não forem os mesmos, o acesso é negado.
            throw new AccessDeniedException("Acesso negado. Você só pode modificar seus próprios dados.");
        }

        // 4. Se a verificação passar, prossiga com a lógica de atualização
        userToUpdate.setName(updateData.getName());
        // ... atualizar outros campos permitidos
        
        User updatedUser = userRepository.save(userToUpdate);
        
        return new UserResponseDto(updatedUser); // Exemplo de retorno
    }
}
```

- **Vantagens:** A lógica é muito explícita e fácil de depurar. Fica claro, ao ler o método, qual é a regra de segurança.
    
- **Desvantagens:** Mistura a lógica de negócio com a lógica de segurança, o que pode "poluir" a classe de serviço.
    

---

### Abordagem 2: Segurança Declarativa com `@PreAuthorize` (Recomendado)

Esta é a maneira mais limpa e poderosa, aproveitando todo o potencial do Spring Security. Você "declara" as regras de segurança diretamente nos métodos, e o Spring cuida de aplicá-las antes que o método seja executado.

**Princípio:** Usar a anotação `@PreAuthorize` com Spring Expression Language (SpEL) para definir a condição de acesso.

**Como Implementar:**

**Passo 1: Habilitar a Segurança em Nível de Método**

Primeiro, você precisa habilitar essa funcionalidade na sua classe `SecurityConfig`.

Java

```
// Em SecurityConfig.java

import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity // <-- ADICIONE ESTA ANOTAÇÃO!
public class SecurityConfig {
    // ... todo o resto do seu código ...
}
```

**Passo 2: Aplicar `@PreAuthorize` no Endpoint ou no Serviço**

Agora você pode usar a anotação. A expressão SpEL dentro da anotação tem acesso direto ao objeto `authentication`.

- `authentication.name`: Retorna o nome de usuário (username) do principal.
    
- `hasAuthority('ROLE_ADMIN')`: Verifica se o usuário tem um determinado papel/permissão.
    

**Exemplo de Código (Abordagem Híbrida com Admin):**

Vamos criar um endpoint no `UserController` para a atualização. A regra será: "O usuário pode atualizar os dados se ele for o dono da conta **OU** se ele for um administrador (`ROLE_ADMIN`)".

Primeiro, vamos ajustar a classe `UserAuthenticated` para retornar papéis reais.

Java

```
// Em UserAuthenticated.java
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {
    // Em vez de hardcoded, deveria vir do seu objeto User
    // Exemplo: user.getRoles().stream().map(role -> new SimpleGrantedAuthority(role.getName())).collect(Collectors.toList());
    
    // Por enquanto, para o exemplo, vamos adicionar um papel de usuário comum
    return List.of(() -> "ROLE_USER"); 
}
```

Agora, o `UserController` com o endpoint de atualização:

Java

```
// Em um novo UserController.java ou no seu AuthenticationController
@RestController
@RequestMapping("/users")
public class UserController {

    @Autowired
    private UserService userService;

    // ... outros métodos ...
    
    // O endpoint recebe o ID do usuário como uma variável de caminho
    @PutMapping("/{id}")
    @PreAuthorize("hasAuthority('ROLE_ADMIN') or @userService.isOwner(#id)") // Exemplo avançado, vamos simplificar
    public UserResponseDto updateUser(...) { /* ... */ }
}
```

A expressão acima é um pouco complexa. Uma forma **muito mais comum e limpa** é basear-se no nome de usuário, que é um identificador único.

**Exemplo de Código Melhorado (a melhor prática):**

Java

```
// Em um UserController.java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    // Endpoint para buscar dados de um usuário. Só o próprio usuário ou um admin pode ver.
    @GetMapping("/{username}")
    @PreAuthorize("hasAuthority('ROLE_ADMIN') or #username == authentication.name")
    public ResponseEntity<UserResponseDto> getUserByUsername(@PathVariable String username) {
        // ... lógica para buscar e retornar o usuário
    }

    // Endpoint para atualizar dados. Só o próprio usuário ou um admin pode atualizar.
    @PutMapping("/{username}")
    @PreAuthorize("hasAuthority('ROLE_ADMIN') or #username == authentication.name")
    public ResponseEntity<UserResponseDto> updateUser(
            @PathVariable String username, 
            @RequestBody UserUpdateRequestDto updateData) 
    {
        // Como a verificação já foi feita pelo @PreAuthorize,
        // o serviço pode focar apenas na lógica de negócio.
        UserResponseDto updatedUser = userService.updateUserByUsername(username, updateData);
        return ResponseEntity.ok(updatedUser);
    }
}
```

**Analisando a Expressão Mágica: `@PreAuthorize("hasAuthority('ROLE_ADMIN') or #username == authentication.name")`**

- `hasAuthority('ROLE_ADMIN')`: Retorna `true` se o usuário logado tiver a permissão "ROLE_ADMIN".
    
- `or`: Operador lógico "OU".
    
- `#username`: Refere-se ao parâmetro do método que tem o nome `username`.
    
- `==`: Operador de comparação.
    
- `authentication.name`: Pega o `username` (neste caso, o e-mail) do usuário que está autenticado na sessão atual.
    

A expressão inteira significa: "Permita a execução deste método apenas se o usuário autenticado tiver o papel de 'ROLE_ADMIN' **ou** se o `username` passado na URL for igual ao `username` do usuário autenticado."

- **Vantagens:** Mantém a lógica de segurança separada da lógica de negócio. É extremamente poderosa e flexível. As regras ficam centralizadas e fáceis de auditar.
    
- **Desvantagens:** Requer aprendizado da SpEL. A magia pode esconder a complexidade para iniciantes.
    

### Resumo e Conclusão

|Característica|Abordagem 1: Programática (Service)|Abordagem 2: Declarativa (`@PreAuthorize`)|
|---|---|---|
|**Local da Lógica**|Dentro do método de serviço|Na anotação, fora do corpo do método|
|**Clareza**|Muito explícito, fácil de seguir o fluxo|Limpo, mas requer conhecimento de SpEL|
|**Separação de Interesses**|Mistura segurança com negócio|Excelente separação|
|**Manutenção**|A lógica de segurança pode se repetir em vários métodos|Regras reutilizáveis e fáceis de modificar em um só lugar|
|**Flexibilidade**|Limitada à lógica Java padrão|Extremamente flexível com SpEL (chamar beans, etc.)|
|**Recomendação**|Bom para regras muito complexas e dinâmicas que não se encaixam na SpEL|**Preferível na maioria dos casos** por ser mais limpo e padrão do Spring|

Para o seu projeto, a **Abordagem 2 com `@PreAuthorize` é a mais recomendada**. Ela representa o "jeito Spring" de lidar com autorização, resultando em um código mais limpo, seguro e de fácil manutenção.