A estrutura geral implementa um sistema de autenticação e registro de usuários usando **Spring Security** com **JSON Web Tokens (JWT)**. A autenticação inicial é feita via **HTTP Basic Auth** para obter um token, e as requisições subsequentes são autenticadas com esse token (Bearer Token).

---

### `AuthenticationController.java`

Esta classe é a porta de entrada da API para as funcionalidades de autenticação e registro.

Java

```
package com.willians.ListifyShop.controller;

import //...

// Anotações da Classe
@RestController // (1)
@RequestMapping("/authentication") // (2)
public class AuthenticationController {

    // Injeção de Dependências
    @Autowired // (3)
    private AuthenticationService authenticationService;

    @Autowired
    private UserService userService;

    // Endpoint de Autenticação
    @GetMapping("/auth") // (4)
    public String authenticate(Authentication authentication){ // (5)
        return authenticationService.autheticate(authentication); // (6)
    }

    // Endpoint de Registro de Usuário
    @PostMapping(consumes = MediaType.MULTIPART_FORM_DATA_VALUE) // (7)
    public UserResponseDto addUser(
            @Valid @RequestPart("user") UserRequestDto userRequest, // (8)
            @RequestPart(value = "image", required = false) MultipartFile image // (9)
    ){
        return this.userService.addUser(userRequest, image); // (10)
    }
}
```

1. `@RestController`: Combina `@Controller` e `@ResponseBody`. Indica que esta classe é um controlador REST, e os retornos dos seus métodos serão automaticamente serializados para JSON e escritos no corpo da resposta HTTP.
    
2. `@RequestMapping("/authentication")`: Define o prefixo do URL para todos os endpoints dentro desta classe. Ou seja, todos os endpoints começarão com `/authentication`.
    
3. `@Autowired`: Realiza a injeção de dependência. O Spring irá procurar por um `Bean` do tipo `AuthenticationService` e `UserService` e injetá-los automaticamente nesta classe.
    
4. `@GetMapping("/auth")`: Mapeia requisições HTTP `GET` para `/authentication/auth`.
    
5. `public String authenticate(Authentication authentication)`: Este método lida com a autenticação. O parâmetro `Authentication` é populado pelo Spring Security **após** o usuário ser autenticado com sucesso (neste caso, via HTTP Basic, conforme configurado no `SecurityConfig`). Ele contém os detalhes do usuário autenticado (username, authorities).
    
6. `return authenticationService.autheticate(authentication)`: Delega a lógica de geração do token para a classe `AuthenticationService`, passando o objeto `Authentication` que contém os dados do usuário.
    
7. `@PostMapping(consumes = MediaType.MULTIPART_FORM_DATA_VALUE)`: Mapeia requisições HTTP `POST` para `/authentication`. O `consumes` indica que este endpoint espera dados no formato `multipart/form-data`, que é usado para enviar arquivos junto com dados de formulário.
    
8. `@Valid @RequestPart("user") UserRequestDto userRequest`: Extrai a parte `user` da requisição multipart e a converte para um objeto `UserRequestDto`. `@Valid` aciona as validações definidas no DTO (ex: `@NotBlank`, `@Email`).
    
9. `@RequestPart(value = "image", required = false)`: Extrai a parte `image` da requisição, que é um arquivo (`MultipartFile`). É opcional (`required = false`).
    
10. `return this.userService.addUser(userRequest, image)`: Chama o `UserService` para processar a lógica de criação do novo usuário e o upload da imagem, retornando um `UserResponseDto`.
    

---

### `AuthenticationService.java`

Esta classe de serviço é responsável pela lógica de autenticação, atuando como uma ponte entre o `Controller` e o `JwtService`.

Java

```
package com.willians.ListifyShop.security.authentication;

import //...

@Service // (1)
public class AuthenticationService {

    @Autowired // (2)
    private UserRepository userRepository;

    JwtService jwtService;

    public AuthenticationService(JwtService jwtService){ // (3)
        this.jwtService = jwtService;
    }

    public String autheticate(Authentication authentication){ // (4)
        return jwtService.genereteToken(authentication);
    }
}
```

