Teremos que cria uma rota que recebe a imagem

```java
@PostMapping(value = "/image")
publica ResponseEntity<Void> uploadImg(@ResquestPart("image") MultipartFile image){
	try {  
     if (!image.getOriginalFilename().endsWith(".png") && !image.getOriginalFilename().endsWith(".jpg")){  
         throw new RuntimeException("O arquivo enviado não é uma imagem png ou jpg");  
     }  
     String fileName = UUID.randomUUID() + image.getOriginalFilename();  
     Path uploadPath = Paths.get(path);  
     Path saveLocal = uploadPath.resolve(fileName);  
  
     image.transferTo(saveLocal);  
     return ResponseEntity.ok().build();  
} catch (IOException e) {  
    throw new RuntimeException(e);  
}
}
```

---

## Documentação da Função de Upload de Imagem

Este documento explica o funcionamento do método `uploadImg`, que é responsável por receber e armazenar arquivos de imagem.

### Visão Geral

O método `uploadImg` é um endpoint Spring Boot que permite o upload de imagens nos formatos PNG e JPG. Ele recebe a imagem como parte da requisição HTTP, valida o formato do arquivo, gera um nome único para a imagem e a salva em um diretório local especificado pela variável `path`.

### Parâmetros

- **`@RequestPart("image") MultipartFile image`**: Este parâmetro representa o arquivo de imagem enviado na requisição.
    
    - **`image`**: É o objeto que contém os dados da imagem, incluindo o nome original do arquivo, o tipo de conteúdo e o próprio conteúdo binário da imagem.
        
    - **`@RequestPart("image")`**: Anotação que indica que o arquivo `image` será enviado como uma parte (`part`) de uma requisição `multipart/form-data`, sob o nome "image".
        

### Retorno

- **`ResponseEntity<Void>`**: O método retorna um `ResponseEntity` sem corpo (`Void`).
    
    - Em caso de sucesso, ele retorna um `fileName` (o nome único gerado para a imagem salva). _Observação: O código fornecido retorna diretamente `fileName`, o que não corresponde ao tipo de retorno `ResponseEntity<Void>`. Para ser consistente com a assinatura, o ideal seria retornar `ResponseEntity.ok().build()` ou similar, e o nome do arquivo poderia ser retornado no corpo da resposta se o tipo de retorno fosse `ResponseEntity<String>`._
        

### Fluxo de Execução

1. **Validação do Formato do Arquivo**:
    
    - O método verifica se o nome original do arquivo (`image.getOriginalFilename()`) termina com ".png" ou ".jpg".
        
    - Se o arquivo não for PNG nem JPG, uma `RuntimeException` é lançada com a mensagem "O arquivo enviado não é uma imagem png ou jpg".
        
2. **Geração de Nome Único para o Arquivo**:
    
    - Um nome de arquivo único é gerado combinando um `UUID.randomUUID()` (Identificador Universalmente Único) com o nome original do arquivo. Isso garante que não haverá conflitos de nomes se múltiplos usuários fizerem upload de arquivos com o mesmo nome.
        
3. **Definição do Caminho de Armazenamento**:
    
    - `Path uploadPath = Paths.get(path);`: Converte a string `path` (que deve conter o caminho do diretório onde as imagens serão salvas) em um objeto `Path`.
        
    - `Path saveLocal = uploadPath.resolve(fileName);`: Constrói o caminho completo onde o arquivo será salvo, combinando o diretório de upload com o nome de arquivo único gerado.
        
4. **Salvamento do Arquivo**:
    
    - `image.transferTo(saveLocal);`: Este é o passo crucial onde o conteúdo binário da imagem recebida (`MultipartFile image`) é transferido e salvo no caminho `saveLocal` no sistema de arquivos.
        
5. **Retorno de Sucesso**:
    
    - Em caso de sucesso no salvamento, o método retorna o `fileName` (o nome único da imagem salva). _Conforme observado acima, para estar em conformidade com `ResponseEntity<Void>`, o retorno deveria ser ajustado._
        
6. **Tratamento de Erros**:
    
    - Um bloco `try-catch` envolve a operação de salvamento (`image.transferTo`).
        
    - Se ocorrer uma `IOException` durante a transferência do arquivo (por exemplo, permissão negada, disco cheio), uma nova `RuntimeException` é lançada, encapsulando a exceção original.
        

### Exemplo de Uso (Requisição HTTP)

Para usar este endpoint, você enviaria uma requisição HTTP `POST` com o `Content-Type` `multipart/form-data`. O corpo da requisição conteria a imagem sob a parte nomeada "image".

**Exemplo com `curl`:**

Bash

```
curl -X POST -F "image=@/caminho/para/sua/imagem.png" http://localhost:8080/image
```

---

