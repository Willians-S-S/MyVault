Para criarmos uma exception personalizada devemos criar uma classe com o nome do nosso erro, ela irá herdar da classe RuntimeException, no caso o nosso erro vai acontecer em tempo de execução.

```java
package com.willians.ListifyShop.exception;  
  
public class NotFoundException extends RuntimeException{  
    public NotFoundException(String message) {  
        super(message);  
    }  
}
```

Para implementar podemos fazer como demostrado no exemplo abaixo:
```java
public ResponseEntity<UserResponseDto> findUserById(UUID id){  
    User user = this.userRepository.findById(id).orElseThrow(() -> new NotFoundException("Usuário não encontrado."));  
  
    return ResponseEntity.ok(mapper.userToResponse(user));  
}
```

Buscamos o usuário pelo id, caso não encontramos é lançado o erro NotFoundException, para retornar a mensagem de erro personalizada siga os passos [[Mensagem de Erro personalizado]]

## Referências:
https://comoprogramarjava.com.br/excecoes-personalizadas-em-java/