1. `@Service`: Anotação que marca a classe como um componente de serviço na camada de negócio. É um estereótipo do `@Component`, o que a torna elegível para ser detectada pelo `component-scanning` do Spring e gerenciada como um `Bean`.
    
2. `@Autowired private UserRepository userRepository`: Injeta o repositório de usuários. **Observação**: A injeção está sendo feita no campo, mas a outra (`JwtService`) é feita via construtor. O ideal é manter a consistência (veja a seção de melhores práticas).
    
3. `public AuthenticationService(JwtService jwtService)`: Construtor que recebe uma instância de `JwtService`. Esta é a **injeção de dependência via construtor**, considerada uma melhor prática.
    
4. `public String autheticate(...)`: Método que recebe o objeto `Authentication` e simplesmente repassa a responsabilidade de gerar o token JWT para o `JwtService`.
    

---

### `JwtService.java`

Componente dedicado exclusivamente a criar e assinar os tokens JWT.

Java

```
package com.willians.ListifyShop.security.authentication;

import //...

@Service
public class JwtService {
    private final JwtEncoder encoder; // (1)

    public JwtService(JwtEncoder encoder){ // (2)
        this.encoder = encoder;
    }

    public String genereteToken(Authentication authentication){
        Instant now = Instant.now(); // (3)
        long expire = 3600L; // (4)

        // (5) Coleta as "authorities" (permissões/papéis) do usuário
        String scopes = authentication.getAuthorities()
                .stream().map(GrantedAuthority::getAuthority)
                .collect(Collectors.joining(" "));

        // (6) Monta o payload (claims) do JWT
        var claims = JwtClaimsSet.builder()
                .issuer("spring-security-jwt") // Emissor do token
                .issuedAt(now) // Data e hora de emissão
                .expiresAt(now.plusSeconds(expire)) // Data e hora de expiração
                .subject(authentication.getName()) // O "dono" do token (username)
                .claim("scope", scopes) // Claim customizada com as permissões
                .build();

        // (7) Codifica e assina as claims, retornando o valor do token como String
        return encoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
    }
}
```

1. `private final JwtEncoder encoder`: Campo final que armazena o codificador de JWT. O `JwtEncoder` é uma interface do Spring Security que abstrai a criação de JWTs.
    
2. `public JwtService(JwtEncoder encoder)`: Recebe via injeção de dependência o `Bean` do `JwtEncoder` que foi configurado no `SecurityConfig`.
    
3. `Instant now = Instant.now()`: Pega o timestamp atual.
    
4. `long expire = 3600L`: Define o tempo de expiração do token em segundos (aqui, 1 hora).
    
5. `String scopes = ...`: Obtém as permissões do usuário (ex: "ROLE_ADMIN", "READ") do objeto `Authentication`, transforma-as em uma `String` separada por espaços e a armazena. Isso é crucial para o controle de acesso baseado em papéis/permissões.
    
6. `var claims = JwtClaimsSet.builder()...`: Cria o conjunto de "claims" (informações) que comporão o payload do JWT. Claims padrão como `iss` (issuer), `iat` (issued at), `exp` (expires at) e `sub` (subject) são definidas aqui, além de uma claim customizada `scope`.
    
7. `encoder.encode(...)`: Usa o `JwtEncoder` para assinar as `claims` com a chave privada (configurada no `SecurityConfig`) e gerar o token final.
    

---

### `SecurityConfig.java`

Esta é a classe central da configuração de segurança. Ela define como o Spring Security deve se comportar.

Java