# Baixar (Download) imagem 
Teremos que fazer uma rota para baixar a imagem
```java
import org.springframework.core.io.UrlResource;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import java.net.MalformedURLException;
import java.nio.file.Path;
import java.nio.file.Paths;

@RestController // Assumindo que este método estará em um Controller
public class ImageController {

    // Ajuste: A variável 'path' deve ser injetada ou definida. Exemplo:
    private final String imageStoragePath = "/caminho/para/suas/imagens"; // Substitua pelo seu caminho real

    @GetMapping("/images/{imageName}") // Mapeia para uma requisição GET em /images/{nomeDaImagem}
    public ResponseEntity<UrlResource> getImage(@PathVariable String imageName) {
        try {
            // Constrói o caminho completo para a imagem
            Path imagePath = Paths.get(this.imageStoragePath).resolve(imageName);
            UrlResource resource = new UrlResource(imagePath.toUri());

            // Verifica se o recurso (imagem) existe e é acessível
            if (resource.exists() && resource.isReadable()) {
                // Tenta determinar o tipo de conteúdo da imagem (PNG, JPG, etc.)
                // Para uma solução mais robusta, você pode usar Tika ou inferir pelo nome
                String contentType = determineContentType(imageName);

                return ResponseEntity.ok()
                        .contentType(MediaType.parseMediaType(contentType)) // Define o tipo de conteúdo dinamicamente
                        .body(resource);
            } else {
                // Retorna 404 Not Found se a imagem não existir ou não puder ser lida
                return ResponseEntity.notFound().build();
            }
        } catch (MalformedURLException e) {
            // Loga a exceção para depuração e retorna um erro 500 Internal Server Error
            System.err.println("Erro ao criar URL para a imagem: " + e.getMessage());
            throw new RuntimeException("Erro interno ao processar a requisição da imagem.", e);
        }
    }

    // Método auxiliar para tentar determinar o tipo de conteúdo
    private String determineContentType(String fileName) {
        if (fileName.toLowerCase().endsWith(".png")) {
            return MediaType.IMAGE_PNG_VALUE;
        } else if (fileName.toLowerCase().endsWith(".jpg") || fileName.toLowerCase().endsWith(".jpeg")) {
            return MediaType.IMAGE_JPEG_VALUE;
        }
        // Fallback ou tratamento para outros tipos de imagem
        return MediaType.APPLICATION_OCTET_STREAM_VALUE; // Tipo genérico para download
    }
}
```

1. **Anotações Spring Boot (`@GetMapping`, `@PathVariable`, `@RestController`)**:
    
    - **`@RestController`**: É uma anotação de conveniência que combina `@Controller` e `@ResponseBody`. Ela indica que esta classe é um controlador e que os valores de retorno dos métodos devem ser serializados diretamente para o corpo da resposta HTTP.
        
    - **`@GetMapping("/images/{imageName}")`**: Mapeia requisições HTTP GET para este método. O `{imageName}` é uma **variável de caminho** que será extraída da URL.
        
    - **`@PathVariable String imageName`**: Esta anotação vincula o valor da variável de caminho `imageName` da URL ao parâmetro `imageName` do método. Isso torna a rota mais limpa e flexível (ex: `/images/minha-foto.png`).
        
2. **Variável `imageStoragePath`**:
    
    - Renomeamos `this.path` para `this.imageStoragePath` para maior clareza. É fundamental que essa variável **seja configurada corretamente** (por exemplo, injetada via `@Value` ou lida de um arquivo de propriedades) para apontar para o diretório onde suas imagens estão salvas.
        
3. **Verificação `resource.isReadable()`**:
    
    - Além de `resource.exists()`, adicionamos `resource.isReadable()`. Isso garante que o servidor não só encontre o arquivo, mas também tenha as **permissões necessárias** para lê-lo, evitando erros de permissão no momento do download.
        
4. **Determinação Dinâmica do `ContentType`**:
    
    - Seu código original usava `MediaType.IMAGE_PNG`, o que significa que todas as imagens seriam enviadas como PNG, mesmo que fossem JPG.
        
    - Aprimoramos isso com um método auxiliar `determineContentType` que tenta inferir o tipo de mídia (MIME type) com base na extensão do arquivo (`.png`, `.jpg`, `.jpeg`). Isso é crucial para que o navegador do usuário saiba como lidar com o arquivo (ex: exibir a imagem no navegador ou iniciar o download).
        
    - Para uma solução de nível de produção, você pode usar bibliotecas como Apache Tika para uma detecção de tipo de conteúdo mais robusta, que examina o conteúdo do arquivo, e não apenas sua extensão.
        
5. **Tratamento de Exceções Aprimorado**:
    
    - Mudamos a mensagem da `RuntimeException` para ser mais informativa em caso de `MalformedURLException`.
        
    - Adicionamos um `System.err.println` para **logar o erro** no console do servidor. Em uma aplicação real, você usaria uma biblioteca de logging (como SLF4J/Logback) para registrar esses erros, o que é fundamental para depuração e monitoramento.
        