```
package com.willians.ListifyShop.security.config;

import //...

@Configuration // (1)
@EnableWebSecurity // (2)
public class SecurityConfig {

    @Value("${jwt.public.key}") // (3)
    private RSAPublicKey key;
    @Value("${jwt.private.key}")
    private RSAPrivateKey priv;
    
    // (4) O Bean que define a cadeia de filtros de segurança principal
    @Bean
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception{
        http.csrf(csrf -> csrf.disable()) // (5)
                .authorizeHttpRequests(
                        auth -> auth.requestMatchers("/authentication/**").permitAll() // (6)
                                .anyRequest().authenticated() // (7)
                ).httpBasic(Customizer.withDefaults()) // (8)
                .oauth2ResourceServer(
                        conf -> conf.jwt(Customizer.withDefaults()) // (9)
                );
        return http.build();
    }
    
    // (10) Define como decodificar (validar) um JWT
    @Bean
    JwtDecoder decoder(){
        return NimbusJwtDecoder.withPublicKey(key).build();
    }
    
    // (11) Define como codificar (criar) um JWT
    @Bean
    JwtEncoder jwtEncoder(){
        var jwk = new RSAKey.Builder(key).privateKey(priv).build();
        var jwks = new ImmutableJWKSet<>(new JWKSet(jwk));
        return new NimbusJwtEncoder(jwks);
    }
    
    // (12) Define o codificador de senhas
    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

1. `@Configuration`: Indica que a classe contém definições de `Beans` para o contexto do Spring.
    
2. `@EnableWebSecurity`: Habilita o suporte à segurança web do Spring Security.
    
3. `@Value(...)`: Injeta valores do arquivo `application.properties`. Aqui, está injetando as chaves RSA pública e privada, que são usadas para assinar e validar os JWTs.
    
4. `@Bean SecurityFilterChain`: Este é o `Bean` mais importante. Ele define a cadeia de filtros de segurança que interceptará todas as requisições.
    
5. `csrf(csrf -> csrf.disable())`: Desabilita a proteção contra CSRF (Cross-Site Request Forgery). Isso é comum em APIs REST stateless, onde a autenticação baseada em token já oferece uma camada de proteção.
    
6. `.requestMatchers("/authentication/**").permitAll()`: Permite que todas as requisições para URLs que começam com `/authentication/` sejam acessadas sem autenticação. Isso é necessário para que os usuários possam se registrar e fazer login.
    
7. `.anyRequest().authenticated()`: Exige que qualquer outra requisição (`anyRequest`) seja autenticada.
    
8. `.httpBasic(...)`: Habilita a autenticação HTTP Basic. O navegador (ou cliente de API) enviará um cabeçalho `Authorization: Basic <base64-encoded-username:password>`. O Spring usará isso para autenticar o usuário no endpoint `/authentication/auth`.
    
9. `.oauth2ResourceServer(...).jwt(...)`: Configura a aplicação como um "Resource Server" que protege seus recursos usando JWTs. Ele automaticamente adiciona um filtro que intercepta requisições com o cabeçalho `Authorization: Bearer <token>`, valida o token usando o `JwtDecoder`, e estabelece o contexto de segurança.
    
10. `@Bean JwtDecoder`: Cria um `Bean` que sabe como validar um JWT. Ele é configurado com a **chave pública** para verificar a assinatura do token.
    
11. `@Bean JwtEncoder`: Cria um `Bean` que sabe como criar um JWT. Ele é configurado com o par de chaves (pública e privada) para poder **assinar** o token.
    
12. `@Bean PasswordEncoder`: Define `BCryptPasswordEncoder` como o `Bean` para codificar senhas. Toda vez que uma senha precisar ser verificada ou salva, este `Bean` será usado.
    

---

### `UserDetailsServiceImpl.java` & `UserAuthenticated.java`

Essas duas classes trabalham juntas para integrar o modelo de usuário da sua aplicação (`User`) com o modelo de segurança do Spring (`UserDetails`).

#### `UserDetailsServiceImpl.java`

Java

```
package com.willians.ListifyShop.security.userdetails;

import //...

@Service
public class UserDetailsServiceImpl implements UserDetailsService { // (1)

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException { // (2)
        return userRepository.findByEmail(username) // (3)
                .map(UserAuthenticated::new) // (4)
                .orElseThrow(() -> new NotFoundException("Usuário não encotrado.")); // (5)
    }
}
```

1. `implements UserDetailsService`: Implementa a interface `UserDetailsService`, que tem um único método. O Spring Security usa esta implementação para carregar os detalhes de um usuário a partir de uma fonte de dados (neste caso, o banco de dados via `UserRepository`).
    
2. `loadUserByUsername(String username)`: Este método é chamado pelo Spring Security (especificamente pelo `DaoAuthenticationProvider` por trás do `httpBasic`) durante o processo de autenticação para buscar um usuário pelo seu nome de usuário (que aqui é o e-mail).
    
3. `userRepository.findByEmail(username)`: Busca o usuário no banco de dados pelo e-mail.
    
4. `.map(UserAuthenticated::new)`: Se o usuário for encontrado, ele o "embrulha" em uma instância de `UserAuthenticated`. Este é o padrão de adaptação (Adapter Pattern).
    
5. `.orElseThrow(...)`: Se o usuário não for encontrado, lança uma exceção.
    

#### `UserAuthenticated.java`

Java

```
package com.willians.ListifyShop.security.userdetails;

//...
public class UserAuthenticated implements UserDetails { // (1)

    private final User user;