6. **Retorno `ResponseEntity`**:
    
    - **`ResponseEntity.ok()`**: Cria uma resposta HTTP com status `200 OK`, indicando sucesso.
        
    - **`.contentType(MediaType.parseMediaType(contentType))`**: Define o cabeçalho `Content-Type` da resposta, usando o tipo de mídia determinado dinamicamente.
        
    - **`.body(resource)`**: Define o corpo da resposta HTTP como o `UrlResource`, que contém os bytes da imagem.
        
    - **`ResponseEntity.notFound().build()`**: Retorna uma resposta HTTP com status `404 Not Found` se a imagem não for encontrada ou não puder ser lida, o que é uma prática recomendada para APIs REST.
        

---
#### Receber dto e imagem no mesmo end point
```java
@PostMapping(consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public UserResponseDto addUser(
            @Valid @RequestPart("user") UserRequestDto userRequest,
            @RequestPart(value = "image", required = false) MultipartFile image
    ){
        return this.userService.addUser(userRequest, image);
    }
```

### Visão Geral

O método `addUser` é um **endpoint POST** que aceita requisições com o tipo de conteúdo `multipart/form-data`. Ele espera receber dois "partes" na requisição:

1. Os dados do usuário em formato JSON (mapeados para `UserRequestDto`).
    
2. Opcionalmente, um arquivo de imagem para o perfil do usuário.
    

Ele então delega a lógica de negócio para o serviço `userService` para realmente adicionar o usuário e processar a imagem.

### Detalhes dos Componentes

- **`@PostMapping(consumes = MediaType.MULTIPART_FORM_DATA_VALUE)`**
    
    - **`@PostMapping`**: Esta anotação indica que este método irá lidar com requisições HTTP **POST**. Requisições POST são tipicamente usadas para **criar novos recursos** no servidor.
        
    - **`consumes = MediaType.MULTIPART_FORM_DATA_VALUE`**: Este atributo especifica que o endpoint espera receber dados no formato `multipart/form-data`. Este formato é comumente usado para **upload de arquivos**, pois permite enviar dados textuais (como JSON) e arquivos binários (como imagens) na mesma requisição.
        
- **`public UserResponseDto addUser(...)`**
    
    - **`UserResponseDto`**: Este é o tipo de retorno do método. Indica que, após o usuário ser adicionado com sucesso, a resposta HTTP conterá um objeto `UserResponseDto`, que provavelmente inclui os dados do usuário recém-criado (e talvez o caminho da imagem, se aplicável).
        
- **`@Valid @RequestPart("user") UserRequestDto userRequest`**
    
    - **`@RequestPart("user")`**: Esta anotação indica que a parte da requisição `multipart/form-data` com o nome `"user"` deve ser extraída e vinculada ao objeto `userRequest`. Essa parte normalmente conteria os dados JSON do usuário.
        
    - **`UserRequestDto userRequest`**: É um Objeto de Transferência de Dados (DTO) que representa os dados de entrada necessários para criar um usuário (ex: nome, email, senha).
        
    - **`@Valid`**: Esta anotação, parte do Bean Validation (JSR 303/349), instrui o Spring a **validar** o objeto `userRequest` de acordo com as regras de validação definidas dentro da classe `UserRequestDto` (por exemplo, `@NotNull`, `@Email`, `@Size`). Se a validação falhar, o Spring lançará uma exceção, e a requisição não prosseguirá.
        
- **`@RequestPart(value = "image", required = false) MultipartFile image`**
    
    - **`@RequestPart(value = "image", required = false)`**: Similar ao anterior, esta anotação extrai a parte da requisição chamada `"image"` e a vincula a um objeto `MultipartFile`.
        
    - **`MultipartFile image`**: Representa o arquivo enviado (neste caso, a imagem de perfil).
        
    - **`required = false`**: Este atributo é **muito importante**. Ele significa que a parte `"image"` na requisição `multipart/form-data` é **opcional**. O usuário pode criar uma conta sem fornecer uma imagem de perfil imediatamente.
        
- **`return this.userService.addUser(userRequest, image);`**
    
    - Esta linha demonstra a **separação de responsabilidades**. O controller (`addUser` neste caso) apenas recebe e valida a entrada. A lógica de negócio real para adicionar o usuário ao banco de dados, processar a imagem, etc., é delegada ao serviço `this.userService`. O método `addUser` do serviço provavelmente cuidará de:
        
        - Salvar os dados do `userRequest` no banco de dados.
            
        - Salvar a `image` em um diretório de arquivos (se presente).
            
        - Retornar os dados do usuário criado, possivelmente incluindo o caminho da imagem salva, como um `UserResponseDto`.
## Referências:
https://medium.com/@dulanjayasandaruwan1998/uploading-images-in-a-spring-boot-project-a-step-by-step-guide-8a55248ea520