    public UserAuthenticated(User user){
        this.user = user;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() { // (2)
        return List.of(()->"read");
    }

    @Override
    public String getPassword() { // (3)
        return user.getPassword();
    }

    @Override
    public String getUsername() { // (4)
        return user.getEmail();
    }

    // Métodos de status da conta (5)
    @Override public boolean isAccountNonExpired() { return true; }
    @Override public boolean isAccountNonLocked() { return true; }
    @Override public boolean isCredentialsNonExpired() { return true; }
    @Override public boolean isEnabled() { return true; }
}
```

1. `implements UserDetails`: Implementa a interface `UserDetails` do Spring Security. Esta classe atua como um **adaptador** entre a sua entidade `User` e o que o Spring Security espera.
    
2. `getAuthorities()`: Retorna as permissões/papéis do usuário. **Aqui está hardcoded para `"read"`**, mas em uma aplicação real, isso viria de `user.getRoles()`.
    
3. `getPassword()`: Retorna a senha (já codificada) do usuário.
    
4. `getUsername()`: Retorna o nome de usuário (neste caso, o e-mail).
    
5. `isAccountNonExpired`, `isAccountNonLocked`, etc.: Retornam o status da conta. Aqui estão hardcoded para `true`, mas poderiam verificar campos booleanos na sua entidade `User` (ex: `user.isAtivo()`).
    

---

### Como os Componentes se Relacionam (Fluxo de Autenticação)

1. **Login para obter Token (HTTP Basic):**
    
    - O cliente envia uma requisição `GET /authentication/auth` com o cabeçalho `Authorization: Basic dXNlcjpwYXNzd29yZA==`.
        
    - O `SecurityFilterChain` (`SecurityConfig`) intercepta a requisição. O filtro `httpBasic` é acionado.
        
    - O Spring Security chama `UserDetailsServiceImpl.loadUserByUsername("user")` para buscar o usuário no banco.
        
    - O serviço retorna um `UserAuthenticated` com os dados do usuário, incluindo a senha _hash_.
        
    - O Spring Security compara o hash da senha do banco com a senha fornecida pelo cliente (usando o `PasswordEncoder`).
        
    - Se as senhas corresponderem, a autenticação é bem-sucedida. O Spring cria um objeto `Authentication`.
        
    - A requisição finalmente chega ao `AuthenticationController.authenticate(authentication)`.
        
    - O controller chama `AuthenticationService`, que chama `JwtService` para gerar um token JWT assinado com a chave privada.
        
    - O token JWT é retornado como uma `String` na resposta.
        
2. **Acesso a um Recurso Protegido (Bearer Token):**
    
    - O cliente envia uma requisição (ex: `GET /api/products`) com o cabeçalho `Authorization: Bearer <token_jwt>`.
        
    - O `SecurityFilterChain` intercepta a requisição. O filtro `oauth2ResourceServer` é acionado.
        
    - Este filtro usa o `JwtDecoder` (configurado com a chave pública) para validar o token: verifica a assinatura, a data de expiração e o emissor.
        
    - Se o token for válido, ele extrai as `claims` (subject, scope) e cria um objeto `Authentication`.
        
    - A requisição prossegue para o controller do recurso solicitado (ex: `ProductController`), que agora pode acessar os detalhes do usuário autenticado se precisar.
        
    - Se o token for inválido, a requisição é rejeitada com um erro `401 Unauthorized`.
        

---

### Possíveis Variações e Melhorias

#### Variações

- **Autenticação com Login/Senha no Corpo da Requisição:** Em vez de `httpBasic`, você poderia criar um endpoint `POST /login` que recebe um DTO com `username` e `password`. Nesse caso, você injetaria o `AuthenticationManager` do Spring e o chamaria manualmente.
    
- **JWT com Chave Simétrica (HMAC):** Em vez de um par de chaves RSA (assimétrica), você pode usar um único segredo (chave simétrica) com o algoritmo HMAC-SHA. A configuração no `SecurityConfig` para o encoder/decoder seria mais simples, usando `SecretKeySpec`. É menos seguro se o segredo vazar, pois a mesma chave que assina também valida.
    
- **Biblioteca `jjwt`:** Em vez de usar as classes `NimbusJwt` nativas do Spring, muitos usam a biblioteca `io.jsonwebtoken:jjwt` por sua API fluente e facilidade de uso para criar e validar tokens.
    

#### Melhores Práticas de Segurança e Organização

- **Injeção de Dependência via Construtor:** Use a injeção via construtor (como feito com `JwtService`) em vez de `@Autowired` em campos. Isso torna as dependências explícitas, as classes mais fáceis de testar e os campos podem ser declarados como `final`.
    
- **Armazenamento de Chaves:** **Nunca** armazene chaves, senhas ou segredos diretamente no `application.properties` em um repositório Git. Use variáveis de ambiente ou, idealmente, um serviço de gerenciamento de segredos como HashiCorp Vault, AWS Secrets Manager ou Azure Key Vault.
    
- **Refresh Tokens:** Tokens JWT de curta duração (5-15 minutos) são mais seguros. Para manter o usuário logado, implemente um mecanismo de "refresh token". Este é um token de longa duração, opaco e armazenado de forma segura, que é usado para obter um novo JWT de acesso sem que o usuário precise digitar a senha novamente.
    
- **Permissões Granulares:** Em `UserAuthenticated`, as `authorities` estão fixas (`"read"`). O ideal é que elas venham da entidade `User`, que teria uma relação com uma entidade `Role` ou `Permission`.
    
- **Tratamento de Exceções:** Use `@ControllerAdvice` para criar um handler de exceções global, que pode capturar `NotFoundException`, `AuthenticationException`, etc., e retornar respostas de erro HTTP padronizadas e limpas.
    
- **Configuração de Expiração:** O tempo de expiração do JWT está fixo no código (`3600L`). Mova isso para o `application.properties` para que possa ser configurado por ambiente.

## Implementar autorização
[[Autorização java]]

## Referências:
https://evoila.com/blog/spring-boot-3-jwt-authentication-leveraging-spring-securitys-support/
https://medium.com/@felipeacelinoo/protegendo-sua-api-rest-com-spring-security-e-autenticando-usu%C3%A1rios-com-token-jwt-em-uma-aplica%C3%A7%C3%A3o-d70e5b0331f9
https://www.youtube.com/watch?v=kEJ8a1w4a2Q
https://g.co/gemini/share/e8f787d3845e
https://www.geeksforgeeks.org/springboot/spring-boot-3-0-jwt-authentication-with-spring-security-using-mysql-database/